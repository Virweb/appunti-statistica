---
tags: [database, capitolo-7, distribuito, nosql, data-warehouse, atzeni]
capitolo: 7
titolo: Architetture Distribuite, NoSQL e Data Warehouse
libro: "Basi di Dati - Atzeni, Ceri, Paraboschi, Torlone"
data_creazione: 2026-09-29
---

# Capitolo 7 — Architetture Distribuite, NoSQL e Data Warehouse

> [!NOTE] Oltre il DB centralizzato
> Quando un singolo server non basta, si usano architetture distribuite. I database NoSQL abbandonano il modello relazionale per scalabilità massima. Il Data Warehouse serve per analisi, non per transazioni operative.

---

## Parte A — Architetture Distribuite (Cap. 10)

## 10.1 Architettura Client-Server

```
┌───────────────┐        ┌───────────────┐
│    CLIENT     │◄──────>│    SERVER     │
│ (interfaccia) │  SQL   │  (DBMS + DB)  │
└───────────────┘        └───────────────┘
```

**Due livelli (2-tier):** client parla direttamente con il DB server.  
**Tre livelli (3-tier):** client → application server → DB server.  
Il 3-tier è lo standard moderno (es: browser → webapp → PostgreSQL).

---

## 10.2 Basi di Dati Distribuite

I dati sono distribuiti su **più nodi** (server) geograficamente separati.

### Frammentazione

Dividere la relazione su nodi diversi:

**Frammentazione orizzontale:** ogni frammento contiene un sottoinsieme di **righe**
```sql
-- Nodo Nord
CLIENTI_NORD = σ_{Regione='Nord'}(CLIENTI)
-- Nodo Sud
CLIENTI_SUD = σ_{Regione='Sud'}(CLIENTI)
```

**Frammentazione verticale:** ogni frammento contiene un sottoinsieme di **colonne** (con la PK replicata)
```
CLIENTI_BASE(CodCliente, Nome, Cognome)   -- Nodo A
CLIENTI_FINANZA(CodCliente, Saldo, Fido)  -- Nodo B
```

**Frammentazione mista:** combinazione orizzontale + verticale.

### Replicazione

Le stesse copie dei dati sono mantenute su più nodi:

✅ **Vantaggi:** alta disponibilità, letture più veloci (serve il nodo più vicino)  
❌ **Svantaggi:** aggiornamenti complessi (propagare su tutte le repliche), rischio di inconsistenza

### Proprietà di un DB Distribuito

- **Trasparenza alla localizzazione:** gli utenti non sanno (e non devono sapere) dove sono i dati
- **Trasparenza alla frammentazione:** le query funzionano indipendentemente dalla frammentazione
- **Trasparenza alla replicazione:** un dato aggiornato è automaticamente aggiornato ovunque

---

## 10.4 Protocollo Two-Phase Commit (2PC)

Per garantire l'atomicità di una transazione distribuita su più nodi:

```
                      COORDINATOR
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
   PARTICIPANT 1      PARTICIPANT 2      PARTICIPANT 3

FASE 1 (Prepare/Voting):
  Coordinator → tutti: "Siete pronti per il COMMIT?"
  Participant i → Coordinator: "YES" | "NO" (o timeout)

FASE 2 (Decision):
  Se tutti YES → Coordinator → tutti: "COMMIT"
  Se anche solo 1 NO o timeout → Coordinator → tutti: "ABORT"
```

**Problema:** se il coordinator crasha nella fase 2 → i participant in stato "ready" restano bloccati.  
**Soluzione:** 3PC (Three-Phase Commit) o Paxos/Raft per sistemi distribuiti moderni.

---

## Parte B — NoSQL (Cenni)

### Motivazione

I DBMS relazionali hanno difficoltà a scalare **orizzontalmente** (aggiungere server) su miliardi di record distribuiti globalmente. I sistemi NoSQL sacrificano parte delle garanzie ACID per ottenere:
- Scalabilità orizzontale massima
- Alta disponibilità
- Schema flessibile

### Teorema CAP

> Un sistema distribuito può garantire **al massimo 2 delle 3** proprietà:
> - **C**onsistency — tutti i nodi vedono gli stessi dati nello stesso momento
> - **A**vailability — ogni richiesta riceve una risposta (anche se non l'ultima versione)
> - **P**artition Tolerance — il sistema funziona anche con perdita di messaggi di rete

I sistemi NoSQL scelgono tipicamente **AP** (sacrificano consistenza forte per disponibilità).

### Famiglie NoSQL

| Tipo | Struttura | Esempi | Uso tipico |
|---|---|---|---|
| **Key-Value** | `key → value` blob | Redis, DynamoDB | Cache, sessioni |
| **Document** | JSON/BSON semi-strutturato | MongoDB, CouchDB | Cataloghi, CMS |
| **Column-family** | Colonne raggruppate per famiglia | Cassandra, HBase | Time-series, IoT |
| **Graph** | Nodi + archi con proprietà | Neo4j, Amazon Neptune | Social network, raccomandazioni |

### Eventual Consistency

La maggior parte dei sistemi NoSQL garantisce **eventual consistency**: le modifiche si propagano eventualmente a tutti i nodi, ma potrebbero non essere immediatamente visibili ovunque.

---

## Parte C — Data Warehouse e Data Mining (Cap. 13)

## 13.1 Architettura del Data Warehouse

Il **Data Warehouse (DW)** è un DB **separato** dal DB operazionale (OLTP), ottimizzato per l'**analisi** (OLAP).

```
Sistemi operazionali (OLTP)
  DBMS vendite │ DBMS HR │ File CSV │ Sorgenti esterne
              ↓
         ┌──────────┐
         │  ETL     │  Extract → Transform → Load
         │ processo │
         └────┬─────┘
              ↓
      ┌───────────────┐
      │  Data Staging │  Area temporanea, pulizia dati
      └───────┬───────┘
              ↓
      ┌───────────────┐
      │  Data Warehouse│  DB analitico centrale
      └───────┬───────┘
              ↓
      ┌───────────────┐
      │  Data Mart    │  DW per singolo dipartimento/area
      └───────┬───────┘
              ↓
      Strumenti OLAP, report, dashboard, ML
```

**OLTP vs OLAP:**

| | OLTP | OLAP |
|---|---|---|
| Scopo | Operazioni quotidiane | Analisi, decisioni |
| Operazioni | INSERT/UPDATE/DELETE | SELECT aggregati |
| Dati | Correnti, dettagliati | Storici, aggregati |
| Utenti | Impiegati, sistemi | Analisti, manager |
| Ottimizzazione | Scritture, transazioni | Letture, aggregazioni |
| Schema | Normalizzato (3NF/BCNF) | Denormalizzato (star/snowflake) |

## 13.2 Schemi per Data Warehouse

### Schema a Stella (Star Schema)

```
                  ┌──────────┐
                  │  DIM_    │
                  │  TEMPO   │
                  └────┬─────┘
                       │
┌──────────┐     ┌─────┴──────┐     ┌──────────┐
│ DIM_     ├────>│   FATTO    │<────┤  DIM_    │
│ PRODOTTO │     │  VENDITE   │     │ CLIENTE  │
└──────────┘     └─────┬──────┘     └──────────┘
                       │
                  ┌────┴─────┐
                  │  DIM_    │
                  │  NEGOZIO │
                  └──────────┘
```

- **Tabella dei fatti:** misure numeriche (Importo, Quantità, Sconto)
- **Tabelle dimensione:** contesto delle misure (chi, cosa, quando, dove)
- **Chiave della tabella dei fatti:** PK composta dalle FK delle dimensioni

### Schema a Fiocco di Neve (Snowflake Schema)

Le dimensioni sono normalizzate (gerarchie esplose in più tabelle):
```
DIM_PRODOTTO → DIM_CATEGORIA → DIM_SUPERCATEGORIA
```
✅ Meno ridondanza | ❌ Più join per le query

## 13.3 Operazioni OLAP

| Operazione | Descrizione | Esempio |
|---|---|---|
| **Roll-up** | Aggrega a livello superiore | Da giorno → mese → anno |
| **Drill-down** | Dettaglia a livello inferiore | Da regione → città → negozio |
| **Slice** | Filtra su una dimensione | Solo anno 2024 |
| **Dice** | Filtra su più dimensioni | Anno 2024, Regione Nord |
| **Pivot** | Ruota le dimensioni | Scambia righe e colonne |

**Cubo OLAP:** rappresentazione multidimensionale dei dati.

```sql
-- Esempio query OLAP in SQL
SELECT
    D_Tempo.Anno,
    D_Prodotto.Categoria,
    SUM(F_Vendite.Importo)      AS TotaleVendite,
    COUNT(F_Vendite.IdOrdine)   AS NumeroOrdini
FROM Fatto_Vendite F_Vendite
    JOIN Dim_Tempo  D_Tempo   ON F_Vendite.IdTempo   = D_Tempo.IdTempo
    JOIN Dim_Prodotto D_Prod  ON F_Vendite.IdProdotto = D_Prod.IdProdotto
GROUP BY ROLLUP(D_Tempo.Anno, D_Prod.Categoria)
ORDER BY D_Tempo.Anno, D_Prod.Categoria;
```

## 13.5 Data Mining

**Data Mining** = estrazione automatica di **pattern** e conoscenza da grandi dataset.

**Tipologie principali:**

| Tecnica | Obiettivo | Algoritmi |
|---|---|---|
| **Classificazione** | Prevedere categoria di nuovi elementi | Decision tree, SVM, Random Forest |
| **Clustering** | Raggruppare elementi simili senza supervisione | K-means, DBSCAN, hierarchical |
| **Association rules** | Trovare co-occorrenze frequenti | Apriori, FP-Growth |
| **Regressione** | Prevedere valore numerico | Regressione lineare, reti neurali |
| **Anomaly detection** | Trovare outlier | Isolation Forest, Autoencoder |

**Metriche association rules:**
- **Support:** frequenza relativa del pattern nel dataset
- **Confidence:** P(B|A) — se A allora B con quale probabilità
- **Lift:** confidence / P(B) — quanto A "aiuta" a prevedere B

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **2PC** = guarantisce atomicità distribuita a costo di possibile blocco
> - **CAP theorem** = Consistency + Availability + Partition tolerance: solo 2 delle 3
> - **Star schema** = tabella fatti centrale + dimensioni → efficiente per query OLAP
> - **ETL** = Extract, Transform, Load — la pipeline da sorgenti a DW
> - **Roll-up/Drill-down** = aggregare/disaggregare lungo le gerarchie dimensionali
> - **OLTP** = transazioni operative; **OLAP** = analisi aggregata

## Link Interni

- [[Capitolo 6 - Tecnologia del DBMS]]
- [[00 - INDICE BASI DI DATI]]
