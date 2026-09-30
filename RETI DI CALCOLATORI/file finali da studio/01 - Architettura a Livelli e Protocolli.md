---
materia: Reti di Calcolatori
tipo: Appunti di studio approfonditi
lezione: "1 - Architettura a Livelli"
riferimenti_slide: "Lezione 1 - Architettura a Livelli.pdf"
capitoli_libro: ["Capitolo 1 - Reti di Calcolatori e Internet (1.1, 1.5)"]
tags: [reti, architettura-a-livelli, iso-osi, tcp-ip, protocolli, incapsulamento, obsidian]
---

# 01 — Architettura a Livelli e Protocolli di Rete

> [!NOTE] Nota Metodologica e Didattica
> La presente nota integra la **Lezione 1 (Prof. Filippo Lanubile)** con la trattazione teorica e formale del testo **Kurose & Ross (8ª ed., Sezione 1.5)**. Vengono analizzati i concetti di astrazione, stratificazione (*layering*), la formalizzazione di sintassi e semantica dei protocolli, la doppia rappresentazione delle dipendenze (grafo vs pila), il processo di incapsulamento/decapsulamento, il modello ISO/OSI a 7 livelli, i dispositivi intermedi ai vari strati e i principi guida dell'architettura Internet.

---

## 1. Il Principio di Stratificazione (Layering) e l'Astrazione

Una rete di calcolatori è un sistema estremamente complesso composto da hardware eterogeneo (cavi, microonde, switch, router), software applicativo e molteplici tecnologie di trasmissione. Per gestire tale complessità senza sopraffare il progettista applicativo, le reti adottano il principio di **stratificazione** (*layering*).

```
+-----------------------------------------+
|     LIVELLO APPLICATIVO (Processi)      |
|  Nasconde topologia, ritardi e fisicità |
+--------------------+--------------------+
                     | Interfaccia API
                     v
+-----------------------------------------+
|     LIVELLO DI TRASPORTO (End-to-End)   |
|  Canale logico affidabile/unreliable    |
+--------------------+--------------------+
                     | Interfaccia API
                     v
+-----------------------------------------+
|       LIVELLO DI RETE (Routing & IP)    |
|   Instradamento tra nodi intermedi      |
+--------------------+--------------------+
                     | Interfaccia API
                     v
+-----------------------------------------+
|  LIVELLO DI COLLEGAMENTO E FISICO (HW)  |
|   Trasmissione fisica di frame e bit    |
+-----------------------------------------+
```

### 1.1 Concetto di Astrazione nei Sistemi Complessi
- **Obiettivo Fondamentale:** Fornire modelli di astrazione progressivi che nascondano la complessità dei livelli sottostanti. Il progettista di un'applicazione Web (es. un browser HTTP) non deve preoccuparsi delle modulazioni di frequenza del WiFi né della gestione delle perdite di pacchetti sul cavo sottomarino.
- **Entità (Entities):** Elementi attivi (processi software, moduli hardware o chip di rete) presenti all'interno di un determinato livello su un dato calcolatore.
- **Servizio (Service):** Insieme di capacità ed operazioni che un livello $N$ offre al livello superiore $(N+1)$. I dettagli su *come* il servizio sia implementato rimangono rigorosamente celati al livello $(N+1)$.
- **Interfaccia (SAP - Service Access Point):** Il punto di contatto e l'insieme di primitive attraverso cui due livelli adiacenti sullo stesso calcolatore scambiano dati e comandi.

---

## 2. Formalizzazione dei Protocolli di Rete

Un **protocollo di rete** è un insieme formale di regole che specificano il formato, l'ordine di invio e ricezione dei messaggi, e le azioni da intraprendere a seguito dell'invio/ricezione di un messaggio o di un evento temporale.

```
Host A (Liv. N) <==== Prot. Liv. N ====> Host B (Liv. N)
     |                                        ^
     v Servizio (N-1)                         | Servizio (N-1)
Host A (Liv. N-1) ---------------------> Host B (Liv. N-1)
```

### 2.1 Sintassi, Semantica e Sincronizzazione
Ogni protocollo di rete è definito in modo rigoroso da tre componenti essenziali:

1. **Sintassi (Syntax):** Definisce la struttura, la codifica e il formato dei dati e delle intestazioni (*header*). Specifica la posizione esatta dei campi all'interno del pacchetto (es. i primi 32 bit indicano l'IP sorgente, i successivi 32 bit l'IP destinazione).
2. **Semantica (Semantics):** Definisce il significato del contenuto di ciascuna intestazione e le azioni che l'entità deve intraprendere quando riceve un determinato campo o messaggio (es. *"Se il flag SYN è 1, inizia la procedura di handshake"*).
3. **Sincronizzazione / Temporizzazione (Timing / Synchronization):** Specifica l'ordine temporale degli eventi, la gestione dei timeout e l'adeguamento delle velocità di trasmissione (es. controllo di flusso e gestione delle ritrasmissioni).

### 2.2 Comunicazione Orientata alla Connessione vs Priva di Connessione

A seconda del modello di servizio offerto, i protocolli si dividono in due grandi paradigmi:

| Proprietà Architetturale | Connection-Oriented (Orientato alla Connessione) | Connectionless (Privo di Connessione) |
|---|---|---|
| **Fase di Setup** | Obbligatoria (Handshake preventivo prima di inviare i dati) | Assente (I dati vengono inviati immediatamente) |
| **Stato della Comunicazione** | Le entità mantengono uno stato (*stateful*) | Le entità non mantengono uno stato (*stateless*) |
| **Ordine dei Messaggi** | Garantito (Consegna sequenziale ordinata) | Non garantito (I pacchetti possono giungere disordinati) |
| **Esempio Tipico** | Protocollo **TCP** (Transmission Control Protocol) | Protocollo **UDP** (User Datagram Protocol) / **IP** |

---

## 3. Rappresentazione delle Famiglie di Protocolli: Grafo vs Pila

Un insieme di protocolli interdipendenti progettati per lavorare in sinergia costituisce una **Famiglia (o Suite) di Protocolli** (es. *Internet Protocol Suite TCP/IP*).

### 3.1 Rappresentazione mediante Grafo delle Dipendenze
- **Nodi del Grafo:** Rappresentano i singoli protocolli (es. HTTP, TCP, UDP, IP, Ethernet).
- **Archi Orientati (Dipendenze d'Uso):** Un arco diretto dal protocollo $A$ al protocollo $B$ ($A \to B$) indica che il protocollo $A$ **dipende ed utilizza i servizi** forniti dal protocollo $B$.

```
[Grafo delle Dipendenze]
       HTTP          DNS
         \          /   \
          v        v     v
            TCP          UDP
             \            /
              v          v
                 IP
                  |
                  v
              Ethernet
```

### 3.2 Rappresentazione mediante Pila (Protocol Stack)
- I protocolli vengono organizzati in blocchi rettangolari sovrapposti verticalmente.
- Le dipendenze d'uso sono **implicite**: un blocco poggiato sopra un altro indica che il livello superiore sfrutta il livello sottostante.
- **Proprietà di AciClicità:** In una pila di protocolli **non sono ammesse dipendenze circolari**. L'elaborazione procede strettamente verso l'alto (in ricezione) o verso il basso (in trasmissione).

---

## 4. Dettaglio del Meccanismo di Incapsulamento e Decapsulamento

Il trasferimento dei dati tra entità remote avviene attraverso il passaggio verticale dei messaggi lungo la pila di protocolli.

```
[FLUSSO DI INCAPSULAMENTO]

(Applicazione) --> Messaggio M
                        |
                        v
(Trasporto)    --> [ H_tcp | M ]              (Segmento)
                        |
                        v
(Rete)         --> [ H_ip | H_tcp | M ]       (Datagramma)
                        |
                        v
(Link)         --> [ H_eth | H_ip | H_tcp | M | T_eth ] (Frame)
                        |
                        v
(Fisico)       --> Bit (1010101101...) sul canale
```

### 4.1 Fasi Operative del Processo
1. **Fase di Spedizione (Incapsulamento - Encapsulation):**
   - Il livello applicativo genera il messaggio $M$ e lo passa al livello di trasporto.
   - Il livello di trasporto accetta $M$, **aggiunge la propria intestazione** $H_t$ (contenente porte, checksum, ecc.), ed eventualmente **frammenta** il messaggio se supera la dimensione massima consentita (*MSS - Maximum Segment Size*). La nuova PDU prende il nome di **Segmento**.
   - Il livello di rete accetta il segmento, aggiunge l'header $H_n$ (con gli indirizzi IP sorgente e destinazione), creando il **Datagramma**.
   - Il livello di collegamento aggiunge la propria intestazione di frame $H_l$ e spesso una coda di chiusura $T_l$ (*Trailer* contenente il codice di controllo di errore CRC), forming la **Trama (Frame)**.
   - Il livello fisico trasforma il frame in una sequenza numerica di impulsi elettrici, ottici o elettromagnetici (**Bit**).

2. **Fase di Ricezione (Decapsulamento - Decapsulation):**
   - Il livello fisico del destinatario riceve i bit e li riassembla nel frame.
   - Ciascun livello esamina ed **elimina la propria intestazione di competenza** (*strip header*), esegue i controlli di correttezza e passa il carico utile (*Payload*) al livello superiore.

---

## 5. Il Modello di Riferimento ISO/OSI (7 Livelli)

Sviluppato dall'**ISO** (*International Organization for Standardization*), il modello **OSI** (*Open Systems Interconnection*) definisce uno standard aperto volto a garantire la completa interoperabilità tra sistemi eterogenei.

```
+---------------------------------------------------+
| 7. APPLICAZIONE (Application Layer)               |
|    Interfaccia utente e processi distribuiti.     |
+---------------------------------------------------+
| 6. PRESENTAZIONE (Presentation Layer)             |
|    Sintassi, cifratura, compressione dati.        |
+---------------------------------------------------+
| 5. SESSIONE (Session Layer)                       |
|    Sincronizzazione dialogo e checkpoint.          |
+---------------------------------------------------+
| 4. TRASPORTO (Transport Layer)                    |
|    Comunicazione end-to-end e controllo flusso.   |
+---------------------------------------------------+
| 3. RETE (Network Layer)                           |
|    Instradamento (routing) datagrammi tra reti.   |
+---------------------------------------------------+
| 2. COLLEGAMENTO DATI (Data Link Layer)            |
|    Trasferimento frame tra nodi adiacenti (MAC).  |
+---------------------------------------------------+
| 1. FISICO (Physical Layer)                        |
|    Trasmissione bit grezzi sul canale hardware.   |
+---------------------------------------------------+
```

---

## 6. Dispositivi Intermedi di Rete collocati ai vari Livelli OSI

I nodi intermedi di una rete commutata svolgono compiti differenti in base al **livello massimo della pila OSI** che sono in grado di analizzare ed elaborare.

```
[Livello 7] ---> APPLICATION GATEWAY (Proxy, WAF)
[Livello 3] ---> ROUTER / LAYER-3 SWITCH
[Livello 2] ---> SWITCH LAYER-2 / BRIDGE
[Livello 1] ---> REPEATER / HUB
```

### 6.1 Classificazione dei Dispositivi

| Dispositivo Intermedio | Livello OSI Operativo | PDU Esaminata | Funzione e Comportamento Logico |
|---|---|---|---|
| **Repeater / Hub** | **Livello 1 (Fisico)** | Bit | Rigenera ed amplifica il segnale elettrico. Non esamina indirizzi; trasmette in broadcast cieco. |
| **Switch L2 / Bridge** | **Livello 2 (Data Link)** | Frame (Trama) | Esamina gli **indirizzi MAC**. Isola i domini di collisione ed inoltra le trame via MAC learning. |
| **Router / Switch L3** | **Livello 3 (Rete)** | Datagramma | Esamina gli **indirizzi IP**. Interroga le tabelle di routing per inoltrare i pacchetti tra reti distinte. |
| **Application Gateway** | **Livello 7 (Applicazione)** | Messaggio | Ricostruisce l'intera pila fino al L7 per analizzare il contenuto applicativo (es. Proxy, WAF). |

---

## 7. Architettura di Internet: Principi Guida e Standardizzazione

L'architettura reale di Internet (la suite **TCP/IP**) differisce dal rigido modello OSI ed è stata concepita guidata da due principi cardine:

```
+-----------------------------------------+
|      Minimalismo ed Autonomia Reti      |
+--------------------+--------------------+
                     |
                     v
+-----------------------------------------+
|      Modello Best-Effort (IP)           |
|    Router Stateless & Decentralizz.     |
+--------------------+--------------------+
                     |
                     v
+-----------------------------------------+
|      Organizzazione IETF / Standard     |
|      Documentazione ufficiale RFC       |
+-----------------------------------------+
```

### 7.1 Principi Cardine dell'Architettura Internet
1. **Minimalismo e Autonomia delle Reti sottostanti:** Per interconnettere una nuova rete a Internet **non è richiesta alcuna modifica interna** alla tecnologia di tale rete. Internet agisce come un meta-livello di astrazione sopra reti eterogenee (*Internetworking*).
2. **Modello di Servizio Best-Effort (Massimo Impegno):** Il livello di rete IP non garantisce che i pacchetti giungano a destinazione, né che giungano in ordine o privi di duplicati. I router mantengono un'architettura **stateless** (senza stato delle sessioni), massimizzando la velocità di inoltro e la tolleranza ai guasti.

### 7.2 Entità di Standardizzazione e Documentazione RFC
- **IETF (Internet Engineering Task Force):** Organismo internazionale aperto costituito da tecnici e ricercatori che sviluppa e promuove gli standard di Internet (http://www.ietf.org).
- **RFC (Request for Comments):** Documenti ufficiali numerati che definiscono i protocolli, le architetture e gli standard di Internet (es. RFC 791 per IP, RFC 793 per TCP, RFC 2616 per HTTP/1.1).
- **Wireshark:** Strumento software open-source di *Packet Sniffing* ed analisi di protocollo che permette di catturare ed ispezionare visivamente l'incapsulamento dei pacchetti ai vari livelli della pila in tempo reale.

---

## 8. Domande di Verifica ed Esercizi di Consolidamento d'Esame

### Domanda 1: Perché in una pila di protocolli non sono ammesse dipendenze circolari?
**Risposta Didattica:** 
Una dipendenza circolare (es. il protocollo A che dipende da B, e B che dipende direttamente da A) creerebbe un ciclo infinito nel processo di incapsulamento/decapsulamento e nell'invocazione delle primitive di interfaccia. La stratificazione richiede una gerarchia strettamente aciclica in cui i livelli superiori consumano in modo unidirezionale i servizi forniti dai livelli sottostanti.

### Domanda 2: Qual è la differenza fondamentale tra un dispositivo di Livello 2 (Switch) e un dispositivo di Livello 3 (Router)?
**Risposta Didattica:** 
Uno **Switch di Livello 2** opera nel livello di collegamento: analizza l'intestazione delle trame (indirizzi fisici MAC) ed inoltra i dati all'interno della medesima rete locale (LAN). Non modifica l'intestazione di rete.
Un **Router di Livello 3** opera nel livello di rete: esamina gli indirizzi logici IP contenuti nei datagrammi, consulta le tabelle di instradamento ed inoltra i pacchetti **tra reti distinte ed eterogenee**, decrementando il campo TTL e riscrivendo l'intestazione del frame di livello 2 a ogni salto.

### Domanda 3: Perché nel modello TCP/IP i livelli di Presentazione e Sessione (presenti in OSI) non esistono come strati autonomi?
**Risposta Didattica:** 
Perché l'architettura Internet rispetta il principio di essenzialità e modularità. Le funzionalità di presentazione (cifratura, compressione) e sessione (gestione del dialogo) non sono ritenute necessarie per tutte le applicazioni di rete. Laddove richieste, esse vengono implementate direttamente all'interno del software di **livello applicativo** (es. TLS/SSL gestito internamente dall'applicazione Web sopra il livello di trasporto TCP).

---

## Riferimenti Incrociati

- [[00 - Introduzione alle Reti di Calcolatori]]
- [[Capitolo 1 - Reti di Calcolatori e Internet]]
- [[Capitolo 2 - Livello di Applicazione]]
- [[Capitolo 3 - Livello di Trasporto]]
