---
materia: Reti di Calcolatori
tipo: Appunti di studio approfonditi
lezione: "4 - World Wide Web e Protocollo HTTP"
riferimenti_slide: "Lezione 4 - WorldWideWeb.pdf"
capitoli_libro: ["Capitolo 2 - Livello di Applicazione (2.2)"]
registrazione: "reti2.aac"
tags: [reti, livello-applicazione, web, http, cookie, web-cache, proxy, http-non-persistente, http-persistente, obsidian]
---

# 04 — World Wide Web e il Protocollo HTTP

> [!NOTE] Nota Metodologica e Didattica
> La presente nota sintetizza la registrazione d'aula della seconda parte del **Livello di Applicazione** integrandola con i contenuti della **Lezione 4 (Slide del docente)** e il testo teorico di riferimento **Kurose & Ross (8ª ed., Sezione 2.2)**. Vengono trattati in dettaglio i fondamenti del Web, l'anatomia delle transazioni HTTP (sintassi di richiesta e risposta, codici di stato), il confronto prestazionale tra connessioni non persistenti e persistenti con modello RTT, la gestione dello stato mediante Cookie, il funzionamento delle Web Cache (Proxy HTTP) con validazione condizionale `If-Modified-Since`, e l'evoluzione verso HTTP/2 e HTTP/3.

---

## 1. Fondamenti e Componenti del World Wide Web

Il **World Wide Web (WWW)** è un'applicazione distribuita su Internet basata sul paradigma Client-Server che consente la condivisione e la fruizione di documenti ipermediali collegati tra loro.

```
+-------------------------------------------------------------------------+
|                       I COMPONENTI BASE DEL WEB                         |
+-------------------------------------------------------------------------+
| 1. PAGINA WEB (Web Page):                                               |
|    Documento base (HTML) che fa da contenitore e fa riferimento a       |
|    molteplici oggetti collegati mediante hyperlink.                     |
+-------------------------------------------------------------------------+
| 2. OGGETTO (Object):                                                    |
|    Qualsiasi file digitale accessibile via rete:                        |
|    - File HTML base                                                     |
|    - Immagini grafiche (JPEG, PNG, SVG)                                 |
|    - Fogli di stile (CSS)                                               |
|    - Script eseguibili lato client (JavaScript)                         |
|    - File multimediali (audio, video, PDF)                              |
+-------------------------------------------------------------------------+
| 3. URL (Uniform Resource Locator):                                      |
|    Indirizzo globale univoco che identifica un oggetto sulla rete:      |
|    http://www.uniba.it:80/dipartimento/informatica/index.html           |
|    \___/  \___________/\_/ \________________________________/           |
|    Schema    Host     Porta                 Percorso                    |
+-------------------------------------------------------------------------+
```

### 1.1 Il Ruolo di Client e Server
- **Web Browser (Client HTTP):** Agisce da User Agent (Chrome, Firefox, Safari). Invia messaggi di richiesta HTTP al Web Server, riceve gli oggetti e si occupa del loro rendering grafico sullo schermo.
- **Web Server (Server HTTP):** Processo server in ascolto perenne (Apache, Nginx, IIS) che memorizza gli oggetti Web e li invia ai client che ne fanno richiesta tramite messaggi di risposta HTTP.

---

## 2. Il Protocollo HTTP (HyperText Transfer Protocol)

Il protocollo **HTTP** (definito nelle RFC 1945 per HTTP/1.0, RFC 2616 e RFC 7230 per HTTP/1.1) è il protocollo di livello applicativo che definisce la struttura e la semantica dei messaggi scambiati tra browser e web server.

```
       [ Client Web (Browser) ]                     [ Server Web (Host) ]
                  |                                           |
                  | ===== 1. Stabilisce Connessione TCP =====> | (Porta 80 / 443)
                  |                                           |
                  | ===== 2. HTTP Request (GET /index.html) => |
                  |                                           |
                  | <==== 3. HTTP Response (200 OK + Dati) === |
                  |                                           |
```

### 2.1 Caratteristiche Architetturali Chiave
1. **Poggia sul Protocollo di Trasporto TCP:** HTTP utilizza un canale affidabile su socket stream (porta standard **80** per HTTP in chiaro, porta **443** per HTTPS cifrato con TLS). Il client apre prima la connessione TCP; una volta stabilita, invia i messaggi HTTP. Nessun dato va perduto.
2. **Protocollo Privo di Stato (Stateless):** Il server HTTP non conserva alcuna informazione né memoria storica sulle richieste precedentemente inviate da uno specifico client. Se il client richiede lo stesso oggetto dieci volte di seguito, il server lo rispedisce dieci volte identico.
   - *Vantaggio:* Semplifica enormemente la progettazione del server, riduce l'occupazione di memoria e consente ai server di gestire contemporaneamente decine di migliaia di connessioni TCP senza rischio di incoerenza dello stato.
   - *Svantaggio:* Impossibilità nativa di implementare sessioni utente (carrelli e-commerce, login protetti), problema brillantemente risolto con l'invenzione dei **Cookie**.

---

## 3. Connessioni HTTP: Non Persistenti vs Persistenti

Una delle scelte progettuali più rilevanti per le prestazioni di caricamento di una pagina Web riguarda la gestione delle connessioni TCP sottostanti.

```
+--------------------------------------------------------------------+
|                CONFRONTO CONVERSIONI TCP IN HTTP                   |
+-----------------------------------+--------------------------------+
| HTTP NON PERSISTENTE (HTTP/1.0)   | HTTP PERSISTENTE (HTTP/1.1)    |
+-----------------------------------+--------------------------------+
| Ogni oggetto richiede l'apertura  | Più oggetti vengono inviati    |
| e chiusura di una NUOVA           | attraverso la STESSA           |
| connessione TCP dedicata.         | connessione TCP aperta.        |
+-----------------------------------+--------------------------------+
```

### 3.1 HTTP Non Persistente (Analisi del Tempo di Risposta RTT)

Definiamo **RTT (Round Trip Time)** come il tempo impiegato da un piccolo pacchetto di controllo per viaggiare dal client al server e ritornare indietro.

Supponiamo che una pagina Web sia composta da un **file HTML base** e contenga riferimenti a **10 immagini JPEG**:

```
Client                                                   Server
  | ---------- 1. TCP SYN (Handshake) -------------------> |
  | <--------- 2. TCP SYNACK ----------------------------- | [1 RTT per TCP]
  | ---------- 3. HTTP Request (GET) + ACK --------------> |
  | <--------- 4. HTTP Response (File HTML) -------------- | [1 RTT per HTTP]
  |                                                        |
  | [Chiusura TCP]                                         |
  
  Tempo per l'HTML base = 2 * RTT + Tempo_Trasmissione_File
```

Per scaricare i 10 oggetti collegati in modalità non persistente:
- Vengono aperte **altre 10 connessioni TCP separate** (ciascuna richiede altri $2 \text{ RTT}$).
- **Overhead Complessivo Elevato:** Sovraccarico notevole sul server e sul sistema operativo del client (creazione di socket, allocazione buffer TCP e variabili di stato).
- **Lentezza della fase Slow-Start:** Ogni nuova connessione TCP inizia con la finestra di congestione al minimo (*TCP slow-start*), non sfruttando mai la piena capacità della banda.

---

### 3.2 HTTP Persistente (Default in HTTP/1.1)

Con l'avvento di **HTTP/1.1**, la connessione TCP rimane aperta dopo che il server ha inviato la risposta. Tutte le richieste successive per gli oggetti appartenenti allo stesso server viaggiano sulla medesima connessione già stabilita:
- **Senza Pipelining:** Il client invia una nuova richiesta solo dopo aver ricevuto la risposta della richiesta precedente ($1 \text{ RTT}$ per ciascun oggetto referenziato).
- **Con Pipelining:** Il client invia tutte le richieste per gli oggetti referenziati una dopo l'altra senza attendere le risposte intermedie.
- **Formula Sintetica del Tempo Totale:**
  $$\text{Tempo}_{\text{Persistente}} \approx \text{RTT}_{\text{Handshake}} + \text{RTT}_{\text{HTML}} + \sum \text{RTT}_{\text{Oggetti}} + \text{Tempi Trasmissione}$$

| Parametro Prestazionale | HTTP Non Persistente | HTTP Persistente |
|---|---|---|
| **Connessioni TCP necessarie** | 1 per ciascun oggetto | 1 unica per l'intera sessione |
| **Tempo di Handshake TCP** | $1\text{ RTT}$ per ogni oggetto | $1\text{ RTT}$ una tantum all'inizio |
| **Impatto sui Buffer Server** | Alto (continuo alloc/dealloc) | Basso e costante |
| **Header di controllo** | `Connection: close` | `Connection: keep-alive` (default in 1.1) |

---

## 4. Anatomia dei Messaggi HTTP: Richiesta e Risposta

I messaggi HTTP/1.x sono composti da testo ASCII puro, rendendoli direttamente leggibili ed ispezionabili dagli analisti di rete.

### 4.1 Messaggio di Richiesta HTTP (HTTP Request)

```http
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Connection: keep-alive
Accept-Language: it-IT,it;q=0.9,en;q=0.8
[Riga vuota: CRLF (\r\n)]
[Corpo dell'entità opzionale: Entity Body]
```

```
+--------------------------------------------------------------------------+
|                      STRUTTURA DEL MESSAGGIO HTTP REQUEST                |
+--------------------------------------------------------------------------+
| 1. RIGA DI RICHIESTA (Request Line):                                     |
|    [ METODO ] [ URL / PERCORSO RISORSA ] [ VERSIONE HTTP ]               |
|    - Metodi principali:                                                  |
|      * GET     : Richiede la risorsa specificata dall'URL.               |
|      * POST    : Invia dati nel corpo (es. campi form, upload file).     |
|      * HEAD    : Identico a GET, ma il server omette il corpo.           |
|      * PUT     : Carica o sovrascrive un oggetto all'URL specificato.    |
|      * DELETE  : Cancella la risorsa specificata sul server.             |
+--------------------------------------------------------------------------+
| 2. RIGHE DI INTESTAZIONE (Header Lines):                                  |
|    - Host: Specifica il nome di dominio (obbligatorio per Virtual Host).  |
|    - User-Agent: Identifica tipo e versione del browser richiedente.     |
|    - Accept-Language: Preferenza linguistica del client.                 |
|    - Connection: keep-alive (mantiene TCP aperta) oppure close.         |
+--------------------------------------------------------------------------+
| 3. RIGA VUOTA (CRLF - Carriage Return Line Feed: \r\n)                   |
|    Indica formalmente la fine dell'header HTTP.                          |
+--------------------------------------------------------------------------+
| 4. CORPO DELL'ENTITÀ (Entity Body):                                      |
|    Vuoto nelle richieste GET; valorizzato nei POST con dati form o json. |
+--------------------------------------------------------------------------+
```

---

### 4.2 Messaggio di Risposta HTTP (HTTP Response)

```http
HTTP/1.1 200 OK
Date: Fri, 16 Oct 2026 12:00:00 GMT
Server: Apache/2.4.52 (Ubuntu)
Last-Modified: Thu, 15 Oct 2026 18:30:00 GMT
ETag: "737023-14c-5d2b1f8"
Content-Length: 6821
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
[Riga vuota: CRLF]
<!DOCTYPE html><html><head><title>Home</title>...</html>
```

#### Codici di Stato HTTP (Status Codes)

La riga di stato iniziale contiene il codice numerico a tre cifre che sintetizza l'esito dell'elaborazione:

| Classe Codice | Categoria di Risposta | Esempi Fondamentali d'Esame |
|---|---|---|
| **`1xx`** | **Informativi** | `100 Continue` (il server autorizza l'invio del body) |
| **`2xx`** | **Successo** | **`200 OK`**: Richiesta elaborata con successo, corpo presente. |
| **`3xx`** | **Reindirizzamento** | **`301 Moved Permanently`**: Risorsa spostata stabilmente.<br>**`304 Not Modified`**: Risorsa non modificata (per Web Cache). |
| **`4xx`** | **Errore del Client** | **`400 Bad Request`**: Sintassi richiesta errata.<br>**`403 Forbidden`**: Accesso non consentito.<br>**`404 Not Found`**: Risorsa non trovata sul server. |
| **`5xx`** | **Errore del Server** | **`500 Internal Server Error`**: Errore imprevisto lato server.<br>**`503 Service Unavailable`**: Server sovraccarico o spento. |

---

## 5. Mantenimento dello Stato Utente: I Cookie

Poiché HTTP è intrinsecamente stateless, i siti web impiegano la tecnologia dei **Cookie** (RFC 6265) per associare richieste distinte alla medesima sessione utente.

```
CLIENT (Browser)                                         SERVER WEB
      |                                                      |
      | ------ 1. Prima richiesta HTTP ordinaria -----------> |
      |                                                      | (Server genera ID sessione: 1678)
      |                                                      | (Salva record nel database interno)
      | <----- 2. HTTP Response (Set-Cookie: id=1678) ------- |
      |                                                      |
  [Browser memorizza cookie                                  |
   nel file locale per il dominio]                           |
      |                                                      |
      | ------ 3. Richiesta successiva (Cookie: id=1678) ---> |
      |                                                      | (Server riconosce l'utente: ID 1678)
      | <----- 4. HTTP Response personalizzata -------------- |
```

### 5.1 Le 4 Componenti del Meccanismo Cookie
1. **Intestazione di risposta HTTP `Set-Cookie:`** Inviata dal server nel messaggio di risposta per comunicare la chiave/valore al browser.
2. **Memorizzazione locale lato Client:** Il browser salva il cookie in un database locale associandolo al nome di dominio.
3. **Intestazione di richiesta HTTP `Cookie:`** Inserita automaticamente dal browser in tutte le successive richieste dirette a quel medesimo dominio.
4. **Database di sessione sul Web Server:** Tabella che correla l'identificatore del cookie alle informazioni sull'utente (prodotti nel carrello, preferenze, autenticazione attiva).

---

## 6. Web Caching e Proxy Server

Una **Web Cache (o Proxy Server)** è un'entità di rete che si interpone tra il browser client e il web server di origine, memorizzando localmente copie degli oggetti richiesti di recente.

```
       [ Client 1 ] ----+
                        |
       [ Client 2 ] ----+---> [ PROXY CACHE LOCALE ] ======= [ INTERNET ] ======= [ SERVER ORIGINE ]
                        |     (Rete Aziendale / Campus)
       [ Client 3 ] ----+
```

### 6.1 Vantaggi Architetturali del Proxy
1. **Riduzione dei Tempi di Risposta:** Se l'oggetto è già presente in cache (*Cache Hit*), esso viene recapitato al client ad altissima velocità su LAN locale senza dover attraversare Internet.
2. **Abbattimento del Traffico sulla Rete di Accesso:** Riduce drasticamente il traffico verso l'ISP esterno, prevenendo la saturazione della banda e riducendo i costi infrastrutturali.

---

### 6.2 Validazione Condizionale (GET Condizionale)

Il problema fondamentale del caching è la **freschezza del dato**: come fa il proxy a sapere se l'oggetto presente nella sua memoria cache è ancora identico a quello presente sul server originale?

La soluzione è il **GET Condizionale**:

```
Proxy Cache                                               Server Origine
     |                                                          |
     | ---- GET /index.html ----------------------------------> |
     |      If-Modified-Since: Thu, 15 Oct 2026 18:30:00 GMT    |
     |                                                          |
     | Caso A: Oggetto NON Modificato                           |
     | <--- HTTP/1.1 304 Not Modified ------------------------- |
     |      (Corpo vuoto: risparmio totale di banda!)           |
     |                                                          |
     | Caso B: Oggetto Modificato                               |
     | <--- HTTP/1.1 200 OK + [Nuovo Corpo File] -------------- |
```

- **Header `Last-Modified:`** Inviato inizialmente dal server per indicare la data/ora di modifica del file.
- **Header `If-Modified-Since:`** Inserito dal proxy nel GET condizionale: ordina al server di rispedire il file solo se è stato alterato successivamente a quella data.
- **Risposta `304 Not Modified`:** Se il documento non è cambiato, il server invia un pacchetto privo di corpo dati, confermando alla cache di riutilizzare la copia locale.

---

## 7. Evoluzione del Protocollo: Da HTTP/1.1 a HTTP/2 e HTTP/3

| Versione Protocollo | Anno | Livello Trasporto | Innovazioni e Risoluzione Colli di Bottiglia |
|---|---|---|---|
| **HTTP/1.0** | 1996 | TCP | Connessioni non persistenti per default; 1 TCP per oggetto. |
| **HTTP/1.1** | 1999 | TCP | Connessioni persistenti (*keep-alive*); caching condizionale; pipelining. Soffre di blocco in testa applicativo (*HOL Blocking*). |
| **HTTP/2** | 2015 | TCP | **Multiplexing binario** su singola connessione TCP (flussi paralleli indipendenti); compressione header HPACK; Server Push. |
| **HTTP/3** | 2022 | **QUIC (UDP)** | Sostituisce TCP con QUIC basato su UDP. Elimina completamente l'HOL Blocking anche a livello di trasporto ed integra TLS 1.3 nativamente. |

---

## 8. Domande di Verifica ed Esercizi di Consolidamento d'Esame

### Domanda 1: Perché HTTP è definito un protocollo "stateless" e in quale modo i Cookie superano questo vincolo architetturale?
**Risposta Didattica:** 
HTTP è *stateless* perché il server elabora ogni richiesta in totale isolamento, senza registrare né ricordare lo storico delle transazioni precedenti con lo stesso client. I **Cookie** superano questa limitazione consentendo al server di associare un codice identificativo di sessione al client tramite l'header `Set-Cookie`. Il client memorizza localmente tale identificatore e lo include automaticamente nelle successive richieste HTTP mediante l'header `Cookie:`, permettendo al server di ricostruire l'identità e lo stato della sessione (es. profilo loggato, articoli nel carrello) interrogando il proprio database.

### Domanda 2: Qual è il vantaggio quantitativo, in termini di RTT, dell'uso di una connessione HTTP persistente rispetto a una non persistente per il caricamento di una pagina contenente $N$ oggetti?
**Risposta Didattica:** 
In una connessione **non persistente**, ogni singolo oggetto richiede l'apertura di una connessione TCP dedicata ($1\text{ RTT}$ per l'handshake SYN-SYNACK) seguita dalla richiesta HTTP ($1\text{ RTT}$), per un totale di $2\text{ RTT}$ per oggetto. Il tempo complessivo (trascurando i tempi di trasmissione) è di $2(N+1)\text{ RTT}$ per l'HTML base più gli $N$ oggetti.
In una connessione **persistente con pipelining**, dopo l'handshake iniziale ($1\text{ RTT}$) e la ricezione del file base HTML ($1\text{ RTT}$), tutti gli $N$ oggetti vengono richiesti contemporaneamente sulla medesima connessione, impiegando idealmente soltanto **$1\text{ RTT}$ complessivo aggiuntivo**. Il tempo totale si riduce a circa $3\text{ RTT}$, ottenendo un risparmio di quasi $2N\text{ RTT}$.

### Domanda 3: In che cosa differisce la risposta HTTP con codice di stato `304 Not Modified` da una risposta `200 OK` durante una richiesta con GET Condizionale?
**Risposta Didattica:** 
La risposta `200 OK` attesta che la risorsa sul server è nuova o è stata modificata: essa include sia l'intestazione HTTP sia l'intero **corpo dei dati** (il file completo), consumando banda di trasmissione. La risposta `304 Not Modified`, al contrario, conferma alla Web Cache che l'oggetto memorizzato localmente è ancora perfettamente aggiornato: la risposta contiene unicamente l'intestazione HTTP e ha il **corpo vuoto**, azzerando il consumo di banda di rete per il trasferimento dei dati e velocizzando la fruizione per il client finale.

---

## Riferimenti Incrociati

- [[00 - Introduzione alle Reti di Calcolatori]]
- [[01 - Architettura a Livelli e Protocolli]]
- [[02 - Storia di Internet e del Web]]
- [[03 - Introduzione alle Applicazioni di Rete]]
- [[Capitolo 2 - Livello di Applicazione]]
