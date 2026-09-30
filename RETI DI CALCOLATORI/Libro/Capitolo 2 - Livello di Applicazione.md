---
tags: [reti, capitolo-2, applicazione, kurose-ross]
capitolo: 2
titolo: Livello di Applicazione
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# Capitolo 2 — Livello di Applicazione

> [!NOTE] Dove vivono le app
> Il livello applicazione è dove risiedono le applicazioni di rete. I protocolli qui definiscono il formato e l'ordine dei messaggi scambiati tra processi su host diversi.

---

## 2.1 Principi delle Applicazioni di Rete

### Architetture delle Applicazioni

**Client-Server:**
- Server sempre attivo, IP fisso, scalabile con data center
- Client comunicano con il server, non tra loro
- Esempi: HTTP (Web), FTP, email (SMTP)

**Peer-to-Peer (P2P):**
- Nessun server dedicato — gli host si parlano direttamente
- Auto-scalante: più peer = più capacità
- Esempi: BitTorrent, Skype (originale)
- Sfide: sicurezza, affidabilità, incentivi

### Processi e Socket

- **Processo** = programma in esecuzione su un host
- Comunicazione inter-processo sullo stesso host → SO
- Comunicazione tra host diversi → **messaggi di rete**
- **Socket** = porta tra il processo e la rete (API)
  - Il processo invia/riceve messaggi attraverso il socket
  - Analogia: porta di casa

### Indirizzamento dei Processi

Per ricevere messaggi, un processo ha bisogno di:
1. **Indirizzo IP** — identifica l'host (32 bit in IPv4)
2. **Numero di porta** — identifica il processo sull'host
   - HTTP: 80 | HTTPS: 443 | SMTP: 25 | DNS: 53

### Servizi di Trasporto Richiesti dalle App

| Requisito | Descrizione | App che lo richiedono |
|---|---|---|
| **Affidabilità** | Dati arrivano corretti e completi | Email, file transfer, web |
| **Throughput** | Banda minima garantita | Video streaming, VoIP |
| **Temporizzazione** | Ritardo massimo garantito | Gaming, VoIP |
| **Sicurezza** | Cifratura, integrità | Banche, e-commerce |

### TCP vs UDP per le App

| | TCP | UDP |
|---|---|---|
| Orientato alla connessione | ✓ | ✗ |
| Trasferimento affidabile | ✓ | ✗ |
| Controllo del flusso | ✓ | ✗ |
| Controllo della congestione | ✓ | ✗ |
| Temporizzazione garantita | ✗ | ✗ |
| Throughput garantito | ✗ | ✗ |
| Uso tipico | Web, email, FTP | Streaming, DNS, VoIP |

---

## 2.2 Il Web e HTTP

### HTTP (HyperText Transfer Protocol)

- Livello applicazione del Web
- Modello **client-server**: browser (client) ↔ web server
- Usa **TCP** come trasporto (porta 80/443)
- **Stateless** — il server non mantiene stato del client tra richieste

**Connessioni HTTP:**

| Tipo | Descrizione | Pro/Con |
|---|---|---|
| **Non persistente** | Nuova connessione TCP per ogni oggetto | + Semplice, − Lento (2 RTT/oggetto) |
| **Persistente** | Stessa connessione TCP per più oggetti | + Efficiente, − più stato |
| **Persistente con pipelining** | Più richieste senza aspettare risposta | + Molto efficiente |

**Ritardo HTTP non persistente:**
- 2 RTT + tempo trasmissione file = RTT (SYN) + RTT (HTTP req/primo byte) + T_trasmissione

### Formato Messaggi HTTP

**Richiesta:**
```
GET /index.html HTTP/1.1\r\n
Host: www.esempio.it\r\n
Connection: keep-alive\r\n
Accept-Language: it\r\n
\r\n
```

**Risposta:**
```
HTTP/1.1 200 OK\r\n
Content-Type: text/html\r\n
Content-Length: 6821\r\n
\r\n
[corpo del messaggio...]
```

**Metodi HTTP principali:**
- `GET` — richiede risorsa (parametri nell'URL)
- `POST` — invia dati al server (nel corpo)
- `PUT` — carica risorsa
- `DELETE` — elimina risorsa
- `HEAD` — come GET ma senza corpo (solo headers)

**Codici di stato principali:**
| Codice | Significato |
|---|---|
| 200 OK | Successo |
| 301 Moved Permanently | Redirect permanente |
| 304 Not Modified | Oggetto in cache valido |
| 400 Bad Request | Richiesta malformata |
| 404 Not Found | Risorsa non trovata |
| 505 HTTP Version Not Supported | |

### Cookie

Meccanismo per aggiungere **stato** a HTTP:
1. Server invia `Set-Cookie: id=1678` nella risposta
2. Browser salva il cookie
3. Ogni richiesta successiva include `Cookie: id=1678`
4. Server identifica l'utente tramite l'id

Usi: sessioni di login, carrelli, personalizzazione  
Controversia: privacy (tracking terze parti)

### Web Cache (Proxy)

Il proxy si interpone tra client e server:
- Soddisfa richieste con **copie locali** (riduce traffico verso Internet)
- **Hit rate** = % richieste soddisfatte dalla cache
- Riduce RTT e banda dell'ISP

**Intestazione If-Modified-Since:**
- Il client chiede: "l'oggetto è cambiato da questa data?"
- Server risponde 304 (non modificato) o 200 + nuovo oggetto
- Evita trasferimento dati non necessario

### HTTP/2 e HTTP/3

**HTTP/2:**
- Multiplexing — più richieste sulla stessa connessione TCP (no Head-of-Line blocking applicativo)
- Compressione degli header (HPACK)
- Server push — il server invia risorse prima che il client le chieda

**HTTP/3:**
- Usa **QUIC** (su UDP) invece di TCP
- Elimina HoL blocking a livello di trasporto
- Migliore gestione della perdita di pacchetti

---

## 2.3 DNS — Domain Name System

### Funzione

Traduce nomi di dominio leggibili dall'uomo in indirizzi IP:
```
www.google.com  →  142.250.185.78
```

### Architettura Distribuita e Gerarchica

```
              ┌──────────────────┐
              │   Root DNS       │  ~13 cluster, gestiti da 12 organizzazioni
              └──────┬───────────┘
        ┌────────────┼────────────┐
   ┌────┴───┐   ┌────┴───┐   ┌───┴────┐
   │  .com  │   │  .org  │   │  .edu  │  ← TLD (Top-Level Domain)
   └────┬───┘   └────┬───┘   └───┬────┘
   ┌────┴───┐   ┌────┴───┐
   │amazon  │   │pbs     │           ← Authoritative DNS
   │.com    │   │.org    │
   └────────┘   └────────┘
```

**Tipi di server DNS:**
1. **Root DNS** — 13 indirizzi IP (cluster ridondanti), conosce i TLD
2. **TLD DNS** — gestisce .com, .org, .it, .edu ecc.
3. **Authoritative DNS** — fornisce IP definitivi per un dominio
4. **Local DNS** — il tuo resolver (ISP o 8.8.8.8), non appartiene alla gerarchia

### Query DNS: Iterativa vs Ricorsiva

**Query iterativa (tipica):**
```
Host → Local DNS: "chi è www.amazon.com?"
Local DNS → Root: "chi è .com?"
Root → Local DNS: "chiedi ai TLD .com"
Local DNS → TLD .com: "chi è amazon.com?"
TLD → Local DNS: "chiedi all'authoritative di Amazon"
Local DNS → Auth Amazon: "chi è www.amazon.com?"
Auth → Local DNS: 205.251.242.103
Local DNS → Host: 205.251.242.103
```

**DNS Caching:**
- I local DNS cachano le risposte per il TTL specificato
- Riduce drasticamente il traffico DNS globale
- I Root server vengono raramente interrogati grazie alla cache

### Record DNS (Resource Records)

```
(Name, Value, Type, TTL)
```

| Tipo | Name | Value | Uso |
|---|---|---|---|
| **A** | hostname | indirizzo IPv4 | Mappa nome → IP |
| **AAAA** | hostname | indirizzo IPv6 | Mappa nome → IPv6 |
| **NS** | dominio | hostname del DNS auth | Delega |
| **CNAME** | alias | nome canonico | Alias |
| **MX** | dominio | mail server | Email |

### Sicurezza DNS

- **DNS amplification attack** — attacchi DDoS sfruttando risposte DNS amplificate
- **DNS cache poisoning** — inserire record falsi nella cache
- **DNSSEC** — firma crittografica dei record DNS

---

## 2.4 Email — SMTP, POP3, IMAP

### Componenti del Sistema Email

```
[Alice] → [User Agent] → [Mail Server Alice] → [Mail Server Bob] → [User Agent] → [Bob]
              SMTP →              SMTP →                  ← POP3/IMAP
```

- **User Agent** — client email (Outlook, Thunderbird, Gmail web)
- **Mail Server** — invia, riceve, memorizza email (coda di uscita + mailbox)
- **SMTP** — protocollo per inviare email tra mail server

### SMTP (Simple Mail Transfer Protocol)

- Usa **TCP**, porta **25**
- Trasferimento **push** (il mittente "spinge" verso il destinatario)
- Messaggi in **ASCII 7 bit** (MIME estende per allegati binari)
- **Connessioni persistenti** tra server dello stesso ISP

**Dialogo SMTP:**
```
S: 220 hamburger.edu
C: HELO crepes.fr
S: 250 Hello crepes.fr, pleased to meet you
C: MAIL FROM: <alice@crepes.fr>
S: 250 alice@crepes.fr... Sender ok
C: RCPT TO: <bob@hamburger.edu>
S: 250 bob@hamburger.edu... Recipient ok
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Do you like ketchup?
C: .
S: 250 Message accepted for delivery
C: QUIT
S: 221 hamburger.edu closing connection
```

### POP3 vs IMAP vs Webmail

| Protocollo | Caratteristiche |
|---|---|
| **POP3** | Download email → elimina dal server. Stateless. Semplice. 3 fasi: autorizzazione, transazione, aggiornamento |
| **IMAP** | Email rimangono sul server. Gestione cartelle remote. Stateful. Più complesso ma potente |
| **Webmail** | HTTP tra browser e mail server. Email sempre sul server |

---

## 2.5 DNS per Applicazioni Peer-to-Peer

### BitTorrent

**Architettura:**
- **Torrent** = insieme di peer che si scambiano chunk di un file
- **Tracker** — server che tiene traccia dei peer nel torrent
- Ogni file diviso in chunk da 256 KB

**Meccanismi chiave:**

*Rarest First* — scarica prima i chunk più rari nel torrent → migliora disponibilità

*Tit-for-Tat* (incentivo alla collaborazione):
- Alice "sblocca" i 4 peer che la forniscono con la banda maggiore (unchoked)
- Ogni 30s sceglie un peer a caso (optimistic unchoke) → scopre peer migliori
- Chi non fornisce dati viene "soffocato" (choked)

---

## 2.6 Streaming Video e CDN

### Streaming Video Adattivo (DASH)

**DASH** (Dynamic Adaptive Streaming over HTTP):
- Video codificato a multiple velocità (bit rate)
- Suddiviso in chunk (tipicamente 2-10 secondi)
- Client misura banda disponibile e sceglie la qualità chunk per chunk
- File **manifest** descrive gli URL dei chunk per ogni qualità

**Logica del client DASH:**
```
mentre (video in riproduzione):
    misura banda disponibile
    seleziona versione con bitrate < banda
    scarica chunk successivo in quella versione
```

### CDN (Content Delivery Network)

**Problema:** Come distribuire video a milioni di utenti simultanei nel mondo?

**Soluzione CDN:**
- Rete di **migliaia di server** distribuiti geograficamente
- Contenuto replicato nei server CDN (push o pull)
- Utente servito dal server CDN più vicino/efficiente

**Architetture CDN:**
- **Enter Deep** — server CDN dentro le reti degli ISP di accesso (vicini agli utenti)
- **Bring Home** — server CDN grandi in pochi punti (IXP e data center)

**Selezione del server CDN:**
Il DNS del CDN redireziona il client al server appropriato basandosi su:
- Geolocalizzazione IP
- Condizioni di rete misurate
- Carico corrente dei server

---

## Concetti Chiave

> [!IMPORTANT] Da sapere
> - **HTTP è stateless** — i cookie aggiungono stato
> - **DNS è distribuito e gerarchico** — root → TLD → authoritative → local
> - **SMTP è push, POP3/IMAP è pull**
> - **DASH** gestisce adattività dello streaming lato client
> - **CDN** porta i contenuti vicino agli utenti

## Formule Essenziali

| Situazione | Formula |
|---|---|
| HTTP non persistente (senza pipelining) | `2 RTT + T_trasm` per ogni oggetto |
| HTTP persistente con pipelining | `RTT + T_trasm` per N oggetti (dopo prima connessione) |

## Link Interni

- [[Capitolo 1 - Reti di Calcolatori e Internet]]
- [[Capitolo 3 - Livello di Trasporto]]
