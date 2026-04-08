## <font color="#244061">Lezione 13: Approximate Nearest Neighbor (AkNN) Search</font>

### Similarity Search e la Maledizione della Dimensionalità

La Similarity Search (o Nearest Neighbor Search) è il compito fondamentale nel recupero denso: dato un vettore di query $q \in \mathbb{R}^d$, trovare il vettore (o i $k$ vettori) nel dataset $\mathcal{X}$ che minimizzano una distanza metrica, tipicamente la distanza Euclidea o l'inverso della Cosine Similarity.

$$NN(q) = \arg\min_{x \in \mathcal{X}} \text{dist}(q, x)$$

In spazi a bassa dimensionalità, strutture come i KD-Trees offrono prestazioni logaritmiche. Tuttavia, quando la dimensionalità $d$ cresce (es. $d > 100$, tipico dei word embedding o descrittori visuali), queste strutture soffrono della "Maledizione della Dimensionalità" (Curse of Dimensionality), degenerando in una scansione lineare $O(N)$. Poiché la scansione lineare è insostenibile per grandi dataset (milioni o miliardi di vettori), si ricorre alla Approximate Nearest Neighbor (AkNN) Search. In questo scenario, si accetta una piccola perdita di accuratezza (non è garantito trovare il vero vicino più prossimo) in cambio di un miglioramento drastico nei tempi di risposta e nell'efficienza della memoria.

### Tecniche AkNN: Alberi, Clustering e Hashing

Esistono diverse famiglie di approcci per risolvere il problema AkNN.

Gli approcci basati su **Alberi**, come **ANNOY** (Approximate Nearest Neighbors Oh Yeah), partizionano lo spazio ricorsivamente utilizzando iperpiani casuali. Per mitigare il problema dei vettori che cadono vicino ai confini di partizione (dove il vicino reale potrebbe trovarsi "dall'altra parte" del muro), ANNOY costruisce una foresta di alberi. Durante la ricerca, si attraversano più alberi contemporaneamente e si aggregano i candidati trovati nelle foglie.

Gli approcci basati su **Clustering**, come l'**IVF** (Inverted File Index), utilizzano un quantizzatore grossolano (coarse quantizer) basato su K-means per dividere lo spazio in celle di Voronoi. I vettori del dataset vengono assegnati al centroide più vicino e memorizzati in liste invertite associate a quel centroide. Durante la query, il sistema confronta $q$ solo con i centroidi, identifica i $nprobe$ centroidi più vicini ed esplora solo le liste associate, ignorando la vasta maggioranza del dataset.

Gli approcci basati su **Hashing**, in particolare **LSH** (Locality Sensitive Hashing), utilizzano funzioni hash progettate in modo che vettori simili abbiano un'alta probabilità di "collidere" nello stesso bucket hash, mentre vettori dissimili abbiano bassa probabilità di collisione. Costruendo molteplici tabelle hash con diverse funzioni randomizzate, si aumenta la probabilità di intercettare i vicini corretti.

### Grafi di Prossimità e HNSW

Attualmente, i metodi basati su grafi offrono le migliori prestazioni in termini di trade-off tra velocità e recall. L'idea base è il grafo **NSW** (Navigable Small World), dove ogni vettore è un nodo collegato ai suoi vicini. Grazie alla proprietà "Small World" (presenza di link a lungo raggio oltre a quelli locali), è possibile navigare il grafo in modo greedy avvicinandosi rapidamente alla destinazione.

L'evoluzione più efficace è l'**HNSW** (Hierarchical Navigable Small World). Ispirato dalle **Skip Lists** di Pugh (che utilizzano livelli multipli di liste collegate per velocizzare la ricerca lineare), HNSW organizza i nodi in una gerarchia di livelli.

- I livelli superiori sono sparsi e contengono solo un sottoinsieme di nodi con collegamenti molto lunghi, permettendo grandi salti nello spazio ("autostrade").
    
- I livelli inferiori sono sempre più densi e contengono collegamenti locali a corto raggio.

La ricerca inizia dal livello più alto. L'algoritmo esegue una ricerca greedy trovando il nodo più vicino a $q$ in quel livello, poi "scende" al livello sottostante usando quel nodo come punto di ingresso, raffinando progressivamente la posizione fino a raggiungere il livello base (livello 0) dove si ottengono i vicini finali.

### Quantizzazione: Product Quantization (PQ)

Per gestire dataset che non entrano in memoria RAM o per accelerare drasticamente il calcolo delle distanze, si utilizza la **Product Quantization** (PQ). PQ decomprime il vettore originale ad alta dimensionalità in $m$ sottovettori di dimensione ridotta. Per ogni sottospazio, si esegue un clustering (es. K-means con 256 centroidi) per creare un codebook. Ogni sottovettore viene quindi sostituito dall'ID del centroide più vicino (che occupa solo 1 byte).

Durante la ricerca, non si ricostruiscono i vettori originali. Si utilizza invece la **Asymmetric Distance Computation (ADC)**. Prima di scansionare il dataset, si calcola la distanza tra i sottovettori della query $q$ e tutti i centroidi dei codebook, salvando i risultati in una tabella di lookup. La distanza approssimata tra $q$ e un documento $d$ diventa quindi una semplice somma di valori letti dalla tabella, eliminando le costose operazioni in virgola mobile e riducendo l'I/O di memoria.

### AkNN su Sparsi: Algoritmo Seismic

Mentre HNSW e PQ dominano nel retrieval denso, la ricerca approssimata su vettori sparsi (come quelli generati da SPLADE o Learned Sparse Retrieval) presenta sfide diverse a causa dell'altissima dimensionalità e della scarsità dei dati. L'algoritmo **Seismic** adatta i principi dell'indice invertito al calcolo vettoriale geometrico.

Seismic organizza i dati utilizzando un'idea di **Concentration of Importance**: nei vettori sparsi, pochi componenti contribuiscono alla maggior parte del punteggio di similarità. L'algoritmo utilizza una struttura a indice invertito ma partiziona le posting lists in blocchi basati su proprietà geometriche. Introduce tecniche di **Blocking** e **Pruning** aggressive simili a WAND, ma adattate al prodotto scalare: ogni blocco di posting list memorizza un sommario dei massimi pesi e, durante la ricerca, interi blocchi vengono saltati se il loro contributo massimo stimato non è sufficiente a superare la soglia corrente dei top-$k$ risultati. Questo permette di eseguire query su vettori sparsi con latenze nell'ordine dei millisecondi, rendendo modelli come SPLADE utilizzabili in produzione.