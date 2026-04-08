## <font color="#3f3151">Lezione 5: Ponderazione (Weighting) e BM25</font>

### <font color="#5f497a">Term Frequency (tf) e Ponderazione Log-frequenza</font>

Nel modello a spazio vettoriale, la frequenza grezza di un termine all'interno di un documento, indicata come $tf_{t,d}$, rappresenta il numero di volte che il termine $t$ appare nel documento $d$. Tuttavia, utilizzare direttamente la frequenza grezza pone un problema di rilevanza: la presenza di un termine per dieci volte non rende il documento necessariamente dieci volte più rilevante rispetto a una singola occorrenza. La rilevanza non cresce linearmente con la frequenza.

Per mitigare l'impatto delle occorrenze multiple e modellare meglio l'importanza del termine, si applica una ponderazione log-frequenza (Log-frequency weighting). Il peso $w_{t,d}$ viene calcolato utilizzando il logaritmo della frequenza:
## $$w_{t,d} = \begin{cases} 1 + \log_{10}(\text{tf}_{t,d}) & \text{se } \text{tf}_{t,d} > 0 \\ 0 & \text{altrimenti} \end{cases}$$
In questo modo, il peso cresce molto più lentamente all'aumentare delle occorrenze. Ad esempio, passando da 1 a 1000 occorrenze, il peso aumenta solo da 1 a 4, riducendo drasticamente l'effetto distorsivo dei termini molto frequenti. Il punteggio totale di un documento rispetto a una query può essere calcolato sommando i pesi logaritmici dei termini che compaiono sia nella query che nel documento.
### <font color="#5f497a">Inverse Document Frequency (idf)</font>

La sola frequenza del termine nel documento non è sufficiente a determinare la rilevanza, poiché non tutti i termini hanno lo stesso contenuto informativo. I termini rari all'interno della collezione sono considerati più informativi rispetto ai termini frequenti. Ad esempio, in una collezione di documenti generici, termini comuni come "il" o "essere" appaiono in quasi tutti i documenti e hanno un potere discriminante nullo, mentre termini specifici come "aracnocentrico" appaiono raramente e caratterizzano fortemente il contenuto.

Per quantificare questo concetto si introduce la Inverse Document Frequency (idf). Definito $\text{df}_t$ come la document frequency (il numero di documenti nella collezione che contengono il termine $t$) e $N$ come il numero totale di documenti, l'$idf$ è definito come:
## $$\text{idf}_t = \log_{10}\left(\frac{N}{\text{df}_t}\right)$$
L'$idf$ di un termine raro sarà alto, mentre l'$idf$ di un termine molto frequente sarà basso (prossimo a 0 se il termine appare in tutti i documenti). È fondamentale notare che l'$idf$ è una misura globale, unica per ogni termine nell'intera collezione, e non dipende dal singolo documento.
### <font color="#5f497a">Schema TF-IDF</font>

La combinazione delle due misure precedenti genera lo schema di ponderazione tf-idf, che rappresenta uno dei metodi più diffusi nell'Information Retrieval. Questo schema assegna un peso elevato ai termini che appaiono frequentemente in un documento specifico (alto $tf$) ma raramente nel resto della collezione (alto $idf$). Il peso $w_{t,d}$ di un termine in un documento è dato dal prodotto:
## $$w_{t,d} = (1 + \log(\text{tf}_{t,d})) \times \log\left(\frac{N}{\text{df}_t}\right)$$
In un modello a spazio vettoriale, ogni documento è rappresentato come un vettore di questi pesi tf-idf. Poiché esistono diverse varianti nel calcolo di tf e idf, così come diverse strategie di normalizzazione, si utilizza spesso la notazione SMART per specificare esattamente lo schema utilizzato. La notazione è composta da tre lettere (ad esempio lnc.ltc), dove la prima terna descrive la ponderazione del documento e la seconda quella della query. Le lettere indicano rispettivamente il fattore di frequenza del termine (es. l per logaritmico), il fattore di frequenza del documento (es. t per idf o n per nullo) e il fattore di normalizzazione (es. c per coseno).
### <font color="#5f497a">Misure di Similarità: Distanza Euclidea vs Cosine Similarity</font>

Una volta rappresentati documenti e query come vettori nello spazio, è necessario quantificare la loro similarità per ordinare i risultati (ranking). Un approccio intuitivo potrebbe essere la **Distanza Euclidea**, che misura la distanza geometrica tra gli estremi dei vettori. Tuttavia, questo metodo è problematico nell'Information Retrieval perché è sensibile alla lunghezza del documento. Un documento che ripete il contenuto della query molte volte avrà un vettore molto lungo (grande magnitudine) e potrebbe risultare "distante" da una query breve, anche se il contenuto è altamente rilevante. La Distanza Euclidea penalizza i vettori lunghi a causa della grande distanza spaziale dagli altri punti.

Per ovviare a questo problema, si utilizza la Cosine Similarity. Questa misura calcola il coseno dell'angolo tra il vettore della query $\vec{q}$ e il vettore del documento $\vec{d}$.
## $$\text{sim}(\vec{d}, \vec{q}) = \frac{\vec{d} \cdot \vec{q}}{|\vec{d}| |\vec{q}|}$$
Il denominatore di questa formula normalizza i vettori, rendendoli vettori unitari. Ciò significa che la lunghezza del documento non influenza più il punteggio di similarità; conta solo l'orientamento del vettore, ovvero la distribuzione relativa dei termini. Un documento lungo e uno breve con la stessa distribuzione di termini avranno vettori con lo stesso orientamento e quindi una cosine similarity pari a 1 (angolo 0).
### <font color="#5f497a">Best Match 25 (BM25)</font>

Il **BM25** (Best Match 25) è una funzione di ranking basata sul modello probabilistico, considerata lo stato dell'arte per i metodi basati sulla frequenza dei termini prima dell'avvento dei modelli neurali. Il BM25 affronta e risolve alcune limitazioni del classico tf-idf, introducendo concetti come la saturazione della frequenza e la normalizzazione della lunghezza.

La **saturazione della frequenza** nel BM25 impedisce che il peso di un termine cresca indefinitamente all'aumentare del $tf$. Mentre nel tf-idf il peso logaritmico continua a crescere, nel BM25 l'influenza del termine si avvicina asintoticamente a un valore massimo controllato dal parametro $k_1$. Se un termine è già molto frequente, aggiungerne ulteriori occorrenze ha un impatto minimo sul punteggio.

La **normalizzazione della lunghezza** è gestita in modo più sofisticato rispetto alla semplice normalizzazione coseno. I documenti lunghi non sono penalizzati indiscriminatamente (potrebbero essere lunghi perché ricchi di contenuti diversi), ma vengono confrontati con la lunghezza media dei documenti nella collezione ($avgdl$). Il parametro $b$ controlla l'intensità di questa normalizzazione: se $b=0$ non c'è normalizzazione, se $b=1$ la normalizzazione è completa.

La formula per calcolare il punteggio (Retrieval Status Value - RSV) di un documento $d$ rispetto a una query $Q$ è:
## $$\text{RSV}_d = \sum_{t \in Q} \text{IDF}_t \cdot \frac{\text{tf}_{t,d} \cdot (k_1 + 1)}{\text{tf}_{t,d} + k_1 \cdot (1 - b + b \cdot \frac{|d|}{avgdl})}$$
Dove $k_1$ è tipicamente impostato tra 1.2 e 2.0, e $b$ è solitamente impostato a 0.75.

### <font color="#5f497a">Il Dataset (Toy Example)</font>

Immaginiamo una collezione di 5 documenti e 1 query.

Vocabolario ridotto: {gatto, cane, topo}.

- **D1:** "il gatto insegue il cane"
    
- **D2:** "gatto gatto gatto"
    
- **D3:** "il cane dorme"
    
- **D4:** "il topo mangia"
    
- **D5:** "il gatto e il topo"

**Query ($q$):** "gatto cane"

---

### <font color="#5f497a">1. Binary Term-Document Incidence Matrix</font>

Questa matrice rappresenta semplicemente la **presenza (1) o assenza (0)** di un termine nel documento. Ignora quante volte il termine appare.

|**Termine**|**D1**|**D2**|**D3**|**D4**|**D5**|
|---|---|---|---|---|---|
|**gatto**|1|1|0|0|1|
|**cane**|1|0|1|0|0|
|**topo**|0|0|0|1|1|

A cosa serve: È la base del Boolean Retrieval.

Concetto chiave: Risponde alla domanda "Il documento contiene il termine?". Non permette di ordinare i risultati per rilevanza (ranking): D1 e D2 sono identici per il termine "gatto", anche se D2 ne parla molto di più.

---

### <font color="#5f497a">2. Term-Document Count Matrix</font>

Qui contiamo le **occorrenze grezze** ($tf$, raw term frequency). Ogni documento è un vettore di conteggi ($\mathbb{N}^v$).

|**Termine**|**D1**|**D2**|**D3**|**D4**|**D5**|
|---|---|---|---|---|---|
|**gatto**|1|3|0|0|1|
|**cane**|1|0|1|0|0|
|**topo**|0|0|0|1|1|

A cosa serve: Introduce il concetto di Bag of Words.

Concetto chiave: Ora possiamo distinguere D2 da D1. D2 sembra molto più focalizzato sui gatti. Tuttavia, la frequenza grezza ha un problema: un documento che ripete una parola 10 volte non è necessariamente 10 volte più rilevante .

---

### <font color="#5f497a">3. Term-Document Frequency Matrix (Log-Weighted)</font>

Spesso per "Frequency Matrix" in IR si intende la matrice dei pesi **TF** normalizzati o logaritmici, pronti per essere usati nel calcolo finale. Usiamo la **Log-frequency weighting** definita nel corso come $w_{t,d} = 1 + \log_{10}(tf_{t,d})$ se $tf > 0$, altrimenti $0$.

_Calcoli:_

- D2 (gatto): $tf=3 \rightarrow 1 + \log_{10}(3) \approx 1.477$
    
- D1 (gatto): $tf=1 \rightarrow 1 + \log_{10}(1) = 1$

|**Termine**|**D1**|**D2**|**D3**|**D4**|**D5**|
|---|---|---|---|---|---|
|**gatto**|1.0|1.48|0|0|1.0|
|**cane**|1.0|0|1.0|0|0|
|**topo**|0|0|0|1.0|1.0|

**A cosa serve:** "Schiaccia" le frequenze alte per evitare che la ripetizione ossessiva di una parola domini il punteggio (saturazione).

---

### <font color="#5f497a">4. TF-IDF: Caratteristiche e Calcolo</font>

Il TF-IDF combina due intuizioni:

1. **TF (Term Frequency):** Premia i termini frequenti nel documento (locale).
    
2. **IDF (Inverse Document Frequency):** Penalizza i termini che appaiono in troppi documenti (globale), considerandoli poco informativi (es. "il", "e", o parole molto comuni nel contesto specifico).

Formula IDF: $IDF_t = \log_{10}(N / df_t)$.

Dove $N$ è il numero totale di documenti ($N=5$) e $df_t$ è il numero di documenti che contengono il termine $t$.

#### Calcolo dell'IDF sul nostro esempio:

- gatto: appare in D1, D2, D5 ($df=3$).
    $$IDF_{gatto} = \log_{10}(5/3) = \log_{10}(1.66) \approx \mathbf{0.22}$$
    
- cane: appare in D1, D3 ($df=2$).
    $$IDF_{cane} = \log_{10}(5/2) = \log_{10}(2.5) \approx \mathbf{0.40}$$
    
- _(Nota: "cane" è più raro di "gatto", quindi pesa quasi il doppio)._
#### Calcolo del punteggio TF-IDF ($TF \times IDF$):

Calcoliamo lo score per la query **"gatto cane"** sui documenti rilevanti (D1 e D2).

**Documento D1:**

- **gatto:** $TF_{log}(1) \times IDF(0.22) = 1.0 \times 0.22 = 0.22$
    
- **cane:** $TF_{log}(1) \times IDF(0.40) = 1.0 \times 0.40 = 0.40$
    
- **Score Totale D1:** $0.22 + 0.40 = \mathbf{0.62}$

**Documento D2:**

- **gatto:** $TF_{log}(1.48) \times IDF(0.22) = 1.48 \times 0.22 \approx 0.33$
    
- **cane:** $0 \times 0.40 = 0$
    
- **Score Totale D2:** $0.33 + 0 = \mathbf{0.33}$
### Conclusione Teorica

In questo esempio, D1 vince su D2 (0.62 vs 0.33).

Anche se D2 parla molto di "gatti", D1 menziona entrambi i termini della query. Inoltre, il termine "cane" (più raro nella collezione) ha spinto in alto il punteggio di D1 grazie all'IDF. Questo dimostra come il TF-IDF bilanci la frequenza locale con la rarità globale.