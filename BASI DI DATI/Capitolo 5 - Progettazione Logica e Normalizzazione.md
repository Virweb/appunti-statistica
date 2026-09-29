---
tags: [database, capitolo-5, progettazione-logica, normalizzazione, atzeni]
capitolo: 5
titolo: Progettazione Logica e Normalizzazione
libro: "Basi di Dati - Atzeni, Ceri, Paraboschi, Torlone"
data_creazione: 2026-09-29
---

# Capitolo 5 — Progettazione Logica e Normalizzazione

> [!NOTE] Dal modello E-R alle tabelle
> La progettazione logica traduce lo schema E-R in uno schema relazionale. La normalizzazione analizza e corregge i difetti dello schema risultante, eliminando ridondanze e anomalie.

---

## Parte A — Progettazione Logica

## Ristrutturazione dello Schema E-R

Prima della traduzione in relazionale, lo schema E-R viene **ristrutturato** per facilitare la traduzione e migliorare le prestazioni.

### 1. Analisi delle Ridondanze

**Ridondanza** = dato derivabile da altri dati presenti nello schema.

Pro della ridondanza: velocizza alcune query (evita join costosi)  
Contro: appesantisce gli aggiornamenti, occupa più spazio, rischio di inconsistenza

Decisione: mantenere o eliminare dipende dal profilo di carico (accessi vs aggiornamenti).

### 2. Eliminazione delle Generalizzazioni

Il modello relazionale non ha un costrutto diretto per la generalizzazione IS-A. Tre strategie:

**Opzione A — Accorpamento delle figlie nel genitore:**
```sql
PERSONA(CodFisc, Nome, Tipo, Matricola, Stipendio)
-- Tipo ∈ {'studente', 'docente'}
-- Matricola e Stipendio possono essere NULL (dipende dal Tipo)
```
✓ Semplice | ✗ Molti NULL se entità molto diverse

**Opzione B — Accorpamento del genitore nelle figlie:**
```sql
STUDENTE(CodFisc, Nome, Matricola, ...)
DOCENTE(CodFisc, Nome, Stipendio, ...)
-- Dati di PERSONA replicati in ogni figlia
```
✓ Nessun NULL | ✗ Ridondanza se attributi genitore sono molti | ✗ Solo se generalizzazione totale

**Opzione C — Mantenimento delle entità separate:**
```sql
PERSONA(CodFisc, Nome, ...)
STUDENTE(CodFisc → PERSONA, Matricola, ...)
DOCENTE(CodFisc → PERSONA, Stipendio, ...)
```
✓ Fedele al modello | ✗ Join necessari per recuperare info complete

La scelta dipende dalla cardinalità (quante entità figlie vs genitore) e dal tipo di accessi.

### 3. Partizionamento/Accorpamento di Entità

**Partizionamento verticale:** dividere un'entità con molti attributi in due tabelle (attributi più acceduti vs meno acceduti) → riduce la dimensione dei record, migliora le prestazioni.

**Accorpamento:** fondere due entità in 1:1 stretta → elimina un join.

---

## Traduzione E-R → Schema Relazionale

### Regola 1: Entità → Relazione

```
Entità STUDENTE (Matricola, Nome, Cognome, DataNasc)
   ↓
STUDENTE(Matricola, Nome, Cognome, DataNasc)
PK: Matricola
```

### Regola 2: Associazione 1:N → Chiave Esterna

La FK va nella relazione del lato **N** (molti).

```
[DOCENTE] ─(1,1)──<TIENE>──(0,N)─ [CORSO]
   ↓
DOCENTE(CodDoc, Nome, ...)
CORSO(CodCorso, Titolo, ..., CodDoc → DOCENTE)
                                    ^^^^^ FK sul lato N
```

### Regola 3: Associazione N:M → Nuova Relazione

```
[STUDENTE] ─(0,N)──<SOSTIENE>──(0,N)─ [ESAME]
con attributi: Voto, Data
   ↓
STUDENTE(Matricola, Nome, ...)
ESAME(CodEsame, Titolo, ...)
SOSTIENE(Matricola → STUDENTE, CodEsame → ESAME, Voto, Data)
PK: (Matricola, CodEsame)
```

### Regola 4: Associazione 1:1

```
[SEDE] ─(1,1)──<DIRIGE>──(0,1)─ [RESPONSABILE]
```
La FK va nella relazione con partecipazione obbligatoria (min=1):
```
SEDE(CodSede, Città, CodResp → RESPONSABILE)
```
Se entrambe obbligatorie (1,1)-(1,1) → possibile accorpamento in unica relazione.

### Regola 5: Attributi Multivalore → Nuova Relazione

```
PERSONA ha attributo multivalore Telefoni
   ↓
TELEFONO(NumTelefono, CodPersona → PERSONA)
PK: (NumTelefono, CodPersona)
```

### Regola 6: Entità Debole

```
[ORDINE] ─(1,1)──<CONTIENE>──(1,N)─ [RIGA]
dove RIGA è identificata da (NumRiga + CodOrdine)
   ↓
RIGA(NumRiga, CodOrdine → ORDINE, Quantità, Prezzo)
PK: (NumRiga, CodOrdine)
```

---

## Parte B — Normalizzazione

## 8.1 Ridondanze e Anomalie

**Problema:** schemi relazionali mal progettati contengono ridondanze che causano anomalie.

Esempio di schema problematico:
```
IMPIEGATO_PROGETTO(CodImp, NomeImp, CodProg, NomeProg, Ore)
```

| CodImp | NomeImp | CodProg | NomeProg   | Ore |
|--------|---------|---------|------------|-----|
| E1     | Mario   | P1      | Alpha      | 40  |
| E1     | Mario   | P2      | Beta       | 20  |
| E2     | Laura   | P1      | Alpha      | 30  |

**Anomalie:**
- **Inserimento:** non posso inserire un progetto senza impiegati
- **Cancellazione:** cancellare l'ultimo impiegato di un progetto perde i dati del progetto
- **Aggiornamento:** cambiare NomeProg richiede di aggiornare tutte le righe (inconsistenza se si dimentica una)

---

## 8.2 Dipendenze Funzionali

**Dipendenza funzionale (DF):** $X \rightarrow Y$ significa che i valori di X **determinano univocamente** i valori di Y.

Nello schema `IMPIEGATO_PROGETTO`:
- `CodImp → NomeImp` (un impiegato ha un nome)
- `CodProg → NomeProg` (un progetto ha un nome)
- `CodImp, CodProg → Ore` (per ogni coppia imp-prog, le ore sono uniche)

**Assiomi di Armstrong** (derivazione di nuove DF):
1. **Riflessività:** se Y ⊆ X allora X → Y
2. **Aumento:** se X → Y allora XZ → YZ
3. **Transitività:** se X → Y e Y → Z allora X → Z

**Chiusura di X (X⁺):** insieme di tutti gli attributi determinati da X.

---

## 8.3 Forma Normale di Boyce-Codd (BCNF)

> Una relazione R è in **BCNF** se per ogni dipendenza funzionale non banale $X \rightarrow Y$, X è una **superchiave** di R.

**BCNF** elimina **tutte** le ridondanze derivanti da DF.

### Algoritmo di Decomposizione in BCNF

```
Input: schema R, insieme F di DF
Output: decomposizione in BCNF

while esiste relazione Ri non in BCNF:
    trova DF X → Y che viola BCNF in Ri
    sostituisci Ri con:
        R1 = XY  (con DF X → Y)
        R2 = Ri − Y + X  (chiave = (chiave originale di Ri) − Y)
```

**Esempio:**
```
IMPIEGATO_PROGETTO(CodImp, NomeImp, CodProg, NomeProg, Ore)
DF: CodImp → NomeImp, CodProg → NomeProg, (CodImp,CodProg) → Ore

Viola BCNF: CodImp → NomeImp (CodImp non è superchiave)

Decomposizione:
  IMPIEGATO(CodImp, NomeImp)
  IMP_PROG(CodImp, CodProg, NomeProg, Ore)

IMP_PROG viola ancora: CodProg → NomeProg
  PROGETTO(CodProg, NomeProg)
  PARTECIPAZIONE(CodImp, CodProg, Ore)

Risultato finale in BCNF:
  IMPIEGATO(CodImp, NomeImp)       PK: CodImp
  PROGETTO(CodProg, NomeProg)      PK: CodProg
  PARTECIPAZIONE(CodImp, CodProg, Ore)  PK: (CodImp, CodProg)
```

> [!WARNING] Limite della BCNF
> La decomposizione in BCNF **non sempre preserva le DF**! Può essere necessario accettare una forma meno restrittiva (3NF) per garantire la preservazione.

---

## 8.4 Terza Forma Normale (3NF)

> Una relazione R è in **3NF** se per ogni DF non banale $X \rightarrow A$:
> - X è una superchiave, **oppure**
> - A è un attributo primo (membro di qualche chiave candidata)

**3NF è meno restrittiva di BCNF** → garantisce sempre la preservazione delle DF.

**Algoritmo di sintesi in 3NF:**
1. Calcola la copertura minimale F_min di F
2. Per ogni DF X → Y in F_min: crea relazione (XY) con PK = X
3. Se nessuna relazione contiene una chiave di R, aggiungi una relazione con una chiave
4. Elimina relazioni ridondanti (contenute in altre)

---

## 8.5 Prima e Seconda Forma Normale

**1NF (Prima Forma Normale):**
- Tutti gli attributi hanno domini **atomici** (nessun attributo multivalore o composto)
- Condizione minima per essere nel modello relazionale

**2NF (Seconda Forma Normale):**
- In 1NF + nessun attributo non-chiave dipende **parzialmente** dalla chiave primaria
- Rilevante solo se la PK è composta
- `(CodImp, CodProg) → NomeImp` è dipendenza **parziale** (basta CodImp)

**Gerarchia:**
$$\text{BCNF} \subset \text{3NF} \subset \text{2NF} \subset \text{1NF}$$

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **N:M** → nuova relazione con PK composta dalle FK delle due entità
> - **1:N** → FK nella relazione sul lato N
> - **Generalizzazione** → 3 strategie: accorpa figlie, accorpa genitore, mantieni separate
> - **DF X→Y**: X determina funzionalmente Y
> - **BCNF**: per ogni DF non banale X→Y, X è superchiave
> - **3NF**: come BCNF oppure A è attributo primo → preserva sempre le DF
> - **Decomposizione**: senza perdita (lossless) + preservazione DF

## Formule e Notazioni

| Simbolo | Significato |
|---|---|
| `X → Y` | X determina funzionalmente Y |
| `X⁺` | Chiusura di X rispetto a F |
| `F_min` | Copertura minimale di F |

## Link Interni

- [[Capitolo 4 - Progettazione Concettuale (E-R)]]
- [[Capitolo 6 - Tecnologia del DBMS]]
