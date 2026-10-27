---
tags: [statistica, probabilita, lezione-04, variabili-discrete, bernoulli, binomiale, overbooking, bayes]
lezione: 4
titolo: "Variabili Aleatorie Discrete, Distribuzioni di Bernoulli e Binomiale"
docente: "Prof. Vito Onofrio Crismale"
---

# Lezione 04 - Variabili Aleatorie Discrete, Distribuzioni di Bernoulli e Binomiale

> ⬅️ **Precedente:** [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]] | 🏠 **Indice:** [[00 - INDICE STATISTICA]] | **Successiva:** [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]] ➡️ | 🛠️ **Pratica:** [[Guida Pratica agli Esercizi d'Esame e Strategie Risolutive]]

---
**Docente:** Vito Onofrio Crismale (`vitoonofrio.crismale@uniba.it`)  
**Testi di riferimento:**
- Ross, *Statistica per l'Ingegneria e le Scienze*
- Baldi, *Calcolo delle Probabilità e Statistica*

---

## 1. Variabili Aleatorie Discrete e Funzione di Probabilità

> [!abstract] L'Idea in Parole Semplici
> Una variabile aleatoria è **discreta** quando i valori numerici che può assumere sono "isolati" e si possono elencare uno per uno (sono in numero finito, come $\{0, 1, \dots, n\}$, oppure infinito numerabile, come tutti i numeri naturali $\{0, 1, 2, 3, \dots\}$). In questo caso, per conoscere tutta la distribuzione della variabile basta conoscere la probabilità di ciascun singolo valore!

### Definizione di Variabile Aleatoria Discreta

Una variabile aleatoria $X: \Omega \longrightarrow \mathbb{R}$ è detta **discreta** se il suo **codominio** (o supporto), indicato con $\text{cod}(X) = X(\Omega)$, è un insieme **finito** o **infinito numerabile**.

Per ogni valore $t \in \text{cod}(X)$, l'insieme:
$$\{X = t\} = \{\omega \in \Omega \mid X(\omega) = t\} \in \mathcal{F}$$
è un evento ben definito, e dunque ha senso calcolare la probabilità $P(X = t)$.

---

### Definizione di Funzione di Probabilità (fp)

Data una v.a. discreta $X$, possiamo definire l'applicazione:
$$p_X: \text{cod}(X) \longrightarrow [0, 1]$$
tale che per ogni $t \in \text{cod}(X)$:
$$\boxed{p_X(t) := P(X = t)}$$
La funzione $p_X$ è detta **funzione di probabilità** (abbreviata in **fp**, o anche *densità discreta* / *massa di probabilità*) di $X$.

---

### Proprietà Fondamentali della Funzione di Probabilità

1. **Condizione di Normalizzazione:**  
   La somma delle probabilità su tutti i possibili valori del codominio vale sempre $1$:
   $$\boxed{\sum_{t \in \text{cod}(X)} p_X(t) = 1}$$

2. **La fp determina univocamente la Legge $P_X$:**  
   > [!note] Osservazione Teorica
   > Conoscere la funzione di probabilità $p_X$ di una v.a. discreta $X$ **equivale** a conoscere l'intera sua legge $P_X$.  
   > Infatti, per qualunque sottoinsieme $B \subseteq \mathbb{R}$, l'evento $\{X \in B\}$ si scrive come unione disgiunta numerabile dei singoletti $\{X = t\}$ al variare di $t \in B \cap \text{cod}(X)$:
   > $$P_X(B) = P(X \in B) = P\left( \bigcup_{t \in B \cap \text{cod}(X)} \{X = t\} \right) = \sum_{t \in B \cap \text{cod}(X)} P(X = t) = \sum_{t \in B \cap \text{cod}(X)} p_X(t)$$
   > **Conclusione operativa:** Per individuare o assegnare la legge di una variabile aleatoria discreta $X$, **basta individuare il suo codominio $\text{cod}(X)$ e la sua funzione di probabilità $p_X(t)$**!

3. **Andamento della Funzione di Distribuzione $F_X(t)$ per v.a. discrete:**  
   Per qualsiasi variabile aleatoria discreta $X$, la funzione di distribuzione $F_X(t) = P(X \le t) = \sum_{x \le t} p_X(x)$ è sempre una **funzione costante a tratti (a gradini)**, che salta esattamente nei punti $t \in \text{cod}(X)$ con ampiezza del salto pari a $p_X(t) = P(X = t)$.

---

## 2. Variabile Aleatoria di Bernoulli: $X \sim \text{Ber}(p)$

> [!abstract] Il Mattone Fondamentale del Caso Discreto
> La variabile aleatoria di Bernoulli modella l'esperimento casuale più semplice in assoluto: una singola prova dicotomica che può concludersi solo con un **Successo ($1$)** oppure un **Insuccesso ($0$)**.

### Definizione

Una variabile aleatoria $X$ si dice **Bernoulliana** se assume **solo i valori $0$ e $1$**, ovvero $\text{cod}(X) = \{0, 1\}$.

In altre parole, fissato un evento $E \in \mathcal{F}$, $X$ è la variabile aleatoria che vale $1$ se l'evento $E$ si è verificato, e $0$ altrimenti. Formalmente coincide con la **funzione indicatrice** dell'evento $E$:
$$\boxed{X = \mathbf{1}_E \qquad \text{dove} \qquad \mathbf{1}_E(\omega) := \begin{cases} 1 & \text{se } \omega \in E \\ 0 & \text{se } \omega \in E^c \end{cases}}$$

### Parametro e Funzione di Probabilità
Si dice **parametro $p$** (con $0 < p < 1$) della v.a. Bernoulliana la quantità:
$$p := P(E) = P(X = 1)$$
La sua funzione di probabilità è quindi:
$$\boxed{\begin{cases} P(X = 1) = p \\ P(X = 0) = 1 - p \end{cases}}$$
*(Notazione compatta: $X \sim \text{Ber}(p)$ oppure $X \sim b(1, p)$).*

#### Esempio: Lancio di una moneta equilibrata
Lancio una moneta e considero la v.a. $X$ che vale $1$ se esce **Testa** e $0$ se esce **Croce**.  
Questa variabile aleatoria è **Bernoulliana con parametro $p = \frac{1}{2}$**.

---

### Funzione di Distribuzione $F_X(t)$ di una Bernoulliana

Ricordiamo che la funzione di distribuzione di $X \sim \text{Ber}(p)$ è data da $F_X(t) = P(X \le t)$:

$$F_X(t) = \begin{cases}
0 & \text{se } t < 0 \\
1 - p & \text{se } 0 \le t < 1 \\
1 & \text{se } t \ge 1
\end{cases}$$

Rappresentazione grafica di $F_X(t)$:
```
    F_X(t) ^
         1 |              *----------------->
           |              |  (salto di ampiezza p in t = 1)
     1 - p |       *------o
           |       |  (salto di ampiezza 1-p in t = 0)
         0 +-------o------+-----------------> t
                   0      1
```

> [!info] Osservazione Grafica
> Come si vede dal grafico, $F_X(t)$ è una **funzione costante a tratti** (continua da destra). Il fatto che $F_X$ sia costante a tratti vale per **ogni variabile aleatoria discreta** $\forall X$ discreta!

---

## 3. Variabile Aleatoria Binomiale: $X \sim \text{Bin}(n, p)$

> [!abstract] Dal Singolo Tentativo a $n$ Prove Ripetute
> Cosa succede se ripetiamo $n$ volte, in modo **indipendente** e nelle stesse condizioni, un esperimento di tipo successo/insuccesso (Bernoulli di parametro $p$) e contiamo il **numero totale di successi** ottenuti? Otteniamo una variabile aleatoria **Binomiale**!

### Definizione

Una variabile aleatoria $X$ si dice **Binomiale** di parametri $n \in \mathbb{N} \setminus \{0\}$ e $p$ (con $0 < p < 1$), e si scrive $X \sim b(n, p)$ oppure $X \sim \text{Bin}(n, p)$, se:
1. $X$ assume valori interi tra $0$ ed $n$:
   $$\text{cod}(X) = \{0, 1, 2, \dots, n\}$$
2. La sua funzione di probabilità è data, per ogni $k \in \{0, 1, \dots, n\}$, da:
   $$\boxed{P(X = k) = \binom{n}{k} p^k (1 - p)^{n - k}}$$
   dove $\binom{n}{k} = \frac{n!}{k!(n-k)!}$ è il coefficiente binomiale.

*(Nota: per $n = 1$, la distribuzione Binomiale $b(1, p)$ coincide esattamente con la distribuzione di Bernoulli $\text{Ber}(p)$).*

---

### Legame Strutturale tra Bernoulli e Binomiale (Somma di v.a. Indipendenti)

> [!important] Osservazione 1: Somma di Bernoulliane Indipendenti
> Siano $X_1, X_2, \dots, X_n$ variabili aleatorie **Bernoulliane indipendenti** tutte di parametro $p$ ($X_i \sim b(1, p)$).  
> Allora la variabile aleatoria somma:
> $$\boxed{X = X_1 + X_2 + \dots + X_n = \sum_{i=1}^n X_i \sim b(n, p)}$$
> è una v.a. **Binomiale di parametri $n$ e $p$**: essa conta esattamente il numero di successi nella ripetizione $n$ volte indipendenti di un fenomeno aleatorio di tipo successo/insuccesso.
> 
> **Viceversa:** Se $X \sim b(n, p)$, allora esistono $X_1, \dots, X_n \sim b(1, p)$ indipendenti tali che $X = \sum_{i=1}^n X_i$.

> [!important] Osservazione 2: Somma di Binomiali Indipendenti (Stesso Parametro $p$)
> Siano $X \sim b(n_1, p)$ e $Y \sim b(n_2, p)$ due variabili aleatorie Binomiali **indipendenti** aventi **lo stesso parametro di successo $p$**. Allora:
> $$\boxed{X + Y \sim b(n_1 + n_2, \, p)}$$
> **Perché?**  
> Perché $X$ è la somma di $n_1$ Bernoulliane indipendenti di parametro $p$, e $Y$ è la somma di $n_2$ Bernoulliane indipendenti di parametro $p$; essendo $X$ e $Y$ indipendenti tra loro, $X + Y$ è la somma complessiva di $n_1 + n_2$ Bernoulliane indipendenti di parametro $p$!

---

## 4. Esercizi Svolti a Lezione sulla Binomiale

### Esercizio 1: Lancio di una moneta 10 volte

**Testo:**  
Lancio una moneta equilibrata $10$ volte. Qual è la probabilità di ottenere **esattamente 4 teste**?

**Svolgimento rigoroso tramite decomposizione in Bernoulliane:**
1. Per ogni lancio $i \in \{1, 2, \dots, 10\}$, definiamo la variabile aleatoria:
   $$X_i = \begin{cases} 1 & \text{se all'}i\text{-esimo lancio ho Testa} \\ 0 & \text{altrimenti (Croce)} \end{cases}$$
2. Le variabili $X_i$ sono v.a. **Bernoulliane** di parametro $p = \frac{1}{2}$ e sono tra loro **indipendenti** (i lanci non si influenzano).
3. Consideriamo la variabile somma:
   $$X = X_1 + X_2 + \dots + X_{10}$$
   $X$ conta il numero totale di teste in $10$ lanci. Per la proprietà vista, $X$ è una v.a. **Binomiale** di parametri $n = 10$ e $p = \frac{1}{2}$:
   $$X \sim b\left(10, \, \frac{1}{2}\right)$$
4. Calcoliamo la probabilità richiesta:
   $$\boxed{P(X = 4) = \binom{10}{4} \left(\frac{1}{2}\right)^4 \left(\frac{1}{2}\right)^6 = \binom{10}{4} \left(\frac{1}{2}\right)^{10} = \frac{210}{1024} = \frac{105}{512} \approx 0.2051}$$

---

### Esercizio 2: Overbooking Aereo — Caso 1 (Aereo da 10 posti, 11 biglietti venduti)

**Testo:**  
Un piccolo aereo ha $10$ posti disponibili, ma la compagnia vende $11$ biglietti sapendo che ogni passeggero si presenta effettivamente all'imbarco con probabilità $p = 0.9$. Qual è la probabilità che si verifichi overbooking (cioè che si presentino tutte e 11 le persone)?

**Svolgimento:**
1. Per ogni passeggero prenotato $i \in \{1, \dots, 11\}$, sia:
   $$X_i = \begin{cases} 1 & \text{se la persona } i\text{-esima si presenta all'imbarco} \\ 0 & \text{altrimenti} \end{cases}$$
2. Assumiamo per semplicità che le $X_i$ siano **indipendenti** (ciascun passeggero viaggia per conto proprio). Ogni $X_i$ è una v.a. Bernoulliana di parametro $p = 0.9$.
3. Sia:
   $$X = X_1 + \dots + X_{11}$$
   la v.a. che conta il numero totale di persone che si presentano all'imbarco.  
   Dato che $X$ è somma di $11$ Bernoulliane indipendenti, $X$ è una v.a. **Binomiale di parametri $n = 11$ e $p = 0.9$**:
   $$X \sim b(11, \, 0.9)$$
4. L'overbooking si verifica se si presentano più di $10$ persone, ovvero esattamente $11$:
   $$\boxed{P(X = 11) = \binom{11}{11} (0.9)^{11} (0.1)^0 = (0.9)^{11} \approx 0.3138 \quad (31.38\%)}$$

---

### Esercizio 3: Overbooking Aereo — Caso 2 (Aereo da 20 posti, 22 biglietti venduti)

**Testo:**  
Consideriamo ora un aereo da $20$ posti per il quale vengono venduti $22$ biglietti, sempre con probabilità individuale di presenza $p = 0.9$ e indipendenza tra i passeggeri. Qual è la probabilità di andare in overbooking?

**Svolgimento:**
1. Siano $Y_i$ con $i \in \{1, \dots, 22\}$ le Bernoulliane che indicano la presenza dell'$i$-esimo passeggero.
2. La variabile somma:
   $$Y = Y_1 + \dots + Y_{22} \sim b(22, \, 0.9)$$
   è **Binomiale di parametri $n = 22$ e $p = 0.9$**.
3. Si ha overbooking quando il numero di passeggeri presenti supera i $20$ posti disponibili ($Y > 20$), cioè quando se ne presentano $21$ oppure $22$:
   $$P(Y > 20) = P(Y = 21) + P(Y = 22)$$
   $$\boxed{P(Y > 20) = \binom{22}{21} (0.9)^{21} (0.1)^1 + \binom{22}{22} (0.9)^{22} = 22 \cdot (0.9)^{21} \cdot (0.1) + (0.9)^{22} \approx 0.3392}$$

---

### Esercizio 4: Richiesta di Connessioni da 24 Operatori

**Testo:**  
In un ufficio ci sono $24$ operatori. In un dato istante, ciascun operatore richiede la connessione alla rete con probabilità $p = 0.6$, in modo indipendente dagli altri. Qual è la probabilità che almeno $20$ operatori richiedano la connessione contemporaneamente?

**Svolgimento:**
1. Per ogni $i \in \{1, \dots, 24\}$, sia $X_i$ la v.a. che vale $1$ se l'operatore $i$-esimo ha richiesto la connessione e $0$ altrimenti.
2. Assumiamo che le $X_i$ siano **indipendenti**; ciascuna $X_i$ è Bernoulliana di parametro $p = 0.6$.
3. La variabile totale:
   $$X = X_1 + \dots + X_{24} \sim b(24, \, 0.6)$$
   è una v.a. **Binomiale** di parametri $n = 24$ e $p = 0.6$.
4. La probabilità che ci siano almeno $20$ operatori connessi è:
   $$\boxed{P(X \ge 20) = \sum_{i=20}^{24} P(X = i) = \sum_{i=20}^{24} \binom{24}{i} (0.6)^i (0.4)^{24 - i}}$$

---

### Esercizio 5 ( Fondamentale d'Esame!): Scatola di Componenti da Due Linee con Indipendenza Condizionata e Bayes

**Testo:**  
Una fabbrica produce componenti su due linee: **Linea A** (che produce il $30\%$ dei lotti) e **Linea B** (che produce il restante $70\%$).
- I componenti prodotti dalla Linea A sono difettosi con probabilità $0.1$.
- I componenti prodotti dalla Linea B sono difettosi con probabilità $0.17$.

I componenti vengono confezionati in scatole da $10$ pezzi ciascuna (tutti i $10$ pezzi di una stessa scatola provengono dalla stessa linea).  
Si prende una scatola a caso, la si controlla e si trova **esattamente $1$ componente difettoso**. Qual è la probabilità che la scatola provenga dalla Linea A?

#### Svolgimento Dettagliato e Analisi Concettuale:

1. **Formalizzazione per un singolo componente:**
   - $A = \text{"Il componente/scatola proviene dalla Linea A"}$ $\implies P(A) = 0.3$
   - $A^c = B = \text{"Il componente/scatola proviene dalla Linea B"}$ $\implies P(A^c) = 0.7$
   - $D = \text{"Il componente è difettoso"}$
     - $P(D \mid A) = 0.1$
     - $P(D \mid A^c) = 0.17$
   *(Se prendessimo un singolo componente isolato, la probabilità che sia difettoso sarebbe $P(D) = P(D \mid A)P(A) + P(D \mid A^c)P(A^c)$).*

2. **Analisi della scatola da 10 pezzi (Attenzione all'Indipendenza!):**
   Per $i \in \{1, \dots, 10\}$, sia $X_i$ la v.a. che vale $1$ se l'$i$-esimo componente della scatola è difettoso, e $0$ altrimenti.  
   Sia:
   $$X = X_1 + \dots + X_{10}$$
   la v.a. che registra il **numero totale di componenti difettosi** nella scatola da $10$ pezzi.

   > [!danger] Osservazione Cruciale del Prof (Scatola nel Riquadro Tratteggiato)
   > **Le variabili $X_1, \dots, X_{10}$ NON SONO INDIPENDENTI in senso assoluto!**  
   > Perché? Perché tutti i 10 pezzi stanno nella stessa scatola: se scoprissi che i primi 9 pezzi sono tutti difettosi, diventerebbe molto più probabile che la scatola provenga dalla Linea B (più difettosa), modificando la probabilità che anche il 10° pezzo sia difettoso!  
   > **PERÒ**, **sapendo che la scatola viene dalla Linea A** (cioè condizionatamente ad $A$), le variabili $X_i$ **sono indipendenti** e $X$ segue una distribuzione **Binomiale**!

3. **Leggi Condizionate di $X$:**
   - **Sapendo che la scatola viene dalla Linea A:**  
     $$X \mid A \sim b(10, \, 0.1)$$
     Quindi la probabilità di trovare esattamente $1$ pezzo difettoso sapendo che viene da $A$ è:
     $$P(X = 1 \mid A) = \binom{10}{1} (0.1)^1 (0.9)^9 = 10 \cdot (0.1) \cdot (0.9)^9 = (0.9)^9$$
   - **Sapendo che la scatola viene dalla Linea B ($A^c$):**  
     $$X \mid A^c \sim b(10, \, 0.17)$$
     Quindi la probabilità di trovare esattamente $1$ pezzo difettoso sapendo che viene da $B$ è:
     $$P(X = 1 \mid A^c) = \binom{10}{1} (0.17)^1 (0.83)^9$$

4. **Applicazione del Teorema di Bayes:**
   Vogliamo calcolare $P(A \mid X = 1)$:
   $$P(A \mid X = 1) = \frac{P(X = 1 \mid A) P(A)}{P(X = 1)}$$
   dove, per la Formula delle Probabilità Totali:
   $$P(X = 1) = P(X = 1 \mid A) P(A) + P(X = 1 \mid A^c) P(A^c)$$
   Sostituendo i valori:
   $$\boxed{P(A \mid X = 1) = \frac{\binom{10}{1} (0.1)^1 (0.9)^9 \cdot 0.3}{\binom{10}{1} (0.1)^1 (0.9)^9 \cdot 0.3 + \binom{10}{1} (0.17)^1 (0.83)^9 \cdot 0.7} \approx 0.3426 \quad (34.26\%)}$$

---

## 5. Checkpoint & Trabocchetti d'Esame

> [!danger] Trabocchetto 1: Sommare Binomiali con parametro $p$ diverso
> La proprietà $X + Y \sim b(n_1 + n_2, p)$ vale **SOLO SE**:
> 1. $X$ e $Y$ sono **indipendenti**, E
> 2. Hanno **lo stesso identico parametro $p$**!  
> Se $X \sim b(n_1, p_1)$ e $Y \sim b(n_2, p_2)$ con $p_1 \neq p_2$, la loro somma **NON** è una variabile Binomiale!

> [!danger] Trabocchetto 2: Dimenticare il caso $k = 0$
> Il supporto di una Binomiale $X \sim b(n, p)$ parte sempre da **$0$** ($\text{cod}(X) = \{0, 1, \dots, n\}$), e $P(X = 0) = (1 - p)^n > 0$. Quando calcoli il complementare di $\{X \ge 2\}$, ricorda che $\{X < 2\} = \{X = 0\} \cup \{X = 1\}$: non dimenticare mai il termine $P(X = 0)$!

---

## 🔗 Link Interni (Mappa del Grafo)

- 🏠 **Indice Generale:** [[00 - INDICE STATISTICA]]
- ⬅️ **Lezione Precedente (Prove di Bernoulli, Variabili Aleatorie e FdR $F_X$):** [[Lezione 03 - Prove di Bernoulli, Variabili Aleatorie e Funzione di Ripartizione]]
- ⬅️ **Prerequisiti (Probabilità Condizionata e Teorema di Bayes):** [[Lezione 02 - Probabilità Condizionata, Teorema di Bayes ed Eventi Indipendenti]]
- ➡️ **Lezione Successiva (V.A. Geometrica e Mancanza di Memoria):** [[Lezione 05 - Variabile Aleatoria Geometrica e Mancanza di Memoria]]
- 🔗 **Limite della Binomiale per $n \to \infty, p \to 0$ (Poisson):** [[Lezione 06 - Variabile Aleatoria di Poisson e Legge dei Casi Rari]]
- 🛠️ **Strategie d'Esame:** [[Guida Pratica agli Esercizi d'Esame e Strategie Risolutive]]
