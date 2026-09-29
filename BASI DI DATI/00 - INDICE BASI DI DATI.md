---
tags: [database, indice, atzeni, overview]
titolo: "Indice - Basi di Dati"
libro: "Basi di Dati - Atzeni, Ceri, Paraboschi, Torlone"
data_creazione: 2026-09-29
---

# 🗄️ Basi di Dati
## Atzeni, Ceri, Paraboschi, Torlone

> [!NOTE] Come usare queste note
> Ogni capitolo ha la sua nota con definizioni, esempi SQL, algoritmi e tabelle di confronto. I link interni connettono i concetti correlati.

---

## 🗂️ Struttura del Corso

```
┌─────────────────────────────────────────────┐
│         PARTE 1: BASI RELAZIONALI           │
│  Cap 1: Introduzione ai DB e DBMS           │
│  Cap 2: Modello Relazionale                 │
│  Cap 3: Algebra Relazionale + SQL completo  │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│         PARTE 2: PROGETTAZIONE              │
│  Cap 4: Progettazione Concettuale (E-R)     │
│  Cap 5: Progettazione Logica + BCNF/3NF     │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│         PARTE 3: TECNOLOGIA                 │
│  Cap 6: ACID, Concorrenza, Recovery, Indici │
│  Cap 7: Distribuito, NoSQL, DW, Mining      │
└─────────────────────────────────────────────┘
```

---

## 📚 Capitoli

| # | Titolo | Temi Principali |
|---|---|---|
| [[Capitolo 1 - Introduzione alle Basi di Dati]] | Introduzione | DB, DBMS, modelli, DDL/DML, ANSI/SPARC |
| [[Capitolo 2 - Il Modello Relazionale]] | Modello Relazionale | Relazione, tuple, PK, FK, NULL, vincoli |
| [[Capitolo 3 - Algebra Relazionale e SQL]] | Algebra + SQL | σ π ⋈ ÷, SELECT, JOIN, GROUP BY, subquery |
| [[Capitolo 4 - Progettazione Concettuale (E-R)]] | E-R Model | Entità, associazioni, cardinalità, IS-A |
| [[Capitolo 5 - Progettazione Logica e Normalizzazione]] | Log. + Normal. | ER→Relazionale, DF, BCNF, 3NF |
| [[Capitolo 6 - Tecnologia del DBMS]] | Tecnologia | ACID, 2PL, deadlock, ARIES, B-Tree, optimizer |
| [[Capitolo 7 - Architetture Distribuite e Data Warehouse]] | Distribuito + OLAP | 2PC, CAP, NoSQL, star schema, ETL, data mining |

---

## 🔑 Concetti Chiave per Esame

### Definizioni Core

| Termine | Definizione rapida |
|---|---|
| **Relazione** | Sottoinsieme del prodotto cartesiano dei domini |
| **Chiave** | Insieme minimale di attributi che identifica univocamente ogni tupla |
| **PK** | Chiave primaria scelta — NON può avere NULL |
| **FK** | Attributo che referenzia la PK di un'altra relazione |
| **NULL** | Valore sconosciuto/assente — non è 0 né '' |
| **Dipendenza funzionale** | X→Y: i valori di X determinano univocamente i valori di Y |
| **BCNF** | Per ogni DF X→Y (non banale), X è superchiave |
| **3NF** | Come BCNF oppure Y è attributo primo |
| **Transazione** | Sequenza atomica di operazioni — tutto o niente |
| **ACID** | Atomicità, Consistenza, Isolamento, Durabilità |
| **2PL** | Locking a due fasi — garantisce serializzabilità |

### Quick Reference SQL

```sql
-- Struttura query completa
SELECT [DISTINCT] attributi / funzioni aggregate
FROM tabelle
[JOIN altre_tabelle ON condizione]
[WHERE condizione]
[GROUP BY attributi]
[HAVING condizione_aggregata]
[ORDER BY attributi [ASC|DESC]]
[LIMIT n];

-- Tipi di join
INNER JOIN  -- solo corrispondenze
LEFT JOIN   -- tutte le righe a sinistra + NULL a destra se non c'è match
RIGHT JOIN  -- tutte le righe a destra
FULL JOIN   -- tutte le righe di entrambi

-- Subquery
WHERE col IN (SELECT ...)
WHERE EXISTS (SELECT 1 FROM ...)
WHERE col = (SELECT MAX(...) ...)

-- Insiemi
UNION / UNION ALL / EXCEPT / INTERSECT

-- Aggregazioni
COUNT(*), COUNT(DISTINCT col), SUM, AVG, MAX, MIN
```

### Algoritmi da Sapere

| Algoritmo | Cosa fa | Capitolo |
|---|---|---|
| **Decomposizione BCNF** | Decompone fino a BCNF — non preserva sempre le DF | [[Capitolo 5 - Progettazione Logica e Normalizzazione]] |
| **Sintesi 3NF** | Produce schema in 3NF preservando le DF | [[Capitolo 5 - Progettazione Logica e Normalizzazione]] |
| **2PC** | Commit atomico distribuito | [[Capitolo 7 - Architetture Distribuite e Data Warehouse]] |
| **ARIES (Analysis→Redo→Undo)** | Recovery da crash | [[Capitolo 6 - Tecnologia del DBMS]] |
| **Conflict graph** | Verifica serializzabilità schedule | [[Capitolo 6 - Tecnologia del DBMS]] |

### Confronti Fondamentali

| A vs B | Differenza chiave |
|---|---|
| BCNF vs 3NF | BCNF più restrittivo, non preserva sempre DF; 3NF la preserva sempre |
| 2PL vs MVCC | 2PL blocca; MVCC usa versioni multiple — no block su letture |
| B-Tree vs Hash Index | B-Tree: range query ✓; Hash: solo uguaglianza, più veloce |
| OLTP vs OLAP | Operativo vs analitico; normalizzato vs denormalizzato |
| Star vs Snowflake | Star: più semplice/veloce; Snowflake: meno ridondanza |
| SQL UNION vs UNION ALL | UNION elimina duplicati (più lento); UNION ALL li mantiene |

---

## 📝 Domande d'Esame Frequenti

> [!IMPORTANT] Tipici quesiti d'esame
> 1. Dato uno schema E-R, tradurlo in schema relazionale
> 2. Dati degli attributi e DF, trovare le chiavi e verificare BCNF
> 3. Scrivere query SQL con JOIN e aggregazioni
> 4. Spiegare le anomalie di uno schema non normalizzato
> 5. Calcolare X⁺ (chiusura di X rispetto alle DF)
> 6. Descrivere il protocollo 2PC e i suoi limiti
> 7. Spiegare le anomalie della concorrenza e i livelli di isolamento
> 8. Differenza tra outer join e inner join con esempio
