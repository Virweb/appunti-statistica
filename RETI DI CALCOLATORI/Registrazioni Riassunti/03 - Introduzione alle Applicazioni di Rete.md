---
materia: Reti di Calcolatori
tipo: Appunti di studio approfonditi
lezione: "3 - Introduzione alle Applicazioni di Rete"
riferimenti_slide: "Lezione 3 - Introduzione alle Applicazioni di Rete.pdf"
capitoli_libro: ["Capitolo 2 - Livello di Applicazione (2.1)"]
registrazione: "reti.aac"
tags: [reti, livello-applicazione, client-server, peer-to-peer, socket, architetture-distribuite, tcp-vs-udp, obsidian]
---

# 03 — Introduzione alle Applicazioni di Rete e Architetture Distribuite

> [!NOTE] Nota Metodologica e Didattica
> La presente nota sintetizza la registrazione d'aula della prima parte del **Livello di Applicazione** integrandola con i contenuti della **Lezione 3 (Slide del docente)** e il testo teorico di riferimento **Kurose & Ross (8ª ed., Sezione 2.1)**. Vengono approfonditi i principi di progettazione delle applicazioni distribuite, i paradigmi architetturali (Client-Server vs P2P), l'interfaccia Socket come frontiera del software utente, i requisiti di trasporto e il confronto tassonomico tra TCP e UDP a supporto dei processi applicativi.

---

## 1. Principi Fondamentali delle Applicazioni di Rete

Il **Livello di Applicazione** costituisce lo strato sommitale (Livello 5 nel modello Internet a 5 strati, Livello 7 nel modello ISO/OSI) dell'architettura di rete. È qui che risiede il software applicativo ed è l'unico livello con cui l'utente finale interagisce direttamente.

```
+-------------------------------------------------------------------+
|                  SISTEMI TERMINALI (END SYSTEMS)                  |
|                                                                   |
|   [ Applicazione Utente (Processo Client) ]                       |
|                 | (Socket API)                                    |
|                 v                                                 |
|   [ Livello di Trasporto: TCP o UDP ]                             |
|                 |                                                 |
|                 v                                                 |
|   [ Livello di Rete: IP ]                                         |
+-----------------+-------------------------------------------------+
                  | (Datagrammi IP)
                  v
       +--------------------+          +--------------------+
       |  ROUTER NUCLEO L3  | -------- |  ROUTER NUCLEO L3  |
       |  (Solo L1-L2-L3)   |          |  (Solo L1-L2-L3)   |
       +--------------------+          +--------------------+
                  |
                  v
+-----------------+-------------------------------------------------+
|   [ Livello di Rete: IP ]                                         |
|                 |                                                 |
|                 v                                                 |
|   [ Livello di Trasporto: TCP o UDP ]                             |
|                 | (Socket API)                                    |
|                 v                                                 |
|   [ Applicazione Utente (Processo Server) ]                       |
|                                                                   |
|                  SISTEMI TERMINALI (END SYSTEMS)                  |
+-------------------------------------------------------------------+
```

### 1.1 Sviluppo al Bordo della Rete (Edge vs Core)
Uno dei principi architetturali più eleganti di Internet risiede nella rigida separazione tra la periferia e il nucleo:
- **I programmi applicativi girano esclusivamente sui sistemi terminali (End Systems / Host):** Computer desktop, server, laptop, smartphone e dispositivi IoT.
- **Il nucleo della rete (Network Core) non esegue codice applicativo:** Router e switch instradano esclusivamente pacchetti a livello di rete (L3) e trame a livello di collegamento (L2), rimanendo completamente agnostici rispetto alla natura dei dati trasportati.
- **Vantaggio dell'approccio:** Questa suddivisione consente lo sviluppo, l'innovazione e il rilascio rapido di nuove applicazioni (Web, streaming multimediale, reti social, videoconferenza) senza dover modificare l'infrastruttura di rete né aggiornare il software interno dei router.

---

## 2. Paradigmi Architetturali delle Applicazioni di Rete

Per realizzare la cooperazione distribuita tra host remoti, i progettisti di rete adottano principalmente due grandi modelli architetturali: **Client-Server** e **Peer-to-Peer (P2P)**, oltre a combinazioni ibride.

```
   [ ARCHITETTURA CLIENT-SERVER ]             [ ARCHITETTURA PEER-TO-PEER (P2P) ]

         +---------------+                           +--------+        +--------+
         | Server Fisso  |                           | Peer A | <====> | Peer B |
         | (IP Statico)  |                           +--------+        +--------+
         +-------+-------+                               ^                 ^
            /    |    \                                  |       \ /       |
           /     |     \                                 |        X        |
          v      v      v                                v       / \       v
      +----+  +----+  +----+                         +--------+        +--------+
      | C1 |  | C2 |  | C3 |                         | Peer C | <====> | Peer D |
      +----+  +----+  +----+                         +--------+        +--------+
       Client (IP Dinamici)                       (Nodi paritetici: Client + Server)
```

### 2.1 Architettura Client-Server

Nel modello Client-Server vi è una netta asimmetria funzionale e temporale tra i due nodi partecipanti:

#### A. Il Server
- **Sempre attivo (Always-on host):** Rimane in perenne ascolto su una porta nota in attesa che giungano richieste.
- **Indirizzo IP noto e statico:** Deve poter essere raggiunto in modo deterministico da tutti i client.
- **Infrastruttura di Data Center:** Poiché un singolo calcolatore fisico collasserebbe sotto il carico di milioni di richieste simultanee (effetto *bottleneck* / punto singolo di fallimento), i server moderni sono ospitati all'interno di vaste server farm o data center distribuiti, virtualizzati e bilanciati (load balancer).

#### B. I Client
- **Comunicazione intermittente:** Possono collegarsi e scollegarsi arbitrariamente.
- **Indirizzi IP dinamici:** L'IP può cambiare a ogni connessione (assegnato dinamicamente via DHCP dall'ISP).
- **Assenza di comunicazione diretta tra Client:** Due client non dialogano mai direttamente tra loro; ogni interazione avviene transitando per il server.
- **Esempi:** Il Web (protocollo HTTP), la posta elettronica (protocollo SMTP), il trasferimento file (FTP), il servizio nomi (DNS).

---

### 2.2 Architettura Peer-to-Peer (P2P)

Nel modello Peer-to-Peer non vi è alcuna dipendenza rigida da un'infrastruttura server centrale sempre attiva:
- **Nodi Paritetici (Servent = Server + Client):** Ogni host (chiamato *peer*) agisce contemporaneamente da client (quando richiede blocchi di dati ad altri nodi) e da server (quando invia i blocchi memorizzati localmente ad altri nodi richiedenti).
- **Comunicazione Diretta:** I peer comunicano direttamente tra loro su connessioni arbitrarie senza intermediari centrali.
- **Auto-scalabilità (Self-Scalability):** All'aumentare del numero di nodi che richiedono dati, aumenta in modo proporzionale anche la capacità elaborativa e di banda complessiva della rete, poiché ogni nuovo peer porta con sé la propria banda di upload.
- **Criticità e Sfide:** Gestione degli indirizzi IP dinamici fortemente mutevoli (*churn rate* elevato per nodi che entrano ed escono continuamente), attraversamento dei NAT/Firewall e necessità di meccanismi di incentivazione (es. algoritmi *Tit-for-Tat* in BitTorrent) per contrastare il fenomeno dei *free-rider* (utenti che scaricano senza condividere).
- **Esempi:** BitTorrent, reti blockchain e criptovalute (Bitcoin, Ethereum), prime generazioni di Skype.

---

### 2.3 Architetture Ibride
Alcune applicazioni combinano i vantaggi di entrambi i paradigmi:
- **Ricerca Centralizzata, Trasferimento P2P (es. Napster storico):** Un server centrale indicizza i file posseduti da ciascun utente; una volta individuato il peer che possiede il brano, il trasferimento del file avviene direttamente in P2P.
- **Instant Messaging:** La gestione della presenza online (utente online/offline) e l'autenticazione avvengono tramite server centralizzato; il flusso di dati (chiamata vocale o video) può tentare l'instradamento diretto P2P per minimizzare la latenza.

---

## 3. Comunicazione tra Processi e l'Interfaccia Socket

A livello operativo, la comunicazione di rete non avviene genericamente tra due computer fisici, ma tra **processi software in esecuzione** sui rispettivi sistemi operativi.

```
HOST MITTENTE                                                 HOST DESTINATARIO
+---------------------------------------+                     +---------------------------------------+
|  Processo Applicativo (Spazio Utente) |                     |  Processo Applicativo (Spazio Utente) |
+-------------------+-------------------+                     +-------------------+-------------------+
                    | invia messaggi                                              ^ riceve messaggi
                    v                                                             |
            [ SOCKET APPLICATIVO ]                                        [ SOCKET APPLICATIVO ]
                    | (Frontiera API: Controllata dal SO)                         ^
~~~~~~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~~~~~~
                    v (Spazio Kernel)                                             | (Spazio Kernel)
+-------------------+-------------------+                     +-------------------+-------------------+
|  Livello di Trasporto (TCP / UDP)     |                     |  Livello di Trasporto (TCP / UDP)     |
+-------------------+-------------------+                     +-------------------+-------------------+
|  Livello di Rete (IP)                 | ==================> |  Livello di Rete (IP)                 |
+---------------------------------------+                     +---------------------------------------+
```

### 3.1 Cos'è una Socket?
Una **Socket** è l'interfaccia di programmazione (API) software che agisce come punto di contatto e porta d'ingresso/uscita tra il processo utente (nello spazio utente) e i protocolli del livello di trasporto (nello spazio kernel del sistema operativo).
- **Analogia Pratica:** La socket è paragonabile alla porta di una casa: il processo applicativo prepara la lettera (il messaggio) e la fa uscire attraverso la porta (la socket); dall'altra parte dell'uscio, l'infrastruttura postale (il livello di trasporto e di rete sottostante) prende in carico la busta per recapitarla alla porta dell'abitazione del destinatario.
- **Controllo dello Sviluppatore:** Il programmatore ha il controllo totale su tutto ciò che sta all'interno del livello applicativo (formato dati, logica di elaborazione), ma all'atto della creazione della socket può impostare solo:
  1. La scelta del protocollo di trasporto sottostante (tipicamente `SOCK_STREAM` per TCP o `SOCK_DGRAM` per UDP).
  2. Alcuni parametri specifici del socket (es. dimensione massima del buffer, timeout).

---

### 3.2 Indirizzamento e Identificazione dei Processi (IP + Porta)

Perché un messaggio inviato da un host sorgente possa raggiungere inequivocabilmente uno specifico processo in esecuzione su un host remoto, occorre una coppia di identificatori gerarchici:

1. **Indirizzo IP (32 bit in IPv4, 128 bit in IPv6):** Identifica univocamente l'interfaccia di rete dell'**Host di destinazione** all'interno dell'intera rete globale Internet.
2. **Numero di Porta (16 bit: interi da $0$ a $65535$):** Identifica univocamente lo **specifico processo (o socket)** in esecuzione all'interno di quel determinato host.

```
                     +----------------------------------------+
                     |     HOST A (Indirizzo IP: 193.204.5.10) |
                     +-------------------+--------------------+
                                         |
                                         +---> Porta 80   (Processo Web Server HTTP)
                                         +---> Porta 443  (Processo Web Server HTTPS)
                                         +---> Porta 25   (Processo Mail Server SMTP)
                                         +---> Porta 53   (Processo Name Server DNS)
                                         +---> Porta 52144 (Processo Browser Client)
```

#### Tassonomia delle Porte IANA:
- **Porte Ben Note (Well-Known Ports, 0 – 1023):** Riservate a protocolli applicativi standardizzati globali e gestite con privilegi amministrativi (root/admin).
  - `Porta 80`: **HTTP** (HyperText Transfer Protocol)
  - `Porta 443`: **HTTPS** (HTTP Secure via TLS/SSL)
  - `Porta 25`: **SMTP** (Simple Mail Transfer Protocol)
  - `Porta 53`: **DNS** (Domain Name System)
  - `Porta 21`: **FTP Control** (File Transfer Protocol)
  - `Porta 22`: **SSH** (Secure Shell)
- **Porte Registrate (Registered Ports, 1024 – 49151):** Assegnate da IANA a specifiche applicazioni commerciali o di comunità (es. DBMS MySQL: 3306).
- **Porte Dinamiche/Effimere (Ephemeral Ports, 49152 – 65535):** Assegnate automaticamente dal sistema operativo ai processi client per la durata della singola sessione di comunicazione.

---

## 4. Requisiti di Trasporto delle Applicazioni di Rete

Non tutte le applicazioni di rete hanno le medesime esigenze. Durante la progettazione di un protocollo applicativo, l'ingegnere deve valutare quattro dimensioni critiche di qualità del servizio (QoS):

```
+------------------------------------------------------------------------------------+
|                DIMENSIONI DEI REQUISITI DI TRASPORTO DELLE APPLICAZIONI            |
+------------------------------------------------------------------------------------+
| 1. AFFIDABILITÀ TRASMISSIVA (Loss-Tolerant vs Data Integrity)                     |
|    - App sensibili alle perdite: File Transfer, E-commerce, Email, Web.            |
|    - App tolleranti alle perdite: Audio/Video Streaming in tempo reale, Gaming.    |
+------------------------------------------------------------------------------------+
| 2. AMPIEZZA DI BANDA / THROUGHPUT (Bandwidth-Sensitive vs Elastic)                 |
|    - App sensibili alla banda: Video 4K/8K, realtà virtuale (richiedono throughput |
|      minimo garantito per evitare freeze).                                         |
|    - App elastiche: Web, Email, SSH (utilizzano qualunque banda disponibile).     |
+------------------------------------------------------------------------------------+
| 3. TEMPORIZZAZIONE E LATENZA (Timing / Latency-Sensitive)                          |
|    - Vincoli rigidi di ritardo end-to-end: Telefonia Internet (VoIP), Gaming online|
|      multiplayer, videoconferenze (ritardi > 150-200 ms causano deperimento).      |
+------------------------------------------------------------------------------------+
| 4. SICUREZZA (Confidentiality & Integrity)                                         |
|    - Crittografia end-to-end, autenticazione dell'endpoint, integrità dei messaggi.|
+------------------------------------------------------------------------------------+
```

### Matrice di Raffronto delle Applicazioni Tipiche

| Applicazione | Tolleranza alle Perdite Dati | Requisito di Throughput | Sensibilità al Ritardo (Latenza) |
|---|---|---|---|
| **Trasferimento File (FTP, SFTP)** | **Nessuna** (Integrità assoluta) | Elastico | No |
| **Pagine Web (HTTP / HTTPS)** | **Nessuna** (Testi/immagini intatti) | Elastico | Parziale (pochi secondi) |
| **Posta Elettronica (SMTP)** | **Nessuna** | Elastico | No |
| **Audio/Video Streaming (YouTube, Netflix)** | **Tollerante** a perdite lievi | Sensibile (Mbps minimi) | Moderata (buffer iniziale) |
| **Telefonia IP / Videoconferenza (VoIP)** | **Tollerante** a perdite lievi | Sensibile ($64 - 1000\text{ kbps}$) | **Critica** ($< 150\text{ ms}$) |
| **Giochi Online Interattivi** | **Tollerante** | Basso / Moderato | **Critica** ($< 50-100\text{ ms}$) |

---

## 5. Tassonomia dei Servizi di Trasporto Internet: TCP vs UDP

L'architettura Internet mette a disposizione del livello applicativo due protocolli di trasporto alternativi: **TCP** (*Transmission Control Protocol*) e **UDP** (*User Datagram Protocol*).

```
=============================================================================
                  CONFRONTO STRUTTURALE TRA TCP E UDP
=============================================================================
CRITERIO                | TCP (RFC 793)             | UDP (RFC 768)
------------------------+---------------------------+------------------------
Orientazione            | Orientato alla connessione| Privo di connessione
                        | (3-Way Handshake iniziale)| (Nessun handshake)
Stato dei nodi          | Stateful (Mantiene stato) | Stateless
Affidabilità dati       | Trasferimento Affidabile  | Best-effort
                        | (ACK, Ritrasmissioni, CRC)| (Possibile perdita/dupl.)
Ordinamento messaggi    | Garantito in sequenza     | Non garantito
Controllo di Flusso     | Sì (Finestra di ricezione)| No
Controllo Congestione   | Sì (Adatta tasso di invio)| No (Spara a piena banda)
Overhead Header         | Alto (minimo 20 byte)     | Bassissimo (8 byte)
Garanzia di Throughput  | NESSUNA                   | NESSUNA
Garanzia Temporale      | NESSUNA                   | NESSUNA
=============================================================================
```

> [!IMPORTANT] Cosa NON Forniscono né TCP né UDP
> Gli studenti d'esame commettono frequentemente l'errore di pensare che TCP fornisca garanzie temporali o di banda. **Né TCP né UDP forniscono alcuna garanzia di ampiezza di banda minima né garanzie di ritardo massimo (latenza limitata)**. L'architettura Internet originale adotta il modello di servizio a pacchetto *Best-Effort*.

---

## 6. Cosa Definisce un Protocollo di Livello Applicativo?

Un **protocollo di livello applicativo** formalizza il modo in cui i processi residenti su host distinti si scambiano messaggi. In particolare, specifica:
1. **Tipi di messaggi scambiati:** Ad esempio messaggi di richiesta (*Request*) e messaggi di risposta (*Response*).
2. **Sintassi dei messaggi:** Quali campi compongono il messaggio e come sono delimitati (formato ASCII leggibile dall'uomo oppure binario compatto).
3. **Semantica dei campi:** Il significato rigoroso delle informazioni contenute all'interno dei campi.
4. **Regole di invio e risposta:** Quando un processo deve inviare un messaggio e come deve reagire all'arrivo di una risposta o al verificarsi di un timeout.

### Standard Pubblici (RFC) vs Proprietari
- **Protocolli Pubblici (Standard IETF):** Definiti in documenti RFC aperti, accessibili a tutti. Garantiscono l'**interoperabilità** tra implementazioni e sistemi operativi eterogenei (es. un browser Firefox su Linux comunica perfettamente con un server Apache su Windows via HTTP/1.1 o HTTP/2).
- **Protocolli Proprietari:** Creati da singole aziende con specifiche segrete e non aperte, volti a proteggere il modello di business o accelerare funzionalità sperimentali (es. il protocollo originale Skype o WhatsApp).

---

## 7. Quadro Sinottico delle Applicazioni e dei Relativi Protocolli

| Servizio / Applicazione | Protocollo Livello Applicativo | Protocollo Livello Trasporto | Modalità Connessione |
|---|---|---|---|
| **Navigazione Web** | HTTP / HTTPS | TCP | Affidabile, connesso |
| **Posta Elettronica** | SMTP (invio), POP3 / IMAP (ricezione) | TCP | Affidabile, connesso |
| **Trasferimento File** | FTP / SFTP | TCP | Affidabile, connesso |
| **Risoluzione Nomi** | DNS | UDP (prevalente per query veloci) / TCP (zone transfer) | Veloce, leggero |
| **Streaming Web Moderno** | HTTP (DASH) | TCP (HTTP/2) o UDP via QUIC (HTTP/3) | Ibrido / Adattivo |
| **Videoconferenza / VoIP** | SIP, RTP, protocolli proprietari | UDP o TCP incapsulato | Priorità alla bassa latenza |

---

## 8. Domande di Verifica ed Esercizi di Consolidamento d'Esame

### Domanda 1: Perché i router e gli switch all'interno del nucleo della rete (Core) non implementano il livello di applicazione?
**Risposta Didattica:** 
Per rispettare il principio architetturale di **separazione delle responsabilità** e garantire prestazioni estreme. Il compito del *Network Core* è esclusivamente quello di commutare e instradare i pacchetti il più velocemente possibile tra nodi intermedi. Elaborare intestazioni o dati applicativi richiederebbe una decapsulazione completa fino al Livello 7, aumentando drasticamente la complessità hardware, la memoria richiesta e i ritardi di elaborazione nei nodi dorsali.

### Domanda 2: Per quale motivo un'applicazione di streaming video in tempo reale o una chiamata VoIP potrebbe preferire UDP a TCP?
**Risposta Didattica:** 
Perché TCP privilegia l'affidabilità assoluta rispetto alla tempestività temporale. In caso di perdita di un pacchetto, TCP blocca l'avanzamento dei dati successivi (*Head-of-Line blocking*) e attende la ritrasmissione del pacchetto mancante. Nelle conversazioni in tempo reale, un frammento audio che arriva con $300\text{ ms}$ di ritardo è del tutto inutile ed introduce fastidiosi balbettii o freeze. UDP trasmette immediatamente i dati senza handshake e senza ritrasmissioni, tollerando perdite minime impercettibili all'orecchio umano.

### Domanda 3: Quali sono i due parametri indispensabili che una socket deve conoscere per recapitare un messaggio a un processo remoto?
**Risposta Didattica:** 
1. L'**Indirizzo IP** dell'host di destinazione, necessario al livello di rete (IP) per instradare il pacchetto attraverso Internet fino all'interfaccia dell'host corretto.
2. Il **Numero di Porta** di destinazione, indispensabile al livello di trasporto (TCP o UDP) per demultiplexare il segmento ed indirizzarlo precisamente alla socket del processo applicativo ricevente.

---

## Riferimenti Incrociati

- [[00 - Introduzione alle Reti di Calcolatori]]
- [[01 - Architettura a Livelli e Protocolli]]
- [[Capitolo 2 - Livello di Applicazione]]
- [[04 - World Wide Web e Protocollo HTTP]]
