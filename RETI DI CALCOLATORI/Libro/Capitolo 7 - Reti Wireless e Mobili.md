---
tags: [reti, capitolo-7, wireless, mobile, 4G, 5G, kurose-ross]
capitolo: 7
titolo: Reti Wireless e Mobili
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# Capitolo 7 — Reti Wireless e Mobili

> [!NOTE] Il segnale nell'aria
> Le reti wireless affrontano sfide uniche: attenuazione del segnale, interferenze, multi-path fading. La mobilità aggiunge la complessità del handoff e della gestione della posizione.

---

## 7.1 Introduzione

### Elementi di una Rete Wireless

```
Host wireless ──radio──> Stazione base ──────────────> Rete cablata
                         (AP WiFi, BS 4G/5G)           (Internet)
```

- **Host wireless** — smartphone, laptop, IoT device
- **Stazione base (BS)** — punto di accesso radio (AP WiFi, eNodeB 4G, gNB 5G)
- **Link wireless** — collegamento radio host-BS
- **Infrastruttura** — rete cablata behind the BS

### Classificazione Reti Wireless

| Caratteristica | WiFi (802.11) | 4G LTE | 5G NR |
|---|---|---|---|
| Raggio | ~100m | ~km | Varia (mmWave: ~50m) |
| Velocità | Fino a Gbps (WiFi 6) | ~150 Mbps DL | ~Gbps |
| Frequenza | 2.4/5/6 GHz | 700MHz - 3.5GHz | Sub-6GHz + mmWave |
| Mobilità | Bassa | Alta | Alta |
| Standard | IEEE 802.11 | 3GPP Release 8+ | 3GPP Release 15+ |

---

## 7.2 Link Wireless e Caratteristiche

### Problemi del Canale Radio

**Path loss (attenuazione):**
- Il segnale si attenua con la distanza: potenza ∝ 1/d²  (spazio libero)
- In ambienti reali l'esponente è 2-4

**Fading multi-path:**
- Il segnale rimbalza su edifici, oggetti → arriva con percorsi multipli e ritardi diversi
- Interferenza costruttiva e distruttiva → potenza del segnale variabile

**Interferenza:**
- Stessa banda → interferenza da altre sorgenti radio (altri AP, microonde, Bluetooth a 2.4GHz)

**Problema della stazione nascosta:**
```
A ─────────── B ─────────── C
      [range A]   [range C]
A e C non si sentono → trasmettono entrambi a B → collisione
```

### SNR e BER

**SNR** (Signal-to-Noise Ratio): rapporto segnale/rumore in dB

Per un dato SNR:
- Modulazione più aggressiva (64-QAM) → più bit/s ma più errori (BER alto)
- Modulazione conservativa (BPSK) → meno bit/s ma BER basso

Il sistema si adatta: **AMC** (Adaptive Modulation and Coding) sceglie la modulazione in base al SNR misurato.

---

## 7.3 WiFi — 802.11 (approfondimento)

### Standard 802.11

| Standard | Frequenza | Velocità max | Note |
|---|---|---|---|
| 802.11b | 2.4 GHz | 11 Mbps | 2000 |
| 802.11a | 5 GHz | 54 Mbps | Meno diffuso |
| 802.11g | 2.4 GHz | 54 Mbps | 2003 |
| 802.11n (WiFi 4) | 2.4+5 GHz | 600 Mbps | MIMO 4×4 |
| 802.11ac (WiFi 5) | 5 GHz | ~3.5 Gbps | MU-MIMO, beamforming |
| 802.11ax (WiFi 6/6E) | 2.4+5+6 GHz | ~9.6 Gbps | OFDMA, BSS coloring |

### MIMO (Multiple Input Multiple Output)

- Antenne multiple su trasmettitore e ricevitore
- **Spatial multiplexing** — trasmissioni parallele = aumento del throughput
- **Diversity** — robustezza contro il fading

### Risparmio Energetico 802.11

- Il nodo segnala all'AP di voler "dormire" (TIM bit nel beacon)
- AP bufferizza i frame per i nodi dormienti
- Il nodo si sveglia periodicamente per controllare il beacon

---

## 7.4 Mobilità: Principi

### Problemi della Mobilità

1. **Handoff** — il dispositivo si sposta da una BS a un'altra mantenendo le connessioni
2. **Localizzazione** — la rete deve sapere dove si trova l'utente
3. **Indirizzamento** — l'IP del dispositivo non cambia con il movimento (idealmente)

### Approcci alla Gestione della Mobilità

**Scenario 1: Il dispositivo si muove nella stessa rete (WiFi)**
- Cambia AP → la subnet rimane la stessa
- IP non cambia → nessun problema TCP

**Scenario 2: Il dispositivo si muove tra reti diverse**
- Cambia subnet → IP cambierebbe
- TCP connessioni interrotte!

### Mobile IP

Standard per mantenere lo stesso IP con la mobilità tra reti:

**Componenti:**
- **Home Agent (HA)** — router nella rete di casa
- **Foreign Agent (FA)** — router nella rete visitata
- **Care-of-Address (CoA)** — IP temporaneo nella rete visitata

**Funzionamento:**
1. Il nodo mobile arriva in rete visitata → ottiene CoA dal FA
2. FA registra CoA con HA
3. Traffico verso IP permanente → HA lo "tunnelizza" verso CoA
4. Risposta dal nodo → diretta al corrispondente (triangle routing)

**Problemi:** triangle routing inefficiente, overhead tunnel

---

## 7.5 Reti Cellulari: 4G LTE

### Architettura 4G

```
[UE] ──air──> [eNodeB] ──S1──> [EPC: SGW, PGW, MME, HSS] ──> Internet
  (telefono)   (Base       (Evolved Packet Core)
               Station)
```

**Componenti EPC:**
- **MME** (Mobility Management Entity) — gestisce autenticazione e mobilità
- **SGW** (Serving Gateway) — instrada il traffico dati locale
- **PGW** (PDN Gateway) — punto di uscita verso Internet, assegna IP
- **HSS** (Home Subscriber Server) — database abbonati (IMSI, profilo)

### LTE Radio

- **OFDM** (Orthogonal Frequency Division Multiplexing) — downlink
- **SC-FDMA** — uplink (riduce il PAPR per risparmio energetico)
- **Resource Block** — unità di allocazione: 12 sottoportanti × 0.5ms

**LTE-A (LTE Advanced):**
- **Carrier Aggregation** — combina più bande
- **MIMO avanzato**
- **HetNet** — small cell (picocell, femtocell) per offload traffico

### Handoff in 4G

**X2 Handoff (tra eNodeB):**
```
1. eNodeB source misura qualità segnale del UE
2. Decide handoff → contatta eNodeB target via X2
3. Target prepara risorse
4. UE esegue handoff (associazione a nuovo eNodeB)
5. Rerouting del traffico
6. Rilascio risorse al source
```

---

## 7.6 Reti Cellulari: 5G

### Novità di 5G

**5G NR (New Radio):**
- **eMBB** (Enhanced Mobile Broadband) — alta velocità (Gbps)
- **URLLC** (Ultra-Reliable Low-Latency Communication) — <1ms, per industria/auto
- **mMTC** (massive Machine Type Communication) — IoT, milioni di dispositivi/km²

### Frequenze 5G

- **Sub-6 GHz** — simile a 4G, buona copertura, velocità moderate (≤1 Gbps)
- **mmWave (24-100 GHz)** — altissima velocità (≤10 Gbps), copertura limitata (~50-100m), attraversa male gli ostacoli

### Architettura 5G SA (Standalone)

```
[gNB] ──N2──> [AMF] ──N11──> [SMF] ──N4──> [UPF] ──> Internet
              (Access &       (Session     (User Plane
               Mobility)      Mgmt)         Function)
```

**Service-Based Architecture (SBA):**
- Funzioni di rete implementate come microservizi HTTP/2
- Flessibilità e scalabilità cloud-native
- **Network Slicing** — creare reti virtuali dedicate per diversi use case

### Network Slicing

Un'unica infrastruttura fisica supporta più "slice" logiche indipendenti:
- Slice eMBB per streaming
- Slice URLLC per veicoli autonomi
- Slice mMTC per sensori IoT

Ogni slice ha risorse, QoS e protocolli dedicati.

---

## 7.7 Confronto Tecnologie Wireless

| Tecnologia | Range | Throughput | Mobilità | Uso tipico |
|---|---|---|---|---|
| **Bluetooth** | ~10m | ~3 Mbps | Bassa | Auricolari, periferiche |
| **Zigbee (802.15.4)** | ~100m | ~250 kbps | Bassa | IoT, smart home |
| **WiFi 6 (802.11ax)** | ~100m | ~9.6 Gbps | Media | LAN indoor |
| **LTE/4G** | ~km | ~150 Mbps | Alta | Mobile broadband |
| **5G sub-6** | ~km | ~1 Gbps | Alta | Mobile broadband |
| **5G mmWave** | ~50m | ~10 Gbps | Bassa | Dense urban |
| **Starlink (LEO)** | Globale | ~100-300 Mbps | Media | Rurale, marittimo |

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **Multi-path fading** e **path loss** sono le sfide principali del wireless
> - **Stazione nascosta** richiesta RTS/CTS o CSMA/CA per mitigarla
> - **AMC** adatta modulazione e coding rate al SNR
> - **4G** = OFDM, EPC, eNodeB | **5G** = NR, SBA, slicing, mmWave
> - **Handoff** = cambio di BS mantenendo la connessione
> - **Network slicing** in 5G permette infrastrutture virtuali separate

## Link Interni

- [[Capitolo 6 - Livello di Collegamento e LAN]]
- [[Capitolo 8 - Sicurezza nelle Reti]]
