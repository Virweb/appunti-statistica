---
materia: Reti di Calcolatori
tipo: Appunti di studio approfonditi
lezione: "2 - Storia di Internet"
riferimenti_slide: "Lezione 2 - Storia di Internet.pdf"
capitoli_libro: ["Capitolo 1 - Reti di Calcolatori e Internet (1.7)"]
tags: [reti, storia-internet, arpanet, tcp-ip, web, rfc, obsidian]
---

# 02 — Storia di Internet e del World Wide Web

> [!NOTE] Nota Metodologica e Didattica
> La presente nota di studio sintetizza ed approfondisce i contenuti della **Lezione 2 (Dott.ssa Teresa Mallardo)** integrando il contesto teorico e storico trattato in **Kurose & Ross (8ª ed., Sezione 1.7)**. La trattazione ricostruisce la genesi geopolitica delle reti, la nascita della commutazione di pacchetto, l'evoluzione di ARPANET, l'invenzione dell'architettura TCP/IP, l'introduzione dei protocolli applicativi fondamentali (DNS, SMTP, FTP) e la rivoluzione ipermediale del World Wide Web.

---

## 1. Il Contesto Storico e Geopolitico (1957–1966)

La nascita delle reti di calcolatori non è un evento tecnologico isolato, ma la diretta conseguenza della competizione scientifica e militare durante la **Guerra Fredda** tra Stati Uniti e Unione Sovietica.

```
       [4 Ottobre 1957: Lancio Sputnik 1 (URSS)]
                          |
                          v
         [1958: Nascita ARPA negli USA]
         (Advanced Research Projects Agency)
                          |
                          v
       [12 Aprile 1961: Primo uomo nello spazio]
       (Yuri Gagarin, URSS) ---> Nascita NASA
                          |
                          v
    [1966: Progetto Taylor per la condivisione]
    (Interconnessione computer universitari ARPA)
```

### 1.1 L'Evento Scatenante e la Nascita dell'ARPA
- **4 Ottobre 1957:** L'Unione Sovietica lancia lo **Sputnik 1**, il primo satellite artificiale della storia. L'evento genera una crisi tecnologica negli Stati Uniti (*Sputnik Crisis*).
- **1958:** Il Presidente Eisenhower e i consiglieri Killian e McElroy fondano l'**ARPA** (*Advanced Research Projects Agency*, dipendente dal Dipartimento della Difesa USA) con l'obiettivo di garantire la supremazia scientifica e tecnologica americana.
- **12 Aprile 1961:** L'URSS invia il primo uomo nello spazio (Yuri Gagarin). Gli USA rispondono con l'istituzione della NASA e l'avvio del Programma Apollo.
- **La Questione Informatica in ARPA (1966):** ARPA finanziava centri di ricerca informatica in diverse università americane, ma i mainframes sparsi sul territorio erano isolati ed incompatibili tra loro. **Bob Taylor** propone un progetto per consentire la condivisione diretta di risorse hardware ed algoritmi tra i calcolatori dei vari laboratori.

---

## 2. I Pionieri della Commutazione di Pacchetto (1959–1964)

Prima della nascita di ARPANET, la comunicazione a lunga distanza si basava esclusivamente sulla commutazione di circuito (rete telefonica analogica). Tre ricercatori indipendenti svilupparono i fondamenti concettuali della **commutazione di pacchetto**.

```
+---------------------------------------------------+
|               I PIONIERI DEL PACKET               |
+---------------------------------------------------+
| 1. PAUL BARAN (RAND Corp, 1959)                   |
|    Reti distribuite vulnerabilità-zero.           |
+---------------------------------------------------+
| 2. DONALD DAVIES (NPL Londra, 1959)               |
|    Connessione terminali e conio del termine      |
|    "Packet" (Pacchetto).                          |
+---------------------------------------------------+
| 3. LEONARD KLEINROCK (MIT, 1961)                  |
|    Teoria matematica delle code per reti a        |
|    commutazione di pacchetto.                     |
+---------------------------------------------------+
```

### 2.1 Le Tre Linee di Ricerca Fondative

1. **Paul Baran (RAND Corporation, 1959–1964):**
   - Incaricato dall'aeronautica militare americana di progettare un sistema di comunicazione vocale in grado di **sopravvivere ad un attacco nucleare**.
   - Dimostra che le reti centralizzate e gerarchiche sono vulnerabili alla distruzione dei nodi centrali. Propone una **rete magliata distribuita** basata sul frazionamento del messaggio in blocchi digitali scritti ed inoltrati in modo autonomo (*Store-and-Forward*).

2. **Donald Davies (National Physical Laboratory - NPL, UK, 1959–1965):**
   - Sviluppa in Gran Bretagna un concetto analogo per la connessione di terminali utente a computer centrali.
   - Conia ufficialmente il termine **"Packet"** (Pacchetto) per descrivere i piccoli blocchi di dati inviati sulla rete.

3. **Leonard Kleinrock (MIT / UCLA, 1961–1964):**
   - Pubblica la prima tesi di dottorato e la prima opera teorica formale sulle reti a commutazione di pacchetto (*"Information Flow in Large Communication Nets"*).
   - Sviluppa i **modelli matematici della teoria delle code** per analizzare quantitativamente i ritardi di accodamento ed il rendimento dei canali dati.

---

## 3. L'Architettura di ARPANET e le prime RFC (1969)

```
[PRIMA RETE ARPANET - DICEMBRE 1969]

   (UCLA) <=================> (SRI)
     ||                        ||
     ||                        ||
   (UCSB) <=================> (UTAH)

  Nodi collegati tramite IMP (BBN Hardware)
```

### 3.1 L'IMP (Interface Message Processor)
- Ideato da **Lawrence Roberts** e **J.C.R. Licklider** (MIT) e costruito dalla società **Bolt Beranek & Newman (BBN)**.
- L'**IMP** è storicamente il **primo commutatore di pacchetto (antenato del router moderno)**. Veniva interposto tra il mainframe universitario (Host) e le linee telefoniche dedicate a $50\text{ kbps}$.

### 3.2 Le RFC e il Primo Protocollo NCP
- **Request For Comments (RFC):** ARPANET adottò una politica di documentazione tecnica aperta e democratica. Chiunque poteva proporre specifiche o miglioramenti.
- **7 Aprile 1969:** **Steve Crocker** (allievo di Kleinrock all'UCLA) scrive la **RFC 001** (*"Host Software"*). Da quel momento le RFC diventeranno gli standard ufficiali gestiti dall'IETF.
- **NCP (Network Control Protocol):** Il primo protocollo tra nodi host-to-host implementato per gestire la comunicazione su ARPANET.

### 3.3 Il Primo Evento di Connessione Reale
- **2 Settembre 1969:** Installato il primo nodo IMP all'UCLA sotto la direzione di Kleinrock.
- **29 Ottobre 1969 (ore 22:30):** Primo tentativo di connessione remota tra l'UCLA (Charley Kline) e lo Stanford Research Institute (SRI).
  - L'operatore iniziò a digitare la parola **"LOGIN"**.
  - Digitò **"L"** (ricevuta e confermata per telefono dallo SRI).
  - Digitò **"O"** (ricevuta).
  - Alla digitazione della **"G"**, il sistema andò in crash.
  - Il primo messaggio trasmesso nella storia di Internet fu dunque **"LO"**.
- **Dicembre 1969:** Costituita la prima rete a 4 nodi: **UCLA**, **SRI** (Stanford), **UCSB** (Santa Barbara) e **University of Utah**.

---

## 4. La Nascita di Internet: Il Protocollo TCP/IP (1973–1983)

Agli inizi degli anni '70 emersero reti a commutazione di pacchetto eterogenee su mezzi trasmissivi diversi (es. **ALOHAnet** alle Hawaii su onde radio, reti satellitari). ARPANET era una rete chiusa vincolata agli IMP: non era in grado di comunicare con tali reti esterne.

```
       [ARPANET]          [ALOHAnet]          [SATNET]
       (Cavo 50k)         (Radio)             (Satellite)
           \                 |                 /
            \                v                /
          +-------------------------------------+
          |   PROTOCOLLO TCP/IP (Cerf & Kahn)   |
          | Interconnessione Reti Eterogenee   |
          +-------------------------------------+
```

### 4.1 L'Invenzione di Cerf e Kahn
- **Vinton Cerf** (UCLA) e **Robert Kahn** (BBN/ARPA) progettano un'architettura universale di interconnessione (*Internetworking*).
- **1973:** Pubblicano lo storico articolo *"A Protocol for Packet Network Intercommunication"*, definendo il **TCP** (*Transmission Control Protocol*).
- **1978:** Il protocollo monolitico TCP viene scomposto in due livelli gerarchici distinti:
  1. **TCP (Livello 4 - Trasporto):** Gestisce l'affidabilità end-to-end, il riordinamento e il controllo di flusso.
  2. **IP (Livello 3 - Rete):** Gestisce l'indirizzamento e l'inoltro dei datagrammi senza connessione (*connectionless routing*).
- **1 Gennaio 1983 (Flag Day):** ARPANET dismette definitivamente il vecchio protocollo NCP ed adotta **TCP/IP** come standard obbligatorio. Questo giorno segna ufficialmente la **nascita di Internet**.

---

## 5. L'Esplosione dei Protocolli Applicativi e delle Dorsali (1980–1990)

```
=====================================================
CRONOLOGIA DEI PROTOCOLLI APPLICATIVI FONDAMENTALI
=====================================================
  1982 | SMTP (RFC 821 - Jon Postel)
       | Posta Elettronica
  -----+---------------------------------------------
  1983 | DNS (Domain Name System - Postel, Mockapetris)
       | Traduzione Nomi -> Indirizzi IP
  -----+---------------------------------------------
  1985 | FTP (RFC 959 - Postel & Reynolds)
       | Trasferimento File tra Host
=====================================================
```

### 5.1 Protocolli Applicativi di Prima Generazione
- **SMTP (Simple Mail Transfer Protocol - 1982):** Definito da **Jon Postel** per standardizzare l'invio e lo scambio di e-mail tra host remoti.
- **DNS (Domain Name System - 1983):** Ideato da Postel, Craig Partridge e **Paul Mockapetris**. Sostituisce il file monolitico `HOSTS.TXT` con un sistema gerarchico e distribuito per associare nomi leggibili (es. `www.uniba.it`) ad indirizzi IP numerici.
- **FTP (File Transfer Protocol - 1985):** Definito da Postel e Reynolds per consentire il download/upload di file tra calcolatori remoti.

### 5.2 Scissione di ARPANET e Nascita di NSFnet
- **1983:** ARPANET si scinde in due reti separate:
  - **MILNET:** Rete militare chiusa ed ad alta sicurezza.
  - **ARPANET:** Rete civile dedicata esclusivamente alla ricerca accademica.
- **NSFnet (1986):** La **National Science Foundation** (NSF) crea una dorsale scientifica ad alta velocità ($56\text{ kbps}$, poi potenziata a **T1 - 1.544 Mbps** nel 1989 e **T3 - 45 Mbps** nel 1991).
- **1990:** ARPANET viene ufficialmente dismessa. NSFnet diventa il backbone portante di quella che ormai viene chiamata **Internet**.

---

## 6. La Rivoluzione del World Wide Web (1989–1995)

```
                 +-----------------------------------+
                 |    1989: TIM BERNERS-LEE (CERN)   |
                 |   Invenzione del World Wide Web   |
                 +-----------------+-----------------+
                                   |
                                   v
                 +-----------------------------------+
                 |     I TRE PILASTRI DEL WEB        |
                 |  1. HTTP (Protocollo di Rete)     |
                 |  2. HTML (Linguaggio Ipertesto)   |
                 |  3. URI/URL (Indirizzamento)      |
                 +-----------------+-----------------+
                                   |
                                   v
                 +-----------------------------------+
                 |  1993: MOSAIC (Andreessen & Bina) |
                 |    Primo Browser Grafico Popolare |
                 +-----------------------------------+
```

### 6.1 L'Invenzione di Tim Berners-Lee al CERN
- **1989:** Presso il **CERN** di Ginevra, lo scienziato britannico **Tim Berners-Lee** propone un sistema ipermediale distribuito per consentire ai fisici delle particelle di condividere ed aggiornare la documentazione scientifica.
- Prende spunto dal programma *Enquire* (creato da lui stesso) e dalle teorie concettuali di **Vannevar Bush** (*Memex*, 1945) e **Ted Nelson** (*Xanadu*, 1960).

### 6.2 I Tre Pilastri del Web
1. **HTTP (HyperText Transfer Protocol):** Protocollo di livello applicativo per la richiesta e risposta di risorse ipertestuali.
2. **HTML (HyperText Markup Language):** Linguaggio di formattazione per strutturare documenti contenenti collegamenti ipertestuali (*hyperlinks*).
3. **URI / URL (Uniform Resource Identifier / Locator):** Schema di indirizzamento globale univoco per identificare qualsiasi risorsa sulla rete.

### 6.3 I Primi Browser e la Commercializzazione
- **1993 - MOSAIC:** **Marc Andreessen** ed **Eric Bina** al NCSA (Università dell'Illinois) sviluppano **Mosaic**, il primo browser grafico intuitivo e gratuito. Mosaic trasforma Internet da strumento per accademici a fenomeno di massa.
- **1994:** Nasce **Netscape Navigator** e viene fondata **Yahoo!**.
- **1995:** Apertura ufficiale di Internet al commercio privato e nascono i primi motori di ricerca (Lycos, Altavista).
- **1996:** Microsoft rilascia la prima versione di **Internet Explorer**, avviando la "guerra dei browser".

---

## 7. Quadro Sinottico Cronologico d'Esame

| Anno | Evento Chiave | Protagonisti / Entità | Impatto Architetturale |
|---|---|---|---|
| **1957** | Lancio Sputnik 1 | URSS | Avvio della gara tecnologica; nascita ARPA (1958) |
| **1961** | Teoria delle Code | Leonard Kleinrock | Modellazione matematica della commutazione di pacchetto |
| **1969** | Primo nodo ARPANET & RFC 001 | Roberts, Crocker, BBN | Nascita del primo commutatore (IMP) e delle RFC |
| **1973** | Progetto TCP | Vinton Cerf, Robert Kahn | Architettura per interconnettere reti eterogenee |
| **1983** | Flag Day (NCP $\to$ TCP/IP) | ARPANET / IETF | Nascita ufficiale di Internet; introduzione del DNS |
| **1986** | Backbone NSFnet | National Science Found. | Dorsale ad alta velocità; transizione verso la rete civile |
| **1989** | Invenzione del WWW | Tim Berners-Lee (CERN) | Introduzione di HTTP, HTML e URL |
| **1993** | Browser Mosaic | Marc Andreessen (NCSA) | Diffusione di massa del Web grafico |

---

## 8. Domande di Verifica ed Esercizi di Consolidamento d'Esame

### Domanda 1: Qual è stata la motivazione tecnica per cui il protocollo TCP originario è stato sbloccato e scomposato in TCP ed IP nel 1978?
**Risposta Didattica:** 
Il protocollo TCP originario (1973) gestiva in modo monolitico sia l'indirizzamento/routing che il controllo di affidabilità e flusso. Con l'emergere di applicazioni in tempo reale o su reti wireless/satellitari (come la voce su pacchetto o ALOHAnet), ci si rese conto che non tutte le comunicazioni necessitavano del controllo d'errore e del riordinamento rigido del TCP. Scomponendo il protocollo, il livello **IP** si occupò del semplice inoltro *connectionless* (garantendo massima efficienza e flessibilità), lasciando al livello **TCP** (e successivamente a **UDP**) la gestione del trasporto end-to-end.

### Domanda 2: Per quale motivo l'invenzione dell'IMP (Interface Message Processor) è considerata la pietra miliare delle reti moderne?
**Risposta Didattica:** 
L'IMP ha introdotto il concetto di **separazione tra la logica di calcolo (Host) e la logica di comunicazione (Network)**. Prima dell'IMP, si tentava di far comunicare direttamente i vari computer centrali. L'IMP ha agito come un dispositivo dedicato di commutazione autonomo (l'antenato del router), sollevando i sistemi terminali dal carico dell'instradamento e definendo l'architettura a commutazione di pacchetto *Store-and-Forward*.

### Domanda 3: Qual è la differenza concettuale tra Internet e World Wide Web?
**Risposta Didattica:** 
- **Internet** è l’infrastruttura globale di rete fisica e logica (hardware, router, cavi, protocolli di livello rete IP e trasporto TCP/UDP) che collega tra loro miliardi di calcolatori.
- Il **World Wide Web (WWW)** è soltanto **una delle tante applicazioni** che funzionano al di sopra dell'infrastruttura di Internet (livello 7). Il Web è un sistema ipermediale distribuito basato sul protocollo HTTP che consente la fruizione di documenti tramite browser. Altre applicazioni su Internet includono la posta elettronica (SMTP), il trasferimento file (FTP) o la messaggistica istantanea.

---

## Riferimenti Incrociati

- [[00 - Introduzione alle Reti di Calcolatori]]
- [[01 - Architettura a Livelli e Protocolli]]
- [[Capitolo 1 - Reti di Calcolatori e Internet]]
