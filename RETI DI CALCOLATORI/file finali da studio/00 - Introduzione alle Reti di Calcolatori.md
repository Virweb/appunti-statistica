---
materia: Reti di Calcolatori
tipo: Appunti di studio approfonditi
lezione: "0 - Introduzione"
riferimenti_slide: "Lezione 0 - Introduzione Reti di Calcolatori.pdf"
capitoli_libro: ["Capitolo 1 - Reti di Calcolatori e Internet (1.1 - 1.7)"]
tags: [reti, introduzione, architettura-a-livelli, commutazione, obsidian]
---

# 00 — Introduzione alle Reti di Calcolatori

> [!NOTE] Nota Metodologica e Didattica
> La presente nota integra in maniera sistematica la visione d'insieme presentata nelle **Slide della Lezione 0 (Prof. Filippo Lanubile)** con la trattazione teorica e formale del testo di riferimento **Kurose & Ross (8ª ed., Capitolo 1)**. Il documento analizza l'architettura delle reti dal livello fisico a quello logico, fornendo modelli matematici per le prestazioni, confronti prestazionali tra paradigmi di commutazione e dettagli sull'organizzazione in livelli di protocollo.

---

## 1. Definizione e Vista Duale della Rete

Una **rete di calcolatori** è una collezione di sistemi autonomi interconnessi mediante una pluralità di mezzi trasmissivi, in grado di scambiare informazioni attraverso protocolli condivisi.

```
+---------------------------------------+
|       APPLICAZIONI DISTRIBUITE        |
|   (Web, Email, Streaming, P2P, IoT)   |
+-------------------+-------------------+
                    | [Socket API]
                    v
+---------------------------------------+
|           NUCLEO DELLA RETE           |
|   [Router A] <-> [Router B] <-> ...   |
+-------------------+-------------------+
                    | [Rete di Accesso]
                    v
+---------------------------------------+
|           SISTEMI TERMINALI           |
|      (Client, Server, Mobile)         |
+---------------------------------------+
```

### 1.1 La Vista dei Servizi (Nuts and Bolts vs Service View)

Nello studio della disciplina si distinguono due prospettive complementari:

1. **Vista dell'Infrastruttura (Hardware/Software Nuts & Bolts):**
   - **Host (Sistemi Periferici / Edge Devices):** Elaboratori che eseguono i processi applicativi (PC, Server nei data center, Smartphone, dispositivi IoT).
   - **Nodi Intermedi di Commutazione (Packet Switches):** Dispositivi che instradano il traffico senza ospitare applicazioni utente (Router nel livello di rete, Switch nel livello di collegamento).
   - **Canali di Comunicazione (Links):** Mezzi fisici (rame, fibra, etere) caratterizzati da un *Transmission Rate* espresso in bit al secondo ($R \text{ bps}$).
   - **Protocolli:** Insiemi di regole rigorose che governano il formato, l'ordine di invio/ricezione dei messaggi e le azioni intraprese a seguito di ciascun evento. Gli standard Internet ufficiali sono definiti nelle **RFC** (*Request for Comments*) curate dall'**IETF** (*Internet Engineering Task Force*).

2. **Vista dei Servizi forniti alle Applicazioni:**
   - La rete si presenta come un'**infrastruttura di comunicazione trasparente** che offre canali logici ai processi applicativi.
   - Fornisce alle applicazioni un'Interfaccia di Programmazione (**API - Socket Interface**), ovvero un insieme di primitive con cui un programma sorgente chiede alla rete di recapitare dati a una destinazione, nascondendo la complessità sottostante del routing e del livello fisico.

---

## 2. Architettura Geometrica e Tassonomia delle Reti

### 2.1 Struttura della Rete: Edge, Access e Core

L'architettura globale di Internet è suddivisa in tre macro-componenti strutturali:

```
+-----------------------------------------+
|        SISTEMI TERMINALI (EDGE)         |
|       Client & Server applicativi       |
+--------------------+--------------------+
                     |
                     v
+-----------------------------------------+
|             RETI DI ACCESSO             |
|   (DSL, Cavo HFC, FTTH, WiFi, 5G/LTE)   |
+--------------------+--------------------+
                     |
                     v
+-----------------------------------------+
|          NUCLEO DELLA RETE (CORE)       |
|    Router magliati e Gerarchia ISP      |
+-----------------------------------------+
```

#### A. Reti di Accesso (Access Networks)
Collegano fisicamente i sistemi terminali al primo nodo del nucleo (*Edge Router*):
- **Accesso Residenziale:**
  - **DSL (Digital Subscriber Line):** Utilizza il preesistente doppino telefonico in rame. Il multiplexer **DSLAM** (*DSL Access Multiplexer*) presso la centrale telefonica separa i canali in frequenza: frequenza vocale ($0\text{--}4\text{ kHz}$), canale dati upstream e downstream.
  - **Cavo (HFC - Hybrid Fiber Coax):** Rete ibrida con fibra fino al nodo di quartiere e cavo coassiale condiviso fino alle abitazioni. Gestita dal **CMTS** (*Cable Modem Termination System*). Essendo un mezzo broadcast condiviso, soffre di degradazione delle prestazioni in caso di elevata attività simultanea degli utenti del quartiere.
  - **FTTH (Fiber To The Home):** Fibra ottica stesa fino all'utente finale. Può utilizzare architetture **AON** (*Active Optical Network*, con switch alimentati) o **PON** (*Passive Optical Network*, con *splitter* ottici passivi non alimentati). Velocità nell'ordine dei $\text{Gbps}$.
- **Accesso Aziendale (LAN):**
  - **Ethernet (IEEE 802.3):** Tecnologie cablate su cavi UTP (Unshielded Twisted Pair) Cat5e/Cat6/Cat7 collegati a switch Ethernet centrali (velocità da $100\text{ Mbps}$ a $100\text{ Gbps}$).
  - **WiFi (IEEE 802.11):** Accesso wireless a corto raggio collegato alla LAN cablata tramite *Access Point* (AP).
- **Accesso Mobile:**
  - Reti cellulari **3G / 4G LTE / 5G**, in cui gli host comunicano direttamente con le stazioni base (*Base Stations / gNodeB*) distribuite sul territorio.

#### B. Mezzi di Trasmissione
- **Mezzi Guidati (Guided Media):** Le onde elettromagnetiche sono instradate lungo un percorso solido.
  - *Doppino Intrecciato (Twisted Pair):* Due fili di rame isolati ed elicoidali per ridurre l'interferenza elettromagnetica.
  - *Cavo Coassiale:* Due conduttori concentrici in rame, altamente schermato per elevati tassi di trasmissione.
  - *Fibra Ottica:* Impulsi luminosi propagati in un sottile filamento di vetro. Bassa attenuazione, immune a interferenze elettromagnetiche, banda trasmissiva teoricamente immensa ($\text{Tbps}$).
- **Mezzi Non Guidati (Unguided / Wireless Media):** Le onde si propagano nello spazio elettromagnetico.
  - *Canali Radio Terrestri:* WiFi, Bluetooth, Reti cellulari, Microonde.
  - *Canali Satellitari:* Satelliti **GEO** (Geostazionari, quota $\approx 36.000\text{ km}$, RTT elevato $\approx 270\text{ ms}$) e **LEO** (Low Earth Orbit, Starlink, quota inferiore, minori latenze).

---

### 2.2 Tipologie di Collegamenti e Classificazione Geografica

#### Collegamenti Diretti vs Indiretti
- **Punto a Punto (Point-to-Point):** Collegamento dedicato esclusivo tra una singola coppia di nodi. In una rete a maglia completa di $N$ calcolatori, il numero di collegamenti necessari cresce quadraticamente:
  $$L = \frac{N(N - 1)}{2} = \Theta(N^2)$$
- **Ad Accesso Multiplo (Shared Broadcast Link):** Un unico canale di comunicazione fisico è condiviso da $N$ calcolatori (es. bus condiviso, rete WiFi). Richiede protocolli MAC (*Medium Access Control*) per prevenire e gestire le collisioni.
- **Collegamenti Indiretti (Switching Networks):** I nodi non sono collegati da linee dedicate individuali, ma comunicano attraversando una rete di nodi intermedi (*switch* o *router*) che effettuano la commutazione dei messaggi.

#### Classificazione per Estensione Geografica

| Tipologia | Raggio d'Azione | Mezzo / Caratteristiche | Esempio / Standard |
|---|---|---|---|
| **PAN** (*Personal Area Network*) | $\approx 10\text{ m}$ (Ambito personale) | Wireless a corto raggio, infrarossi | Bluetooth (IEEE 802.15.1) |
| **LAN** (*Local Area Network*) | Edificio, Campus (senza suolo pubblico) | Elevata velocità ($100\text{ Mbps} \text{ -- } 10\text{ Gbps}$), rame/fibra/WiFi | Ethernet (802.3), WiFi (802.11) |
| **MAN** (*Metropolitan Area Network*) | Ambito cittadino | Dorsali in fibra ottica, accordi locali | Reti civiche, Anelli di quartiere |
| **WAN** (*Wide Area Network*) | Nazionale, Continentale, Globale | Collegamenti indiretti commutati, fibra, satellite | Internet backbone |

---

## 3. Tecniche di Multiplazione e Condivisione delle Risorse

Per trasmettere più flussi indipendenti su un unico mezzo fisico condiviso occorre applicare una tecnica di **multiplazione** (multiplexing).

```
[FDM: Divisione in Frequenza]
Freq ^
  f3 | [ Canale 3 (continuo) ]
  f2 | [ Canale 2 (continuo) ]
  f1 | [ Canale 1 (continuo) ]
     +-------------------------> Tempo

[TDM: Slot Temporali Fissi]
Freq ^
  f0 | [Sl 1][Sl 2][Sl 3][Sl 1][Sl 2]...
     +-------------------------> Tempo

[Multiplazione Statistica]
Freq ^
  f0 | [Pkt A1][Pkt B1][Pkt A2][Pkt C1]...
     +-------------------------> Tempo
```

### 3.1 Multiplazione a Divisione di Frequenza (FDM - Frequency Division Multiplexing)
- Lo spettro di frequenza del mezzo viene suddiviso in canali (bande) di ampiezza fissa.
- Ogni comunicazione dispone **in modo continuo ed esclusivo** della propria banda di frequenza per l'intera durata della sessione.
- *Tipico uso:* Radiodiffusione AM/FM, TV via cavo, trasmissione analogica legacy.

### 3.2 Multiplazione a Divisione di Tempo Fissa (TDM - Time Division Multiplexing)
- Il tempo viene suddiviso in *frame* temporali periodici, ciascuno partizionato in un numero fisso di *slot temporali* (time slots).
- A ciascuna connessione viene assegnato uno slot dedicato in ogni frame a rotazione ciclica.
- *Svantaggio dell'allocazione statica (FDM/TDM):* Se la sorgente non trasmette dati (silenzio), lo slot o la banda assegnati rimangono inutilizzati, causando uno spreco inefficiente della risorsa trasmissiva.

### 3.3 Multiplazione Statistica (Statistical TDM / Packet Switching)
- Non vi è alcuna prenotazione preventiva o prefissata del tempo o della frequenza.
- Le risorse sono allocate **on-demand** (su richiesta dinamica). I flussi applicativi vengono segmentati in **pacchetti**.
- I pacchetti provenienti da sorgenti diverse vengono inviati sul link di uscita nell'ordine in cui arrivano (*First-Come, First-Served* o secondo politiche di accodamento).
- *Vantaggio:* Massimizza l'efficienza di allocazione quando il traffico è di tipo **bursty** (caratterizzato da picchi improvvisi e lunghi periodi di inattività).

---

## 4. Paradigmi di Commutazione: Circuito vs Pacchetto

La scelta del paradigma di commutazione definisce l'architettura fondamentale del nucleo della rete.

```
                  RETE COMMUTATA
                        |
        +---------------+---------------+
        |                               |
  Commutazione di                Commutazione di
     Circuito                       Pacchetto
   (FDM / TDM)              (Multiplazione Statistica)
                                        |
                         +--------------+--------------+
                         |                             |
                 Circuiti Virtuali                Datagramma
                (Connection-Oriented)           (Connectionless - IP)
```

---

### 4.1 Reti a Commutazione di Circuito (Circuit Switching)

Utilizzata storicamente nelle **reti telefoniche tradizionali (PSTN)**.

#### Fasi operative
1. **Instaurazione del circuito (Connection Setup):** Viene ricercato e riservato in modo end-to-end un percorso fisico/logico (canale FDM o slot TDM) tra mittente e destinatario.
2. **Trasferimento dati (Data Transfer):** I bit fluiscono a tasso costante lungo la risorsa riservata.
3. **Abbattimento del circuito (Teardown):** Rilascio formale delle risorse allocate lungo i nodi intermedi.

#### Caratteristiche Fondamentali
- **Risorse Riservate (Bandwidth Guarantee):** Banda e risorse nei nodi sono dedicate in via esclusiva.
- **Assenza di Overhead di Intestazione nei Nodi:** Non serve inviare l'indirizzo di destinazione in ciascun bit di dati, poiché il cammino fisico è predefinito.
- **Assenza di Accodamento ed Elaborazione nei Nodi:** I nodi non memorizzano i dati; non vi sono ritardi di accodamento imprevedibili.
- **Gestione delle Contese mediante Bloccaggio:** Se lungo il percorso tutte le risorse/canali sono occupate, la richiesta di connessione viene **rifiutata** (segnale di occupato).

---

### 4.2 Reti a Commutazione di Pacchetto (Packet Switching)

Utilizzata nelle **reti di calcolatori e in Internet**.

#### Caratteristiche Fondamentali
- **Store-and-Forward (Salva e Inoltra):** Il nodo intermedio (router/switch) deve **ricevere completamente l'intero pacchetto** nel proprio buffer prima di poter iniziare a trasmetterne il primo bit sul link di uscita.
- **Overhead dei Pacchetti:** Ogni pacchetto porta con sé una testata (*header*) contenente informazioni di controllo (indirizzo sorgente, destinazione, checksum, numero di sequenza).
- **Gestione delle Contese mediante Accodamento:** Se la velocità di arrivo dei pacchetti al router supera la velocità del link di uscita $R$, i pacchetti si accumulano in una coda di memoria buffer (*queuing*).
- **Perdita di Pacchetti (Packet Loss):** Se il buffer del router si riempie completamente (overflow), i pacchetti in arrivo vengono scartati (*dropped*).

---

### 4.3 Confronto Dettagliato: Datagramma vs Circuito Virtuale

All'interno della commutazione di pacchetto si distinguono due approcci architetturali:

```
[Approccio Datagramma (IP)]
Host A --> Router 1 --> Router 2 --> Router 3 --> Host B
  Pkt 1: via Router 1 -> Router 2 -> Router 3
  Pkt 2: via Router 1 -> Router 4 -> Router 3 (Percorsi autonomi)

[Approccio Circuito Virtuale (ATM/MPLS)]
Host A =[VC:5]=> Router 1 =[VC:12]=> Router 3 =[VC:3]=> Host B
  Tutti i pacchetti seguono la stessa rotta prefissata nel setup!
```

#### A. Modalità Datagramma (Connectionless - es. Protocollo IP in Internet)
- **Assenza di Stato nei Router:** I router non mantengono alcuna informazione sullo stato della connessione tra gli host periferici (*stateless core*).
- **Routing Indipendente:** Ogni pacchetto viene trattato come un'entità autonoma (*datagramma*). Pacchetti appartenenti allo stesso messaggio applicativo possono seguire percorsi geografici differenti a seconda delle tabelle di instradamento correnti.
- **Header Completo:** Ogni pacchetto deve contenere l'indirizzo IP di destinazione completo ($32\text{ bit}$ in IPv4, $128\text{ bit}$ in IPv6).
- **Robustezza e Resilienza:** Se un router o un link guasta durante una sessione, le tabelle di routing si aggiornano dinamiche e i pacchetti successivi aggirano il guasto senza far cadere la comunicazione.
- **Affidabilità delegata agli Host:** Non c'è garanzia di consegna in ordine o senza perdite a livello di rete. Il controllo errori/riordinamento è demandato agli host periferici (es. protocollo TCP al livello di trasporto).

#### B. Modalità Circuito Virtuale (Connection-Oriented - es. ATM, Frame Relay, MPLS)
- **Fase di Setup e Stato nei Router:** Prima di inviare i dati, si instaura una connessione logica (*Virtual Circuit*). I router lungo il percorso aggiornano tabelle interne di VC (*VC Table*).
- **VC Identifier (VCI):** Ogni pacchetto contiene non l'indirizzo finale completo, ma un identificatore di circuito virtuale locale (**VCI**) a breve raggio, riscritto a ogni salto (*label swapping*).
- **Garanzie di Qualità del Servizio (QoS):** Permette di negoziare banda, ritardo massimo e di riservare risorse al momento della connessione (o rifiutarla se satura).

---

### 4.4 Tabella Comparativa Sinottica d'Esame

| Parametro Architetturale | Commutazione di Circuito | Commutazione di Pacchetto (Datagramma) | Commutazione di Pacchetto (Circuito Virtuale) |
|---|---|---|---|
| **Prenotazione Risorse** | Esclusiva / Dedicata | Nessuna (Statistica) | Negoziazione al setup |
| **Stato nei Router/Nodi** | Stato di circuito fisico | Nessuno (*Stateless*) | Stato di circuito virtuale |
| **Overhead per Pacchetto** | Nullo (dopo setup) | Alto (Indirizzi IP compl.) | Ridotto (Identificatore VCI) |
| **Risoluzione Contesa** | Bloccaggio (Occupato) | Accodamento e Ritardo | Rifiuto connessione in setup |
| **Gestione Guasti** | Connessione interrotta | Dinamica ed autonoma | Riconfigurazione o caduta VC |
| **Idoneità Applicativa** | Traffico continuo (Voce) | Traffico a raffica (*Bursty*) | Traffico a garanzia di QoS |

> [!TIP] Perché la Commutazione di Pacchetto Domina in Internet
> Assumiamo un link da $1\text{ Mbps}$ e utenti che richiedono $100\text{ kbps}$ quando attivi, ma sono attivi solo per il $10\%$ del tempo.
> - Con **Commutazione di Circuito (TDM/FDM)**, il link supporta al massimo $\frac{1\text{ Mbps}}{100\text{ kbps}} = 10\text{ utenti}$ simultanei.
> - Con **Commutazione di Pacchetto**, se vi sono $35$ utenti, la probabilità che più di 10 siano attivi contemporaneamente è calcolabile tramite la distribuzione binomiale ed è inferiore allo $0.004$ ($0.4\%$). Pertanto, la commutazione di pacchetto consente di servire quasi 4 volte gli utenti nello stesso canale con un rischio trascurabile di congestione.

---

## 5. Gerarchia degli ISP e Struttura Globale di Internet

Internet è definita formale come una **"Rete di Reti"** organizzata in una gerarchia commerciale e logica di *Internet Service Provider* (ISP).

```
          +-----------------------------------+
          |            ISP TIER 1             | (Backbone Globale,
          |    AT&T, Level 3, NTT, Telia      |  Transit Free)
          +-----------------+-----------------+
                            |
           +----------------+----------------+
           | Point of Presence (PoP) / IXP   |
           +----------------+----------------+
                            |
          +-----------------+-----------------+
          |            ISP TIER 2             | (Dorsali Nazionali/
          |    Vodafone, Telecom, Fastweb     |  Regionali)
          +-----------------+-----------------+
                            |
          +-----------------+-----------------+
          |        ISP TIER 3 / LOCALI        | (Last Hop Access)
          +-----------------+-----------------+
                            |
          +-----------------+-----------------+
          |    SISTEMI TERMINALI (HOSTS)      |
          +-----------------------------------+
```

### 5.1 Livelli Gerarchici degli ISP
1. **Tier-1 ISP:** Costituiscono la spina dorsale (*backbone*) mondiale di Internet. Possiedono coperture geografiche continentali o intercontinentali (es. AT&T, Level 3, NTT, Sprint).
   - **Peering Paritetico:** I Tier-1 sono interconnessi tra loro in modo diretto e si scambiano il traffico a titolo gratuito (*Settlement-Free Peering*).
   - Non acquistano transito da nessun altro provider (*Transit-Free Networks*).
2. **Tier-2 ISP:** Provider su scala nazionale o regionale. Si collegano ad uno o più ISP Tier-1 acquistando l'accesso al resto della rete globale (*Customer-Provider Relationship*). Possono effettuare tra loro *peering* locale per evitare il costo del transito verso il Tier-1.
3. **Tier-3 / ISP di Accesso (Local ISP):** Reti "dell'ultimo salto" (*last hop*), più vicine agli utenti finali (residenziali e aziendali). Acquistano connettività dagli ISP di livello superiore.

### 5.2 Elementi di Interconnessione
- **PoP (Point of Presence):** Gruppo di uno o più router nel sito di un ISP cliente che si collegano alla rete dell'ISP fornitore.
- **IXP (Internet Exchange Point):** Infrastruttura fisica terza in cui molteplici ISP (Tier-2, Tier-3, CDN) si incontrano per scambiare traffico direttamente tramite accordi di *peering* senza pagare un ISP di rango superiore.
- **CDN (Content Delivery Network):** Reti private gestite da grandi fornitori di contenuti (es. Google, Netflix, Akamai) che installano *server cache* direttamente all'interno delle reti degli ISP locali per servire i dati a bassissima latenza, bypassando il nucleo della rete.
- **GARR (Gestione Ampliamento Rete Ricerca):** Esempio di rete accademica e di ricerca italiana (evidenziata nelle slide della docente) che interconnette Università e Centri di Ricerca italiani collegandosi al backbone europeo GÉANT.

---

## 6. Misure di Prestazione: Ritardi, Throughput e Strumenti Diagnostici

---

### 6.1 Analisi Quantitativa del Ritardo Nodal (Nodal Delay)

Il ritardo totale subito da un pacchetto nel transito attraverso un nodo di commutazione è dato dalla somma di quattro componenti disgiunte:

$$d_{\text{nodale}} = d_{\text{elab}} + d_{\text{accod}} + d_{\text{trasm}} + d_{\text{prop}}$$

```
[Pkt] -> [Elaborazione] -> [Buffer Coda] -> [Trasmissione] -> Mezzo
             d_elab            d_accod          d_trasm        d_prop
```

#### 1. Ritardo di Elaborazione ($d_{\text{elab}}$ - Processing Delay)
- **Definizione:** Tempo necessario al router per esaminare l'intestazione (*header*) del pacchetto, verificare la presenza di bit errati (tramite checksum/CRC) e determinare l'interfaccia di uscita interrogando la tabella di forwarding.
- **Dinamica:** Nell'ordine dei microsecondi ($\mu\text{s}$), implementato oggi via hardware dedicato (ASIC / TCAM).

#### 2. Ritardo di Accodamento ($d_{\text{accod}}$ - Queuing Delay)
- **Definizione:** Tempo trascorso dal pacchetto nella coda del buffer dell'interfaccia di uscita in attesa di essere trasmesso sul canale.
- **Dinamica:** Estremamente variabile e stocastico. Dipende dal livello di congestione della rete.

#### Intensità di Traffico ($I$)
Formalizzata matematicamente tramite il rapporto:

$$I = \frac{L \cdot a}{R}$$

Dove:
- $L$ = Dimensione media del pacchetto ($\text{bit}$)
- $a$ = Tasso medio di arrivo dei pacchetti ($\text{pacchetti/secondo}$)
- $R$ = Velocità trasmissiva del link di uscita ($\text{bit/secondo}$)

```
Ritardo Accodamento (d_accod)
      ^
      |                             / <-- I -> 1 (Asintoto)
      |                            /
      |                           /
      +---------------------------------> Intensità (I = L*a / R)
     0                 0.8       1.0
```

- **$I \to 0$:** Arrivi rari e scaglionati. Ritardo di accodamento quasi nullo ($d_{\text{accod}} \approx 0$).
- **$I \to 1$:** Arrivi vicini alla capacità massima. La coda cresce indefinitamente; il ritardo tende all'infinito ($d_{\text{accod}} \to \infty$).
- **$I > 1$:** Il lavoro in ingresso supera la capacità di smaltimento del nodo. Il buffer si satura e si verifica la **perdita di pacchetti (Packet Loss)**.

#### 3. Ritardo di Trasmissione ($d_{\text{trasm}}$ - Transmission Delay)
- **Definizione:** Tempo necessario per iniettare tutti i bit del pacchetto sul mezzo fisico. Dipende dalla lunghezza del pacchetto e dalla velocità del trasmettitore.
- **Formula:**
  $$d_{\text{trasm}} = \frac{L}{R}$$
  - $L$ = Lunghezza del pacchetto in bit ($\text{bit}$)
  - $R$ = Velocità del link in bit al secondo ($\text{bps}$)

#### 4. Ritardo di Propagazione ($d_{\text{prop}}$ - Propagation Delay)
- **Definizione:** Tempo impiegato dal singolo bit per viaggiare dall'inizio alla fine del mezzo fisico di collegamento. Dipende dalla distanza e dalla velocità della luce nel mezzo.
- **Formula:**
  $$d_{\text{prop}} = \frac{d}{s}$$
  - $d$ = Distanza fisica tra nodo trasmettitore e ricevitore ($\text{m}$)
  - $s$ = Velocità di propagazione del segnale nel mezzo ($\approx 2 \times 10^8 \text{ m/s}$ in fibra/rame, $\approx 3 \times 10^8 \text{ m/s}$ nell'aria/vuoto)

> [!IMPORTANT] Distinzione Frequente all'Esame: $d_{\text{trasm}}$ vs $d_{\text{prop}}$
> - **Ritardo di Trasmissione ($d_{\text{trasm}} = L/R$):** È una funzione della dimensione del pacchetto e della capacità del trasmettitore. Non dipende dalla distanza geografica!
> - **Ritardo di Propagazione ($d_{\text{prop}} = d/s$):** È una funzione della distanza geografica e della fisica del mezzo. Non dipende dalla dimensione del pacchetto né dalla banda del trasmettitore!

---

### 6.2 Ritardo End-to-End e Concetto di Store-and-Forward

Considerando un percorso formato da $N$ link identici in cascata (quindi $N-1$ router intermedi) trascurando elaborazione e accodamento, il ritardo totale end-to-end per l'invio di un singolo pacchetto è:

$$d_{\text{end-to-end}} = N \cdot \left( \frac{L}{R} + \frac{d}{s} \right)$$

Se trasmettiamo un messaggio composto da $P$ pacchetti distinti su $N$ hop:
$$d_{\text{totale}} = (N + P - 1) \cdot \frac{L}{R}$$

---

### 6.3 Round Trip Time (RTT) e Strumenti Diagnostici

- **Round Trip Time (RTT):** Il tempo necessario a un pacchetto di prova per viaggiare dal mittente al destinatario e ritornare al mittente.

#### Strumenti di Misura Diagnostici
1. **`ping`:**
   - Invia pacchetti di tipo **ICMP Echo Request** verso un host di destinazione e attende la risposta **ICMP Echo Reply**.
   - Calcola l'RTT complessivo andata/ritorno e la percentuale di pacchetti persi per messaggi di dimensione standard ($32\text{ B}$ o $64\text{ B}$).
   - *Se il ping fallisce (Richiesta scaduta / Request timed out):* L'host destinatario o un firewall intermedio rifiutano i pacchetti ICMP, oppure c'è assenza di instradamento.
2. **`traceroute` (o `tracert` su Windows):**
   - Mappa l'intero percorso dei router intermedi (*hops*) verso una destinazione, misurando l'RTT parziale per ciascun nodo.
   - **Meccanismo di Funzionamento:** Invia una serie di pacchetti dati (UDP o ICMP) impostando il campo **TTL** (*Time To Live*) dell'header IP a valori incrementali ($TTL = 1, 2, 3, \dots$).
     - Quando il pacchetto con $TTL = k$ raggiunge il $k$-esimo router, il nodo decrementa il TTL a 0, scarta il pacchetto ed invia al mittente un pacchetto di errore **ICMP Time Exceeded**.
     - Il mittente registra l'indirizzo IP del router e calcola il tempo trascorso (RTT per il salto $k$).

---

### 6.4 Rendimento (Throughput) e Larghezza di Banda (Bandwidth)

- **Larghezza di Banda (Bandwidth):** La capacità massima teorica di trasmissione di un canale ($R\text{ bps}$).
- **Rendimento (Throughput):** Il tasso effettivo al quale i bit vengono correttamente trasferiti tra sorgente e destinazione nell'unità di tempo.
  - *Throughput Istantaneo:* Velocità misurata in un preciso istante temporale.
  - *Throughput Medio:* Rapporto $F / T$ dove $F$ sono i bit trasferiti nel periodo $T$.

#### Il Principio del Collo di Bottiglia (Bottleneck Link)
In un percorso rettilineo con connessioni in serie aventi capacità trasmissive $R_1, R_2, \dots, R_n$, il throughput complessivo end-to-end è limitato dal link con la **capacità minima**:

$$\text{Throughput End-to-End} = \min(R_1, R_2, \dots, R_n)$$

```
[Server] ==(Rs = 100 Mbps)==> [Router] --(Rc = 1.5 Mbps)--> [Client]
                                       ^
                                COLLO DI BOTTIGLIA!
                         Throughput effettivo = 1.5 Mbps
```

---

## 7. Architetture a Livelli: Modello ISO/OSI e TCP/IP

La complessità dei sistemi di rete rende indispensabile la progettazione modulare basata su **architetture a livelli di protocollo (Protocol Stacks)**.

```
MODELLO ISO/OSI (7 Livelli)       MODELLO TCP/IP (5 Livelli)
+-------------------------+       +-------------------------+
|  7. Applicazione        |       |  5. Applicazione        | (HTTP, DNS, SMTP)
+-------------------------+       +-------------------------+
|  6. Presentazione       |       |  4. Trasporto           | (TCP, UDP)
+-------------------------+       +-------------------------+
|  5. Sessione            |       |  3. Rete                | (IP, ICMP)
+-------------------------+       +-------------------------+
|  4. Trasporto           |       |  2. Collegamento (Link) | (Ethernet, WiFi)
+-------------------------+       +-------------------------+
|  3. Rete                |       |  1. Fisico              | (Rame, Fibra)
+-------------------------+       +-------------------------+
|  2. Collegamento (Link) |
+-------------------------+
|  1. Fisico              |
+-------------------------+
```

### 7.1 Principi della Stratificazione (Layering)
- Ogni livello $N$:
  1. Offre **servizi** al livello immediatamente superiore $(N+1)$.
  2. Si avvale dei **servizi** forniti dal livello inferiore $(N-1)$.
  3. Comunica con la propria entità pari-grado (*Peer Entity*) sul nodo remoto implementando un **protocollo del livello $N$**.
- **Vantaggi:**
  - *Modularità e Astrazione:* Cambiare l'implementazione fisica di un livello (es. passare da WiFi a Ethernet) non richiede modifiche ai livelli superiori (es. le applicazioni Web o il protocollo TCP restano inalterati).
  - *Manutenibilità:* Semplifica il testing e lo sviluppo di standard aperti.

---

### 7.2 Confronto Dettagliato ISO/OSI vs TCP/IP

#### I 7 Livelli del Modello ISO/OSI
1. **Fisico:** Trasmissione di bit grezzi (*raw bits*) sul canale di comunicazione hardware.
2. **Collegamento Dati (Data Link):** Trasferimento affidabile di **frame** tra nodi adiacenti (stessa rete locale); gestione dell'indirizzamento fisico (MAC) e dell'accesso al mezzo.
3. **Rete (Network):** Instradamento (**routing**) dei **datagrammi** sorgente-destinazione attraverso reti multiple.
4. **Trasporto:** Comunicazione logica processo-a-processo (**end-to-end**), gestione di multiplexing/demultiplexing via porte, controllo di flusso e congestione.
5. **Sessione:** Gestione del dialogo, sincronizzazione, controllo del checkpoint della sessione tra applicazioni.
6. **Presentazione:** Traduzione dei formati dati, sintassi, cifratura e compressione.
7. **Applicazione:** Interfaccia diretta per l'utente e per i programmi di rete.

#### L'Architettura Reale Internet TCP/IP (5 Livelli)
Internet adotta l'architettura pragmatica a 5 livelli. Le funzioni di **Presentazione** e **Sessione** del modello OSI non possiedono livelli dedicati autonomi in TCP/IP: se necessarie, vengono implementate direttamente **all'interno dell'applicazione stessa** (es. TLS/SSL per la cifratura in HTTP $\to$ HTTPS).

---

### 7.3 Il Processo di Incapsulamento e Decapsulamento

Man mano che un messaggio applicativo scende lungo la pila trasmettitrice, ciascun livello aggiunge la propria intestazione di controllo (**Header**), creando una nuova **PDU** (*Protocol Data Unit*).

```
[FLUSSO DI INCAPSULAMENTO]

(Applicazione) --> Messaggio M
                        |
                        v
(Trasporto)    --> [ H_tcp | M ]              (Segmento)
                        |
                        v
(Rete)         --> [ H_ip | H_tcp | M ]       (Datagramma)
                        |
                        v
(Link)         --> [ H_eth | H_ip | H_tcp | M | T_eth ] (Frame)
                        |
                        v
(Fisico)       --> Bit (1010101101...) sul canale
```

#### Nomenclatura delle PDU per Livello

| Livello Protocollo | PDU (*Protocol Data Unit*) | Intestazioni tipiche contenute |
|---|---|---|
| **5. Applicazione** | **Messaggio (Message)** | Dati applicativi utente (es. `GET /index.html`) |
| **4. Trasporto** | **Segmento (Segment)** / Datagramma UDP | Porte Sorgente e Destinazione, Sequence Number, Checksum |
| **3. Rete** | **Datagramma (Datagram)** | Indirizzo IP Sorgente, Indirizzo IP Destinazione, TTL |
| **2. Collegamento** | **Frame (Trama)** | Indirizzi MAC Sorgente e Destinazione, CRC trailer |
| **1. Fisico** | **Bit** | Segnali elettrici, ottici o frequenze radio |

---

## 8. Domande di Verifica ed Esercizi di Consolidamento d'Esame

### Domanda 1: Qual è la differenza sostanziale tra ritardo di trasmissione e ritardo di propagazione?
**Risposta Didattica:** 
Il **ritardo di trasmissione** ($d_{\text{trasm}} = L/R$) è il tempo impiegato dal trasmettitore per spingere tutti i bit che compongono il pacchetto sul canale trasmissivo. Dipende esclusivamente dalla lunghezza del pacchetto $L$ e dalla velocità di cifra $R$ dell'interfaccia.
Il **ritardo di propagazione** ($d_{\text{prop}} = d/s$) è il tempo impiegato dal primo bit spinto sul canale per viaggiare attraverso il mezzo fisico dal punto A al punto B. Dipende dalla distanza geografica $d$ e dalla velocità del segnale nel mezzo $s$.

### Domanda 2: Per quale motivo la commutazione di pacchetto è più efficiente della commutazione di circuito per il traffico Web?
**Risposta Didattica:** 
Il traffico di rete generato dalle applicazioni Web è di tipo *bursty* (invia richieste repentine seguite da lunghi intervalli di inattività o lettura dell'utente). Nella commutazione di circuito, le risorse (frequenza o slot temporale) rimangono riservate ed inutilizzate durante i periodi di silenzio. La commutazione di pacchetto sfrutta la **multiplazione statistica**: condivide dinamicamente le risorse trasmissive su richiesta, consentendo a un numero elevato di utenti di condividere lo stesso collegamento con probabilità di congestione minima.

### Domanda 3: Perché Internet adotta un modello "Connectionless" a livello di Rete (IP) delegando il controllo errore al livello di Trasporto?
**Risposta Didattica:** 
Questo riflette il principio di progettazione noto come **End-to-End Principle**: mantenere il nucleo della rete (*core routers*) il più semplice, veloce e scalabile possibile (*stateless core*). Spostando la gestione dello stato della connessione, il controllo dell'affidabilità e il riordinamento dei pacchetti agli host periferici (tramite il protocollo TCP al livello di trasporto), i router intermedi devono soltanto effettuare l'inoltro rapido dei datagrammi IP, garantendo un'elevatissima scalabilità e resilienza ai guasti dei singoli nodi.

---

## Riferimenti Incrociati

- [[Capitolo 1 - Reti di Calcolatori e Internet]]
- [[Capitolo 2 - Livello di Applicazione]]
- [[Capitolo 3 - Livello di Trasporto]]
- [[Capitolo 4 - Livello di Rete (Piano dei Dati)]]
- [[Capitolo 6 - Livello di Collegamento e LAN]]
