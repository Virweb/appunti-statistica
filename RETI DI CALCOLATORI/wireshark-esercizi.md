# 🦈 Wireshark — Esercizi Pratici per LO

> Ogni esercizio ha: **obiettivo**, **setup**, **istruzioni**, **domande** e **soluzione completa**.
> Fai prima l'esercizio da solo, poi scorri alla soluzione per verificare.

---

## Indice

### 🟢 Livello Principiante
1. [Prima cattura e filtro DNS](#esercizio-1--prima-cattura-e-filtro-dns)
2. [Identificare il tuo IP e MAC address](#esercizio-2--identificare-il-tuo-ip-e-mac-address)
3. [Analizzare un ping ICMP](#esercizio-3--analizzare-un-ping-icmp)
4. [Seguire una connessione HTTP](#esercizio-4--seguire-una-connessione-http)
5. [Vedere cosa fa un'app in background](#esercizio-5--vedere-cosa-fa-unapp-in-background)

### 🟡 Livello Intermedio
6. [Analizzare il TCP Three-Way Handshake](#esercizio-6--analizzare-il-tcp-three-way-handshake)
7. [Trovare richieste HTTP POST](#esercizio-7--trovare-richieste-http-post)
8. [Mappare i dispositivi in rete con ARP e mDNS](#esercizio-8--mappare-i-dispositivi-in-rete-con-arp-e-mdns)
9. [Usare Protocol Hierarchy per capire il traffico](#esercizio-9--usare-protocol-hierarchy-per-capire-il-traffico)
10. [Analizzare traffico QUIC di YouTube](#esercizio-10--analizzare-traffico-quic-di-youtube)

### 🔴 Livello Avanzato
11. [Analisi forense di un PCAP pubblico](#esercizio-11--analisi-forense-di-un-pcap-pubblico)
12. [Ricostruire un file trasferito via HTTP](#esercizio-12--ricostruire-un-file-trasferito-via-http)
13. [Trovare credenziali in chiaro con tshark](#esercizio-13--trovare-credenziali-in-chiaro-con-tshark)
14. [Rilevare una scansione di rete](#esercizio-14--rilevare-una-scansione-di-rete)

---

---

# 🟢 LIVELLO PRINCIPIANTE

---

## Esercizio 1 — Prima cattura e filtro DNS

### 🎯 Obiettivo
Capire come avviare una cattura, fermarla, e usare un filtro di base per isolare solo il traffico DNS.

### 🛠️ Setup
- Wireshark installato e aperto
- Connessione internet attiva

### 📋 Istruzioni

1. Apri Wireshark
2. Seleziona la tua interfaccia attiva (WiFi o Ethernet) — quella con il grafico che si muove
3. Avvia la cattura cliccando il **requin blu** (shark fin) in alto a sinistra
4. Apri il browser e visita questi 3 siti uno dopo l'altro:
   - `https://www.wikipedia.org`
   - `https://www.github.com`
   - `https://www.reddit.com`
5. Torna su Wireshark e ferma la cattura (quadrato rosso)

### ❓ Domande

1. Quanti pacchetti totali hai catturato?
2. Scrivi nella barra filtri: `dns` — quanti pacchetti rimangono visibili?
3. Riesci a trovare la query DNS per `wikipedia.org`? Che tipo di query è (A o AAAA)?
4. Qual è l'IP che Wireshark ha restituito per `github.com`?
5. Qual è l'IP del server DNS che sta rispondendo alle tue query?

---

### ✅ SOLUZIONE

**1. Pacchetti totali**
Dipende da quanto traffico hai in background — di solito tra 500 e 5000 in 30 secondi. Non c'è una risposta unica, l'importante è che siano tanti.

**2. Filtro `dns`**
Dopo il filtro vedi solo le query e le risposte DNS — di solito qualche decina. Tutto il resto scompare dalla vista (ma non è cancellato).

**3. Query per wikipedia.org**
Filtro più preciso:
```
dns && dns.qry.name contains "wikipedia"
```
Trovi una riga `Standard query A wikipedia.org` — tipo **A** = richiesta indirizzo IPv4.
Potresti trovare anche una query **AAAA** = IPv6. Entrambe sono normali.

**4. IP di github.com**
Clicca sulla riga `Standard query response A github.com` → nel pannello Packet Details espandi:
```
▼ Domain Name System (response)
  ▼ Answers
      github.com: type A, class IN, addr X.X.X.X
```
L'IP è lì. GitHub di solito risponde con `140.82.121.x`.

**5. IP del server DNS**
Guarda la colonna **Source** delle righe con `Standard query response` — quell'IP è il tuo server DNS. Di solito è:
- `192.168.1.1` → il tuo router (che fa da DNS proxy)
- `8.8.8.8` → Google DNS (se configurato manualmente)
- `1.1.1.1` → Cloudflare DNS

---

## Esercizio 2 — Identificare il tuo IP e MAC address

### 🎯 Obiettivo
Usare Wireshark per ricavare l'indirizzo IP e MAC del tuo PC direttamente dai pacchetti, senza usare `ipconfig`.

### 🛠️ Setup
- Wireshark aperto
- Connessione internet attiva

### 📋 Istruzioni

1. Avvia una cattura sulla tua interfaccia attiva
2. Aspetta 10 secondi senza fare nulla
3. Ferma la cattura
4. Applica il filtro: `arp`

### ❓ Domande

1. Riesci a trovare il tuo indirizzo IP locale guardando i pacchetti ARP?
2. Riesci a trovare il tuo MAC address?
3. Qual è il MAC address del tuo router (gateway)?
4. Cosa significa `Gratuitous ARP` se ne vedi uno?
5. Verifica: apri un CMD e scrivi `ipconfig /all` — i valori corrispondono?

---

### ✅ SOLUZIONE

**1 e 2. Trovare IP e MAC dal traffico ARP**
Nei pacchetti ARP guarda la colonna **Info**:
```
Who has 192.168.1.1? Tell 192.168.1.10
```
→ Il tuo IP è `192.168.1.10` (quello dopo "Tell")

Nel Packet Details clicca il pacchetto ARP → espandi `Address Resolution Protocol`:
```
Sender MAC address: aa:bb:cc:dd:ee:ff   ← il tuo MAC
Sender IP address:  192.168.1.10        ← il tuo IP
Target IP address:  192.168.1.1         ← il router
```

**3. MAC del router**
Nelle ARP Reply trovi:
```
192.168.1.1 is at xx:xx:xx:xx:xx:xx
```
Quel MAC è il tuo router/gateway.

**4. Gratuitous ARP**
È un ARP Reply non richiesto — un dispositivo annuncia "sono io ad avere questo IP" senza che nessuno abbia chiesto. Normale al boot o dopo una riconnessione. Segnale sospetto se arriva continuamente da un MAC non conosciuto (possibile ARP spoofing).

**5. Verifica con ipconfig**
```
ipconfig /all
```
I valori devono corrispondere esattamente a quelli trovati in Wireshark.

---

## Esercizio 3 — Analizzare un ping ICMP

### 🎯 Obiettivo
Catturare e leggere un ping, capire la struttura Request/Reply e misurare il RTT (Round Trip Time).

### 🛠️ Setup
- Wireshark aperto
- PowerShell o CMD aperto di fianco

### 📋 Istruzioni

1. Avvia la cattura in Wireshark
2. Apri PowerShell e digita:
   ```powershell
   ping 8.8.8.8 -n 5
   ```
   (manda 5 ping a Google DNS)
3. Aspetta che finisca
4. Ferma la cattura
5. Applica il filtro: `icmp`

### ❓ Domande

1. Quanti pacchetti ICMP vedi? (dovrebbero essere 10 — 5 request + 5 reply)
2. Qual è il TTL nelle Echo Reply di Google?
3. Quanto è grande il payload di ogni pacchetto ping? (in byte)
4. Riesci a calcolare il RTT (tempo tra Request e Reply) guardando la colonna Time?
5. Cosa succede se pinghi un IP inesistente tipo `10.0.0.99`? Fai un secondo ping e osserva.

---

### ✅ SOLUZIONE

**1. 10 pacchetti ICMP**
5 `Echo (ping) request` + 5 `Echo (ping) reply` = 10 totali.
Filtro preciso:
```
icmp && ip.addr == 8.8.8.8
```

**2. TTL nelle Reply di Google**
Clicca su una Echo Reply → Packet Details → `Internet Protocol`:
```
Time to Live: 118  (o 119, dipende dal routing)
```
Google parte con TTL 128 (Windows) o 64 (Linux). Se arriva a te con 118, ha attraversato ~10 router.

**3. Dimensione payload**
Windows manda ping con **32 byte** di payload di default.
Nel Packet Details → `Internet Control Message Protocol → Data (32 bytes)`

**4. Calcolare RTT**
Guarda i timestamp nella colonna Time:
- Request al tempo `T`
- Reply al tempo `T + 0.012` (esempio)
→ RTT = 12ms

Oppure usa `Edit → Find Packet` o guarda `tcp.time_delta` — per ICMP leggi direttamente i timestamp.

**5. Ping a IP inesistente**
Se l'IP è nella tua subnet (es. `192.168.1.99` non esistente) → vedi solo ARP Request senza risposta, poi nessun ICMP.
Se è esterno (`10.0.0.99`) → potresti vedere ICMP `Destination Unreachable` dal router.

---

## Esercizio 4 — Seguire una connessione HTTP

### 🎯 Obiettivo
Usare la funzione "Follow HTTP Stream" per leggere una conversazione HTTP completa in chiaro.

### 🛠️ Setup
- Wireshark aperto
- Browser disponibile

> [!NOTE]
> Servono siti in HTTP puro (non HTTPS). Usa questi che ancora rispondono in HTTP:
> - `http://example.com`
> - `http://httpbin.org/get`
> - `http://neverssl.com`

### 📋 Istruzioni

1. Avvia la cattura
2. Apri il browser e vai su `http://example.com`
3. Ferma la cattura
4. Filtro: `http`
5. Trova la riga con `GET / HTTP/1.1`
6. Click destro → **Follow → HTTP Stream**

### ❓ Domande

1. Che metodo HTTP ha usato il browser per richiedere la pagina?
2. Qual è il valore dell'header `Host`?
3. Qual è il `User-Agent` del tuo browser?
4. Qual è il codice di risposta del server?
5. Riesci a leggere l'HTML della risposta nella finestra Follow Stream?
6. Di che colore è il testo della richiesta (client → server) e quello della risposta (server → client)?

---

### ✅ SOLUZIONE

**1. Metodo HTTP**
`GET` — il browser richiede la pagina con una GET request.

**2. Header Host**
```
Host: example.com
```
Indica al server quale dominio si sta richiedendo (essenziale per virtual hosting).

**3. User-Agent**
Qualcosa tipo:
```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 ...
```
Identifica il browser, la versione e il sistema operativo. Un server può bloccare o modificare risposte in base a questo.

**4. Codice di risposta**
```
HTTP/1.1 200 OK
```
200 = successo. Altri codici comuni: 301 (redirect), 404 (non trovato), 500 (errore server).

**5. HTML nella risposta**
Sì — nella parte blu (risposta del server) vedi il codice HTML della pagina di example.com:
```html
<!doctype html>
<html>
<head><title>Example Domain</title>...
```

**6. Colori nel Follow Stream**
- 🔴 **Rosso** = dati dal client (il tuo browser → server)
- 🔵 **Blu** = dati dal server (server → il tuo browser)

---

## Esercizio 5 — Vedere cosa fa un'app in background

### 🎯 Obiettivo
Scoprire a quali server si connette un'applicazione specifica usando DNS e IP filtering.

### 🛠️ Setup
- Wireshark aperto
- Un'app a scelta (Discord, Spotify, Steam, qualsiasi)

### 📋 Istruzioni

1. **Chiudi** l'app che vuoi analizzare (se è aperta)
2. Avvia la cattura in Wireshark
3. **Apri** l'app e lasciala avviare completamente
4. Aspetta 30 secondi senza fare nulla nell'app
5. Ferma la cattura
6. Filtro: `dns`

### ❓ Domande

1. Quanti domini diversi ha contattato l'app durante l'avvio?
2. Riesci a identificare il dominio principale dell'app? (es. `discord.com`, `spotify.com`...)
3. Ci sono domini di terze parti? (analytics, CDN, telemetria...)
4. Trova l'IP principale del server dell'app — usa il DNS per trovarlo, poi filtra con `ip.addr == [IP]`
5. Quanti byte ha trasferito l'app in 30 secondi di idle?

---

### ✅ SOLUZIONE

**1. Domini contattati**
Usa:
```
dns && dns.flags.response == 0
```
(solo le query, non le risposte — evita duplicati)

Poi `Statistics → Endpoints → UDP` per una vista pulita, oppure conta le righe DNS filtrate.

**2. Dominio principale**
Di solito il più frequente nelle query. Per Discord: `discord.com`, `gateway.discord.gg`. Per Spotify: `api.spotify.com`, `apresolve.spotify.com`.

**3. Domini di terze parti**
Quasi tutte le app contattano:
- Servizi di analytics (Amplitude, Segment, Mixpanel)
- CDN (Cloudflare, Fastly, Akamai)
- Crash reporting (Sentry, Crashlytics)
- Pubblicità (se è un'app free)

**4. Trovare l'IP e filtrare il traffico**
```
dns && dns.qry.name contains "spotify"
```
Clicca una risposta DNS → nel Packet Details vedi l'IP nella sezione Answers.
Poi:
```
ip.addr == [IP trovato]
```

**5. Byte trasferiti**
`Statistics → Conversations → TCP`
Ordina per Bytes — trova la conversazione verso l'IP dell'app → colonna `Bytes A→B` e `Bytes B→A`.

---
---

# 🟡 LIVELLO INTERMEDIO

---

## Esercizio 6 — Analizzare il TCP Three-Way Handshake

### 🎯 Obiettivo
Capire e identificare le 3 fasi del TCP handshake, leggere i flag e i numeri di sequenza.

### 🛠️ Setup
- Wireshark aperto

### 📋 Istruzioni

1. Avvia la cattura
2. Apri PowerShell e digita:
   ```powershell
   Invoke-WebRequest http://example.com
   ```
3. Aspetta la risposta
4. Ferma la cattura
5. Filtro:
   ```
   tcp && ip.addr == [IP di example.com]
   ```
   (se non sai l'IP, filtra `dns contains "example"` prima per trovarlo)

### ❓ Domande

1. Identifica i 3 pacchetti del handshake: quale ha flag SYN, quale SYN-ACK, quale ACK?
2. Qual è il numero di sequenza iniziale (ISN) del client?
3. Qual è il numero di sequenza iniziale del server?
4. Dopo il handshake, cosa succede? Quali flag vedi nei pacchetti successivi?
5. Identifica il FIN handshake — come si chiude la connessione?

---

### ✅ SOLUZIONE

**Filtro preciso per vedere solo SYN e SYN-ACK:**
```
tcp.flags.syn == 1
```

**1. I 3 pacchetti del handshake**

| Pacchetto | Direzione | Flag | Significato |
|---|---|---|---|
| 1 | Client → Server | `SYN` | "Voglio connettermi, il mio ISN è X" |
| 2 | Server → Client | `SYN, ACK` | "Ok! Il mio ISN è Y, confermo il tuo X" |
| 3 | Client → Server | `ACK` | "Ricevuto, siamo connessi" |

**2. ISN del client**
Clicca sul pacchetto SYN → Packet Details → TCP:
```
Sequence Number (raw): 123456789   ← ISN del client
```

**3. ISN del server**
Clicca sul SYN-ACK → TCP:
```
Sequence Number (raw): 987654321   ← ISN del server
Acknowledgment Number: 123456790   ← ISN client + 1 (conferma)
```

**4. Dopo il handshake**
Vedi pacchetti con flag `PSH, ACK` — dati veri inviati (la GET request HTTP e la risposta HTML).

**5. FIN handshake — chiusura**
```
tcp.flags.fin == 1
```
Trovi 4 pacchetti: `FIN, ACK` → `ACK` → `FIN, ACK` → `ACK`
(chiusura a 4 vie, simmetrica al handshake di apertura)

---

## Esercizio 7 — Trovare richieste HTTP POST

### 🎯 Obiettivo
Intercettare una richiesta POST HTTP e leggere i dati inviati, incluse potenziali credenziali in chiaro.

### 🛠️ Setup
- Wireshark aperto
- Browser

> [!NOTE]
> Sito di test con login HTTP (non HTTPS): usa `http://httpbin.org/post` per simulare una POST, oppure cerca "http login test page" — molti laboratori di sicurezza offrono pagine HTTP di test.

### 📋 Istruzioni

**Metodo A — Simula una POST con PowerShell:**

1. Avvia la cattura
2. Apri PowerShell e digita:
```powershell
Invoke-WebRequest -Uri "http://httpbin.org/post" `
  -Method POST `
  -Body "username=mario&password=supersegreta123" `
  -ContentType "application/x-www-form-urlencoded"
```
3. Ferma la cattura
4. Filtro: `http.request.method == "POST"`

### ❓ Domande

1. Riesci a trovare il pacchetto POST?
2. Segui il Follow HTTP Stream — riesci a vedere `username=mario&password=supersegreta123` in chiaro?
3. Nella richiesta, qual è il valore dell'header `Content-Type`?
4. Qual è la dimensione del body inviato (Content-Length)?
5. Cosa risponde il server (httpbin.org)? Riesci a leggere la risposta JSON?

---

### ✅ SOLUZIONE

**1. Trovare la POST**
```
http.request.method == "POST"
```
Una sola riga — la tua richiesta POST a httpbin.org.

**2. Credenziali in chiaro**
Follow HTTP Stream → nella parte rossa (client):
```
POST /post HTTP/1.1
Host: httpbin.org
Content-Type: application/x-www-form-urlencoded
Content-Length: 38

username=mario&password=supersegreta123
```
**Sì, tutto leggibile in chiaro.** Questo è esattamente il motivo per cui HTTP senza TLS è pericoloso.

**3. Content-Type**
```
application/x-www-form-urlencoded
```
Il formato classico dei form HTML.

**4. Content-Length**
`38` — la lunghezza in byte della stringa `username=mario&password=supersegreta123`.

**5. Risposta JSON di httpbin**
httpbin.org fa echo dei dati ricevuti in formato JSON:
```json
{
  "form": {
    "password": "supersegreta123",
    "username": "mario"
  },
  "url": "http://httpbin.org/post"
}
```
Leggi tutto nella parte blu (server) del Follow Stream.

---

## Esercizio 8 — Mappare i dispositivi in rete con ARP e mDNS

### 🎯 Obiettivo
Scoprire tutti i dispositivi attivi nella tua LAN, i loro IP, MAC address, nomi host e produttori hardware — solo con Wireshark passivo.

### 🛠️ Setup
- Wireshark aperto
- Più dispositivi connessi alla tua rete (telefono, tablet, smart TV, PC ecc.)

### 📋 Istruzioni

1. Avvia la cattura
2. Aspetta **2-3 minuti** senza fare nulla (lascia che i dispositivi parlino da soli)
3. Ferma la cattura
4. Crea una tabella con quello che trovi

### ❓ Domande

1. Quanti dispositivi distinti riesci a identificare?
2. Usando `Statistics → Endpoints → Ethernet`, trova tutti i MAC address presenti
3. Filtra `mdns` — quanti dispositivi si annunciano? Cosa espongono?
4. Filtra `nbns` — ci sono PC Windows nella rete? Quali nomi hanno?
5. Riesci a indovinare il produttore di ogni dispositivo dal MAC address? (primi 3 byte)

---

### ✅ SOLUZIONE

**1. Trovare dispositivi via ARP**
```
arp
```
Ogni riga ARP Request rivela un IP nella tua subnet. Colonne: `Sender IP` = chi chiede, `Target IP` = chi cerca.

**2. Statistics → Endpoints → Ethernet**
Lista tutti i MAC address visti nella cattura con byte inviati/ricevuti.
- I primi 3 byte (OUI) identificano il produttore
- Lookup su [maclookup.app](https://maclookup.app): es. `B8:27:EB` = Raspberry Pi, `AC:DE:48` = Apple, `FC:AA:14` = Google (Chromecast/Nest)

**3. mDNS — dispositivi Apple, Android, IoT**
```
mdns
```
Cerca righe con `_services._dns-sd._udp.local` — annuncio servizi.
Vedi:
- `_airplay._tcp` → Apple TV o Mac con AirPlay
- `_googlecast._tcp` → Chromecast / Google Home
- `_http._tcp` → qualcosa con un web server (router, NAS, stampante)
- `_smb._tcp` → condivisione file (Mac o Linux)

**4. NetBIOS — PC Windows**
```
nbns
```
Ogni PC Windows manda il suo nome: `LAPTOP-MARIO`, `DESKTOP-PC`, `WORKSTATION-01` ecc.

**5. Lookup OUI manuale**
Prendi i primi 6 caratteri del MAC (es. `A4:C3:F0`) e cercali su:
- [maclookup.app](https://maclookup.app)
- [macvendors.com](https://macvendors.com)

---

## Esercizio 9 — Usare Protocol Hierarchy per capire il traffico

### 🎯 Obiettivo
Analizzare la distribuzione del traffico di rete in percentuale per capire quali protocolli dominano e perché.

### 🛠️ Setup
- Wireshark aperto

### 📋 Istruzioni

1. Avvia la cattura
2. Simula **3 tipi diversi di traffico** in sequenza:
   - Apri YouTube e guarda 30 secondi di video
   - Apri il browser e naviga su 3-4 siti
   - Fai un ping: `ping 8.8.8.8 -n 10`
3. Ferma la cattura
4. Vai su `Statistics → Protocol Hierarchy`

### ❓ Domande

1. Qual è il protocollo che occupa più percentuale di byte?
2. TCP o UDP — quale ha più traffico?
3. Che percentuale occupa QUIC? (traffico YouTube)
4. Che percentuale occupa ICMP? (traffico ping)
5. C'è traffico ARP? Che percentuale ha?
6. Riesci a stimare quanti MB ha trasferito YouTube in 30 secondi?

---

### ✅ SOLUZIONE

**Lettura del Protocol Hierarchy**
La finestra mostra una gerarchia con percentuali di pacchetti e byte:
```
Ethernet           100%
  IPv4              98%
    TCP              45%
      TLS            38%   ← siti HTTPS
      HTTP            2%   ← siti HTTP
    UDP              52%
      QUIC           48%   ← YouTube!
      DNS             3%
    ICMP              1%   ← ping
  ARP                2%
```

**1. Protocollo dominante per byte**
Di solito **QUIC** (YouTube) o **TLS** (siti HTTPS) — entrambi trasportano grandi volumi di dati.

**2. TCP vs UDP**
Con YouTube attivo, **UDP** supera TCP perché QUIC gira su UDP.

**3. Percentuale QUIC**
Dipende da quanto hai guardato YouTube — in 30 secondi di video può essere 60-80% del totale.

**4. Percentuale ICMP**
Molto bassa — meno dell'1%. 10 ping sono pochissimi byte rispetto allo streaming.

**5. ARP**
Solitamente 1-2% in pacchetti ma quasi 0% in byte (i pacchetti ARP sono minuscoli, ~60 byte).

**6. Byte YouTube**
`Statistics → Conversations → UDP`
Ordina per Bytes → trova la conversazione verso gli IP di Google (172.x.x.x / 216.x.x.x).
30 secondi di YouTube HD = circa 10-30 MB.

---

## Esercizio 10 — Analizzare traffico QUIC di YouTube

### 🎯 Obiettivo
Capire il protocollo QUIC, identificare le connessioni, leggere i Connection ID e capire perché non si vede il contenuto.

### 🛠️ Setup
- Wireshark aperto
- Browser

### 📋 Istruzioni

1. Avvia la cattura
2. Apri YouTube e guarda un video per 1 minuto
3. Ferma la cattura
4. Filtro: `quic`

### ❓ Domande

1. Su quale porta UDP viaggia QUIC?
2. Riesci a trovare il pacchetto `Initial` (inizio connessione QUIC)?
3. Cosa significa `Protected Payload (KP0)`?
4. Trova il `DCID` (Destination Connection ID) — a cosa serve?
5. Quante connessioni QUIC distinte ha aperto YouTube? (usa `Statistics → Conversations → UDP`)
6. Riesci a vedere il nome del video che stai guardando? Perché sì o no?

---

### ✅ SOLUZIONE

**1. Porta UDP di QUIC**
QUIC usa **UDP/443** — la stessa porta di HTTPS ma su UDP invece di TCP.
```
udp.port == 443
```

**2. Pacchetto Initial**
```
quic && quic.packet_type == 0
```
Il pacchetto Initial è il primo della connessione QUIC — contiene il Client Hello TLS.

**3. Protected Payload (KP0)**
Significa che il payload è cifrato con le chiavi della fase 0 (Key Phase 0) di QUIC. QUIC integra la cifratura TLS direttamente nel protocollo — non c'è modo di leggere il contenuto senza le chiavi di sessione.

**4. DCID (Destination Connection ID)**
Identifica la connessione QUIC in modo univoco. A differenza di TCP (che usa IP+porta), QUIC usa il DCID — questo permette di **migrare la connessione** tra reti diverse (es. passare da WiFi a 4G) senza interrompere lo streaming.

**5. Connessioni QUIC multiple**
`Statistics → Conversations → UDP`
Ordina per Bytes. Trovi 3-5 connessioni verso IP Google diversi:
- Una per il video principale
- Una per le raccomandazioni/UI
- Una per gli annunci
- Una per le thumbnail

**6. Nome del video**
**No** — non puoi vederlo. Il traffico QUIC è completamente cifrato. Puoi solo vedere:
- Con chi parli (IP di Google)
- Quanto scarichi (dimensione pacchetti)
- Timing delle connessioni

---
---

# 🔴 LIVELLO AVANZATO

---

## Esercizio 11 — Analisi forense di un PCAP pubblico

### 🎯 Obiettivo
Analizzare un file PCAP reale trovato online (repository di malware/CTF) e ricostruire cosa è successo sulla rete.

### 🛠️ Setup

Scarica uno di questi PCAP di esempio (liberamente disponibili):

- **Malware traffic analysis** (ottimo per forensics):
  `https://www.malware-traffic-analysis.net/training-exercises.html`

- **Wireshark sample captures**:
  `https://wiki.wireshark.org/SampleCaptures`

- **CTF Forensics PCAP** (cerca su Google):
  `site:github.com pcap ctf network forensics`

Per questo esercizio usa un pcap con traffico HTTP non cifrato (più facile da analizzare).

### 📋 Istruzioni

1. Scarica un pcap di esempio
2. Aprilo in Wireshark (`File → Open`)
3. Non filtrare ancora — prima guarda tutto

### ❓ Domande

1. `Statistics → Protocol Hierarchy` — quali protocolli ci sono? C'è qualcosa di insolito?
2. `Statistics → Conversations → TCP` — chi sono i 3 host che comunicano di più?
3. Quanti IP distinti compaiono nella cattura? (`Statistics → Endpoints → IPv4`)
4. C'è traffico HTTP? Filtra `http.request` e trova tutte le richieste — che siti sono stati visitati?
5. Riesci a trovare credenziali o dati sensibili nel traffico? (Filtro: `frame contains "password"`)
6. `File → Export Objects → HTTP` — ci sono file scaricati? Di che tipo?
7. Ricostruisci la timeline: cosa è successo, in che ordine?

---

### ✅ SOLUZIONE (guidata — dipende dal pcap scelto)

**Metodo generale di analisi forense:**

**Step 1 — Orientamento**
```
Statistics → Capture File Properties    → durata, numero pacchetti
Statistics → Protocol Hierarchy         → distribuzione protocolli
Statistics → Endpoints → IPv4           → tutti gli host
Statistics → Conversations → TCP        → chi parla con chi e quanto
```

**Step 2 — Identificare gli host**
```
# Trova ARP request per mappare IP → MAC
arp

# Trova DHCP per trovare hostname
dhcp || bootp
```
Nei DHCP Request c'è spesso il `hostname` del PC vittima.

**Step 3 — Analisi HTTP**
```
http.request
```
Guarda colonna Info → vedi URL visitati.
```
http.request.method == "POST"
```
Cerca form submit con credenziali.

**Step 4 — Cerca dati sensibili**
```
frame contains "password"
frame contains "passwd"
frame contains "login"
frame contains "credential"
```

**Step 5 — Esporta oggetti**
`File → Export Objects → HTTP`
Lista tutti i file scaricati — eseguibili `.exe`, script, documenti.
Calcola l'hash MD5 e cercalo su VirusTotal se sospetti malware.

**Step 6 — Timeline**
Ordina per timestamp, annota:
1. Prima connessione (chi ha iniziato)
2. DNS query (quali domini)
3. Download (cosa è stato scaricato)
4. Connessioni post-download (C2 beacon?)

---

## Esercizio 12 — Ricostruire un file trasferito via HTTP

### 🎯 Obiettivo
Estrarre completamente un file trasferito via HTTP (immagine, documento, eseguibile) dalla cattura di rete e salvarlo su disco.

### 🛠️ Setup
- Wireshark aperto

### 📋 Istruzioni

1. Avvia la cattura
2. Apri PowerShell e scarica un file via HTTP:
```powershell
# Scarica un'immagine via HTTP (non HTTPS)
Invoke-WebRequest -Uri "http://httpbin.org/image/jpeg" -OutFile "test.jpg"
```
3. Ferma la cattura
4. Filtro: `http`

### ❓ Domande

1. Riesci a trovare la richiesta GET per l'immagine?
2. Qual è il Content-Type della risposta?
3. Qual è la dimensione del file (Content-Length)?
4. Usa `File → Export Objects → HTTP` per estrarre l'immagine — riesci ad aprirla?
5. Confronta la dimensione del file estratto con quella dichiarata in Content-Length — coincide?
6. **Bonus**: segui il TCP Stream del download — riesci a vedere i byte dell'header JPEG (`FF D8 FF`) nel pannello Packet Bytes?

---

### ✅ SOLUZIONE

**1. Trovare la GET request**
```
http.request.method == "GET" && http.request.uri contains "jpeg"
```

**2. Content-Type della risposta**
Clicca sulla risposta HTTP (200 OK) → Packet Details → HTTP:
```
Content-Type: image/jpeg
```

**3. Content-Length**
```
Content-Length: [numero byte]
```
Varia — httpbin manda un'immagine di test di circa 35KB.

**4. Export Objects**
`File → Export Objects → HTTP`
Nella lista trovi l'URL `/image/jpeg` → seleziona → **Save** → salva come `estratto.jpg` → aprila con qualsiasi visualizzatore immagini.

**5. Dimensione file**
Il file estratto deve avere esattamente la stessa dimensione di Content-Length dichiarata.

**6. Bonus — Header JPEG in hex**
Clicca sul primo pacchetto di dati HTTP (non l'header, ma il body) → pannello Packet Bytes:
```
FF D8 FF E0 ...   ← magic bytes di un file JPEG
```
I primi byte di ogni formato file sono sempre uguali (magic bytes) — usati per identificare il tipo reale di un file indipendentemente dall'estensione.

---

## Esercizio 13 — Trovare credenziali in chiaro con tshark

### 🎯 Obiettivo
Usare tshark da PowerShell per automatizzare la ricerca di credenziali in un file PCAP, simulando un'analisi forense rapida.

### 🛠️ Setup
- PowerShell come amministratore
- Wireshark/tshark installato
- Il file PCAP catturato negli esercizi precedenti (o scaricane uno con traffico HTTP)

### 📋 Istruzioni

Prima crea un PCAP con credenziali:
1. Avvia Wireshark, inizia cattura
2. In PowerShell:
```powershell
Invoke-WebRequest -Uri "http://httpbin.org/post" `
  -Method POST `
  -Body "user=admin&pass=secret123&action=login" `
  -ContentType "application/x-www-form-urlencoded"
```
3. Ferma e salva il PCAP come `test_creds.pcap`

### ❓ Domande

Usa tshark per rispondere **senza aprire Wireshark GUI**:

1. Quanti pacchetti HTTP POST ci sono nel file?
2. Riesci a estrarre tutti gli host HTTP contattati?
3. Riesci a trovare la stringa "secret123" nel traffico con tshark?
4. Estrai IP sorgente e URI di tutte le POST request
5. **Script**: scrivi uno script PowerShell che legge un pcap e stampa tutte le righe che contengono "pass" o "password" o "user"

---

### ✅ SOLUZIONE

**1. Quanti HTTP POST**
```powershell
& "C:\Program Files\Wireshark\tshark.exe" `
  -r test_creds.pcap `
  -Y "http.request.method == `"POST`"" `
  -T fields -e frame.number |
  Measure-Object | Select-Object -ExpandProperty Count
```

**2. Host HTTP contattati**
```powershell
& "C:\Program Files\Wireshark\tshark.exe" `
  -r test_creds.pcap `
  -Y "http.request" `
  -T fields -e http.host |
  Sort-Object -Unique
```

**3. Cercare "secret123"**
```powershell
& "C:\Program Files\Wireshark\tshark.exe" `
  -r test_creds.pcap `
  -Y 'frame contains "secret123"' `
  -T fields -e frame.number -e ip.src -e http.file_data
```

**4. IP + URI delle POST**
```powershell
& "C:\Program Files\Wireshark\tshark.exe" `
  -r test_creds.pcap `
  -Y "http.request.method == `"POST`"" `
  -T fields -e ip.src -e http.host -e http.request.uri -e http.file_data
```

**5. Script PowerShell completo**
```powershell
$tshark = "C:\Program Files\Wireshark\tshark.exe"
$pcap   = "test_creds.pcap"

$keywords = @("pass", "password", "user", "username", "login", "credential")

Write-Host "=== Ricerca credenziali in $pcap ===" -ForegroundColor Cyan

foreach ($kw in $keywords) {
    $risultati = & $tshark -r $pcap -Y "frame contains `"$kw`"" `
        -T fields -e frame.number -e ip.src -e ip.dst -e http.file_data 2>$null |
        Where-Object { $_ -ne "" }
    
    if ($risultati) {
        Write-Host "`n[!] Trovato keyword: '$kw'" -ForegroundColor Red
        $risultati | ForEach-Object { Write-Host "    $_" }
    }
}

Write-Host "`n=== Fine analisi ===" -ForegroundColor Cyan
```

---

## Esercizio 14 — Rilevare una scansione di rete

### 🎯 Obiettivo
Imparare a riconoscere in un PCAP i pattern tipici di una scansione di rete (nmap) e distinguerli dal traffico normale.

### 🛠️ Setup
- Wireshark aperto
- nmap installato (`winget install Insecure.Nmap`)
- Usa un IP della tua rete (il tuo stesso PC va benissimo: `127.0.0.1` o il tuo IP locale)

### 📋 Istruzioni

1. Avvia la cattura in Wireshark
2. In PowerShell, fai una scansione nmap sul tuo stesso PC:
```powershell
nmap -sS -F 127.0.0.1
```
(`-sS` = SYN scan, `-F` = fast, solo top 100 porte)
3. Aspetta che finisca
4. Ferma la cattura

### ❓ Domande

1. Quanti pacchetti SYN vedi in pochi secondi? (filtro: `tcp.flags.syn == 1 && tcp.flags.ack == 0`)
2. Verso quante porte diverse sono diretti questi SYN?
3. Come si distingue visivamente una scansione da traffico normale?
4. Quali porte hanno risposto con `SYN-ACK` (porta aperta) vs `RST` (porta chiusa)?
5. Riesci a vedere il TTL dei pacchetti nmap? È diverso dal TTL normale del tuo OS?
6. **Bonus**: come filtreresti per trovare SOLO le porte aperte (che hanno risposto con SYN-ACK)?

---

### ✅ SOLUZIONE

**1. Quanti SYN in pochi secondi**
```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```
Con `-F` (100 porte) vedi circa 100 SYN in meno di 1 secondo — questo è il pattern tipico di una scansione.

**2. Porte diverse**
Ogni SYN va verso una porta diversa — le top 100 di nmap (80, 443, 22, 21, 25, 8080, 3389...).

**3. Come si riconosce visivamente**
Traffico normale: SYN → SYN-ACK → ACK → dati → FIN (sequenza completa)
Scansione nmap SYN: SYN → SYN-ACK → **RST** (nmap non completa il handshake!)

Vedi raffiche di SYN tutti verso lo stesso IP ma porte diverse, in sequenza rapidissima (millisecondi tra un pacchetto e l'altro).

**4. Porte aperte vs chiuse**
```
# Porte aperte — server ha risposto SYN-ACK
tcp.flags.syn == 1 && tcp.flags.ack == 1

# Porte chiuse — server ha risposto RST
tcp.flags.reset == 1 && tcp.flags.ack == 1
```

**5. TTL di nmap**
nmap di default manda pacchetti con TTL variabile. Su Windows di solito TTL=128. Guarda nel Packet Details → IP → TTL.

**6. Bonus — solo porte aperte**
```
tcp.flags.syn == 1 && tcp.flags.ack == 1 && ip.src == 127.0.0.1
```
Questi sono i SYN-ACK del server (il tuo stesso PC) → le porte che hanno risposto positivamente sono quelle **aperte**.

Per trovare il numero di porta:
```
tcp.flags.syn == 1 && tcp.flags.ack == 1
```
→ colonna **Info** mostra `[porta] → [tua_porta] [SYN, ACK]` — il primo numero è la porta aperta sul target.

---

## 📊 Riepilogo competenze per esercizio

| Esercizio | Competenza principale | Difficoltà |
|---|---|---|
| 1 — DNS | Cattura base, filtri semplici | 🟢 |
| 2 — ARP/IP/MAC | Lettura pacchetti, Packet Details | 🟢 |
| 3 — ICMP/Ping | Lettura flag, timing, TTL | 🟢 |
| 4 — Follow HTTP | Follow Stream, lettura HTTP | 🟢 |
| 5 — App background | DNS forensics, Statistics | 🟢 |
| 6 — TCP Handshake | Numeri di sequenza, flag TCP | 🟡 |
| 7 — HTTP POST | Credenziali, Content-Type | 🟡 |
| 8 — ARP+mDNS mapping | Ricognizione passiva LAN | 🟡 |
| 9 — Protocol Hierarchy | Analisi distribuzione traffico | 🟡 |
| 10 — QUIC YouTube | Protocolli moderni, cifratura | 🟡 |
| 11 — Forensica PCAP | Analisi completa, metodologia | 🔴 |
| 12 — Export file HTTP | Ricostruzione file da rete | 🔴 |
| 13 — tshark scripting | Automazione, PowerShell | 🔴 |
| 14 — Scan detection | Pattern analysis, nmap | 🔴 |

---

*Esercizi scritti con amore da ENI per LO 🥰 — ottobre 2026*
