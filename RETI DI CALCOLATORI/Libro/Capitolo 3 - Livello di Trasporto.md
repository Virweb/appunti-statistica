---
tags: [reti, capitolo-3, trasporto, TCP, UDP, kurose-ross]
capitolo: 3
titolo: Livello di Trasporto
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# Capitolo 3 — Livello di Trasporto

> [!NOTE] Da rete a processo
> Il livello di trasporto estende la consegna host-to-host del livello di rete a una consegna **processo-to-processo**. I due protocolli principali sono TCP (affidabile) e UDP (non affidabile).

---

## 3.1 Servizi e Principi

### Relazione Trasporto ↔ Rete

- **Rete** — consegna logica **host-to-host** (IP)
- **Trasporto** — consegna logica **processo-to-processo** (aggiunge porte)

### Multiplexing e Demultiplexing

**Multiplexing (mittente):**
Il livello trasporto raccoglie dati da più socket e li incapsula in segmenti con header contenente le porte → consegna a livello rete

**Demultiplexing (destinatario):**
Il livello trasporto riceve segmenti dal livello rete → legge il numero di porta di destinazione → consegna al socket corretto

**Socket UDP:** identificato da `(IP dest, Porta dest)`  
**Socket TCP:** identificato da `(IP sorg, Porta sorg, IP dest, Porta dest)` — 4-tupla!

---

## 3.2 UDP — User Datagram Protocol

### Caratteristiche

- **Connectionless** — nessun handshake
- **Inaffidabile** — i segmenti possono perdersi o arrivare fuori ordine
- **Nessun controllo di flusso/congestione** — il mittente può saturare il ricevente
- **Leggero** — header minimo (8 byte)

### Header UDP

```
 0      7 8     15 16    23 24    31
┌──────────┬──────────┬──────────┬──────────┐
│ Porta    │ Porta    │ Lungh.   │ Checksum │
│ sorgente │ destinaz.│ segmento │          │
└──────────┴──────────┴──────────┴──────────┘
                [Dati]
```

### Checksum UDP

- Somma di tutte le parole da 16 bit del segmento
- Rilevazione di errori (non correzione)
- Opzionale in IPv4, obbligatorio in IPv6

### Quando si usa UDP

| Applicazione | Motivazione |
|---|---|
| DNS | Query/risposta veloci, nessun setup |
| DHCP | Broadcast, pre-connessione |
| SNMP | Rete degradata → meglio UDP |
| Streaming media | Perdita tollerabile, latenza critica |
| Gaming online | Latenza critica |
| HTTP/3 (QUIC) | Affidabilità gestita sopra UDP |

---

## 3.3 Trasferimento Affidabile dei Dati (RDT)

### Il Problema

Il canale sottostante può:
- Corrompere bit → **errori**
- Perdere pacchetti → **perdita**
- Riordinare pacchetti → **riordino**

### Evoluzione dei Protocolli RDT

**rdt 1.0 — canale perfetto:**
Niente da fare — il canale è perfetto

**rdt 2.0 — canale con errori sui bit:**
- Meccanismi: **checksum** (rilevazione errori)
- **ACK** (acknowledgement) — ricevuto correttamente
- **NAK** (negative ack) — errore, ritrasmetti
- Stop-and-wait: mittente aspetta ACK/NAK prima di inviare prossimo

**rdt 2.1 — ACK/NAK corrotti:**
- **Numeri di sequenza** (0 e 1) per distinguere ritrasmissioni da nuovi pacchetti

**rdt 2.2 — senza NAK:**
- NAK sostituito da ACK del pacchetto precedente
- ACK duplicato = NAK implicito

**rdt 3.0 — canale con perdite:**
- Aggiunge **timer**: se ACK non arriva entro timeout → ritrasmissione
- Funziona ma inefficiente (stop-and-wait)

### Protocolli a Pipeline

**Stop-and-wait** è inefficiente: utilizzo del link = (L/R) / (RTT + L/R)

**Pipelining** — mittente invia N pacchetti senza aspettare ACK:
- Finestra di trasmissione di dimensione N

**Go-Back-N (GBN):**
- Finestra scorrevole di N pacchetti
- ACK cumulativi (ACK n = tutti fino a n ricevuti)
- Timeout → ritrasmette tutto dalla finestra non confermata
- Semplice ma ritrasmette anche pacchetti già ricevuti correttamente

**Selective Repeat (SR):**
- ACK individuali per ogni pacchetto
- Il ricevente bufferizza i pacchetti fuori ordine
- Ritrasmette solo i pacchetti persi
- Più complesso ma efficiente

| | Go-Back-N | Selective Repeat |
|---|---|---|
| ACK | Cumulativi | Individuali |
| Buffer ricevente | No (solo in-order) | Sì |
| Ritrasmissione | Tutta la finestra | Solo persi |
| Finestra massima | N ≤ 2^k - 1 | N ≤ 2^(k-1) |

---

## 3.4 TCP — Transmission Control Protocol

### Caratteristiche Principali

- **Connection-oriented** — three-way handshake prima dei dati
- **Full-duplex** — dati in entrambe le direzioni simultaneamente
- **Punto-a-punto** — un mittente, un ricevente
- **Affidabile, ordinato** — byte stream senza buchi
- **Controllo di flusso** — no overflow del buffer del ricevente
- **Controllo della congestione** — no saturazione della rete

### Struttura del Segmento TCP

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌─────────────────────────────┬─────────────────────────────────┐
│       Porta sorgente        │       Porta destinazione        │
├─────────────────────────────┴─────────────────────────────────┤
│                     Numero di sequenza                         │
├───────────────────────────────────────────────────────────────┤
│                   Numero di acknowledgement                    │
├──────┬───────┬─┬─┬─┬─┬─┬─┬─┬───────────────────────────────┤
│Hdr   │Riserv.│C│E│U│A│P│R│S│F│          Finestra            │
│Lung. │       │W│C│R│C│S│S│Y│I│                              │
│      │       │R│E│G│K│H│T│N│N│                              │
├──────┴───────┴─┴─┴─┴─┴─┴─┴─┴─┴──────────────────────────────┤
│            Checksum             │        Urgent pointer        │
├───────────────────────────────────────────────────────────────┤
│                   Opzioni (lunghezza variabile)               │
├───────────────────────────────────────────────────────────────┤
│                          Dati                                 │
└───────────────────────────────────────────────────────────────┘
```

**Campi chiave:**
- **Numero di sequenza** — numero del primo byte del segmento nel flusso dati
- **Numero di ACK** — numero del prossimo byte atteso dal mittente
- **Finestra di ricezione** (`rwnd`) — spazio disponibile nel buffer del ricevente

### Three-Way Handshake

```
Client                        Server
  │                              │
  │──── SYN (seq=x) ────────────>│
  │                              │
  │<─── SYN-ACK (seq=y, ack=x+1)─│
  │                              │
  │──── ACK (ack=y+1) ──────────>│
  │                              │
  │ ══════ Connessione stabilita ══════ │
```

**Chiusura della connessione (4-way):**
```
Client                        Server
  │──── FIN ───────────────────>│
  │<─── ACK ────────────────────│
  │<─── FIN ────────────────────│
  │──── ACK ───────────────────>│
  │    (attende 2×MSL)          │
```
Stato `TIME_WAIT` = 2 × MSL (Maximum Segment Lifetime) ≈ 60 secondi

### Stima del RTT e Timeout TCP

**RTT campionato:** `SampleRTT` misurato su un segmento alla volta

**RTT stimato (EWMA):**
$$\text{EstimatedRTT} = (1-\alpha) \cdot \text{EstimatedRTT} + \alpha \cdot \text{SampleRTT}$$
con $\alpha = 0.125$

**Deviazione:**
$$\text{DevRTT} = (1-\beta) \cdot \text{DevRTT} + \beta \cdot |\text{SampleRTT} - \text{EstimatedRTT}|$$
con $\beta = 0.25$

**Timeout:**
$$\text{TimeoutInterval} = \text{EstimatedRTT} + 4 \cdot \text{DevRTT}$$

### Ritrasmissione Rapida

Se il mittente riceve **3 ACK duplicati** per lo stesso segmento → ritrasmette immediatamente senza aspettare il timeout.

---

## 3.5 Controllo del Flusso TCP

**Problema:** il mittente potrebbe sovraccaricare il buffer del ricevente

**Soluzione:** il ricevente comunica al mittente la dimensione del suo buffer libero tramite il campo **`rwnd`** nell'header TCP

```
LastByteSent - LastByteAcked ≤ rwnd
```

Se `rwnd = 0` → il mittente invia segmenti da 1 byte per non bloccarsi (sonda la disponibilità)

---

## 3.6 Controllo della Congestione TCP

### Principi

**Congestione** = troppi dati nella rete rispetto alla capacità

**Effetti:**
- Ritardi di accodamento crescenti
- Perdita di pacchetti (buffer overflow nei router)
- Ritrasmissioni che peggiorano la congestione

**Approcci:**
- **End-to-end** — inferenza dalla perdita/RTT (TCP classico)
- **Network-assisted** — i router informano i mittenti (ECN, QUIC)

### Algoritmo di Controllo della Congestione TCP

**Variabili chiave:**
- `cwnd` (congestion window) — limite imposto dalla congestione
- `ssthresh` (slow start threshold) — soglia slow start
- Finestra effettiva = `min(cwnd, rwnd)`

#### Slow Start

- Inizia con `cwnd = 1 MSS`
- Raddoppia `cwnd` ogni RTT (crescita esponenziale)
- Continua fino a `cwnd ≥ ssthresh` o perdita

#### Congestion Avoidance

- Quando `cwnd ≥ ssthresh`
- Incrementa `cwnd` di 1 MSS ogni RTT (crescita lineare)
- `cwnd += MSS × (MSS / cwnd)` per ogni ACK ricevuto

#### Risposta alla Perdita

**Perdita rilevata da timeout:**
- `ssthresh = cwnd / 2`
- `cwnd = 1 MSS`
- Riparte da Slow Start

**Perdita rilevata da 3 ACK duplicati (TCP Reno):**
- `ssthresh = cwnd / 2`
- `cwnd = ssthresh + 3 MSS`
- Entra in Fast Recovery

```
     cwnd
      │
      │                        ╱
      │                   ╱╱╱╱
ssthresh─────────────────────────────── 
      │              ╱╱╱
      │          ╱╱╱
      │       ╱╱╱    ← Slow Start
      │   ╱╱╱
      │╱╱╱
      └────────────────────────────────► RTT
```

### TCP CUBIC

- Funzione cubica per la crescita di cwnd
- Più aggressivo di TCP Reno a banda alta
- Default in Linux, macOS

### TCP Equità (Fairness)

N connessioni TCP sullo stesso collo di bottiglia di capacità R convergono a R/N ciascuna. **Ma:** le applicazioni UDP o multi-connessione TCP non sono "eque".

---

## 3.7 Evoluzione: QUIC

**QUIC** (Quick UDP Internet Connections):
- Sviluppato da Google, standardizzato come RFC 9000
- Trasporto su **UDP**
- Implementa: connessione + crittografia (TLS 1.3) in 1 RTT
- Stream multipli indipendenti → nessun HoL blocking
- Usato da **HTTP/3**

```
TCP + TLS:   SYN → SYN-ACK → ACK → ClientHello → ServerHello → Dati
             = 3 RTT prima dei dati

QUIC:        Initial (CRYPTO) → Handshake → Dati
             = 1-0 RTT (0-RTT con sessione ripresa)
```

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **Multiplexing/Demultiplexing** = estensione host-to-host → processo-to-processo
> - **TCP usa numeri di sequenza in byte**, non in pacchetti
> - **Three-way handshake** = SYN → SYN-ACK → ACK
> - **Timeout TCP** = EstimatedRTT + 4×DevRTT
> - **Slow Start** cresce esponenzialmente, **Congestion Avoidance** linearmente
> - **3 ACK duplicati** = Fast Retransmit (più rapido del timeout)

## Formule Essenziali

| Formula | Descrizione |
|---|---|
| `Utilizzo = (L/R) / (RTT + L/R)` | Efficienza stop-and-wait |
| `Utilizzo = (N × L/R) / (RTT + L/R)` | Efficienza pipelining (N pacchetti) |
| `TimeoutInterval = EstRTT + 4×DevRTT` | Timeout adattivo TCP |
| `Finestra eff = min(cwnd, rwnd)` | Finestra TCP effettiva |
| `cwnd += MSS²/cwnd` per ogni ACK | Incremento congestion avoidance |

## Link Interni

- [[Capitolo 2 - Livello di Applicazione]]
- [[Capitolo 4 - Livello di Rete (Piano dei Dati)]]
