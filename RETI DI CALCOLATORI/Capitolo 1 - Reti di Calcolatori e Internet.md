---
tags: [reti, capitolo-1, internet, kurose-ross]
capitolo: 1
titolo: Reti di Calcolatori e Internet
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# Capitolo 1 — Reti di Calcolatori e Internet

> [!NOTE] Approccio Top-Down
> Il libro parte dal livello più alto (applicazione) e scende verso i livelli fisici. Questo capitolo introduce la visione d'insieme dell'intera architettura.

---

## 1.1 Cos'è Internet?

Internet è una **rete di reti** — miliardi di dispositivi interconnessi che comunicano usando protocolli standard.

### Componenti fondamentali

| Componente | Descrizione |
|---|---|
| **Host (sistemi periferici)** | Dispositivi finali: PC, smartphone, server, IoT |
| **Router** | Instradano i pacchetti tra reti diverse |
| **Switch** | Collegano dispositivi all'interno di una rete locale |
| **Link di comunicazione** | Cavi in rame, fibra ottica, wireless, satellite |
| **ISP** | Internet Service Provider — la gerarchia che forma Internet |

### Due visioni di Internet

**Vista dei servizi (nuts and bolts):**
- Infrastruttura che fornisce servizi alle applicazioni distribuite
- API per socket programming — le app "parlano" a Internet tramite socket

**Vista dell'infrastruttura:**
- Insieme di protocolli che controllano la trasmissione e ricezione dei dati
- **RFC** (Request for Comments) — standard ufficiali di Internet, gestiti da IETF

---

## 1.2 Ai Confini della Rete — Gli Host

Gli host si dividono in:
- **Client** — richiedono servizi (browser, app mobile)
- **Server** — forniscono servizi (data center, web server)

### Reti di Accesso

Come gli host si collegano al router di bordo (edge router):

#### DSL (Digital Subscriber Line)
- Usa la linea telefonica esistente (doppino in rame)
- Frequenze separate per dati downstream, upstream e voce
- **DSLAM** nel central office separa dati e voce
- Downstream tipico: fino a 24 Mbps | Upstream: fino a 2.5 Mbps

#### Cavo
- Usa la rete HFC (Hybrid Fiber Coax)
- Fibra ottica fino al quartiere, poi coassiale fino alle case
- **CMTS** (Cable Modem Termination System) nel headend
- Canale broadcast condiviso → tutti vedono i pacchetti di tutti
- Downstream: fino a 42.8 Mbps | Upstream: fino a 30.7 Mbps

#### FTTH (Fiber To The Home)
- Fibra ottica direttamente fino a casa
- Architetture:
  - **AON** (Active Optical Network) — switch dedicati
  - **PON** (Passive Optical Network) — splitter ottici passivi
- Velocità: fino a Gbps

#### Reti Aziendali (LAN)
- **Ethernet** — standard dominante per LAN cablate
  - 100 Mbps, 1 Gbps, 10 Gbps, 100 Gbps
  - Switch Ethernet collegano i dispositivi
- **WiFi** (IEEE 802.11) — LAN wireless
  - Access Point collegato alla LAN cablata

#### Reti Wireless Geografiche
- **3G/4G/5G** — reti cellulari, copertura su aree vaste
- **LTE** — Long-Term Evolution, fino a 300 Mbps downstream

### Mezzi di Trasmissione

**Guidati (fisici):**
- **Doppino intrecciato (UTP)** — il più comune, economico, categorie Cat5/Cat6
- **Cavo coassiale** — schermato, banda larga, usato in HFC
- **Fibra ottica** — luce = dati, immune a interferenze EM, bassa attenuazione, altissima banda

**Non guidati (wireless):**
- **Radio a spettro terrestre** — WiFi, cellulare, FM
- **Radio a microonde** — collegamenti punto-punto fino a 45 Mbps
- **Satellite** — GEO (35.786 km, 270ms RTT), LEO (Starlink)

---

## 1.3 Il Nucleo della Rete — Commutazione

### Commutazione di Pacchetto

I dati vengono divisi in **pacchetti** → ogni pacchetto viaggia indipendentemente.

**Store-and-forward:**
- Il router riceve l'intero pacchetto prima di trasmetterlo
- Ritardo di trasmissione = L/R (L = bit del pacchetto, R = velocità del link)
- Per N link: ritardo totale = N × L/R

**Code di attesa (Queuing):**
- I pacchetti si accodano se il link di uscita è occupato
- **Buffer** nel router — se pieno → **packet loss**
- Ritardo di accodamento variabile e imprevedibile

**Forwarding table e protocolli di routing:**
- Ogni router ha una tabella di forwarding (indirizzo dest → link di uscita)
- Protocolli di routing calcolano e aggiornano queste tabelle

### Commutazione di Circuito (a confronto)

- Risorse **riservate** per tutta la durata della comunicazione
- Usata nelle reti telefoniche tradizionali
- **FDM** (Frequency Division Multiplexing) — ogni circuito ha una banda di frequenza
- **TDM** (Time Division Multiplexing) — ogni circuito ha slot temporali

| | Commutazione di Pacchetto | Commutazione di Circuito |
|---|---|---|
| Risorse | Condivise (statistica) | Riservate |
| Efficienza | Alta (nessuno spreco) | Bassa (risorse inutilizzate) |
| Ritardo | Variabile (code) | Fisso e prevedibile |
| Uso | Internet | Rete telefonica |

> [!TIP] Perché i pacchetti vincono
> La commutazione di pacchetto supporta più utenti simultanei rispetto al circuito — statisticamente la maggior parte degli utenti non trasmette continuamente.

### La Gerarchia degli ISP

```
                    ┌─────────────────┐
                    │   Tier-1 ISP    │  (backbone globale)
                    │  AT&T, NTT...   │
                    └────────┬────────┘
                    ╔════════╧════════╗
              ┌─────╢  Peering point  ╟─────┐
              │     ╚═════════════════╝     │
        ┌─────┴──────┐              ┌───────┴────┐
        │  Tier-2    │              │  Tier-2    │  (ISP regionali)
        └─────┬──────┘              └───────┬────┘
              │                             │
        ┌─────┴──────┐              ┌───────┴────┐
        │  Tier-3    │              │  Tier-3    │  (ISP locali/accesso)
        └─────┬──────┘              └───────┬────┘
              │                             │
         [Host]                         [Host]
```

- **IXP** (Internet Exchange Point) — punto fisico dove ISP si interconnettono
- **CDN** (Content Delivery Network) — reti proprie (Google, Netflix) che bypassano ISP

---

## 1.4 Ritardo, Perdita e Throughput

### Tipi di Ritardo nei Nodi

```
Ritardo totale = d_elab + d_accod + d_trasm + d_prop
```

| Tipo | Simbolo | Formula | Note |
|---|---|---|---|
| **Elaborazione** | d_elab | ~microsecondi | Controllo bit errors, lookup |
| **Accodamento** | d_accod | variabile | Dipende dal traffico |
| **Trasmissione** | d_trasm | L/R | L=dimensione pacchetto, R=velocità link |
| **Propagazione** | d_prop | d/s | d=distanza, s=velocità segnale (~2×10⁸ m/s) |

### Intensità di Traffico

$$I = \frac{La}{R}$$

- L = dimensione pacchetto (bit)
- a = tasso di arrivo pacchetti (pkt/s)  
- R = velocità del link (bit/s)

| Intensità | Comportamento |
|---|---|
| I → 0 | Ritardo di accodamento minimo |
| I → 1 | Ritardo di accodamento → ∞ |
| I > 1 | Più lavoro che capacità → perdita pacchetti |

### Traceroute
Strumento diagnostico: misura RTT verso ogni router nel percorso.
- Usa pacchetti UDP con TTL incrementale
- Ogni router che scarta (TTL=0) risponde con ICMP "Time Exceeded"

### Throughput

**Throughput istantaneo** — velocità di ricezione bit in un dato istante  
**Throughput medio** — F bit trasferiti in T secondi = F/T bit/s

**Collo di bottiglia** — il link con capacità minima lungo il percorso:
$$\text{Throughput} = \min(R_s, R_c, \ldots)$$

---

## 1.5 Livelli di Protocollo e Modelli di Servizio

### L'Architettura a Livelli

Ogni livello:
1. Fornisce un **servizio** al livello superiore
2. Usa i **servizi** del livello inferiore
3. Implementa le proprie **funzioni** tramite protocolli

**Vantaggi della stratificazione:**
- Modularità — modificare un livello senza impattare gli altri
- Riuso — protocolli riusabili in contesti diversi

### Il Modello Internet (TCP/IP)

```
┌─────────────────────────┐
│  5. Applicazione        │  HTTP, DNS, SMTP, FTP, BitTorrent
├─────────────────────────┤
│  4. Trasporto           │  TCP, UDP
├─────────────────────────┤
│  3. Rete                │  IP, ICMP, OSPF, BGP
├─────────────────────────┤
│  2. Collegamento        │  Ethernet, WiFi, PPP
├─────────────────────────┤
│  1. Fisico              │  Bit sul cavo/fibra/radio
└─────────────────────────┘
```

### Il Modello OSI (7 livelli)

```
┌─────────────────────────┐
│  7. Applicazione        │
├─────────────────────────┤
│  6. Presentazione       │  (cifratura, compressione, traduzione)
├─────────────────────────┤
│  5. Sessione            │  (sincronizzazione, checkpoint)
├─────────────────────────┤
│  4. Trasporto           │
├─────────────────────────┤
│  3. Rete                │
├─────────────────────────┤
│  2. Collegamento        │
├─────────────────────────┤
│  1. Fisico              │
└─────────────────────────┘
```

> [!NOTE] OSI vs TCP/IP
> Internet reale usa TCP/IP (5 livelli). OSI è un modello teorico. Le funzioni di presentazione e sessione in TCP/IP sono gestite dall'applicazione stessa.

### Incapsulamento (Encapsulation)

```
APP:      [Messaggio                              ]
TRASPORTO: [Hdr-T | Messaggio                    ]  ← Segmento
RETE:     [Hdr-N | Hdr-T | Messaggio             ]  ← Datagramma
LINK:     [Hdr-L | Hdr-N | Hdr-T | Messaggio |Tr]  ← Frame
FISICO:    101010110101010101...                      ← Bit
```

Ogni livello aggiunge la propria intestazione (header) → il livello ricevente la rimuove.

---

## 1.6 Reti sotto Attacco: Sicurezza

### Principali Minacce

**Malware:**
- Virus — si replica allegandosi a file legittimi
- Worm — si replica autonomamente sfruttando vulnerabilità di rete
- Spyware — raccoglie informazioni senza consenso
- Botnet — reti di host compromessi controllati da un attaccante

**Attacchi DoS e DDoS:**
- **DoS** (Denial of Service) — sovraccaricare server/rete con traffico artificiale
- **DDoS** (Distributed) — attacco coordinato da botnet di migliaia di host
- Tre categorie: vulnerability attack, bandwidth flooding, connection flooding

**Packet Sniffing:**
- Un ricevitore promiscuo cattura e legge tutti i pacchetti sul link
- Pericoloso su WiFi e reti broadcast condivise
- Difesa: **cifratura**

**IP Spoofing:**
- Inviare pacchetti con indirizzo sorgente falsificato
- Difesa: **autenticazione endpoint**

**Man-in-the-Middle:**
- L'attaccante si interpone tra due host comunicanti
- Può leggere, modificare, iniettare messaggi

---

## 1.7 Breve Storia di Internet

| Anno | Evento |
|---|---|
| **1961-1972** | Sviluppo della commutazione di pacchetto (Leonard Kleinrock, 1961) |
| **1969** | ARPANET — primo host a host collegato (UCLA ↔ Stanford) |
| **1972** | Prima demo pubblica di ARPANET; primo programma email (Ray Tomlinson) |
| **1974** | TCP/IP — Cerf e Kahn (architettura aperta e interconnessa) |
| **1983** | ARPANET adotta TCP/IP; nasce il DNS |
| **1991** | Tim Berners-Lee inventa il World Wide Web al CERN |
| **1993** | Mosaic — primo browser grafico |
| **1994-2000** | Boom di Internet commerciale, esplosione dot-com |
| **2000s** | Web 2.0, social network, streaming, smartphone |
| **2010s+** | Cloud computing, IoT, 5G, SDN, CDN globali |

---

## Concetti Chiave da Ricordare

> [!IMPORTANT] Definizioni fondamentali
> - **Protocollo** = insieme di regole che definiscono formato, ordine e azione dei messaggi scambiati tra entità comunicanti
> - **RTT** (Round Trip Time) = tempo per inviare un pacchetto piccolo e ricevere risposta
> - **Collo di bottiglia** = link con throughput minimo lungo il percorso
> - **Store-and-forward** = il router deve ricevere tutto il pacchetto prima di ritrasmettere

## Formule Essenziali

| Formula | Descrizione |
|---|---|
| `d_trasm = L/R` | Ritardo di trasmissione (L bit, R bps) |
| `d_prop = d/s` | Ritardo di propagazione (d metri, s ≈ 2×10⁸ m/s) |
| `Throughput = min(R1, R2, ..., Rn)` | Collo di bottiglia |
| `I = La/R` | Intensità di traffico |

## Link Interni

- [[Capitolo 2 - Livello di Applicazione]]
- [[Capitolo 3 - Livello di Trasporto]]
- [[Capitolo 4 - Livello di Rete (Piano dei Dati)]]
- [[Capitolo 5 - Livello di Rete (Piano di Controllo)]]
- [[Capitolo 6 - Livello di Collegamento e LAN]]
- [[Capitolo 7 - Reti Wireless e Mobili]]
- [[Capitolo 8 - Sicurezza nelle Reti]]
