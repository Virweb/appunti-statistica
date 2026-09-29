---
tags: [database, capitolo-4, ER, progettazione-concettuale, atzeni]
capitolo: 4
titolo: Progettazione Concettuale — Modello E-R
libro: "Basi di Dati - Atzeni, Ceri, Paraboschi, Torlone"
data_creazione: 2026-09-29
---

# Capitolo 4 — Progettazione Concettuale e Modello E-R

> [!NOTE] Dall'idea alla struttura
> La progettazione concettuale trasforma i requisiti informali degli utenti in uno schema E-R formale, **indipendente dal DBMS** scelto. È la fase più creativa e critica del processo.

---

## Il Processo di Progettazione di un DB

```
Requisiti utente
      │
      ▼
┌─────────────────────┐
│ Progettazione       │  → Schema E-R (concettuale)
│ Concettuale         │     (modello E-R, UML)
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Progettazione       │  → Schema relazionale (logico)
│ Logica              │     (tabelle, FK, vincoli)
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Progettazione       │  → Schema fisico
│ Fisica              │     (indici, tablespace, partizioni)
└─────────────────────┘
```

---

## Il Modello Entity-Relationship (E-R)

Proposto da **Peter Chen** nel 1976. Descrive la realtà tramite:
- **Entità** — oggetti del mondo reale
- **Associazioni (Relationship)** — legami tra entità
- **Attributi** — proprietà di entità e associazioni

---

## Costrutti del Modello E-R

### Entità

Un'**entità** rappresenta una classe di oggetti con proprietà comuni e autonoma esistenza nel dominio applicativo.

```
┌─────────────┐
│  STUDENTE   │  ← Entità (rettangolo)
└─────────────┘
```

**Istanza di entità:** un singolo oggetto appartenente all'entità (es: lo studente "Mario Rossi").

### Attributi

Proprietà elementari di un'entità o associazione.

**Tipi di attributi:**

| Tipo | Descrizione | Esempio |
|---|---|---|
| **Semplice** | Valore atomico | Nome, Età |
| **Composto** | Struttura con sotto-attributi | Indirizzo (Via, Città, CAP) |
| **Multivalore** | Più valori per una stessa istanza | Telefoni {tel1, tel2} |
| **Derivato** | Calcolato da altri attributi | Età (da DataNascita) |

**Identificatore (chiave concettuale):** attributo (o insieme di attributi) che identifica univocamente ogni istanza dell'entità.

```
STUDENTE
├── Matricola (identificatore — sottolineato nello schema)
├── Nome
├── Cognome
└── DataNascita
```

### Associazioni (Relationship)

Legame logico tra due o più entità.

```
[STUDENTE] ─────────── <SOSTIENE> ─────────── [ESAME]
```

**Attributi di associazione:** proprietà che appartengono all'associazione, non alle entità.
```
<SOSTIENE> ha attributi: DataSostenimento, Voto
```

**Associazioni n-arie:** coinvolgono più di due entità (meno comuni, spesso sostituibili con entità + associazioni binarie).

---

## Cardinalità delle Associazioni

La **cardinalità** specifica quante istanze di un'entità possono partecipare all'associazione per ogni istanza dell'altra entità.

### Notazione (min, max)

```
(min-card, max-card) dove max può essere N (illimitato)
```

| Tipo | Cardinalità | Significato |
|---|---|---|
| **Uno a uno (1:1)** | (1,1) — (1,1) | Ogni A associato a 1 B, ogni B associato a 1 A |
| **Uno a molti (1:N)** | (1,1) — (0,N) | 1 A associato a molti B, ogni B a 1 A |
| **Molti a molti (N:M)** | (0,N) — (0,N) | Molti A a molti B |

### Esempi Pratici

**1:N — Docente tiene Corso:**
```
[DOCENTE] ─(1,1)──<TIENE>──(0,N)─ [CORSO]
```
*Un docente tiene uno o più corsi; ogni corso ha esattamente un docente responsabile.*

**N:M — Studente sostiene Esame:**
```
[STUDENTE] ─(0,N)──<SOSTIENE>──(0,N)─ [ESAME]
```
*Uno studente sostiene più esami; un esame è sostenuto da più studenti.*

**1:1 — Direttore dirige Dipartimento:**
```
[IMPIEGATO] ─(0,1)──<DIRIGE>──(1,1)─ [DIPARTIMENTO]
```
*Ogni dipartimento ha esattamente 1 direttore; un impiegato può dirigere al più 1 dipartimento.*

---

## Generalizzazioni (Gerarchie IS-A)

La **generalizzazione** esprime una relazione di specializzazione/ereditarietà tra entità.

```
         ┌─────────────┐
         │   PERSONA   │  ← Entità genitore
         └──────┬──────┘
                │  IS-A
       ┌────────┴────────┐
       │                 │
┌──────┴──────┐   ┌──────┴──────┐
│  STUDENTE   │   │   DOCENTE   │  ← Entità figlie
└─────────────┘   └─────────────┘
```

**Proprietà ereditate:** le entità figlie ereditano tutti gli attributi e le associazioni del genitore.

### Tipi di Generalizzazione

| Dimensione | Opzioni | Significato |
|---|---|---|
| **Copertura** | Totale / Parziale | Ogni istanza del genitore è anche figlia (totale) o no |
| **Sovrapposizione** | Esclusiva / Sovrapposta | Un'istanza può appartenere a più figli (sovrapposta) o a uno solo (esclusiva) |

Combinazioni:
- **Totale-Esclusiva:** ogni persona è o studente o docente, non entrambi e non né l'uno né l'altro
- **Parziale-Sovrapposta:** ci sono persone che non sono né studenti né docenti, e alcuni possono essere entrambi

---

## Entità Deboli

Un'**entità debole** non ha un identificatore proprio — dipende da un'entità forte.

```
[ORDINE] ─(1,1)──<CONTIENE>──(1,N)─ [RIGA ORDINE]
                                       (entità debole: identificata da NumRiga + CodOrdine)
```

L'entità debole usa come identificatore: (identificatore entità forte + attributo locale).

---

## Documentazione degli Schemi E-R

Un buon schema E-R deve essere accompagnato da:

**Dizionario dei dati:**
- **Entità:** nome, descrizione, attributi, identificatore, entità correlate
- **Associazioni:** nome, entità coinvolte, cardinalità, attributi, descrizione

**Regole aziendali (business rules):**
Vincoli non esprimibili graficamente nel modello E-R:
- "Un impiegato non può lavorare più di 40 ore settimanali"
- "Il voto di un esame deve essere compreso tra 18 e 30 o 'lode'"

---

## Qualità di uno Schema Concettuale

Uno schema E-R di qualità deve essere:

| Proprietà | Descrizione |
|---|---|
| **Correttezza** | Usa correttamente i costrutti del modello |
| **Completezza** | Rappresenta tutti i requisiti |
| **Leggibilità** | Schema comprensibile ai non-tecnici |
| **Minimalità** | Nessuna ridondanza inutile |

**Errori comuni:**
- Usare attributi dove servirebbe un'entità (se l'attributo ha struttura propria)
- Usare un'entità dove basterebbe un attributo
- Associazioni ridondanti (derivabili da composizione di altre)

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **Entità** = classe di oggetti; **Istanza** = singolo oggetto
> - **Associazione** = legame tra entità (può avere attributi propri)
> - **Cardinalità (min, max)** per ogni "verso" dell'associazione
> - **1:1, 1:N, N:M** — le tre cardinalità fondamentali
> - **Generalizzazione** = IS-A con ereditarietà (totale/parziale, esclusiva/sovrapposta)
> - **Entità debole** = identificata solo in relazione all'entità forte

## Link Interni

- [[Capitolo 3 - Algebra Relazionale e SQL]]
- [[Capitolo 5 - Progettazione Logica e Normalizzazione]]
