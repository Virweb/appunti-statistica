---
tags: [database, capitolo-2, modello-relazionale, atzeni]
capitolo: 2
titolo: Il Modello Relazionale
libro: "Basi di Dati - Atzeni, Ceri, Paraboschi, Torlone"
data_creazione: 2026-09-29
---

# Capitolo 2 — Il Modello Relazionale

> [!NOTE] Il cuore teorico
> Il modello relazionale (Codd, 1970) organizza i dati in **relazioni** (tabelle). Ha una solida base matematica nella teoria degli insiemi e nella logica del primo ordine.

---

## 2.1 La Struttura del Modello Relazionale

### Terminologia

| Termine formale | Sinonimo pratico | Descrizione |
|---|---|---|
| **Relazione** | Tabella | Insieme di tuple con lo stesso schema |
| **Tupla** | Riga / Record | Un'istanza della relazione |
| **Attributo** | Colonna / Campo | Proprietà della relazione |
| **Dominio** | Tipo di dato | Insieme dei valori ammessi per un attributo |
| **Schema** | Intestazione | Insieme dei nomi degli attributi + dominio |
| **Istanza** | Contenuto | Insieme delle tuple attuali |

### Definizione Formale

**Prodotto cartesiano:** $D_1 \times D_2 \times \ldots \times D_n$ = insieme di tutte le n-uple $(v_1, v_2, \ldots, v_n)$ con $v_i \in D_i$

**Relazione:** sottoinsieme finito del prodotto cartesiano dei domini

**Schema di relazione:** $R(A_1:D_1, A_2:D_2, \ldots, A_n:D_n)$  
**Esempio:** `STUDENTI(Matricola:INTEGER, Nome:VARCHAR, Età:INTEGER)`

### Proprietà delle Relazioni

- Le tuple sono un **insieme** → **nessun ordine**, **nessun duplicato**
- I valori degli attributi sono **atomici** (prima forma normale — 1NF)
- L'ordine degli attributi è irrilevante (si accede per nome)

### Valori Nulli

**NULL** = valore sconosciuto o non applicabile (non è 0 né stringa vuota!)

Tre interpretazioni possibili:
1. **Valore sconosciuto** — esiste ma non lo sappiamo (es: data di nascita mancante)
2. **Valore inesistente** — non ha senso per questa tupla (es: numero coniuge per single)
3. **Valore non applicabile** — fuori dal dominio

> [!WARNING] Attenzione ai NULL
> `NULL = NULL` è **NULL** (non TRUE) in SQL! Usare `IS NULL` / `IS NOT NULL`. Influenzano aggregazioni, join e comparazioni in modo non intuitivo.

---

## 2.2 Vincoli di Integrità

I vincoli **limitano i valori ammissibili** nel DB, garantendo la coerenza.

### Vincoli Intrarelazionali

Riguardano una singola relazione.

**Vincolo di dominio:** il valore di un attributo deve appartenere al suo dominio
```sql
Età INTEGER CHECK (Età >= 0 AND Età <= 150)
```

**Vincolo di tupla:** condizione che coinvolge attributi della stessa tupla
```sql
CHECK (DataFine >= DataInizio)
```

**Chiave:** insieme minimale di attributi che identifica univocamente ogni tupla

Tipi di chiavi:
- **Superchiave** — insieme di attributi che identificano univocamente una tupla (non necessariamente minimale)
- **Chiave (candidata)** — superchiave minimale
- **Chiave primaria (PK)** — chiave scelta come identificatore principale → **non può avere NULL**
- **Chiave alternativa** — chiave candidata non scelta come primaria

```
Esempio:
STUDENTI(Matricola, CodiceFiscale, Nome, Cognome, Età)
- Superchiavi: {Matricola}, {CodiceFiscale}, {Matricola, Nome}, ...
- Chiavi: {Matricola}, {CodiceFiscale}
- PK: {Matricola}  (scelta convenzionalmente)
```

### Vincoli Interrelazionali — Integrità Referenziale

La **chiave esterna (FK)** collega due relazioni:

```
ESAMI(Matricola → STUDENTI.Matricola, Corso, Voto, Data)
```

**Regola:** ogni valore della FK deve esistere come PK nella relazione referenziata (o essere NULL).

**Azioni alla violazione:**
| Azione | Descrizione |
|---|---|
| `NO ACTION` / `RESTRICT` | Rifiuta l'operazione |
| `CASCADE` | Propaga la modifica/cancellazione |
| `SET NULL` | Imposta la FK a NULL |
| `SET DEFAULT` | Imposta un valore di default |

```sql
FOREIGN KEY (Matricola) REFERENCES STUDENTI(Matricola)
    ON DELETE CASCADE
    ON UPDATE NO ACTION
```

---

## 2.3 Algebra Relazionale — Operatori Base

*(Vedi [[Capitolo 3 - Algebra Relazionale e SQL]] per il dettaglio completo)*

| Operatore | Simbolo | Descrizione |
|---|---|---|
| Selezione | σ | Filtra le tuple per una condizione |
| Proiezione | π | Seleziona un sottoinsieme di attributi |
| Unione | ∪ | Unisce due relazioni compatibili |
| Differenza | − | Tuple in R1 ma non in R2 |
| Prodotto cartesiano | × | Combina ogni tupla di R1 con ogni tupla di R2 |
| Ridenominazione | ρ | Rinomina attributi o relazione |

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **Relazione** = sottoinsieme del prodotto cartesiano dei domini
> - **Chiave** = insieme minimale di attributi identificatore univoco
> - **PK** non può avere NULL; **FK** deve rispettare l'integrità referenziale
> - **NULL** ≠ 0 e ≠ '' — rappresenta l'assenza di valore
> - Le tuple sono un **insieme** → no ordine, no duplicati

## Formule / Notazioni

| Notazione | Significato |
|---|---|
| `R(A₁, A₂, ..., Aₙ)` | Schema della relazione R |
| `t[A]` | Valore dell'attributo A nella tupla t |
| `FK → R.PK` | Chiave esterna che referenzia R |

## Link Interni

- [[Capitolo 1 - Introduzione alle Basi di Dati]]
- [[Capitolo 3 - Algebra Relazionale e SQL]]
