---
tags: [statistica, probabilita, guida-esame, esercizi, formulario, crismale]
titolo: "Guida Pratica agli Esercizi d'Esame e Strategie Risolutive"
docente: "Prof. Vito Onofrio Crismale"
---

# Guida Pratica agli Esercizi d'Esame: Metodi Risolutivi, Strategie e Trabocchetti

> 🏠 **Indice:** [[00 - INDICE STATISTICA]] | 📚 **Lezioni:** [[Lezione 01 - Spazi di Probabilità]] • [[Lezione 02 - Probabilità Condizionata, Teorema di Bayes ed Eventi Indipendenti]] • [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]] • [[Lezione 04 - Variabili Aleatorie Discrete, Bernoulli e Binomiale]] • [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]] • [[Lezione 06 - Variabile Aleatoria di Poisson e Legge dei Casi Rari]]

---
**Corso:** Calcolo delle Probabilità e Statistica  
**Docente:** Prof. Vitonofrio Crismale
**Obiettivo della guida:** Fornire un prontuario operativo orientato alla risoluzione degli esercizi d'esame (scritti e orali), basato esattamente sui pattern, gli esempi e gli schemi visti a lezione.

---

## 🧭 Mappa di Decodifica Rapida (Pattern Recognition)

Quando leggi una traccia d'esame, cerca immediatamente queste **parole spia**:

| Frase nel testo | Cosa significa matematicamente | Strumento da usare |
|---|---|---|
| *"Sapendo che..."*, *"Dato che..."*, *"Dato l'esito..."* | Lo spazio campionario è ridotto ad un evento già avvenuto | **Probabilità Condizionata** o **Teorema di Bayes** |
| *"La causa che ha provocato..."*, *"Quale macchina ha prodotto il pezzo difettoso..."* | Effetto noto, causa incognita (probabilità inversa) | **Teorema di Bayes** |
| *"Almeno uno..."*, *"Almeno due..."*, *"Non tutti..."* | Il calcolo diretto richiederebbe una somma complicata di troppi casi disgiunti | **Passaggio al Complementare:** $P(A) = 1 - P(A^c)$ |
| *"Estrazione con rimessa (con reimbussolamento)"* | Le prove sono **identiche e indipendenti** | **Binomiale** / Prodotto di probabilità fisse |
| *"Estrazione senza rimessa (senza reimbussolamento)"* | La composizione cambia ad ogni estrazione; le prove **NON** sono indipendenti | **Ipergeometrica / Coefficienti Binomiali $\binom{\cdot}{\cdot}$** |
| *"Numero di successi su $n$ tentativi indipendenti"* | Conteggio su numero fisso di prove | **Distribuzione Binomiale** $X \sim \text{Bin}(n, p)$ |
| *"Finché non si ottiene il primo successo..."* | Numero di tentativi fino al 1° evento | **Distribuzione Geometrica** $X \sim \text{Geo}(p)$ |
| *"Arrivi in un intervallo di tempo/spazio"*, *"Tasso medio $\lambda$"*, *"Eventi rari con $n$ grande e $p$ piccolo"* | Fenomeni continui di conteggio o limite della binomiale | **Distribuzione di Poisson** $X \sim \text{Poisson}(\lambda)$ |
| *"Sia $Y = \min(X, k)$"* oppure *"$\max(X, k)$"* | Variabile aleatoria troncata/saturata | **Partizione sui valori critici e salto cumulato nella FdD** |

---

## 1. Tematica: Probabilità Condizionata e Invarianza dell'Ordine

### 🎯 La Trappola Mentale
In problemi con estrazioni successive **senza rimessa** (es. palline bianche e nere), quasi tutti pensano che l'ordine cambi la probabilità marginale di estrazione.

### 💡 Il Consiglio d'Oro
> [!tip] Invarianza della Marginale
> Se non possiedi alcuna informazione su cosa è stato estratto alla prima estrazione, la probabilità che la seconda (o la $k$-esima) pallina sia bianca è **identica** a quella della prima:
> $$P(\text{II estratta bianca}) = P(\text{I estratta bianca}) = \frac{b}{b+n}$$
> Dimostralo sempre allo scritto usando la partizione della prima estrazione!

### 📝 Risoluzione Modello: Estrazione a 2 Fasi senza Rimessa
**Traccia:** Un'urna contiene $b$ palline bianche ed $n$ nere. Si estraggono 2 palline senza rimessa. Calcolare $P(\text{II estratta Bianca})$.

1. **Step 1 - Dichiara sempre gli eventi con precisione:**
   - $A = \{\text{la II estratta è bianca}\}$
   - $B = \{\text{la I estratta è bianca}\}$
   - $B^c = \{\text{la I estratta è nera}\}$
2. **Step 2 - Scrivi la decomposizione:**
   $$P(A) = P(A \mid B)P(B) + P(A \mid B^c)P(B^c)$$
3. **Step 3 - Calcola i 4 termini elementari:**
   - $P(B) = \frac{b}{b+n}$
   - $P(A \mid B) = \frac{b-1}{b+n-1}$ (tolta una bianca, ne restano $b-1$ su $b+n-1$)
   - $P(B^c) = \frac{n}{b+n}$
   - $P(A \mid B^c) = \frac{b}{b+n-1}$ (tolta una nera, tutte le $b$ bianche sono ancora dentro)
4. **Step 4 - Concludi algebricamente semplificando:**
   $$P(A) = \frac{(b-1)b + bn}{(b+n)(b+n-1)} = \frac{b(b-1+n)}{(b+n)(b+n-1)} = \frac{b}{b+n}$$

---

## 2. Tematica: Teorema di Bayes e Probabilità delle Cause

### 🎯 La Trappola Mentale
Scambiare la probabilità del dato/sintomo dato lo stato, con lo stato dato il dato:
$$P(\text{Difetto} \mid \text{Macchina } 2) \ne P(\text{Macchina } 2 \mid \text{Difetto})$$
Inoltre, molti dimenticano di calcolare correttamente il denominatore (la probabilità totale).

### 💡 Il Consiglio d'Oro (Tabella a 3 Colonne per Bayes)
> [!tip] Usa la griglia tabellare
> Invece di scrivere formule kilometriche con indici che possono confondere, imposta una tabella:
> 
> | Causa $C_i$ | Priori $P(C_i)$ | Verosimiglianza $P(D \mid C_i)$ | Prodotto $P(C_i \cap D) = P(D \mid C_i)P(C_i)$ |
> |---|:---:|:---:|:---:|
> | $M_1$ | $P(M_1)$ | $P(D \mid M_1)$ | $A_1$ |
> | $M_2$ | $P(M_2)$ | $P(D \mid M_2)$ | $A_2$ |
> | $M_3$ | $P(M_3)$ | $P(D \mid M_3)$ | $A_3$ |
> | **Totale** | **1** | — | **$P(D) = \sum A_i$** |
> 
> A questo punto la probabilità a posteriori di qualsiasi causa è semplicemente:
> $$P(M_k \mid D) = \frac{A_k}{\sum A_i}$$

---

### 📝 Risoluzione Modello: Bayes applicato a Parametri di Binomiali
Questo è uno dei problemi più frequenti e con il punteggio più alto negli scritti (combinazione tra Bayes e Binomiale).

**Traccia:**  
Una scatola da 10 pezzi proviene con probabilità $0.3$ dalla linea $A$ (tasso difetti $10\%$) e con probabilità $0.7$ dalla linea $B$ (tasso difetti $17\%$). Si estrae la scatola e si trova **esattamente 1 pezzo difettoso** ($X = 1$). Qual è la probabilità che provenga dalla linea $A$?

**Guida allo svolgimento:**
1. **Riconosci il condizionamento della v.a.:**  
   Non puoi trattare i pezzi come incondizionatamente indipendenti, perché sapere che un pezzo è difettoso aumenta la probabilità che l'intera scatola venga dalla linea peggiore ($B$).
   - Se la scatola è di tipo $A$: $X \mid A \sim \text{Bin}(10, \, 0.1)$
   - Se la scatola è di tipo $B = A^c$: $X \mid B \sim \text{Bin}(10, \, 0.17)$
2. **Calcola le verosimiglianze binomiali con $k = 1$:**
   $$P(X=1 \mid A) = \binom{10}{1} (0.1)^1 (0.9)^9 = 10 \cdot 0.1 \cdot (0.9)^9 = (0.9)^9 \approx 0.3874$$
   $$P(X=1 \mid B) = \binom{10}{1} (0.17)^1 (0.83)^9 = 1.7 \cdot (0.83)^9 \approx 0.3186$$
3. **Calcola la Probabilità Totale di trovare 1 difetto:**
   $$P(X=1) = P(X=1 \mid A)P(A) + P(X=1 \mid B)P(B) = (0.3874)(0.3) + (0.3186)(0.7) = 0.1162 + 0.2230 = 0.3392$$
4. **Applica Bayes:**
   $$P(A \mid X=1) = \frac{P(X=1 \mid A)P(A)}{P(X=1)} = \frac{0.1162}{0.3392} \approx 34.26\%$$

---

## 3. Tematica: Evento Complementare ("Almeno uno...")

### 🎯 La Trappola Mentale
Cercare di calcolare direttamente $P(\text{almeno due successi})$ o $P(\text{almeno un compleanno uguale})$ sommando decine di casi disgiunti. Questo porta a formule ingestibili o al classico blocco durante lo scritto.

### 💡 Il Consiglio d'Oro
> [!tip] La Regola del "Nessuno"
> Ogni volta che vedi *"almeno uno"* o *"almeno due"*, ribalta immediatamente la domanda:
> $$P(\text{almeno un evento}) = 1 - P(\text{nessun evento})$$
> $$P(\text{almeno due eventi coincidenti}) = 1 - P(\text{tutti distinti})$$

### 📝 Risoluzione Modello: Paradosso dei Compleanni
**Traccia:** In un'aula ci sono $n$ studenti. Qual è la probabilità che almeno due compiano gli anni lo stesso giorno?

1. **Passa all'evento opposto:**  
   $A^c = \{\text{tutti gli } n \text{ studenti hanno date di nascita distinte}\}$.
2. **Usa il principio moltiplicativo delle scelte ordinate:**
   - 1° studente: 365 giorni disponibili
   - 2° studente: 364 giorni
   - $k$-esimo studente: $365 - k + 1$ giorni
   - Casi favorevoli ad $A^c$: $|A^c| = 365 \cdot 364 \dots (365 - n + 1) = \frac{365!}{(365-n)!}$
3. **Casi totali equiprobabili:**  
   Ogni studente può essere nato in un giorno qualsiasi: $|\Omega| = 365^n$.
4. **Rapporto e complementare:**
   $$P(A) = 1 - \frac{365!}{(365-n)! \cdot 365^n}$$
   *Valore d'esame da ricordare a mente per l'orale:* Con $n=23$, la probabilità supera il $50\%$ ($\approx 0.507$).

---

## 4. Tematica: Variabili Aleatorie e Funzione di Distribuzione (FdD)

### 🎯 La Trappola Mentale
1. Confondere una variabile aleatoria (funzione $X: \Omega \to \mathbb{R}$) con il valore numerico che assume.
2. Pensare che se due v.a. hanno la stessa distribuzione, debbano essere la stessa funzione.
3. Dimenticare che per v.a. discrete, la funzione di ripartizione $F_X(t) = P(X \le t)$ è **costante a tratti** (a gradini) e i salti corrispondono alle probabilità puntuali:
   $$\text{Ampiezza del salto in } t_0 = P(X = t_0) = F_X(t_0) - F_X(t_0^-)$$

### 💡 Il Consiglio d'Oro (Come trattare Minimi e Massimi di v.a.)
> [!tip] Tecnica per $Y = \min(X, c)$
> Quando una variabile viene saturata o limitata superiormente:
> 1. Individua i valori possibili di $Y$: assumerà tutti i valori $X < c$ invariati, e si fermerà a $c$ per tutti i valori $X \ge c$.
> 2. Per il valore soglia $c$, la probabilità puntuale è la coda superiore:
>    $$P(Y = c) = P(X \ge c) = 1 - P(X < c)$$
> 3. Disegna $F_Y(y)$ ricordando che per $y \ge c$ la funzione ha raggiunto stabilmente il valore $1$.

---

### 📝 Risoluzione Modello: $Y = \min(X, 2)$ con $X \sim \text{Poisson}(3)$

1. **Codominio di $Y$:**  
   Poiché $X \in \{0, 1, 2, 3, \dots\}$, $Y = \min(X, 2)$ può assumere **solamente tre valori**: $\{0, 1, 2\}$.
2. **Funzione di probabilità di $Y$:**
   - Per $y = 0$: $Y = 0 \iff X = 0 \implies P(Y=0) = e^{-3}\frac{3^0}{0!} = e^{-3}$
   - Per $y = 1$: $Y = 1 \iff X = 1 \implies P(Y=1) = e^{-3}\frac{3^1}{1!} = 3e^{-3}$
   - Per $y = 2$: $Y = 2 \iff X \ge 2$:
     $$P(Y=2) = 1 - [P(X=0) + P(X=1)] = 1 - 4e^{-3}$$
3. **Scrittura della FdD $F_Y(y) = P(Y \le y)$ a tratti:**
   $$F_Y(y) = \begin{cases}
   0 & \text{se } y < 0 \\
   e^{-3} & \text{se } 0 \le y < 1 \\
   4e^{-3} & \text{se } 1 \le y < 2 \\
   1 & \text{se } y \ge 2
   \end{cases}$$
4. **Verifica di Indipendenza tra $\{Y=1\}$ e $\{X \le 2\}$:**
   - L'evento $\{Y=1\}$ coincide con $\{X=1\}$.
   - Poiché $\{X=1\} \subset \{X \le 2\}$, la loro intersezione è $\{X=1\}$:
     $$P(\{Y=1\} \cap \{X \le 2\}) = P(X=1) = 3e^{-3}$$
   - Verifica se vale la fattorizzazione $P(A \cap B) = P(A)P(B)$:
     $$P(Y=1) \cdot P(X \le 2) = (3e^{-3}) \cdot \left(e^{-3} + 3e^{-3} + \frac{9}{2}e^{-3}\right) = 3e^{-3} \cdot \left(\frac{17}{2}e^{-3}\right) = \frac{51}{2} e^{-6}$$
   - Poiché $3e^{-3} \ne \frac{51}{2}e^{-6}$, **gli eventi NON sono indipendenti**.

---

## 5. Tematica: Le Dimostrazioni Teoriche Chiave (Salva-Orale)

Se prendi tra 16 e 17 allo scritto e devi sostenere l'orale, il prof verificherà quasi certamente una di queste tre dimostrazioni:

### 1. Proprietà di Mancanza di Memoria della Geometrica
> **Enunciato:** Se $X \sim \text{Geo}(p)$, allora $\forall n, m \ge 1$:
> $$P(X > n+m \mid X > m) = P(X > n)$$

**I tre passaggi della dimostrazione:**
1. Esprimi l'evento $\{X > k\}$: significa fare $k$ insuccessi consecutivi, quindi:
   $$P(X > k) = (1-p)^k$$
2. Nota che $\{X > n+m\} \subseteq \{X > m\}$, perciò l'intersezione coincide con il sottoinsieme più piccolo $\{X > n+m\}$.
3. Applica la definizione di condizionata:
   $$P(X > n+m \mid X > m) = \frac{P(X > n+m)}{P(X > m)} = \frac{(1-p)^{n+m}}{(1-p)^m} = (1-p)^n = P(X > n) \quad \blacksquare$$

---

### 2. Somma di Poisson Indipendenti
> **Enunciato:** Se $X \sim \text{Poisson}(\lambda_1)$ e $Y \sim \text{Poisson}(\lambda_2)$ sono indipendenti, allora $X + Y \sim \text{Poisson}(\lambda_1 + \lambda_2)$.

**I tre passaggi della dimostrazione:**
1. Decomponi l'evento $\{X + Y = k\}$ sull'indice $j \in \{0, \dots, k\}$:
   $$P(X+Y=k) = \sum_{j=0}^k P(X=j, Y=k-j) \stackrel{\text{indip.}}{=} \sum_{j=0}^k P(X=j)P(Y=k-j)$$
2. Sostituisci le formule di Poisson:
   $$= \sum_{j=0}^k \left(e^{-\lambda_1} \frac{\lambda_1^j}{j!}\right) \left(e^{-\lambda_2} \frac{\lambda_2^{k-j}}{(k-j)!}\right) = e^{-(\lambda_1+\lambda_2)} \sum_{j=0}^k \frac{\lambda_1^j \lambda_2^{k-j}}{j!(k-j)!}$$
3. Moltiplica e dividi per $k!$ per far comparire il binomio di Newton:
   $$= \frac{e^{-(\lambda_1+\lambda_2)}}{k!} \underbrace{\sum_{j=0}^k \binom{k}{j} \lambda_1^j \lambda_2^{k-j}}_{= (\lambda_1+\lambda_2)^k} = e^{-(\lambda_1+\lambda_2)} \frac{(\lambda_1+\lambda_2)^k}{k!} \quad \blacksquare$$

---

### 3. $P_X = P_Y \centernot\implies X = Y$ (Stessa Legge non implica Stessa Variabile)
Prepara sempre a mente il controesempio del dado a 6 facce:
- $\Omega = \{1, 2, 3, 4, 5, 6\}$ con esiti equiprobabili $1/6$.
- Definisci:
  - $X = 1$ se esce pari $\{2,4,6\}$, altrimenti $0$.
  - $Y = 1$ se esce $\ge 4$ $\{4,5,6\}$, altrimenti $0$.
- Su $\omega = 2$: $X(2) = 1$ ma $Y(2) = 0$, dunque $X \ne Y$ come funzioni su $\Omega$.
- Tuttavia $P(X=0) = P(Y=0) = 1/2$ e $P(X=1) = P(Y=1) = 1/2$, per cui $P_X = P_Y$.

---

## 6. Checklist Finale Pre-Esame

Prima di consegnare lo scritto o rispondere all'orale, controlla:
- [ ] Ho definito formalmente gli eventi con lettere maiuscole ($A, B, M_i$) prima di fare i calcoli?
- [ ] Se ho applicato $P(A \cup B) = P(A) + P(B)$, ho verificato/scritto esplicitamente che $A \cap B = \emptyset$?
- [ ] La probabilità finale ottenuta appartiene all'intervallo $[0, 1]$? (Se viene $<0$ o $>1$, c'è un errore algebrico palese).
- [ ] In problemi con percentuali di successo su $n$ persone (es. overbooking aereo), ho chiarito quale sia la variabile $X$, quanti siano i tentativi $n$ e quale sia $p$?
- [ ] Nella funzione di ripartizione $F_X(t)$ ho verificato che sia non decrescente e che $\lim_{t \to +\infty} F_X(t) = 1$?

---

## 🔗 Link Interni (Mappa del Grafo)

- 🏠 **Indice Generale:** [[00 - INDICE STATISTICA]]
- 📘 **Fondamenti e Spazi Equiprobabili:** [[Lezione 01 - Spazi di Probabilità]]
- 📘 **Probabilità Condizionata, Probabilità Totali, Bayes e Compleanni:** [[Lezione 02 - Probabilità Condizionata, Teorema di Bayes ed Eventi Indipendenti]]
- 📘 **Prove di Bernoulli, Variabili Aleatorie, Legge $P_X$, FdR $F_X$ e Controesempio $P_X = P_Y$:** [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]]
- 📘 **Modelli di Bernoulli e Binomiale, Overbooking e Bayes+Binomiale:** [[Lezione 04 - Variabili Aleatorie Discrete, Bernoulli e Binomiale]]
- 📘 **V.A. Geometrica e Dimostrazione Mancanza di Memoria:** [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]]
- 📘 **V.A. di Poisson, Teorema Limite, Somma di Poisson e $Y = \min(X, 2)$:** [[Lezione 06 - Variabile Aleatoria di Poisson e Legge dei Casi Rari]]