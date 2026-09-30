---
tags: [reti, capitolo-4, rete, IP, forwarding, kurose-ross]
capitolo: 4
titolo: "Livello di Rete: Piano dei Dati"
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# Capitolo 4 — Livello di Rete: Piano dei Dati

> [!NOTE] Due piani distinti
> Il livello di rete ha due funzioni: **forwarding** (piano dei dati — locale, hardware, veloce) e **routing** (piano di controllo — globale, software, lento). Questo capitolo si concentra sul primo.

---

## 4.1 Panoramica del Livello di Rete

### Piano dei Dati vs Piano di Controllo

| | Piano dei Dati | Piano di Controllo |
|---|---|---|
| **Funzione** | Forwarding pacchetti | Calcolo percorsi |
| **Scope** | Locale (singolo router) | Globale (rete intera) |
| **Velocità** | Nanoseccondi (hardware) | Secondi (software) |
| **Implementazione** | ASIC, linee veloci | CPU del router, protocolli routing |
| **Protocolli** | IP | OSPF, BGP, RIP |

### Forwarding vs Routing

- **Forwarding** = processo locale: il router legge l'header del pacchetto e lo instrada all'interfaccia di uscita corretta
- **Routing** = processo globale: determinare il percorso end-to-end tra sorgente e destinazione

### SDN (Software Defined Networking)

Approccio moderno: il piano di controllo è separato e centralizzato in un **controller remoto** (software).
- Router = semplici dispositivi di forwarding
- Controller calcola le tabelle di forwarding e le distribuisce
- Maggiore flessibilità e programmabilità della rete

---

## 4.2 Cosa Fa un Router

### Componenti Interni

```
┌─────────────────────────────────────────────────────────┐
│                        ROUTER                           │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ Porta    │    │  Struttura   │    │   Porta      │  │
│  │ ingresso │───>│  switching   │───>│   uscita     │  │
│  │ (input)  │    │  (fabric)    │    │   (output)   │  │
│  └──────────┘    └──────────────┘    └──────────────┘  │
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │           Processore di routing                    │ │
│  │  (calcola/mantiene tabella forwarding, protocolli) │ │
│  └────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Porte di Ingresso

Funzioni:
1. **Terminazione del link fisico** (livello fisico e collegamento)
2. **Lookup** nella tabella di forwarding → determina interfaccia di uscita
3. **Accodamento** se la struttura di switching è occupata

**Forwarding basato su destinazione:**
- Match più lungo (longest prefix match)
- Es: `11001000 00010111 00010` → interfaccia 0

**Longest Prefix Match:**
```
Prefix          Interface
11001000 00010111 00010     → 0
11001000 00010111 00011000  → 1
11001000 00010111 00011     → 2
altro                       → 3
```
Per `11001000 00010111 00011001` → corrisponde a regola 1 (28 bit) e 2 (26 bit) → sceglie regola 1 (più lunga)

### Strutture di Switching

| Tipo | Descrizione | Velocità |
|---|---|---|
| **Memoria** | CPU copia il pacchetto | Limitata dalla velocità I/O |
| **Bus** | Bus condiviso tra porte | Limitata dalla velocità bus |
| **Crossbar (rete)** | Connessioni parallele | Alta, N² | 

### Code e Perdita di Pacchetti

**Code all'ingresso (HOL blocking):**
- Head-of-Line blocking: un pacchetto bloccato in testa blocca quelli dietro
- Soluzione: Virtual Output Queues (VOQ)

**Code all'uscita:**
- Se arrivano pacchetti più veloce di quanto escono → buffer overflow
- Perdita di pacchetti

**Scheduling dei pacchetti:**
- **FIFO** — First In First Out
- **Priority Queuing** — code a priorità, serve prima la più alta
- **Round Robin / WFQ** (Weighted Fair Queuing) — a turno tra classi

---

## 4.3 Il Protocollo IP (Internet Protocol)

### IPv4

**Formato Datagramma IPv4:**

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌────┬────┬───────────────┬───────────────────────────────────────┐
│Ver │IHL │     TOS/DSCP  │          Lunghezza totale             │
├────┴────┴───────────────┼───────┬─┬─┬─┬───────────────────────┤
│    Identificazione      │Flags  │D│M│ │  Fragment Offset       │
│                         │       │F│F│ │                        │
├────────────────────────┬┴───────┴─┴─┴─┴───────────────────────┤
│         TTL            │    Protocollo   │  Checksum header    │
├────────────────────────┴─────────────────┴────────────────────┤
│                   Indirizzo IP sorgente                        │
├───────────────────────────────────────────────────────────────┤
│                   Indirizzo IP destinazione                    │
├───────────────────────────────────────────────────────────────┤
│              Opzioni (se IHL > 5)                             │
├───────────────────────────────────────────────────────────────┤
│                          Dati                                 │
└───────────────────────────────────────────────────────────────┘
```

**Campi importanti:**
- **TTL** — Time to Live: decrementato di 1 ad ogni router, scartato a 0 (evita loop infiniti)
- **Protocollo** — 6=TCP, 17=UDP, 1=ICMP
- **DF** (Don't Fragment) — se impostato, il router scarta e invia ICMP "fragmentation needed"
- **Identificazione, Flags, Offset** — per la frammentazione

### Frammentazione e Riassemblaggio

**MTU** (Maximum Transmission Unit): dimensione massima del frame al livello link  
(Ethernet: 1500 byte)

Se il datagramma è > MTU → il router frammenta:
- I frammenti vengono riassemblati **solo al destinatario finale**
- Ogni frammento ha: stesso Identification, MF bit (More Fragments), Fragment Offset

```
Datagramma 4000 byte:
[Header 20B | Dati 3980B]

Frammento 1: [Header | Dati 1480B] ID=x, MF=1, Offset=0
Frammento 2: [Header | Dati 1480B] ID=x, MF=1, Offset=185
Frammento 3: [Header | Dati 1020B] ID=x, MF=0, Offset=370
```

### Indirizzamento IPv4

**Indirizzo IPv4:** 32 bit, notazione decimale puntata (es: 192.168.1.1)

**Subnet (sottorete):** insieme di host con stessa maschera di rete
- Interfacce che possono raggiungere l'una l'altra **senza attraversare un router**

**CIDR** (Classless Inter-Domain Routing): notazione `a.b.c.d/x`
- `x` = numero di bit della parte di rete (prefisso)
- Esempio: `200.23.16.0/23` → 9 bit di host → 512 indirizzi

**Indirizzi speciali:**
| Indirizzo | Uso |
|---|---|
| `0.0.0.0/0` | Default route (qualunque) |
| `127.0.0.1` | Loopback (localhost) |
| `255.255.255.255` | Broadcast limitato |
| `192.168.x.x`, `10.x.x.x`, `172.16-31.x.x` | Privati (RFC 1918) |

### DHCP — Dynamic Host Configuration Protocol

Assegna automaticamente IP agli host.

**Processo DHCP (DORA):**
```
Host            DHCP Server
  │                  │
  │─── DISCOVER ────>│  (broadcast, src: 0.0.0.0)
  │                  │
  │<─── OFFER ───────│  (offre IP, maschera, gateway, DNS)
  │                  │
  │─── REQUEST ─────>│  (accetta l'offerta)
  │                  │
  │<─── ACK ─────────│  (conferma, assegna lease)
```

DHCP fornisce anche: indirizzo del **default gateway**, indirizzo del **server DNS**, **maschera di rete**.

### NAT — Network Address Translation

**Problema:** indirizzi IPv4 esauriti  
**Soluzione NAT:** rete locale usa IP privati, router NAT traduce verso IP pubblico

```
Rete locale (10.0.0.x)    Router NAT     Internet
10.0.0.1:3345  ──────────────────────────> 138.76.29.7:5001
10.0.0.2:3346  ──────────────────────────> 138.76.29.7:5002
```

**Tabella di traduzione NAT:**
| IP locale | Porta locale | IP WAN | Porta WAN |
|---|---|---|---|
| 10.0.0.1 | 3345 | 138.76.29.7 | 5001 |
| 10.0.0.2 | 3346 | 138.76.29.7 | 5002 |

**Controversie NAT:**
- Viola il principio end-to-end (router non dovrebbe modificare numeri di porta)
- Problemi per applicazioni P2P (NAT traversal, STUN, TURN)
- IPv6 è la soluzione vera

---

## 4.4 IPv6

### Motivazione

IPv4: 2³² ≈ 4 miliardi di indirizzi → esauriti (2011 IANA, 2012 APNIC)

### Miglioramenti IPv6

- **128 bit** di indirizzo → 2¹²⁸ ≈ 3.4 × 10³⁸ indirizzi
- Header fisso da **40 byte** (più semplice per il forwarding)
- **Nessuna frammentazione** nei router (solo alla sorgente)
- **Nessun checksum** (delegato ai livelli superiori)
- **Flow label** — marcatura dei flussi per QoS
- ICMPv6 integra funzioni ARP e IGMP di IPv4

### Formato Header IPv6

```
┌──────────────────────────────────────────────┐
│ Ver(4) │ Traffic Class(8) │ Flow Label(20)   │
├──────────────────────────────────────────────┤
│ Payload Length (16) │ Next Header(8) │ Hop Limit(8) │
├──────────────────────────────────────────────┤
│            Indirizzo sorgente (128 bit)       │
│                                              │
├──────────────────────────────────────────────┤
│           Indirizzo destinazione (128 bit)   │
│                                              │
└──────────────────────────────────────────────┘
```

### Notazione IPv6

`2001:0db8:0000:0000:0000:ff00:0042:8329`  
Abbreviato: `2001:db8::ff00:42:8329`

**Indirizzi speciali IPv6:**
- `::1` — loopback
- `fe80::/10` — link-local
- `2000::/3` — global unicast

### Transizione IPv4 → IPv6

**Tunneling:** incapsula datagramma IPv6 nel payload di un datagramma IPv4

```
[IPv4 Header | IPv6 Header | Payload]
```
Usato quando due router IPv6 comunicano attraverso router IPv4.

---

## 4.5 Generalizzazione: SDN e OpenFlow

### OpenFlow

Protocollo per programmare le tabelle di forwarding dei router/switch.

**Match-action:** ogni regola specifica:
- **Match** su campi dell'header (IP sorg/dest, porta TCP/UDP, MAC, VLAN...)
- **Action**: forward, drop, modify header, send to controller

```
Match                          Action
IP dest = 10.1.0.0/24    →    Forward porta 1
IP dest = 10.2.0.0/24    →    Forward porta 4
TCP sport = 80            →    Drop (blocca HTTP)
IP sorg = 10.3.0.0/24    →    Send to controller
```

SDN permette di implementare routing, firewall, NAT, load balancing con le stesse regole di forwarding.

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **Forwarding** (locale) ≠ **Routing** (globale)
> - **Longest prefix match** per la selezione dell'interfaccia
> - **IPv4 = 32 bit**, **IPv6 = 128 bit**
> - **TTL** decrementato ad ogni hop → previene loop
> - **DHCP** = configurazione automatica IP (DORA)
> - **NAT** permette IP privati dietro un solo IP pubblico
> - **Frammentazione** in IPv4 avviene nei router, in IPv6 solo alla sorgente

## Formule Essenziali

| Formula | Descrizione |
|---|---|
| `N. host = 2^(32-x)` | Host per sottorete CIDR /x |
| `Offset frammento = primo byte / 8` | Posizione del frammento nel datagramma |

## Link Interni

- [[Capitolo 3 - Livello di Trasporto]]
- [[Capitolo 5 - Livello di Rete (Piano di Controllo)]]
