---
tags: [statistica, probabilita, indice, crismale, overview]
titolo: "Indice - Calcolo delle Probabilità e Statistica"
docente: "Prof. Vito Onofrio Crismale"
libri: ["Ross - Statistica per l'Ingegneria e le Scienze", "Baldi - Calcolo delle Probabilità e Statistica"]
data_creazione: 2026-10-27
---

# 📊 Calcolo delle Probabilità e Statistica
## Prof. Vito Onofrio Crismale — Ross & Baldi

> [!NOTE] Come usare queste note nel Grafo di Obsidian
> Questo file funge da **nodo centrale (Hub)** per tutte le lezioni e le guide di Statistica. Ogni lezione è collegata sia sequenzialmente (Lezione precedente / successiva) sia concettualmente ai prerequisiti teorici e alla guida pratica agli esercizi d'esame.

---

## 🗂️ Struttura del Corso (Finora)

```
┌────────────────────────────────────────────────────────────────────┐
│  BLOCCO 1: FONDAMENTI DI PROBABILITÀ                               │
│  • Lezione 01: Spazi di Probabilità (Ω, F, P), Assiomi, Classica   │
│  • Lezione 02: Probabilità Condizionata, Totali, Bayes, Indipend.  │
└────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌────────────────────────────────────────────────────────────────────┐
│  BLOCCO 2: VARIABILI ALEATORIE E FUNZIONE DI RIPARTIZIONE          │
│  • Lezione 03: Prove di Bernoulli, Variabili Aleatorie, Legge P_X, │
│                Funzione di Ripartizione F_X, Vettori e V.A. Discr. │
└────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌────────────────────────────────────────────────────────────────────┐
│  BLOCCO 3: MODELLI DISCRETI NOTEVOLI                               │
│  • Lezione 04: Distribuzioni di Bernoulli e Binomiale b(n, p),     │
│                Overbooking, Connessioni e Bayes + Binomiale        │
│  • Lezione 05: V.A. Geometrica Geo(p) e Mancanza di Memoria        │
│  • Lezione 06: V.A. di Poisson P(λ), Teorema Limite, Somma di      │
│                Poisson e Trasformazioni Y = min(X, k)              │
└────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌────────────────────────────────────────────────────────────────────┐
│  STRUMENTI OPERATIVI PER L'ESAME                                   │
│  • Guida Pratica agli Esercizi d'Esame e Strategie Risolutive      │
└────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Lezioni e Guide

| File / Nodo | Argomenti Principali | Concetti Chiave |
|---|---|---|
| [[Lezione 01 - Spazi di Probabilità]] | Spazio campionario, eventi, assiomi, spazi equiprobabili | $(\Omega, \mathcal{F}, P)$, $\sigma$-additività, inclusione-esclusione, dadi |
| [[Lezione 02 - Probabilità Condizionata, Teorema di Bayes ed Eventi Indipendenti]] | Probabilità condizionata, formula probabilità totali, Bayes, indipendenza | $P(A\mid B)$, partizione, cause a posteriori, compleanni, urna |
| [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]] | Prove ripetute $S$-$I$, variabili aleatorie, legge $P_X$, FdR $F_X$, vettori aleatori | $P_X = P_Y \centernot\implies X = Y$, proprietà di $F_X$, salti, indipendenza di v.a., pf $p_X$ |
| [[Lezione 04 - Variabili Aleatorie Discrete, Bernoulli e Binomiale]] | Modelli di Bernoulli e Binomiale, somma di Bernoulli/Binomiali, esercizi | $\text{Ber}(p)$, $b(n,p)$, overbooking, 24 operatori, indipendenza condizionata + Bayes |
| [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]] | Tempo di primo successo, serie geometrica, assenza di memoria | $\text{Geo}(p)$, $P(X > n) = (1-p)^n$, $P(X > n+m \mid X > m) = P(X > n)$ |
| [[Lezione 06 - Variabile Aleatoria di Poisson e Legge dei Casi Rari]] | Eventi rari, limite della Binomiale, stabilità per somma, trasformazioni | $P(\lambda)$, Teorema di Poisson, $X+Y \sim P(\lambda_1+\lambda_2)$, $Y = \min(X, 2)$ |
| [[Guida Pratica agli Esercizi d'Esame e Strategie Risolutive]] | Pattern recognition tracce, schemi risolutivi, dimostrazioni salva-orale | Parole spia, tabella di Bayes, complementare, checklist pre-consegna |

---

## 🔑 Formulario Rapido e Riferimenti

| Formula / Teorema | Espressione Matematica | Lezione di Riferimento |
|---|---|---|
| **Principio di Inclusione-Esclusione** | $P(A \cup B) = P(A) + P(B) - P(A \cap B)$ | [[Lezione 01 - Spazi di Probabilità]] |
| **Probabilità Classica (Equiprobabile)** | $P(E) = \frac{\lvert E \rvert}{\lvert \Omega \rvert}$ | [[Lezione 01 - Spazi di Probabilità]] |
| **Probabilità Condizionata** | $P(A \mid B) = \frac{P(A \cap B)}{P(B)}$ | [[Lezione 02 - Probabilità Condizionata, Teorema di Bayes ed Eventi Indipendenti]] |
| **Formula delle Probabilità Totali** | $P(A) = \sum_{i=1}^n P(A \mid B_i)P(B_i)$ | [[Lezione 02 - Probabilità Condizionata, Teorema di Bayes ed Eventi Indipendenti]] |
| **Teorema di Bayes** | $P(B_i \mid A) = \frac{P(A \mid B_i)P(B_i)}{\sum_{k=1}^n P(A \mid B_k)P(B_k)}$ | [[Lezione 02 - Probabilità Condizionata, Teorema di Bayes ed Eventi Indipendenti]] |
| **Eventi Indipendenti** | $P(A \cap B) = P(A) \cdot P(B)$ | [[Lezione 02 - Probabilità Condizionata, Teorema di Bayes ed Eventi Indipendenti]] |
| **Funzione di Ripartizione (FdR)** | $F_X(t) = P(X \le t), \quad P(a < X \le b) = F_X(b) - F_X(a)$ | [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]] |
| **Salto Puntuale di $F_X$** | $P(X = t) = F_X(t) - \lim_{x \to t^-} F_X(x)$ | [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]] |
| **Indipendenza di Variabili Aleatorie** | $F_X(t_1, \dots, t_n) = F_{X_1}(t_1)\cdots F_{X_n}(t_n)$ | [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]] |
| **Bernoulli $\text{Ber}(p)$** | $P(X = 1) = p, \quad P(X = 0) = 1 - p$ | [[Lezione 04 - Variabili Aleatorie Discrete, Bernoulli e Binomiale]] |
| **Binomiale $b(n, p)$** | $P(X = k) = \binom{n}{k}p^k(1-p)^{n-k}, \quad k \in \{0,\dots,n\}$ | [[Lezione 04 - Variabili Aleatorie Discrete, Bernoulli e Binomiale]] |
| **Geometrica $\text{Geo}(p)$** | $P(X = k) = p(1-p)^{k-1}, \quad k \in \mathbb{N}\setminus\{0\}$ | [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]] |
| **Mancanza di Memoria** | $P(X > n+m \mid X > m) = P(X > n)$ | [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]] |
| **Poisson $P(\lambda)$** | $P(X = k) = e^{-\lambda}\frac{\lambda^k}{k!}, \quad k \in \mathbb{N}\cup\{0\}$ | [[Lezione 06 - Variabile Aleatoria di Poisson e Legge dei Casi Rari]] |
| **Somma di Poisson Indipendenti** | $X \sim P(\lambda_1) \perp Y \sim P(\lambda_2) \implies X+Y \sim P(\lambda_1+\lambda_2)$ | [[Lezione 06 - Variabile Aleatoria di Poisson e Legge dei Casi Rari]] |

---

## ⚖️ Tabella di Confronto: Le 4 Distribuzioni Discrete Notevoli

| Distribuzione | Codominio $\text{cod}(X)$ | Parametri | Cosa Conta / Modella | Proprietà Chiave | Lezione |
|---|---|---|---|---|---|
| **Bernoulli** $\text{Ber}(p)$ | $\{0, 1\}$ | $p \in (0,1)$ | Esito di una singola prova dicotomica (Successo/Insuccesso) | $X = \mathbf{1}_E$ | [[Lezione 04 - Variabili Aleatorie Discrete, Bernoulli e Binomiale]] |
| **Binomiale** $b(n, p)$ | $\{0, 1, \dots, n\}$ | $n \ge 1, \, p \in (0,1)$ | Numero di successi su $n$ prove indipendenti fissate | Somma di $n$ $\text{Ber}(p)$ indip.; $b(n_1,p)+b(n_2,p) \sim b(n_1+n_2,p)$ | [[Lezione 04 - Variabili Aleatorie Discrete, Bernoulli e Binomiale]] |
| **Geometrica** $\text{Geo}(p)$ | $\{1, 2, 3, \dots\}$ | $p \in (0,1)$ | Tentativo in cui cade il $1^\circ$ successo | Assenza di memoria; $P(X > n) = (1-p)^n$ | [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]] |
| **Poisson** $P(\lambda)$ | $\{0, 1, 2, \dots\}$ | $\lambda > 0$ | Conteggio di eventi rari in un intervallo continuo ($n \to \infty, p \to 0$) | Limite di $b(n, \lambda/n)$; $P(\lambda_1)+P(\lambda_2) \sim P(\lambda_1+\lambda_2)$ | [[Lezione 06 - Variabile Aleatoria di Poisson e Legge dei Casi Rari]] |
