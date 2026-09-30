---
tags: [reti, capitolo-6, collegamento, ethernet, LAN, MAC, kurose-ross]
capitolo: 6
titolo: Livello di Collegamento e LAN
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# Capitolo 6 — Livello di Collegamento e LAN

> [!NOTE] Da hop a hop
> Il livello di collegamento si occupa del trasferimento dei dati tra nodi **adiacenti** (collegati dallo stesso link). Gestisce errori, accesso al mezzo e indirizzamento fisico (MAC).

---

## 6.1 Introduzione al Livello di Collegamento

### Servizi Offerti

| Servizio | Descrizione | Obbligatorio? |
|---|---|---|
| **Framing** | Incapsula il datagramma in un frame con header/trailer | Sì |
| **Accesso al link** | Protocollo MAC per coordinare accesso al mezzo condiviso | Sì (mezzo condiviso) |
| **Consegna affidabile** | ACK e ritrasmissione hop-by-hop (es: WiFi) | No (wired spesso omette) |
| **Rilevazione errori** | CRC, parità | Sì |
| **Correzione errori** | Hamming, convoluzionali | No (raro nel wired) |

### Dove è Implementato

Il livello di collegamento è implementato nel **NIC** (Network Interface Card):
- Hardware: MAC, codifica, segnalazione
- Driver: parte software del SO

### Terminologia

- **Nodo** = host o router
- **Link** = canale di comunicazione tra nodi adiacenti (wired, wireless, LAN)
- **Frame** = unità di dati del livello collegamento

---

## 6.2 Tecniche di Rilevazione e Correzione Errori

### Bit di Parità

**Parità semplice (1D):**
- Aggiunge 1 bit per rendere il numero di 1 pari (o dispari)
- Rileva solo errori su **numero dispari** di bit

**Parità bidimensionale (2D):**
- Bit organizzati in matrice → parità per ogni riga e colonna
- Rileva **e corregge** errori su 1 bit
- Rileva errori su 2 bit

### CRC — Cyclic Redundancy Check

Tecnica più usata nei protocolli di rete moderni (Ethernet, WiFi).

**Principio:**
1. Mittente e ricevente concordano su un **polinomio generatore G** di grado r
2. Mittente calcola r bit di CRC → aggiunge a D → invia `D·2^r XOR R`
3. Ricevente divide per G → se resto = 0 → nessun errore

**Capacità di rilevazione:**
- Tutti gli errori burst < r+1 bit
- Tutti gli errori con numero dispari di bit (con G opportuno)
- CRC-32 (r=32) usato in Ethernet

### Codici di Hamming (correzione errori)

Aggiunge bit di ridondanza nelle posizioni potenze di 2 (1, 2, 4, 8, ...).  
Permette di **localizzare e correggere** 1 bit errato.

---

## 6.3 Protocolli di Accesso Multiplo

**Problema:** come coordinare l'accesso a un mezzo di trasmissione condiviso?

### Classificazione

```
Protocolli MAC
├── Partizionamento del canale
│   ├── TDMA (Time Division)
│   ├── FDMA (Frequency Division)
│   └── CDMA (Code Division)
├── Accesso casuale
│   ├── ALOHA puro
│   ├── ALOHA slottato
│   ├── CSMA
│   ├── CSMA/CD (Ethernet)
│   └── CSMA/CA (WiFi)
└── A turno
    ├── Polling
    └── Token passing
```

### TDMA e FDMA

**TDMA:** il tempo è diviso in slot, ogni nodo ha il suo slot  
**FDMA:** la banda è divisa in sotto-bande, ogni nodo ha la sua frequenza

Entrambi: efficienti, ma **sprecano risorse** se un nodo non ha dati.

### ALOHA

**ALOHA puro:**
- Trasmetti quando hai dati
- Se collisione → aspetta tempo casuale e ritrasmetti
- Efficienza massima: **18.4%** (1/2e)

**ALOHA Slottato:**
- Trasmissioni allineate a slot temporali
- Collisione → ritrasmetti in slot futuro scelto casualmente
- Efficienza massima: **37%** (1/e)

### CSMA — Carrier Sense Multiple Access

**"Ascolta prima di trasmettere"** — sense the carrier

**CSMA semplice:** se il canale è libero → trasmetti; se occupato → aspetta  
Non elimina le collisioni (ritardo di propagazione → due nodi sentono libero "allo stesso tempo")

**CSMA/CD (Collision Detection):**
- Usato in **Ethernet** cablato
- Se rileva collisione durante trasmissione → stop immediato
- Invia **segnale di jamming** per avvisare tutti
- **Binary Exponential Backoff:**
  - Dopo k-esima collisione → attendi un numero casuale in `{0, 1, ..., 2^k - 1}` slot
  - Max 10 tentativi → poi scarto il frame

**CSMA/CA (Collision Avoidance):**
- Usato in **WiFi** (802.11)
- Non è possibile rilevare collisioni (half-duplex wireless, near/far problem)
- Evita le collisioni prima di trasmettere
- **DCF** (Distributed Coordination Function): DIFS → dati → SIFS → ACK

### Protocolli a Turno

**Polling:**
- Nodo master interroga i nodi a turno
- Efficiente ma: single point of failure, polling overhead

**Token Ring:**
- Un token circola tra i nodi
- Solo chi ha il token può trasmettere
- Nessuna collisione, ma overhead del token + failure

---

## 6.4 LAN Cablate: Ethernet

### Indirizzo MAC (Media Access Control)

- **48 bit** (6 byte), scritto in esadecimale: `1A:2B:3C:4D:5E:6F`
- **Univoco** — assegnato dal produttore (burned into ROM)
- **Broadcast MAC:** `FF:FF:FF:FF:FF:FF`
- Piatto (non gerarchico) — non cambia con la posizione nella rete

### ARP — Address Resolution Protocol

**Problema:** conosco l'IP del vicino, ma non il suo MAC.

**ARP Request (broadcast):**
```
"Chi ha IP 192.168.1.5? Risponde 192.168.1.1"
Dest MAC: FF:FF:FF:FF:FF:FF
```

**ARP Reply (unicast):**
```
"192.168.1.5 è qui, il mio MAC è 00:1A:2B:3C:4D:5E"
```

**ARP Table (cache):** ogni host mantiene una tabella `IP → MAC` con TTL.

**ARP per router:**
- Per inviare a destinazione fuori dalla subnet → ARP verso il **default gateway** (router)
- Il router usa poi ARP per trovare il MAC del prossimo hop

### Frame Ethernet

```
┌──────────┬───────────┬──────┬──────────────────┬─────┐
│ Preambolo│ MAC dest  │ MAC  │ Tipo/Lunghezza   │ CRC │
│  7 byte  │  6 byte   │ sorg │    2 byte        │ 4B  │
│+ 1 SFD   │           │ 6B   │ (IPv4=0x0800,    │     │
│          │           │      │  ARP=0x0806,     │     │
│          │           │      │  IPv6=0x86DD)    │     │
└──────────┴───────────┴──────┴──────────────────┴─────┘
                              │   Dati (46-1500 byte)   │
                              └─────────────────────────┘
```

**MTU Ethernet:** 1500 byte (payload massimo)

### Switch Ethernet

**Caratteristiche:**
- Lavora a livello 2 (frame, MAC)
- **Store-and-forward** — riceve frame completo poi lo ritrasmette
- **Plug-and-play, auto-learning** — nessuna configurazione manuale
- **Full-duplex** — nessuna collisione (ogni porta in dominio di collisione separato)
- **Filtraggio** — non ritrasmette inutilmente

**Self-learning (tabella MAC):**
```
Quando riceve frame dalla porta x con MAC sorgente M:
  → apprende: M si trova sulla porta x (con TTL)
  
Per inviare frame a MAC dest D:
  se D in tabella → forwarda sulla porta corrispondente
  se non in tabella → flood su tutte le porte tranne la sorgente
  se stessa porta → scarta (filtraggio)
```

**Switch vs Router:**

| | Switch | Router |
|---|---|---|
| Livello OSI | 2 (link) | 3 (rete) |
| Indirizzamento | MAC | IP |
| Plug-and-play | Sì | No |
| Routing ottimale | No (spanning tree) | Sì |
| Isolamento broadcast | No | Sì |

### VLAN — Virtual LAN

Permette di creare reti logicamente separate su switch fisici condivisi.

**Motivazione:**
- Isolare reparti senza switch separati
- Ridurre broadcast domain
- Sicurezza e segmentazione

**Port-based VLAN:** ogni porta dello switch assegnata a una VLAN

**802.1Q (VLAN tagging):**
- Frame esteso con **tag VLAN** (4 byte) che include il VLAN ID (12 bit → 4096 VLAN)
- Su link **trunk** tra switch → i frame portano il tag
- Su link **access** verso l'host → il tag viene rimosso

---

## 6.5 WiFi — 802.11

### Architettura WiFi

**Modalità infrastruttura:**
- **BSS** (Basic Service Set) = un AP + host associati
- **ESS** (Extended Service Set) = più BSS connesse dalla stessa rete wired
- **AP** (Access Point) — bridge tra BSS e rete cablata

**Modalità ad-hoc:**
- Host comunicano direttamente senza AP

### Canali e Associazione

Spettro 2.4 GHz diviso in 11 canali (sovrapposti, solo 1-6-11 non si sovrappongono).  
5 GHz ha più canali non sovrapposti.

**Associazione:**
1. AP trasmette periodicamente **Beacon Frame** (SSID, MAC AP)
2. Host scansiona tutti i canali → trova AP disponibili
3. Host sceglie AP → richiesta di associazione → risposta
4. Autenticazione (WPA3, 802.1X)

### 802.11 MAC — CSMA/CA

**Perché non CSMA/CD?**
- Trasmissione e ricezione simultanee impraticabili sul radio
- Problema stazione nascosta (A-B-C: A non sente C)

**DCF con CSMA/CA:**
```
se canale libero per DIFS:
    trasmetti
altrimenti:
    attendi DIFS + backoff casuale (decrementa solo se canale libero)
    poi trasmetti
ricevente attende SIFS → invia ACK
se no ACK → backoff esponenziale → ritrasmetti
```

**RTS/CTS (opzionale, per stazione nascosta):**
1. A invia **RTS** a AP (Request To Send)
2. AP risponde **CTS** (Clear To Send) — broadcast → tutti sentono
3. A trasmette dati
4. AP invia ACK

### Sicurezza WiFi

| Standard | Descrizione |
|---|---|
| WEP | Deprecato, facilmente violabile |
| WPA | Transitorio, TKIP |
| WPA2 | CCMP/AES, robusto |
| WPA3 | SAE handshake, protezione da dictionary attack |
| 802.1X/EAP | Autenticazione enterprise con RADIUS |

---

## 6.6 Reti Datacenter

I grandi datacenter (Google, Amazon, Facebook) hanno reti specializzate:

**Architettura gerarchica:**
```
        [Core switches]
              │
      [Aggregation switches]
              │
      [Top-of-Rack switches]
              │
          [Server rack]
```

**Soluzioni moderne:**
- **Fat-tree** topology — alta ridondanza e bilanciamento
- **ECMP** (Equal-Cost Multi-Path) — load balancing su link multipli
- **SDN** per controllo centralizzato
- **RDMA** (Remote Direct Memory Access) — latenza ultra-bassa

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **MAC address** = 48 bit, piatto, univoco, assegnato dal produttore
> - **ARP** = risolve IP → MAC nella stessa subnet
> - **CSMA/CD** (Ethernet) = rileva collisioni; **CSMA/CA** (WiFi) = le evita
> - **Switch** = auto-learning, lavora a livello 2, plug-and-play
> - **VLAN** = segmentazione logica su switch fisico
> - **CRC-32** usato in Ethernet per rilevazione errori

## Formule Essenziali

| Formula | Descrizione |
|---|---|
| Efficienza ALOHA puro = 1/2e ≈ 18.4% | Throughput massimo |
| Efficienza ALOHA slottato = 1/e ≈ 37% | Throughput massimo |
| MTU Ethernet = 1500 byte | Payload massimo frame |
| Backoff: slot ∈ {0,...,2^k-1} | Dopo k-esima collisione |

## Link Interni

- [[Capitolo 5 - Livello di Rete (Piano di Controllo)]]
- [[Capitolo 7 - Reti Wireless e Mobili]]
