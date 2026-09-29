---
tags: [reti, capitolo-5, routing, OSPF, BGP, SDN, kurose-ross]
capitolo: 5
titolo: "Livello di Rete: Piano di Controllo"
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# Capitolo 5 — Livello di Rete: Piano di Controllo

> [!NOTE] Come si calcolano i percorsi
> Il piano di controllo determina come i pacchetti vengono instradati da sorgente a destinazione. Include algoritmi di routing e protocolli come OSPF (intra-AS) e BGP (inter-AS).

---

## 5.1 Introduzione

Il piano di controllo popola le tabelle di forwarding dei router.

### Due Approcci

**Routing tradizionale (per-router):**
- Ogni router esegue il proprio algoritmo di routing
- I router si scambiano informazioni tramite protocolli (OSPF, BGP)
- Ogni router computa la propria tabella

**SDN (Software-Defined Networking):**
- Controller centralizzato (logicamente) computa le tabelle
- Le distribuisce ai router tramite protocollo (OpenFlow)
- Router = semplici dispositivi di forwarding

---

## 5.2 Algoritmi di Routing

### Classificazione

| Tipo | Caratteristica | Esempio |
|---|---|---|
| **Link-state** | Ogni router conosce la topologia completa | OSPF, IS-IS |
| **Distance vector** | Ogni router conosce solo i vicini diretti | RIP, BGP (concettualmente) |
| **Centralizzato** | Controller esterno calcola i percorsi | SDN |
| **Statico** | Percorsi configurati manualmente | — |

### Dijkstra — Algoritmo Link-State

**Input:** grafo della rete con costi sui link  
**Output:** albero dei percorsi minimi dalla sorgente

**Pseudocodice:**
```
Inizializzazione:
  N' = {u}  (nodi visitati)
  D(v) = costo da u a v (diretto se vicino, ∞ altrimenti)
  
Loop:
  trova w ∉ N' con D(w) minima
  aggiungi w a N'
  per ogni vicino v di w non in N':
    D(v) = min(D(v), D(w) + c(w,v))
  
fino a N' = tutti i nodi
```

**Complessità:** O(n²) — ottimizzabile con heap a O(n log n)

**Oscillazioni:** Se i costi dipendono dal traffico → i percorsi possono oscillare. Soluzione: randomizzare i tempi di advertisement.

### Bellman-Ford — Algoritmo Distance Vector

**Equazione di Bellman-Ford:**
$$d_x(y) = \min_v \{ c(x,v) + d_v(y) \}$$

Ogni nodo x mantiene un vettore delle distanze verso tutti gli altri nodi, aggiornato dai vicini.

**Algoritmo DV:**
```
Ogni nodo x:
  D_x(y) = costo verso y per ogni destinazione y
  
Loop:
  aspetta cambiamento costo link o ricezione DV da vicino
  ricalcola DV: D_x(y) = min_v{c(x,v) + D_v(y)}
  se cambiato → notifica vicini
```

**Count-to-infinity:** Con link failure, i nodi si aggiornano lentamente.
- Soluzione: **Poisoned Reverse** — se x raggiunge z via y, allora x dice a y che D_x(z) = ∞

| | Link-State (LS) | Distance Vector (DV) |
|---|---|---|
| **Messaggi** | O(n×E) per broadcast | scambio solo con vicini |
| **Convergenza** | Rapida, O(n²) | Lenta, possibile count-to-infinity |
| **Robustezza** | Router può pubblicare costi errati per sé | Errore si propaga su tutti |

---

## 5.3 OSPF — Open Shortest Path First

### Caratteristiche

- Protocollo **link-state** per routing **intra-AS** (dentro un Autonomous System)
- Standardizzato da IETF (RFC 2328 per OSPFv2, RFC 5340 per OSPFv3/IPv6)
- Ogni router esegue Dijkstra sulla mappa completa della topologia AS
- Messaggi OSPF trasportati direttamente da **IP** (non TCP/UDP), protocollo 89
- Autenticazione dei messaggi di routing

### Funzionamento

1. Router invia **Hello** ai vicini → scopre adiacenze
2. Router formano adjacency con i vicini
3. Ogni router invia **LSA** (Link State Advertisement) → flood dell'intera topologia
4. Ogni router costruisce il **Link State Database** (mappa completa)
5. Dijkstra → calcola shortest path tree

### OSPF Gerarchico

Per reti grandi, si suddivide l'AS in **aree**:
- **Backbone area (Area 0)** — connette tutte le altre aree
- **Area interne** — routing interno limitato all'area
- **ABR** (Area Border Router) — connette area a backbone
- **ASBR** (AS Boundary Router) — connette a altri AS

Riduce il traffico di routing e la dimensione del LSDB.

---

## 5.4 BGP — Border Gateway Protocol

### Ruolo di BGP

BGP è il protocollo di routing **inter-AS** (tra Autonomous System diversi) — "la colla che tiene insieme Internet".

- **eBGP** — scambia informazioni di raggiungibilità tra AS diversi
- **iBGP** — distribuisce le info eBGP all'interno dell'AS

### Attributi BGP

Ogni route BGP porta attributi:
- **AS-PATH** — lista degli AS attraversati
- **NEXT-HOP** — indirizzo IP del primo router nel path esterno
- **LOCAL-PREF** — preferenza locale (intra-AS, non propagato)
- **MED** (Multi-Exit Discriminator) — suggerisce al vicino il link preferito
- **Community** — tag per policy routing

### Selezione del Percorso BGP

L'algoritmo di selezione esamina gli attributi in ordine:
1. Massima **LOCAL-PREF** (politica locale)
2. **AS-PATH** più corto
3. **Origine** più vicina (IGP < EGP < INCOMPLETE)
4. **MED** più basso
5. **eBGP** preferita a **iBGP**
6. Percorso con next-hop più vicino (hot potato routing)
7. Router ID più basso (tiebreaker)

### Policy Routing e Relazioni tra AS

Gli AS hanno relazioni commerciali che determinano la politica di routing:
- **Customer → Provider** — il customer paga il provider per accesso a Internet
- **Peer → Peer** — scambio di traffico gratuito tra AS di pari livello

**Principio di policy:**
- Un AS non trasporta traffico tra due provider diversi (no transit)
- Un provider non annuncia rotte di un peer a un altro peer

### Hot Potato Routing

Il router sceglie l'uscita più vicina (meno costosa) verso l'AS di destinazione, senza considerare il costo end-to-end. Ogni AS "butta via" il pacchetto il prima possibile.

---

## 5.5 SDN — Piano di Controllo

### Architettura SDN

```
┌─────────────────────────────────────────┐
│          Applicazioni di controllo       │
│    (routing, firewall, load balancer)   │
├─────────────────────────────────────────┤
│         Controller SDN                  │
│  ┌──────────┐ ┌──────────┐ ┌─────────┐ │
│  │Link-state│ │FIB Mgmt  │ │Topology │ │
│  │  Manager │ │  Module  │ │  Mgr    │ │
│  └──────────┘ └──────────┘ └─────────┘ │
│       Northbound API (REST)             │
├─────────────────────────────────────────┤
│       Southbound API (OpenFlow)         │
├───────────────────┬─────────────────────┤
│   Switch/Router 1 │   Switch/Router 2   │
│   (solo forwarding│   (solo forwarding) │
└───────────────────┴─────────────────────┘
```

**Northbound API:** interfaccia verso le applicazioni di controllo (REST, Python libs)  
**Southbound API:** interfaccia verso i dispositivi di rete (OpenFlow, NETCONF)

### OpenFlow — Piano di Controllo

Il controller comunica con gli switch tramite OpenFlow:

**Messaggi controller → switch:**
- `MODIFY_FLOW_TABLE` — aggiunge/modifica regole
- `PACKET_OUT` — invia pacchetto da porta specifica

**Messaggi switch → controller:**
- `PACKET_IN` — invia pacchetto al controller (nessuna regola applicabile)
- `FLOW_REMOVED` — notifica scadenza regola
- `PORT_STATUS` — cambiamento stato porta

---

## 5.6 ICMP — Internet Control Message Protocol

**ICMP** trasporta messaggi di errore e informativi tra host e router.

Incapsulato in IP (protocollo 1).

### Tipi ICMP Principali

| Tipo | Codice | Significato |
|---|---|---|
| 0 | 0 | Echo Reply (ping) |
| 3 | 0-15 | Destination Unreachable (host/net/port/proto unreachable) |
| 4 | 0 | Source Quench (deprecato) |
| 5 | 0-3 | Redirect |
| 8 | 0 | Echo Request (ping) |
| 11 | 0 | TTL Expired (usato da traceroute) |
| 12 | 0-1 | Parameter Problem |

### Traceroute e ICMP

```
Traceroute invia UDP con TTL=1 → Router 1 scarta e risponde ICMP TTL expired
Traceroute invia UDP con TTL=2 → Router 2 scarta e risponde ICMP TTL expired
...
Traceroute invia UDP con TTL=n → Host destinazione risponde ICMP Port Unreachable
```

---

## 5.7 Gestione della Rete — SNMP e NETCONF

### SNMP (Simple Network Management Protocol)

**Architettura:**
- **Manager** — applicazione di gestione (NMS - Network Management System)
- **Agent** — software sul dispositivo gestito (router, switch)
- **MIB** (Management Information Base) — database di oggetti gestibili

**Operazioni SNMP:**
- `GET` — manager legge variabile dall'agent
- `SET` — manager imposta variabile nell'agent
- `TRAP` — agent notifica evento al manager (asincrono)

### NETCONF / YANG

Protocollo moderno di gestione (RFC 6241):
- Basato su XML/JSON
- Transazionale (atomic)
- YANG = linguaggio per modellare dati di configurazione
- Più espressivo di SNMP

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **OSPF** = link-state, intra-AS, usa Dijkstra
> - **BGP** = distance-vector ibrido, inter-AS, policy-based
> - **AS-PATH** in BGP previene i loop
> - **Hot potato routing** = butta via il pacchetto il prima possibile
> - **SDN** separa piano di controllo (controller) da piano dei dati (switch)
> - **OpenFlow** = protocollo southbound per programmare switch SDN

## Formule Essenziali

| Formula | Descrizione |
|---|---|
| `D_x(y) = min_v { c(x,v) + D_v(y) }` | Bellman-Ford (DV) |
| Dijkstra: `D(v) = min(D(v), D(w) + c(w,v))` | Link-State update |

## Link Interni

- [[Capitolo 4 - Livello di Rete (Piano dei Dati)]]
- [[Capitolo 6 - Livello di Collegamento e LAN]]
