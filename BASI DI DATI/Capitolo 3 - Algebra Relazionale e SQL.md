---
tags: [database, capitolo-3, algebra-relazionale, SQL, atzeni]
capitolo: 3
titolo: Algebra Relazionale e SQL
libro: "Basi di Dati - Atzeni, Ceri, Paraboschi, Torlone"
data_creazione: 2026-09-29
---

# Capitolo 3 & 4 — Algebra Relazionale e SQL

> [!NOTE] Interrogare i dati
> L'algebra relazionale è il fondamento teorico; SQL è il linguaggio pratico standardizzato. Questo capitolo copre entrambi in parallelo, mostrando la corrispondenza tra i costrutti.

---

## 3.1 Algebra Relazionale

Linguaggio **procedurale** — specifica *come* ottenere il risultato tramite sequenza di operatori.

### Operatori Unari

#### Selezione (σ)
Restituisce le tuple che soddisfano una condizione.

$$\sigma_{condizione}(R)$$

```
σ_{Età > 25}(STUDENTI)
```
**SQL:** `SELECT * FROM STUDENTI WHERE Età > 25`

#### Proiezione (π)
Restituisce solo gli attributi specificati (elimina le colonne non richieste).

$$\pi_{A_1, A_2, \ldots}(R)$$

```
π_{Nome, Cognome}(STUDENTI)
```
**SQL:** `SELECT Nome, Cognome FROM STUDENTI`

> [!WARNING] Proiezione e duplicati
> In algebra relazionale la proiezione **elimina i duplicati** (risultato è un insieme). In SQL invece `SELECT` senza `DISTINCT` mantiene i duplicati!

#### Ridenominazione (ρ)
Rinomina attributi o la relazione stessa.
$$\rho_{NuovoNome \leftarrow VecchioNome}(R)$$

### Operatori Insiemistici

Richiedono **compatibilità**: stessa struttura (stesso numero di attributi con gli stessi domini).

| Operatore | Simbolo | SQL | Nota |
|---|---|---|---|
| **Unione** | R ∪ S | `UNION` | Tuple in R o in S (no duplicati) |
| **Differenza** | R − S | `EXCEPT` | Tuple in R ma non in S |
| **Intersezione** | R ∩ S | `INTERSECT` | Tuple in R e in S |

### Prodotto Cartesiano (×)

Combina ogni tupla di R con ogni tupla di S:
$$R \times S$$
Se R ha n tuple e S ha m tuple → risultato ha n×m tuple.
**SQL:** `FROM R, S` (senza condizione di join)

### Join

Il join è il cuore dell'algebra relazionale — combina tuple di relazioni diverse.

#### Join Naturale (⋈)
Unisce tuple con **stesso valore** sugli **attributi con lo stesso nome**.
$$R \bowtie S$$

```
STUDENTI ⋈ ESAMI   (unisce su Matricola, che appare in entrambe)
```

#### Theta-Join (⋈_θ)
Join con condizione arbitraria θ:
$$R \bowtie_{\theta} S = \sigma_{\theta}(R \times S)$$

#### Equi-Join
Theta-join con condizione di uguaglianza (caso più comune).

#### Join Esterno (Outer Join)
Conserva anche le tuple che non hanno corrispondenza nell'altra relazione (inserisce NULL).

| Tipo | Simbolo | Conserva |
|---|---|---|
| **Left outer join** | R ⟕ S | Tutte le tuple di R |
| **Right outer join** | R ⟖ S | Tutte le tuple di S |
| **Full outer join** | R ⟗ S | Tutte le tuple di entrambi |

#### Semi-Join e Anti-Join
- **Semi-join (⋉):** tuple di R che hanno almeno un corrispondente in S
- **Anti-join:** tuple di R che NON hanno corrispondenti in S → `NOT EXISTS` in SQL

### Divisione (÷)

Trovare le entità che soddisfano tutte le condizioni di un insieme.
$$R \div S$$
"Tutti gli studenti che hanno superato **tutti** gli esami in S"

In SQL si esprime con doppia negazione: `NOT EXISTS (... NOT EXISTS ...)`

---

## 3.2 SQL — Data Definition Language (DDL)

### Creazione Tabelle

```sql
CREATE TABLE Studenti (
    Matricola   INTEGER      PRIMARY KEY,
    CodiceFisc  CHAR(16)     UNIQUE NOT NULL,
    Nome        VARCHAR(30)  NOT NULL,
    Cognome     VARCHAR(30)  NOT NULL,
    DataNasc    DATE,
    Età         INTEGER      CHECK (Età BETWEEN 17 AND 100),
    Corso       INTEGER      REFERENCES Corsi(CodCorso) 
                             ON DELETE SET NULL ON UPDATE CASCADE
);
```

**Tipi di dato SQL standard:**

| Tipo | Descrizione |
|---|---|
| `INTEGER`, `SMALLINT`, `BIGINT` | Interi |
| `DECIMAL(p,s)`, `NUMERIC(p,s)` | Decimali esatti |
| `REAL`, `FLOAT`, `DOUBLE` | Virgola mobile |
| `CHAR(n)` | Stringa a lunghezza fissa |
| `VARCHAR(n)` | Stringa a lunghezza variabile |
| `DATE`, `TIME`, `TIMESTAMP` | Data/ora |
| `BOOLEAN` | Vero/Falso |
| `BLOB`, `CLOB` | Dati binari/testuali grandi |

**Modificare tabelle:**
```sql
ALTER TABLE Studenti ADD COLUMN Email VARCHAR(100);
ALTER TABLE Studenti DROP COLUMN Età;
ALTER TABLE Studenti ALTER COLUMN Nome SET NOT NULL;
DROP TABLE Studenti;
```

---

## 3.3 SQL — Data Manipulation Language (DML)

### SELECT — La Query Fondamentale

```sql
SELECT [DISTINCT] lista_attributi
FROM   lista_tabelle
[WHERE condizione]
[GROUP BY attributi_raggruppamento]
[HAVING condizione_su_gruppi]
[ORDER BY attributi [ASC|DESC]]
[LIMIT n];
```

**Ordine di esecuzione logica:**
```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

#### Condizioni WHERE

```sql
-- Confronto
WHERE Età > 25 AND Città = 'Roma'

-- LIKE (pattern matching)
WHERE Nome LIKE 'Mar%'        -- inizia con Mar
WHERE Nome LIKE '_aria'       -- _ = un solo carattere

-- BETWEEN
WHERE Voto BETWEEN 18 AND 30

-- IN
WHERE Città IN ('Roma', 'Milano', 'Napoli')

-- IS NULL / IS NOT NULL
WHERE DataLaurea IS NULL

-- EXISTS / NOT EXISTS
WHERE EXISTS (SELECT 1 FROM Esami WHERE Esami.Matr = S.Matr)
```

#### Join in SQL

```sql
-- Inner join (esplicito, raccomandato)
SELECT S.Nome, E.Corso, E.Voto
FROM Studenti S
    INNER JOIN Esami E ON S.Matricola = E.Matricola
WHERE E.Voto >= 27;

-- Left outer join
SELECT S.Nome, E.Voto
FROM Studenti S
    LEFT JOIN Esami E ON S.Matricola = E.Matricola;

-- Self join (join di una tabella con sé stessa)
SELECT A.Nome AS Dipendente, B.Nome AS Capo
FROM Impiegati A
    JOIN Impiegati B ON A.CodCapo = B.CodImpiegato;
```

#### Funzioni Aggregate

| Funzione | Descrizione |
|---|---|
| `COUNT(*)` | Numero di righe |
| `COUNT(DISTINCT A)` | Valori distinti non-NULL |
| `SUM(A)` | Somma dei valori |
| `AVG(A)` | Media dei valori |
| `MAX(A)` | Valore massimo |
| `MIN(A)` | Valore minimo |

> [!WARNING] NULL nelle aggregate
> Le funzioni aggregate **ignorano i NULL** (tranne `COUNT(*)`). `AVG(Voto)` calcola la media solo sui valori non-NULL.

#### GROUP BY e HAVING

```sql
-- Media voti per corso
SELECT Corso, AVG(Voto) AS MediaVoto, COUNT(*) AS NrEsami
FROM Esami
GROUP BY Corso
HAVING AVG(Voto) > 25
ORDER BY MediaVoto DESC;
```

**Regola GROUP BY:** nella `SELECT` si possono usare solo:
- Attributi che compaiono nel `GROUP BY`
- Funzioni aggregate

#### Subquery (Query Annidate)

```sql
-- Subquery scalare (restituisce un solo valore)
SELECT Nome FROM Studenti
WHERE Età = (SELECT MAX(Età) FROM Studenti);

-- Subquery con IN
SELECT Nome FROM Studenti
WHERE Matricola IN (SELECT Matricola FROM Esami WHERE Voto = 30);

-- Subquery correlata con EXISTS
SELECT S.Nome FROM Studenti S
WHERE EXISTS (
    SELECT 1 FROM Esami E
    WHERE E.Matricola = S.Matricola AND E.Voto >= 28
);

-- Tutti gli studenti che hanno superato TUTTI gli esami del CdL 'Informatica'
SELECT S.Nome FROM Studenti S
WHERE NOT EXISTS (
    SELECT E.Corso FROM EsamiCdL E WHERE E.CdL = 'Informatica'
    EXCEPT
    SELECT ES.Corso FROM EsamiSuperati ES WHERE ES.Matricola = S.Matricola
);
```

#### Operatori Insiemistici in SQL

```sql
-- Unione (elimina duplicati)
SELECT Città FROM Studenti
UNION
SELECT Città FROM Docenti;

-- Unione con duplicati
SELECT Città FROM Studenti
UNION ALL
SELECT Città FROM Docenti;

-- Differenza
SELECT Matricola FROM Studenti
EXCEPT
SELECT Matricola FROM EsamiSostenuti;

-- Intersezione
SELECT Matricola FROM Studenti
INTERSECT
SELECT Matricola FROM Tesisti;
```

### INSERT, UPDATE, DELETE

```sql
-- Insert
INSERT INTO Esami (Matricola, Corso, Voto, Data)
VALUES (12345, 'BD01', 28, '2024-06-15');

INSERT INTO EsamiTop
SELECT * FROM Esami WHERE Voto = 30;

-- Update
UPDATE Esami
SET Voto = Voto + 1
WHERE Corso = 'BD01' AND Data = '2024-06-15';

-- Delete
DELETE FROM Studenti
WHERE DataLaurea < '2000-01-01';
```

---

## 3.4 Viste (VIEW)

Una vista è una **tabella virtuale** definita da una query:

```sql
CREATE VIEW EsamiTop AS
    SELECT Matricola, Corso, Voto
    FROM Esami
    WHERE Voto >= 28;

-- Usata come tabella normale
SELECT * FROM EsamiTop WHERE Corso = 'BD01';
```

**Vantaggi:**
- Semplificano query complesse (astrazione)
- Controllo degli accessi (esporre solo alcune colonne/righe)
- Indipendenza logica (la vista si adatta se la tabella cambia)

**Aggiornabilità:** le viste sono aggiornabili solo in casi semplici (nessun GROUP BY, DISTINCT, join di più tabelle, ecc.)

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **Selezione σ** = WHERE | **Proiezione π** = SELECT | **Join ⋈** = FROM + ON
> - **Join naturale** unisce su attributi con stesso nome
> - **Outer join** conserva tuple senza corrispondenza (inserisce NULL)
> - **GROUP BY** + **HAVING** per aggregazioni su gruppi
> - **NULL** non è uguale a niente — usare `IS NULL`
> - Proiezione in algebra: elimina duplicati; `SELECT` in SQL: no (serve `DISTINCT`)

## Equivalenze Algebra ↔ SQL

| Algebra | SQL |
|---|---|
| σ_cond(R) | `SELECT * FROM R WHERE cond` |
| π_{A,B}(R) | `SELECT DISTINCT A, B FROM R` |
| R ∪ S | `SELECT ... UNION SELECT ...` |
| R − S | `SELECT ... EXCEPT SELECT ...` |
| R × S | `FROM R, S` |
| R ⋈ S | `FROM R JOIN S ON cond` |
| R ⟕ S | `FROM R LEFT JOIN S ON cond` |
| R ÷ S | `NOT EXISTS (... EXCEPT ...)` |

## Link Interni

- [[Capitolo 2 - Il Modello Relazionale]]
- [[Capitolo 4 - Progettazione Concettuale (E-R)]]
