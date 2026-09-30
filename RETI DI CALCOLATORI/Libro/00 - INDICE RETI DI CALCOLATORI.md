---
tags: [reti, indice, kurose-ross, overview]
titolo: "Indice - Reti di Calcolatori e Internet"
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# 📡 Reti di Calcolatori e Internet
## Kurose & Ross — Un Approccio Top-Down (8a Edizione)

> [!NOTE] Come usare queste note
> Ogni capitolo ha la sua nota dedicata con definizioni, formule, tabelle di confronto e diagrammi ASCII. I link interni connettono i concetti correlati tra capitoli.

---

## 🗂️ Struttura del Libro

```
Applicazione     → Cap. 2 → HTTP, DNS, SMTP, P2P, CDN
      ↑
Trasporto        → Cap. 3 → TCP, UDP, controllo congestione
      ↑
Rete             → Cap. 4 → IP, NAT, IPv6, forwarding
(Piano dati)         Cap. 5 → OSPF, BGP, SDN, routing
      ↑
Collegamento     → Cap. 6 → Ethernet, MAC, WiFi, Switch
      ↑
Fisico           → Cap. 1 → Media, accesso, infrastruttura
      ↑
Trasversale      → Cap. 7 → Wireless e mobilità
                   Cap. 8 → Sicurezza (tutti i livelli)
```

---

## 📚 Capitoli

| # | Titolo | Temi Principali |
|---|---|---|
| [[Capitolo 1 - Reti di Calcolatori e Internet]] | Reti e Internet | ISP, protocolli, ritardi, throughput, OSI, TCP/IP |
| [[Capitolo 2 - Livello di Applicazione]] | Applicazione | HTTP, DNS, SMTP, POP3, IMAP, P2P, CDN, DASH |
| [[Capitolo 3 - Livello di Trasporto]] | Trasporto | TCP, UDP, RDT, controllo flusso/congestione, QUIC |
| [[Capitolo 4 - Livello di Rete (Piano dei Dati)]] | IP, Forwarding | IPv4, IPv6, NAT, DHCP, ARP, router, CIDR |
| [[Capitolo 5 - Livello di Rete (Piano di Controllo)]] | Routing | Dijkstra, Bellman-Ford, OSPF, BGP, SDN |
| [[Capitolo 6 - Livello di Collegamento e LAN]] | Collegamento | Ethernet, MAC, ARP, switch, VLAN, WiFi 802.11 |
| [[Capitolo 7 - Reti Wireless e Mobili]] | Wireless | WiFi, 4G LTE, 5G NR, mobilità, handoff |
| [[Capitolo 8 - Sicurezza nelle Reti]] | Sicurezza | AES, RSA, TLS, IPsec, WPA3, firewall, IDS |

---

## 🔑 Concetti Chiave Trasversali

### Protocolli per Livello

```
L5 Applicazione:  HTTP/3, DNS, SMTP, FTP, DHCP, SNMP
L4 Trasporto:     TCP, UDP, QUIC
L3 Rete:          IP (v4/v6), ICMP, OSPF, BGP, RIP
L2 Collegamento:  Ethernet, WiFi (802.11), PPP
L1 Fisico:        Fibra, doppino, radio, satellite
```

### Formule da Sapere

| Formula | Significato | Capitolo |
|---|---|---|
| `d_tot = d_elab + d_accod + d_trasm + d_prop` | Ritardo totale nodo | [[Capitolo 1 - Reti di Calcolatori e Internet]] |
| `d_trasm = L/R` | Ritardo trasmissione | [[Capitolo 1 - Reti di Calcolatori e Internet]] |
| `d_prop = d/s` | Ritardo propagazione | [[Capitolo 1 - Reti di Calcolatori e Internet]] |
| `Throughput = min(R₁,...,Rₙ)` | Collo di bottiglia | [[Capitolo 1 - Reti di Calcolatori e Internet]] |
| `TimeoutTCP = EstRTT + 4×DevRTT` | Timeout TCP adattivo | [[Capitolo 3 - Livello di Trasporto]] |
| `D_x(y) = min_v{c(x,v) + D_v(y)}` | Bellman-Ford | [[Capitolo 5 - Livello di Rete (Piano di Controllo)]] |
| `MTU Ethernet = 1500 byte` | Max payload frame | [[Capitolo 6 - Livello di Collegamento e LAN]] |

### Confronti Fondamentali

| A vs B | Differenza chiave | Capitolo |
|---|---|---|
| TCP vs UDP | Affidabilità e controllo congestione | [[Capitolo 3 - Livello di Trasporto]] |
| IPv4 vs IPv6 | Dimensione indirizzo (32 vs 128 bit) | [[Capitolo 4 - Livello di Rete (Piano dei Dati)]] |
| OSPF vs BGP | Intra-AS vs Inter-AS | [[Capitolo 5 - Livello di Rete (Piano di Controllo)]] |
| Switch vs Router | Livello 2 vs Livello 3 | [[Capitolo 6 - Livello di Collegamento e LAN]] |
| CSMA/CD vs CSMA/CA | Rileva vs Evita collisioni | [[Capitolo 6 - Livello di Collegamento e LAN]] |
| 4G vs 5G | EPC vs SBA, slicing, mmWave | [[Capitolo 7 - Reti Wireless e Mobili]] |
| TLS vs IPsec | Trasporto vs Rete | [[Capitolo 8 - Sicurezza nelle Reti]] |

---

## ⚡ Quick Reference — Porte Comuni

| Porta | Protocollo | Trasporto |
|---|---|---|
| 20/21 | FTP | TCP |
| 22 | SSH | TCP |
| 23 | Telnet | TCP |
| 25 | SMTP | TCP |
| 53 | DNS | UDP (anche TCP per zone transfer) |
| 67/68 | DHCP | UDP |
| 80 | HTTP | TCP |
| 110 | POP3 | TCP |
| 143 | IMAP | TCP |
| 161/162 | SNMP | UDP |
| 443 | HTTPS (TLS) | TCP |
| 993 | IMAPS | TCP |
| 995 | POP3S | TCP |

---

## 📝 Note di Studio

> [!TIP] Strategia di studio
> 1. Inizia dal [[Capitolo 1 - Reti di Calcolatori e Internet]] per la visione d'insieme
> 2. Studia cap. 2-3 (applicazione e trasporto) — protocolli più vicini all'esperienza utente
> 3. Cap. 4-5 sono il cuore teorico — IP e routing
> 4. Cap. 6 chiude il giro end-to-end con il livello fisico/link
> 5. Cap. 7-8 sono temi specialistici ma molto d'esame

> [!IMPORTANT] Domande d'esame frequenti
> - Calcola il ritardo end-to-end con N router store-and-forward
> - Spiega il three-way handshake TCP
> - Come funziona il DNS gerarchico (query iterativa)
> - Differenza CSMA/CD e CSMA/CA
> - Dijkstra su un grafo dato
> - Come funziona NAT con un esempio numerico
> - Slow Start e Congestion Avoidance nel controllo della congestione
> - Come funziona TLS (handshake ad alto livello)
