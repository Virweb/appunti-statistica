---
tags: [statistica, probabilita, lezione-06, poisson, eventi-rari, binomio-di-newton, trasformazioni]
lezione: 6
titolo: "Variabile Aleatoria di Poisson, Teorema Limite ed Esercizi sulle Trasformazioni"
docente: "Prof. Vito Onofrio Crismale"
---

# Lezione 06 - Variabile Aleatoria di Poisson, Teorema Limite ed Esercizi sulle Trasformazioni

> ⬅️ **Precedente:** [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]] | 🏠 **Indice:** [[00 - INDICE STATISTICA]] | 🛠️ **Pratica:** [[Guida Pratica agli Esercizi d'Esame e Strategie Risolutive]]

---
**Docente:** Vito Onofrio Crismale (`vitoonofrio.crismale@uniba.it`)  
**Testi di riferimento:**
- Ross, *Statistica per l'Ingegneria e le Scienze*
- Baldi, *Calcolo delle Probabilità e Statistica*

---

## 1. Definizione di Variabile Aleatoria di Poisson: $X \sim P(\lambda)$

> [!abstract] L'Idea in Parole Semplici: "La Legge degli Eventi Rari"
> Cosa succede a una distribuzione Binomiale $b(n, p)$ quando il numero di prove $n$ è **enorme** ($n \to +\infty$), ma la probabilità di successo $p$ in ogni singola prova è **piccolissima** ($p \to 0$), in modo tale che il numero medio di successi $\lambda = n \cdot p$ rimanga costante e moderato?  
> Calcolare $\binom{n}{k}p^k(1-p)^{n-k}$ per $n = 10\,000$ è proibitivo: entra allora in gioco la **distribuzione di Poisson**, che modella il conteggio di eventi rari in un intervallo di tempo o spazio (es. telefonate a un call center, pacchetti a un router, guasti, decadimenti radioattivi).

### Definizione Formale

Sia $\lambda > 0$. Una variabile aleatoria $X$ si dice di **Poisson** di parametro $\lambda$, e si scrive $X \sim P(\lambda)$ (oppure $X \sim \text{Poisson}(\lambda)$), se $X$ è una variabile aleatoria discreta con:
1. Codominio dato da tutti gli interi non negativi:
   $$\text{cod}(X) = \mathbb{N} \cup \{0\} = \{0, 1, 2, 3, \dots\}$$
2. Funzione di probabilità (fp) definita, per ogni $k \in \mathbb{N} \cup \{0\}$, da:
   $$\boxed{p_X(k) = P(X = k) = e^{-\lambda} \frac{\lambda^k}{k!} \qquad \forall k \in \mathbb{N} \cup \{0\}}$$
   *(ricordando per convenzione che $0! = 1$ e $\lambda^0 = 1$, quindi $P(X = 0) = e^{-\lambda}$).*

### Verifica della Condizione di Normalizzazione
Ricordando lo sviluppo in serie di Taylor della funzione esponenziale valido per ogni $\lambda \in \mathbb{R}$:
$$e^\lambda = \sum_{k=0}^{+\infty} \frac{\lambda^k}{k!}$$
verifichiamo immediatamente che la somma delle probabilità su tutto il codominio vale $1$:
$$\sum_{k=0}^{+\infty} p_X(k) = \sum_{k=0}^{+\infty} e^{-\lambda} \frac{\lambda^k}{k!} = e^{-\lambda} \underbrace{\sum_{k=0}^{+\infty} \frac{\lambda^k}{k!}}_{= e^\lambda} = e^{-\lambda} \cdot e^\lambda = 1 \quad \blacksquare$$

---

## 2. Il Teorema di Poisson (Approssimazione della Binomiale)

### Motivazione Empirica
Se si effettua un numero molto alto $n$ di esperimenti indipendenti di tipo successo/insuccesso, ciascuno con probabilità di successo $p$ **molto bassa**, allora la variabile aleatoria $X \sim b(n, p)$ che conta i successi si comporta in realtà come una variabile aleatoria di Poisson:
$$\boxed{X \sim b(n, p) \quad \text{con } n \text{ grande, } p \text{ piccolo} \quad \Longrightarrow \quad X \approx P(\lambda) \quad \text{dove } \lambda = n \cdot p}$$

---

### Enunciato del Teorema di Poisson

> [!note] Teorema (Limite di Poisson / Legge dei Casi Rari)
> Sia $\lambda > 0$ fissato e sia $\{X_n\}_{n \ge 1}$ una successione di variabili aleatorie Binomiali tali che:
> $$X_n \sim b\left(n, \, \frac{\lambda}{n}\right) \qquad \left(\text{cioè } p_n = \frac{\lambda}{n} \iff \lambda = n \cdot p_n\right)$$
> Allora, per ogni intero fissato $k \ge 0$, la funzione di probabilità di $X_n$ converge, per $n \to +\infty$, alla funzione di probabilità di una Poisson di parametro $\lambda$:
> $$\boxed{\lim_{n \to +\infty} p_{X_n}(k) = \lim_{n \to +\infty} P(X_n = k) = e^{-\lambda} \frac{\lambda^k}{k!} \qquad \forall k \ge 0}$$

---

### Dimostrazione Completa Passo-Passo (Svolta a Lezione)

Sia $k \ge 0$ fissato. Scriviamo la funzione di probabilità della Binomiale $X_n \sim b\left(n, \frac{\lambda}{n}\right)$:

1. **Espressione iniziale:**
   $$p_{X_n}(k) = P(X_n = k) = \binom{n}{k} \left(\frac{\lambda}{n}\right)^k \left(1 - \frac{\lambda}{n}\right)^{n - k}$$

2. **Esplicitiamo il coefficiente binomiale e separiamo le potenze:**
   Ricordiamo che $\binom{n}{k} = \frac{n!}{k!(n-k)!} = \frac{n(n-1)(n-2)\cdots(n-k+1)}{k!}$ e che $\left(1 - \frac{\lambda}{n}\right)^{n-k} = \frac{\left(1 - \frac{\lambda}{n}\right)^n}{\left(1 - \frac{\lambda}{n}\right)^k}$:
   $$p_{X_n}(k) = \frac{n(n - 1)\cdots(n - k + 1)}{k!} \cdot \frac{\lambda^k}{n^k} \cdot \frac{\left(1 - \frac{\lambda}{n}\right)^n}{\left(1 - \frac{\lambda}{n}\right)^k}$$

3. **Raccogliamo $n$ da ciascuno dei $k$ fattori al numeratore:**
   Nel prodotto $n(n - 1)(n - 2)\cdots(n - k + 1)$ ci sono esattamente $k$ fattori; raccogliendo $n$ da ciascun fattore otteniamo un $n^k$ davanti che si semplifica con l'$n^k$ a denominatore:
   $$p_{X_n}(k) = \frac{\cancel{n^k} \cdot 1 \cdot \left(1 - \frac{1}{n}\right)\left(1 - \frac{2}{n}\right)\cdots\left(1 - \frac{k - 1}{n}\right)}{\cancel{n^k}} \cdot \frac{\lambda^k}{k!} \cdot \frac{\left(1 - \frac{\lambda}{n}\right)^n}{\left(1 - \frac{\lambda}{n}\right)^k}$$

4. **Passaggio al limite per $n \to +\infty$:**  
   Poiché $k$ è un intero **fissato**, analizziamo il comportamento di ciascun blocco quando $n \to +\infty$:
   - Per ogni $j \in \{1, \dots, k-1\}$, si ha $\frac{j}{n} \to 0$, quindi ciascuno dei fattori $\left(1 - \frac{j}{n}\right) \longrightarrow 1$. Essendo un prodotto di un numero finito ($k$) di termini che tendono a $1$:
     $$\left(1 - \frac{1}{n}\right)\left(1 - \frac{2}{n}\right)\cdots\left(1 - \frac{k - 1}{n}\right) \xrightarrow{n \to +\infty} 1$$
   - Al denominatore, poiché $k$ è fissato e $\frac{\lambda}{n} \to 0$:
     $$\left(1 - \frac{\lambda}{n}\right)^k \xrightarrow{n \to +\infty} 1^k = 1$$
   - Al numeratore compare il **limite fondamentale (notevole) di Nepero**:
     $$\text{Poiché } \left(1 + \frac{1}{n}\right)^n \to e, \quad \text{si ha:} \qquad \boxed{\left(1 - \frac{\lambda}{n}\right)^n \xrightarrow{n \to +\infty} e^{-\lambda}}$$
   - Il termine $\frac{\lambda^k}{k!}$ non dipende da $n$ e resta invariato.

5. **Conclusione:**
   Mettendo insieme i limiti dei singoli fattori:
   $$\lim_{n \to +\infty} p_{X_n}(k) = 1 \cdot \frac{\lambda^k}{k!} \cdot \frac{e^{-\lambda}}{1} = e^{-\lambda} \frac{\lambda^k}{k!} \quad \blacksquare$$

---

## 3. Esempio Modellistico: Telefonate a un Centro di Assistenza

Per capire *perché* i fenomeni di conteggio nel tempo continuo seguono una legge di Poisson, vediamo la costruzione fisica illustrata a lezione.

**Problema:**  
Consideriamo il numero di telefonate che arrivano a un centro di assistenza in un intervallo di tempo $[0, T]$. Perché tale numero è modellato da una variabile di Poisson?

```
Intervallo [0, T] suddiviso in N micro-intervalli di ampiezza T/N:
|---1---|---2---|---3---|  ...  |---i---|  ...  |---N---|
0      T/N    2T/N    3T/N                     (N-1)T/N  T
   X_1     X_2     X_3             X_i             X_N     (ciascuna X_i in {0, 1})
```

**Costruzione del Modello:**
1. Supponiamo di dividere l'intervallo $[0, T]$ in $N$ sotto-intervalli di uguale lunghezza $\frac{T}{N}$, con **$N$ molto grande** (cosicché ciascun micro-intervallo sia brevissimo, ad esempio un millisecondo).
2. Se $N$ è abbastanza grande, in ogni singolo micro-intervallo è fisicamente impossibile che arrivino due o più chiamate simultanee: **arriva al più una telefonata** ($0$ oppure $1$).
3. Possiamo allora considerare, per ogni micro-intervallo $i \in \{1, \dots, N\}$, la variabile aleatoria di Bernoulli:
   $$X_i = \begin{cases} 1 & \text{se arriva una chiamata nell'}i\text{-esimo intervallino} \\ 0 & \text{se non arrivano chiamate} \end{cases}$$
4. Se la probabilità di arrivo nel singolo intervallino è proporzionale alla sua ampiezza, ossia $X_i \sim b\left(1, \frac{\lambda}{N}\right)$, e gli arrivi in intervalli disgiunti sono **indipendenti**, allora il numero totale di chiamate che avvengono in $[0, T]$ è dato dalla somma:
   $$\sum_{i=1}^N X_i \sim b\left(N, \, \frac{\lambda}{N}\right)$$
5. Poiché $N$ è "grande" ($N \to +\infty$), per il **Teorema di Poisson** appena dimostrato:
   $$\boxed{\sum_{i=1}^N X_i \approx P(\lambda)}$$
   Ecco spiegato perché il numero di telefonate (o di accessi a un server, clienti in coda, ecc.) in un intervallo $[0, T]$ segue la distribuzione di Poisson!

---

## 4. Somma di Variabili di Poisson Indipendenti (Riproduttività)

Come per la Binomiale (a parità di $p$), anche la famiglia di Poisson è **chiusa rispetto alla somma di variabili indipendenti**: sommare due flussi di Poisson indipendenti produce un nuovo flusso di Poisson il cui tasso è la somma dei tassi!

### Proposizione

> [!note] Proposizione (Somma di Poisson Indipendenti)
> Siano $\lambda_1, \lambda_2 > 0$ e siano $X \sim P(\lambda_1)$ e $Y \sim P(\lambda_2)$ due variabili aleatorie **indipendenti**. Allora la variabile somma segue ancora una legge di Poisson di parametro $\lambda_1 + \lambda_2$:
> $$\boxed{X + Y \sim P(\lambda_1 + \lambda_2)}$$

---

### Dimostrazione Completa Passo-Passo (Svolta a Lezione)

Poiché $X$ e $Y$ assumono valori in $\mathbb{N} \cup \{0\}$, anche $X + Y$ assume valori interi $k \ge 0$.  
Fissiamo $k \ge 0$ e calcoliamo $P(X + Y = k)$:

1. **Decomposizione dell'evento $\{X + Y = k\}$ in eventi disgiunti:**  
   La somma $X + Y$ vale $k$ se e solo se $X$ assume un certo valore $j \in \{0, 1, \dots, k\}$ e contemporaneamente $Y$ assume il valore complementare $k - j$:
   $$\{X + Y = k\} = \bigcup_{j=0}^k \{X = j, \, Y = k - j\}$$
   Poiché al variare di $j \in \{0, \dots, k\}$ gli eventi $\{X = j, \, Y = k - j\}$ sono **a due a due disgiunti**, per additività si ha:
   $$P(X + Y = k) = P\left( \bigcup_{j=0}^k \{X = j, \, Y = k - j\} \right) = \sum_{j=0}^k P(X = j, \, Y = k - j)$$

2. **Uso dell'ipotesi di indipendenza tra $X$ e $Y$:**
   Poiché $X$ e $Y$ sono indipendenti, la probabilità congiunta si fattorizza nel prodotto delle marginali:
   $$= \sum_{j=0}^k P(X = j) \cdot P(Y = k - j)$$

3. **Sostituzione delle funzioni di probabilità di Poisson:**
   $$= \sum_{j=0}^k \left( e^{-\lambda_1} \frac{\lambda_1^j}{j!} \right) \left( e^{-\lambda_2} \frac{\lambda_2^{k - j}}{(k - j)!} \right)$$

4. **Trucco algebrico: raccoglimento degli esponenziali e moltiplicazione/divisione per $k!$:**
   Portiamo fuori dalla sommatoria il fattore $e^{-\lambda_1} e^{-\lambda_2} = e^{-(\lambda_1 + \lambda_2)}$, e **moltiplichiamo e dividiamo per $k!$** per far comparire il coefficiente binomiale $\binom{k}{j} = \frac{k!}{j!(k-j)!}$:
   $$= \frac{e^{-(\lambda_1 + \lambda_2)}}{k!} \sum_{j=0}^k \underbrace{\frac{k!}{j!(k - j)!}}_{= \binom{k}{j}} \lambda_1^j \lambda_2^{k - j} = \frac{e^{-(\lambda_1 + \lambda_2)}}{k!} \sum_{j=0}^k \binom{k}{j} \lambda_1^j \lambda_2^{k - j}$$

5. **Applicazione della Formula del Binomio di Newton:**
   Ricordando lo sviluppo del binomio di Newton:
   $$\boxed{(x + y)^n = \sum_{k=0}^n \binom{n}{k} x^k y^{n - k}}$$
   la sommatoria coincide esattamente con $(\lambda_1 + \lambda_2)^k$. Pertanto:
   $$P(X + Y = k) = e^{-(\lambda_1 + \lambda_2)} \frac{(\lambda_1 + \lambda_2)^k}{k!}$$
   che è esattamente la funzione di probabilità di una variabile aleatoria di Poisson di parametro $\lambda_1 + \lambda_2$. Dunque **$X + Y \sim P(\lambda_1 + \lambda_2)$** $\blacksquare$.

---

## 5. Esercizio Completo d'Esame (Trasformazione di una v.a. di Poisson)

Questo esercizio (svolto interamente nell'ultima pagina degli appunti) è un **classico assoluto delle tracce scritte d'esame del Prof. Crismale**.

### Testo dell'Esercizio
Sia $X \sim P(3)$ una variabile aleatoria di Poisson di parametro $\lambda = 3$, e si definisca la nuova variabile aleatoria:
$$Y = \min(X, 2)$$
1. **Punto a):** Determinare la funzione di probabilità (fp) $p_Y$ di $Y$, la sua funzione di distribuzione (FdD) $F_Y$ e rappresentarla graficamente.
2. **Punto b):** Stabilire se gli eventi $\{Y = 1\}$ e $\{X \le 2\}$ sono stocasticamente indipendenti.

---

### Svolgimento Passo-Passo

#### Punto a.1 — Codominio e Funzione di Probabilità $p_Y$ di $Y$
Dobbiamo anzitutto capire quali valori può assumere $Y = \min(X, 2)$.  
Poiché $X \sim P(3)$ assume valori in $\{0, 1, 2, 3, 4, \dots\}$:
- Se $X = 0 \implies Y = \min(0, 2) = 0$
- Se $X = 1 \implies Y = \min(1, 2) = 1$
- Se $X \ge 2$ (cioè $X \in \{2, 3, 4, \dots\}$) $\implies Y = \min(X, 2) = 2$

Pertanto il codominio di $Y$ è finito ed è:
$$\boxed{\text{cod}(Y) = \{0, 1, 2\}}$$

Calcoliamo ora la funzione di probabilità $p_Y(k) = P(Y = k)$ nei tre punti del codominio, ricordando che $P(X = k) = e^{-3} \frac{3^k}{k!}$:

1. **Per $k = 0$:**
   $$p_Y(0) = P(Y = 0) = P\big(\min(X, 2) = 0\big) = P(X = 0) = e^{-3} \frac{3^0}{0!} = \boxed{e^{-3}}$$

2. **Per $k = 1$:**
   $$p_Y(1) = P(Y = 1) = P\big(\min(X, 2) = 1\big) = P(X = 1) = e^{-3} \frac{3^1}{1!} = \boxed{3e^{-3}}$$

3. **Per $k = 2$:**  
   Attenzione! Il minimo tra $X$ e $2$ vale $2$ non solo quando $X = 2$, ma **per qualsiasi valore $X \ge 2$**! Passiamo quindi all'evento complementare:
   $$p_Y(2) = P(Y = 2) = P\big(\min(X, 2) = 2\big) = P(X \ge 2) = 1 - P(X < 2)$$
   Poiché $X$ assume solo valori interi non negativi, $\{X < 2\} = \{X = 0\} \cup \{X = 1\}$, quindi:
   $$p_Y(2) = 1 - \big[ P(X = 0) + P(X = 1) \big] = 1 - e^{-3} - 3e^{-3} = \boxed{1 - 4e^{-3}}$$

*(Verifica rapida: $p_Y(0) + p_Y(1) + p_Y(2) = e^{-3} + 3e^{-3} + (1 - 4e^{-3}) = 1$ ✅).*

---

#### Punto a.2 — Funzione di Distribuzione $F_Y(x)$ e Grafico
La funzione di distribuzione $F_Y(x) = P(Y \le x)$ si ottiene cumulando i salti nei punti $0, 1, 2$:

$$\boxed{F_Y(x) = P(Y \le x) = \begin{cases}
0 & \text{se } x < 0 \\
e^{-3} & \text{se } 0 \le x < 1 \quad \big[= P(Y = 0)\big] \\
4e^{-3} & \text{se } 1 \le x < 2 \quad \big[= P(Y = 0) + P(Y = 1) = e^{-3} + 3e^{-3}\big] \\
1 & \text{se } x \ge 2 \quad \big[= P(Y = 0) + P(Y = 1) + P(Y = 2)\big]
\end{cases}}$$

**Rappresentazione grafica di $F_Y(x)$:**
```
    F_Y(x) ^
         1 |                      *------------------>
           |                      |  (salto 1 - 4e^-3)
    4e^-3  |              *-------o
           |              |  (salto 3e^-3)
     e^-3  |      *-------o
           |      |  (salto e^-3)
         0 +------o-------+-------+------------------> x
                  0       1       2
```

---

#### Punto b — Studio dell'Indipendenza tra $\{Y = 1\}$ e $\{X \le 2\}$

Dobbiamo stabilire se vale l'uguaglianza:
$$P\big(\{Y = 1\} \cap \{X \le 2\}\big) \stackrel{?}{=} P(Y = 1) \cdot P(X \le 2)$$

1. **Calcolo della probabilità dell'intersezione $P(Y = 1, \, X \le 2)$:**  
   Ricordiamo che l'evento $\{Y = 1\}$ coincide esattamente con l'evento $\{X = 1\}$.  
   Pertanto:
   $$\{Y = 1\} \cap \{X \le 2\} = \{X = 1\} \cap \{X \le 2\} = \{X = 1\}$$
   *(poiché se $X = 1$, allora è automaticamente vero che $X \le 2$, cioè $\{X = 1\} \subseteq \{X \le 2\}$).*  
   Di conseguenza:
   $$\boxed{P(Y = 1, \, X \le 2) = P(X = 1) = 3e^{-3}}$$

2. **Calcolo del prodotto delle singole probabilità $P(Y = 1) \cdot P(X \le 2)$:**  
   - Sappiamo già che $P(Y = 1) = P(X = 1) = 3e^{-3}$.
   - Calcoliamo $P(X \le 2)$:
     $$P(X \le 2) = P(X = 0) + P(X = 1) + P(X = 2) = e^{-3} + 3e^{-3} + \frac{9}{2}e^{-3} = \frac{17}{2}e^{-3}$$
   - Moltiplicando i due termini:
     $$\boxed{P(Y = 1) \cdot P(X \le 2) = 3e^{-3} \big[ P(X = 0) + P(X = 1) + P(X = 2) \big] = 3e^{-3} \left[ e^{-3} + 3e^{-3} + \frac{9}{2}e^{-3} \right]}$$

3. **Confronto e Conclusione:**  
   Poiché il fattore tra parentesi quadre $P(X \le 2) = \frac{17}{2}e^{-3} \approx 0.423 \neq 1$, abbiamo:
   $$P(Y = 1, \, X \le 2) = 3e^{-3} \neq 3e^{-3} \cdot \left(\frac{17}{2}e^{-3}\right) = P(Y = 1) \cdot P(X \le 2)$$
   Dunque **gli eventi $\{Y = 1\}$ e $\{X \le 2\}$ NON sono indipendenti!** $\blacksquare$

---

## 6. Checkpoint & Trabocchetti d'Esame

> [!danger] Trabocchetto 1: Confondere $\{Y = 2\}$ con $\{X = 2\}$ in $Y = \min(X, 2)$
> Quando una variabile viene "troncata" con il minimo $Y = \min(X, M)$, tutti i valori di $X$ maggiori o uguali a $M$ collassano sul valore $M$! Quindi:
> - Per $k < M$: $\{Y = k\} \iff \{X = k\}$
> - Per $k = M$: $\{Y = M\} \iff \{X \ge M\}$ (e **NON** $\{X = M\}$!). Tutta la coda infinita di $X$ da $M$ in poi confluisce sul punto $M$.

> [!danger] Trabocchetto 2: Calcolare l'intersezione come prodotto quando non c'è indipendenza
> Nel Punto b dell'esercizio, per calcolare $P(Y = 1, X \le 2)$ **non** devi mai moltiplicare le probabilità, ma devi tradurre entrambi gli eventi in termini della stessa variabile $X$: $\{X = 1\} \cap \{X \le 2\} = \{X = 1\}$. Solo *dopo* aver calcolato l'intersezione insiemistica la confronti col prodotto $P(Y=1)P(X \le 2)$!

---

## 🔗 Link Interni (Mappa del Grafo)

- 🏠 **Indice Generale:** [[00 - INDICE STATISTICA]]
- ⬅️ **Lezione Precedente (V.A. Geometrica):** [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]]
- ⬅️ **Origine come Limite della Binomiale $b(n, \lambda/n)$:** [[Lezione 04 - Variabili Aleatorie Discrete, Bernoulli e Binomiale]]
- ⬅️ **Funzione di Ripartizione $F_Y$ e Indipendenza di V.A.:** [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]]
- 🛠️ **Strategie d'Esame:** [[Guida Pratica agli Esercizi d'Esame e Strategie Risolutive]]
