## Question #1: Describe a general learning-to-rank framework used in modern Web search. What are the characteristics of the three main families of algorithms in learning to rank? Please describe, in more technical terms, one of the three families (the one you like the most). 

Il framework generale del **Learning-to-Rank (LtR)** si configura come un sistema di apprendimento supervisionato il cui obiettivo è costruire un modello (ranking function) capace di ordinare un insieme di documenti in base alla loro rilevanza rispetto a una query.
![[Pasted image 20260130181442.png]]
Le diverse strategie di LtR si classificano in base a come formulano la funzione di perdita e l'input considerato durante l'addestramento:

1. **Approccio Pointwise:** È l'approccio più semplice e tratta ogni singolo documento in modo indipendente. Il problema del ranking viene ridotto a un classico problema di regressione (predire lo score esatto di rilevanza) o classificazione (predire la classe di rilevanza). La loss function calcola l'errore sul singolo documento (es. Mean Squared Error) ignorando completamente la posizione relativa degli altri documenti nella lista.
    
2. **Approccio Pairwise:** Sposta il focus sulle relazioni relative tra coppie di documenti $(d_i, d_j)$ per la stessa query. L'obiettivo non è predire il punteggio esatto, ma minimizzare il numero di **inversioni** nella classifica. Il modello cerca di apprendere una funzione tale che, se il documento $d_i$ è più rilevante di $d_j$, allora $f(d_i) > f(d_j)$.
    
3. **Approccio Listwise:** Opera sull'intera lista di documenti associata a una query simultaneamente. Cerca di ottimizzare direttamente (o tramite proxy differenziabili) le metriche di valutazione del ranking come NDCG o MAP. Questo metodo è il più complesso ma teoricamente il più accurato, poiché cattura direttamente la natura posizionale del problema, dando più peso agli errori nelle posizioni alte della classifica.


## Question #2: Detail and explain a general architecture of a query processor employing representation-based methods. Describe the main differences between dense single-vector, dense multi-vector, and sparse representations. What are the pros and cons of the three approaches? 

L'architettura generale di un processore di query basato su **Representation-Based Methods** (spesso riferiti come _Bi-Encoders_ o _Dual Encoders_) si fonda sul principio del disaccoppiamento tra l'elaborazione del documento e quella della query.

In questo schema, l'obiettivo è mappare sia le query che i documenti in uno spazio vettoriale comune (embedding space) dove la rilevanza semantica è misurata tramite una funzione di similarità geometrica, tipicamente il prodotto scalare o la similarità del coseno.
In questa architettura ci sono due fasi, una offline dove vengono processati tutti i documenti e una online dove a tempo di query la query appunto viene trasformata con lo stesso encoder e poi vengono recuperati i documenti che massimizzano la similaritá con quanto ottenuto dalla query.
#### Dense Single-Vector Representation

In questo approccio (es. DPR, ANCE), l'intero contenuto semantico di un documento viene compresso in un **unico vettore denso** di dimensione fissa (ad esempio, 768 dimensioni). Solitamente si utilizza l'embedding del token speciale `[CLS]` o il mean pooling degli output dell'ultimo layer del Transformer.

- **Pro:** È la soluzione più efficiente in termini di memoria e latenza. Poiché ogni documento è un singolo punto nello spazio, la ricerca (Nearest Neighbor Search) è rapidissima e richiede poco spazio di archiviazione per l'indice.
    
- **Contro:** Soffre del cosiddetto **Information Bottleneck**. Comprimere un lungo passaggio in un singolo vettore comporta inevitabilmente una perdita di informazioni e sfumature, rendendo difficile catturare matching precisi tra termini specifici o gestire documenti multi-argomento.
#### Dense Multi-Vector Representation

Questa famiglia (resa celebre da **ColBERT**) rappresenta il documento non come un punto, ma come una **matrice di vettori** (o un "bag of embeddings"), mantenendo un embedding distinto per ogni token del documento. La rilevanza viene calcolata tramite un meccanismo di _Late Interaction_, come la somma delle similarità massime (MaxSim) tra ogni vettore della query e i vettori del documento.

- **Pro:** Offre la massima efficacia (accuracy) tra i metodi representation-based. Mantenendo i vettori a livello di token, preserva una granularità fine che permette di modellare il matching esatto dei termini e le relazioni sintattiche complesse, avvicinandosi alle prestazioni dei costosi Cross-Encoders.
    
- **Contro:** I costi di spazio sono proibitivi per collezioni massive. Memorizzare centinaia di vettori float per ogni documento porta a un'esplosione delle dimensioni dell'indice (spesso due ordini di grandezza superiore al single-vector). Anche la latenza di ricerca è superiore dovendo caricare e confrontare matrici invece di singoli vettori.
#### Sparse Learned Representations

Questi metodi (es. **SPLADE**) proiettano il testo nello spazio del vocabolario (es. 30.000 dimensioni), simile al Bag-of-Words tradizionale, ma i pesi dei termini sono appresi e non statistici (come in TF-IDF). Il modello impara sia a pesare i termini presenti, sia a effettuare **espansione semantica**, assegnando pesi non nulli a termini non presenti nel testo originale ma semanticamente correlati.

- **Pro:** Unisce i vantaggi del neurale con l'efficienza delle strutture dati classiche. È **interpretabile** (possiamo leggere quali termini il modello ha "acceso"), gestisce nativamente il mismatch lessicale tramite l'espansione e utilizza l'Indice Invertito, che è una tecnologia matura e altamente ottimizzata. Ha dimostrato eccellenti capacità di generalizzazione (Zero-Shot).
    
- **Contro:** Sebbene l'indice sia sparso, l'espansione dei termini può rendere le posting list molto più lunghe rispetto a un indice BM25 standard, impattando sulla velocità se non si applicano tecniche di _pruning_ aggressive. Inoltre, non cattura relazioni posizionali complesse come i modelli multi-vettore densi.

## Question #3: Describe why computing the optimal node-split in decision tree computationally expensive. What are the main bottlenecks? Then discuss at least tree possible approaches to improve its efficiency. 

Il calcolo dello split ottimale in un albero decisionale (o in un ensemble come GBRT) è l'operazione più onerosa dell'intera fase di addestramento perché richiede una ricerca esaustiva "greedy". Per ogni nodo che deve essere creato, l'algoritmo deve identificare la coppia $(f, t)$ — dove $f$ è una feature e $t$ è una soglia (threshold) — che massimizza il guadagno informativo (information gain) o minimizza la varianza dell'errore.

In termini formali, se abbiamo un dataset di $N$ istanze e $M$ feature, l'algoritmo standard (pre-sorted algorithm) richiede di scansionare tutte le $M$ feature. Per ciascuna feature, è necessario ordinare le $N$ istanze in base al valore della feature stessa per poter valutare in modo efficiente tutti i possibili $N-1$ punti di split candidati.

Il costo computazionale è dominato dall'operazione di ordinamento, che ha una complessità di $O(M \cdot N \log N)$ per ogni livello dell'albero. Anche se i dati vengono pre-ordinati, il costo rimane elevato a causa della gestione degli indici e della frammentazione dei dati man mano che si scende in profondità nell'albero.

### Principali Colli di Bottiglia (Bottlenecks)

1. **Ordinamento (Sorting Cost):** Come accennato, l'ordinamento dei valori delle feature è il collo di bottiglia algoritmico principale. Dover riordinare (o mantenere ordinati) i dati a ogni nodo o partizione consuma enormi quantità di cicli CPU.
    
2. **Accesso alla Memoria e Cache Miss:** Le feature sono spesso memorizzate in colonne (columnar storage) per facilitare la scansione, ma l'accesso ai gradienti (necessari per calcolare il guadagno dello split) avviene spesso in modo casuale o riga per riga. Quando l'algoritmo scansiona una feature ordinata, deve accedere ai gradienti corrispondenti agli indici dei documenti; se questi non sono contigui in memoria, si verificano frequenti **cache miss**, riducendo drasticamente il throughput della CPU.
    
3. **Data Fragmentation:** Man mano che l'albero cresce e i nodi si dividono, il numero di istanze in ogni nodo foglia diminuisce. Questo causa una frammentazione dei dati in memoria che rende difficile sfruttare il parallelismo delle moderne CPU (SIMD) e riduce l'efficienza del pre-fetching dei dati.
### Approcci per migliorare l'efficienza

Per mitigare questi costi, la letteratura moderna (es. implementazioni come LightGBM o XGBoost) propone diverse strategie. Eccone tre fondamentali:
#### 1. Histogram-based Splitting (Binning)

Questo è l'approccio più efficace per ridurre la complessità computazionale. Invece di valutare lo split su tutti i valori unici continui di una feature (che possono essere $N$), i valori vengono discretizzati in $K$ bin (ceste), dove $K \ll N$ (tipicamente 256).

Durante il training, si costruisce un istogramma per ogni feature che aggrega i gradienti dei dati che cadono in ogni bin. La ricerca dello split ottimale avviene quindi sui bin, non sui singoli dati.

Questo riduce la complessità da $O(N)$ a $O(K)$, rendendo il costo di ricerca dello split quasi indipendente dal numero di istanze. Inoltre, gli istogrammi sono compatti e stanno comodamente nella cache L1/L2 del processore, eliminando quasi del tutto i cache miss.

#### 2. Gradient-based One-Side Sampling (GOSS)

Questa tecnica affronta il problema riducendo il numero di istanze $N$ da considerare, basandosi sull'informazione dei gradienti. L'intuizione è che le istanze con gradienti piccoli hanno già un errore basso e sono ben addestrate, quindi contribuiscono poco al guadagno informativo del nuovo split.

GOSS mantiene tutte le istanze con gradienti grandi (alto errore) e campiona casualmente solo una piccola frazione delle istanze con gradienti piccoli. Per non alterare la distribuzione dei dati, si introduce un peso correttivo per le istanze campionate. Questo permette di focalizzare la potenza di calcolo sui dati "difficili", mantenendo un'accuratezza paragonabile al full training ma con una frazione del costo.

#### 3. Exclusive Feature Bundling (EFB)

Questo approccio mira a ridurre la dimensionalità $M$ (numero di feature). In contesti sparsi (tipici dell'Information Retrieval o del NLP), molte feature non assumono mai valori non-nulli contemporaneamente (sono mutuamente esclusive).

EFB permette di "impacchettare" (bundle) diverse feature esclusive in un'unica feature sintetica densa. Utilizzando offset specifici per i valori delle feature originali, è possibile distinguere i contributi originali all'interno del bundle. Di fatto, questo riduce il numero di istanze di istogrammi da costruire, trasformando un problema sparso ad alta dimensionalità in uno più denso a bassa dimensionalità, velocizzando linearmente la costruzione degli istogrammi senza perdita di informazione.
## Question #4: Answer the following questions about k-nearest neighbors retrieval. ﻿﻿
## - Explain how to efficiently identify the closest point to a given query with Euclidean distance in 1D. ﻿﻿
## - Explain how to efficiently identify the closest point to a given query with Euclidean distance in 2D. 
## - What is product quantization, and how do we compute the distance between a query and an encoded vector? Please provide an example.

### 1. Ricerca efficiente in 1D (Euclidean Distance)

L'identificazione del punto più vicino in uno spazio unidimensionale è un problema banale che può essere risolto con altissima efficienza sfruttando l'ordinamento.

Se i punti del dataset vengono **pre-ordinati** in un array (costo $O(N \log N)$ in fase di indicizzazione), il problema della ricerca del nearest neighbor si riduce a trovare la posizione teorica della query $q$ all'interno di questo array ordinato.

L'algoritmo da utilizzare è la **Ricerca Binaria** (Binary Search). Data la query $q$, si esegue la ricerca binaria per trovare l'indice $i$ in cui $q$ si inserirebbe mantenendo l'ordinamento. Una volta identificato tale indice, il candidato più vicino deve essere necessariamente uno dei due punti adiacenti: il "predecessore" (all'indice $i$) o il "successore" (all'indice $i+1$). È sufficiente calcolare la distanza euclidea $|q - p_i|$ e $|q - p_{i+1}|$ e restituire il punto associato alla distanza minore.

Questo approccio garantisce una complessità di ricerca pari a **$O(\log N)$**.
### 2. Ricerca efficiente in 2D (Euclidean Distance)

Nello spazio bidimensionale (e in generale $N$-dimensionale), non esiste un ordinamento totale dei punti che preservi perfettamente la località spaziale (problema noto come _curse of dimensionality_), quindi non possiamo usare una semplice ricerca binaria su un array lineare.

Per risolvere efficientemente il problema in 2D si ricorre a strutture dati di **partizionamento spaziale**, la più comune delle quali è il **k-d tree** (k-dimensional tree).

Un k-d tree in 2D è un albero binario che partiziona lo spazio ricorsivamente tramite iperpiani (in 2D sono rette) ortogonali agli assi. A ogni livello dell'albero si alterna l'asse di taglio: al livello radice si divide lo spazio in base alla coordinata $x$ (es. tutti i punti con $x < median$ a sinistra, gli altri a destra), al livello successivo in base alla coordinata $y$, poi di nuovo $x$, e così via. Ogni nodo foglia conterrà un piccolo sottoinsieme di punti in una specifica regione rettangolare dello spazio.

In media, questo metodo riduce drasticamente lo spazio di ricerca rispetto alla scansione lineare, offrendo complessità logaritmica $O(\log N)$, sebbene nel caso peggiore (worst-case) possa degenerare.
### 3. Product Quantization (PQ)

La **Product Quantization (PQ)** è una tecnica di compressione _lossy_ (con perdita di informazione) progettata per gestire dataset massivi di vettori ad alta dimensionalità, permettendo di conservarli interamente in memoria RAM e di calcolare le distanze molto velocemente.

L'idea fondamentale è decomporre lo spazio vettoriale originale ad alta dimensionalità (prodotto cartesiano) in un prodotto cartesiano di $m$ sottospazi a bassa dimensionalità.

Dato un vettore originale $x \in \mathbb{R}^D$:

1. Il vettore viene diviso in $m$ sottovettori distinti $u_1, u_2, ..., u_m$, ciascuno di dimensione $D^* = D/m$.
    
2. Per ogni sottospazio, si esegue un algoritmo di clustering (tipicamente K-means) sui dati di training per creare un **codebook** indipendente contenente $K$ centroidi (spesso $K=256$ per poter usare 1 byte per indice).
    
3. Il vettore originale viene "quantizzato": ogni sottovettore $u_j$ viene sostituito dall'indice (ID) del centroide più vicino nel relativo codebook.
    
4. Il vettore compresso è quindi rappresentato solo dalla sequenza di $m$ indici (interi), occupando molto meno spazio dei float originali.

**Calcolo della distanza (Asymmetric Distance Computation - ADC):**

Per calcolare la distanza tra una query $q$ (non compressa, per massima precisione) e un documento codificato $y$, non serve decomprimere $y$.

Si pre-calcola una **Lookup Table (LUT)**. Appena arriva la query $q$, essa viene divisa negli stessi $m$ sottovettori. Per ogni sottovettore della query, si calcola la distanza euclidea al quadrato verso _tutti_ i $K$ centroidi del relativo codebook.

La distanza finale tra $q$ e un documento generico è semplicemente la **somma** delle distanze parziali lette dalla tabella agli indici specificati dalla codifica del documento.