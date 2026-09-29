---
tags: [database, capitolo-6, transazioni, concorrenza, recovery, atzeni]
capitolo: 6
titolo: Tecnologia del DBMS — Transazioni, Concorrenza, Recovery
libro: "Basi di Dati - Atzeni, Ceri, Paraboschi, Torlone"
data_creazione: 2026-09-29
---

# Capitolo 6 — Tecnologia del DBMS

> [!NOTE] Dentro il motore del DB
> Questo capitolo svela il funzionamento interno di un DBMS: come garantisce l'atomicità delle operazioni (transazioni), gestisce gli accessi concorrenti e si riprende dai guasti.

---

## 9.1 Transazioni

### Definizione

Una **transazione** è una sequenza di operazioni sul DB che deve essere eseguita come un'**unità atomica**: o tutte le operazioni vengono completate con successo, o nessuna modifica viene applicata al DB.

```sql
BEGIN TRANSACTION;
    UPDATE ContoA SET Saldo = Saldo - 1000;
    UPDATE ContoB SET Saldo = Saldo + 1000;
COMMIT;
-- Se qualcosa va male → ROLLBACK
```

### Proprietà ACID

| Proprietà | Significato |
|---|---|
| **A — Atomicità** | Tutto o niente: o tutte le operazioni vanno a buon fine o nessuna modifica persiste |
| **C — Consistenza** | La transazione porta il DB da uno stato consistente a un altro stato consistente |
| **I — Isolamento** | Le transazioni concorrenti non si "vedono" (come se fossero seriali) |
| **D — Durabilità** | Una volta eseguita la COMMIT, i dati sopravvivono a qualsiasi guasto |

### Stati di una Transazione

```
                  ┌──────────┐
             ─────> ATTIVA   │
             │    └────┬─────┘
             │         │ operazioni
             │         ▼
             │   ┌──────────────┐
             │   │  PARZIALMENTE│
             │   │  COMMITTED   │◄── tutte le operazioni OK
             │   └──────┬───────┘
             │          │
             │    ┌─────┴──────┐ verifica vincoli
             │    ▼            ▼
             │ COMMITTED    ABORTED
             │ (COMMIT)     (ROLLBACK)
             │                 │
             └─────────────────┘ (retry o terminazione)
```

---

## 9.2 Controllo della Concorrenza

**Problema:** più transazioni eseguono contemporaneamente sugli stessi dati → possibili interferenze.

### Anomalie della Concorrenza

**Lost Update (Aggiornamento perso):**
```
T1: legge X=100; (T2: legge X=100, scrive X=150); T1: scrive X=120
→ L'aggiornamento di T2 è perso!
```

**Dirty Read (Lettura sporca):**
```
T1: scrive X=200; T2: legge X=200; T1: fa ROLLBACK (X torna a 100)
→ T2 ha letto un dato che non è mai stato confermato
```

**Unrepeatable Read (Lettura non ripetibile):**
```
T1: legge X=100; T2: scrive X=200 e fa COMMIT; T1: rilegge X=200
→ T1 legge valori diversi per la stessa query
```

**Phantom Read:**
```
T1: conta righe WHERE età>30 → 5; T2: inserisce riga età=35 e fa COMMIT; T1: riconta → 6
→ Appaiono nuove righe "fantasma"
```

### Serializzabilità

Uno **schedule** (sequenza interleaved di operazioni) è **serializzabile** se produce lo stesso risultato di qualche esecuzione seriale delle transazioni.

**Conflict Serializzabilità:**
Due operazioni sono in **conflitto** se appartengono a transazioni diverse, operano sullo stesso dato e almeno una è una scrittura.

**Conflict graph:** nodo per ogni transazione; arco Ti→Tj se Ti ha un'operazione in conflitto che precede una di Tj.
- Lo schedule è conflict-serializzabile se e solo se il conflict graph è **aciclico**.

### Protocolli di Locking

**2PL (Two-Phase Locking):**

Ogni transazione acquisisce tutti i lock prima di rilasciarne alcuno:
- **Fase di crescita:** acquisisce lock (shared per letture, exclusive per scritture)
- **Fase di decrescita:** rilascia lock (nessun nuovo lock)

```
T1: lock_s(X) → leggi X → lock_x(Y) → scrivi Y → unlock(X) → unlock(Y)
     ──── fase crescita ────         ──── fase decrescita ────
```

**2PL garantisce la serializzabilità.**

**Deadlock:** due transazioni si bloccano reciprocamente aspettando i lock dell'altra.
```
T1 → lock(X), aspetta lock(Y)
T2 → lock(Y), aspetta lock(X)
→ stallo!
```

Rilevazione: **grafo wait-for** → se c'è un ciclo → deadlock → abort una transazione.  
Prevenzione: **timeout** o ordinamento delle risorse.

### Livelli di Isolamento SQL

| Livello | Dirty Read | Unrepeatable Read | Phantom |
|---|---|---|---|
| `READ UNCOMMITTED` | ✓ possibile | ✓ possibile | ✓ possibile |
| `READ COMMITTED` | ✗ | ✓ possibile | ✓ possibile |
| `REPEATABLE READ` | ✗ | ✗ | ✓ possibile |
| `SERIALIZABLE` | ✗ | ✗ | ✗ |

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

## 9.3 Gestione del Buffer

Il DBMS gestisce un **buffer pool** in memoria RAM per ridurre gli accessi a disco.

**Buffer Manager:**
- Mantiene pagine disco in memoria (frame)
- **Dirty bit:** indica se una pagina in buffer è stata modificata
- **Policy di rimpiazzamento:** LRU, Clock, MRU (scelta dipende dal tipo di accesso)

**Regola del WAL (Write Ahead Log):**
> Prima di modificare i dati su disco, scrivi il log.

---

## 9.4 Recovery (Ripristino dopo Guasti)

### Tipi di Guasto

| Tipo | Causa | Effetto |
|---|---|---|
| **Guasto di transazione** | Abort, deadlock | Solo la transazione fallisce |
| **Guasto di sistema** | Crash SO/hardware | Perdita del buffer (RAM) |
| **Guasto di dispositivo** | Disco danneggiato | Perdita di dati persistenti |

### Log (Giornale)

Il **log** registra sequenzialmente tutte le operazioni sul DB:
```
[LSN, TxID, tipo, tabella, pagina, old_value, new_value, prev_LSN]
```

**Record di log:**
- `BEGIN(Ti)` — inizio transazione
- `WRITE(Ti, X, old, new)` — Ti scrive X (da old a new)
- `COMMIT(Ti)` — Ti ha fatto commit
- `ABORT(Ti)` — Ti è stata abortita

### Protocollo ARIES (Algoritmo di Recovery)

**Tre fasi del recovery:**

1. **Analysis Phase:** scandisce il log dall'ultimo checkpoint → determina quali transazioni erano attive al momento del crash (winner: committed; loser: non committed)

2. **Redo Phase:** ripete **tutte** le operazioni dal checkpoint (anche delle loser) per ricostruire lo stato al momento del crash

3. **Undo Phase:** annulla le operazioni delle **loser transactions** (dall'ultima al BEGIN) → garantisce atomicità

```
        Checkpoint          Crash
─────────────────────────────────────────
T1:   ───BEGIN─────COMMIT─────────────────  (winner)
T2:   ─────────BEGIN──────WRITE──write──   (loser → UNDO)
T3:   ──────────────────BEGIN──WRITE────   (loser → UNDO)
                   │← REDO da qui →│
```

---

## 9.5 Strutture di Accesso Fisico

### Indici

Un **indice** è una struttura dati che velocizza la ricerca basata su uno o più attributi.

**Tipi:**

**B-Tree (Albero B+):**
- Struttura ad albero bilanciato
- Foglie contengono i valori dell'attributo indicizzato + puntatori alle tuple
- Foglie collegate in una lista → supporto efficiente a range query
- O(log n) per ricerche, inserimenti, cancellazioni
- **Index universale** — usato per PK, FK, attributi di ricerca frequenti

```
        [30 | 70]
       /    |    \
  [10|20] [40|60] [80|90]
```

**Hash Index:**
- Funzione hash sull'attributo → bucket con le tuple
- O(1) per ricerche esatte
- **Non** supporta range query o ordinamento

**Indice cluster (primario):** i dati sono fisicamente ordinati secondo l'indice → solo uno per tabella  
**Indice non cluster (secondario):** puntatori alle tuple → più lenti ma possono essere molti

### Costo delle Operazioni

| Operazione | Senza indice | Con B-Tree | Con Hash |
|---|---|---|---|
| Ricerca esatta | O(n) | O(log n) | O(1) |
| Range query | O(n) | O(log n + k) | O(n) |
| Ordinamento | O(n log n) | O(1) se cluster | O(n log n) |

---

## 9.6 Ottimizzazione delle Query

Il **query optimizer** trasforma una query SQL in un piano di esecuzione efficiente.

### Fasi dell'Ottimizzazione

1. **Parsing:** verifica sintattica e semantica, costruisce l'albero della query
2. **Riscrittura logica:** applica regole algebriche per semplificare (push-down delle selezioni, eliminazione ridondanze)
3. **Ottimizzazione fisica:** sceglie gli operatori fisici (nested loop join vs hash join vs merge join) e l'ordine dei join
4. **Generazione del piano:** produce il piano di esecuzione (explain plan)

### Regole di Riscrittura

```
-- Push-down della selezione (più selettiva è la selezione, meno dati circolano)
σ_c(R ⋈ S) → σ_c(R) ⋈ S    (se c riguarda solo R)

-- Cascata di selezioni
σ_{c1 AND c2}(R) → σ_c1(σ_c2(R))

-- Commutatività del join
R ⋈ S ≡ S ⋈ R
```

### Stima dei Costi e Statistiche

L'ottimizzatore usa statistiche sul catalogo del DB:
- Cardinalità delle relazioni (numero di tuple)
- Cardinalità degli attributi (numero di valori distinti)
- Istogrammi della distribuzione dei valori

```sql
-- In PostgreSQL, aggiorna le statistiche
ANALYZE Studenti;

-- Vedi il piano di esecuzione
EXPLAIN ANALYZE SELECT * FROM Studenti WHERE Età > 25;
```

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **ACID** = Atomicità, Consistenza, Isolamento, Durabilità
> - **2PL** garantisce serializzabilità; può causare deadlock
> - **WAL** = scrivere il log prima dei dati → fondamentale per il recovery
> - **ARIES** = Analysis → Redo → Undo
> - **B-Tree** = indice universale; **Hash** = solo ricerche esatte
> - **Livelli isolamento**: READ COMMITTED è il default in molti DBMS

## Link Interni

- [[Capitolo 5 - Progettazione Logica e Normalizzazione]]
- [[Capitolo 7 - Architetture Distribuite e NoSQL]]
