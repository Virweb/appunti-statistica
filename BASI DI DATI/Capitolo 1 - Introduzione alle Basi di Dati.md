---
tags: [database, capitolo-1, introduzione, atzeni]
capitolo: 1
titolo: Introduzione alle Basi di Dati
libro: "Basi di Dati - Atzeni, Ceri, Paraboschi, Torlone"
data_creazione: 2026-09-29
---

# Capitolo 1 — Introduzione alle Basi di Dati

> [!NOTE] Il punto di partenza
> Prima di entrare nel tecnico, il capitolo stabilisce il lessico fondamentale: cos'è un dato, cos'è un'informazione, perché esistono i DBMS e quali vantaggi portano rispetto ai file tradizionali.

---

## 1.1 Informazione e Dati

**Dato** = fatto grezzo, registrato e memorizzato (es: `"42"`, `"Rossi"`)  
**Informazione** = dato interpretato in un contesto (es: `"Mario Rossi ha 42 anni"`)

I sistemi informativi gestiscono informazioni per supportare le attività di un'organizzazione. Le **basi di dati** sono il cuore della componente di memorizzazione di qualsiasi sistema informativo moderno.

---

## 1.2 Basi di Dati e DBMS

**Base di dati (DB):**
> Insieme organizzato di dati, utilizzati per supportare lo svolgimento delle attività di un ente (pubblico o privato).

**DBMS (Database Management System):**
> Sistema software che gestisce grandi quantità di dati, persistenti e condivisi, in modo efficiente e affidabile.

### Proprietà fondamentali dei dati in un DB

| Proprietà | Descrizione |
|---|---|
| **Grandi** | Dimensione superiore alla memoria RAM disponibile |
| **Persistenti** | Vivono oltre la singola esecuzione di un programma |
| **Condivisi** | Acceduti da più utenti/applicazioni simultaneamente |

### Obiettivi di un DBMS

- **Efficienza** — accesso rapido ai dati (strutture indice, ottimizzazione query)
- **Affidabilità** — i dati sopravvivono a guasti hardware e software (recovery)
- **Condivisione sicura** — controllo della concorrenza, gestione dei conflitti
- **Privatezza** — controllo accessi tramite autorizzazioni

---

## 1.3 Modelli dei Dati

Un **modello dei dati** è un insieme di costrutti per organizzare i dati e descriverne la struttura e la dinamica.

### Livelli di Astrazione (Architettura ANSI/SPARC)

```
┌──────────────────────────────────┐
│       Schema Esterno (Vista)     │  ← Quello che vede l'utente/app
├──────────────────────────────────┤
│       Schema Logico              │  ← Struttura logica completa del DB
├──────────────────────────────────┤
│       Schema Fisico              │  ← Come i dati sono memorizzati su disco
└──────────────────────────────────┘
```

**Indipendenza logica:** si può modificare lo schema logico senza toccare gli schemi esterni  
**Indipendenza fisica:** si può cambiare l'organizzazione fisica senza modificare lo schema logico

### Modelli principali

| Modello | Epoca | Caratteristica |
|---|---|---|
| **Gerarchico** | Anni '60 | Struttura ad albero, navigazione per puntatori |
| **Reticolare (CODASYL)** | Anni '70 | Grafi, flessibile ma complesso |
| **Relazionale** | 1970 (Codd) | Tabelle, algebra relazionale, standard de facto |
| **A oggetti** | Anni '90 | Oggetti con identità, ereditarietà, metodi |
| **Object-Relational** | Fine '90 | Estensione SQL con tipi complessi |
| **NoSQL** | 2000s | Document, key-value, graph, column-family |

> [!NOTE] Il modello relazionale
> Proposto da **E.F. Codd** nel 1970 (paper "A Relational Model of Data for Large Shared Data Banks"). Basa la rappresentazione su **relazioni** (tabelle) e usa la logica del primo ordine come fondamento teorico. Ancora dominante oggi.

---

## 1.4 Linguaggi e Utenti

### Tipi di Linguaggi

| Tipo | Sigla | Funzione | Esempio |
|---|---|---|---|
| **Data Definition Language** | DDL | Definisce la struttura del DB | `CREATE TABLE`, `ALTER` |
| **Data Manipulation Language** | DML | Interroga e modifica i dati | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **Data Control Language** | DCL | Controlla accessi e transazioni | `GRANT`, `REVOKE`, `COMMIT` |

**SQL** unifica DDL + DML + DCL in un unico linguaggio standardizzato.

### Categorie di Utenti

```
┌────────────────────────────────────────────────┐
│              Utenti finali                      │  Non tecnici, usano interfacce
├────────────────────────────────────────────────┤
│              Programmatori applicativi          │  SQL embedded, API
├────────────────────────────────────────────────┤
│              Amministratore DB (DBA)            │  Schema, autorizzazioni, tuning
└────────────────────────────────────────────────┘
```

**DBA (Database Administrator):**
- Definisce lo schema logico e fisico
- Gestisce autorizzazioni e sicurezza
- Monitora le prestazioni e ottimizza
- Gestisce backup e recovery

---

## 1.5 Vantaggi e Svantaggi dei DBMS

### Vantaggi

✅ **Indipendenza dei dati** — le applicazioni non dipendono dalla struttura fisica  
✅ **Gestione della concorrenza** — più utenti accedono contemporaneamente senza conflitti  
✅ **Controllo dell'integrità** — i vincoli garantiscono la coerenza dei dati  
✅ **Sicurezza** — controllo granulare degli accessi (tabella, riga, colonna)  
✅ **Recovery** — ripristino automatico dopo guasti  
✅ **Riduzione della ridondanza** — i dati sono memorizzati una volta sola  
✅ **Standardizzazione** — SQL come lingua franca  

### Svantaggi

❌ **Costo** — licenze DBMS commerciali (Oracle, SQL Server) sono costose  
❌ **Complessità** — richiede personale specializzato (DBA)  
❌ **Overhead** — per applicazioni semplici, un DBMS può essere eccessivo  
❌ **Inadeguato per dati non strutturati** — immagini, video, testi liberi (→ NoSQL)

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **DB** = dati grandi, persistenti, condivisi
> - **DBMS** = software che gestisce il DB con efficienza, affidabilità, privacy
> - **Architettura a 3 livelli** = esterno (viste) → logico → fisico
> - **Indipendenza** fisica e logica = separazione dei livelli
> - **DDL** per struttura, **DML** per dati, **DCL** per accessi

## Link Interni

- [[Capitolo 2 - Il Modello Relazionale]]
- [[00 - INDICE BASI DI DATI]]
