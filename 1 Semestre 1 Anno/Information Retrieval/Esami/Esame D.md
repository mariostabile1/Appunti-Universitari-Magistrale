### 1. **Build a toy example (5 documents, 1 query) that you can use to introduce and discuss the concepts of binary term-document incidence matrix, term-document count matrix and term-document frequency matrix. What are the main characteristics of TF-IDF? Calculate TF-IDF on your example** 

Immaginiamo una collezione di 5 documenti e 1 query.
Vocabolario ridotto: {gatto, cane, topo}.

- **D1:** "il gatto insegue il cane"
    
- **D2:** "gatto gatto gatto"
    
- **D3:** "il cane dorme"
    
- **D4:** "il topo mangia"
    
- **D5:** "il gatto e il topo"

**Query ($q$):** "gatto cane"
##### <font color="#5f497a">1. Binary Term-Document Incidence Matrix</font>

Questa matrice rappresenta semplicemente la **presenza (1) o assenza (0)** di un termine nel documento. Ignora quante volte il termine appare.

|**Termine**|**D1**|**D2**|**D3**|**D4**|**D5**|
|---|---|---|---|---|---|
|**gatto**|1|1|0|0|1|
|**cane**|1|0|1|0|0|
|**topo**|0|0|0|1|1|

A cosa serve: È la base del Boolean Retrieval.

Concetto chiave: Risponde alla domanda "Il documento contiene il termine?". Non permette di ordinare i risultati per rilevanza (ranking): D1 e D2 sono identici per il termine "gatto", anche se D2 ne parla molto di più.
##### <font color="#5f497a">2. Term-Document Count Matrix</font>

Qui contiamo le **occorrenze grezze** ($tf$, raw term frequency). Ogni documento è un vettore di conteggi ($\mathbb{N}^v$).

|**Termine**|**D1**|**D2**|**D3**|**D4**|**D5**|
|---|---|---|---|---|---|
|**gatto**|1|3|0|0|1|
|**cane**|1|0|1|0|0|
|**topo**|0|0|0|1|1|

A cosa serve: Introduce il concetto di Bag of Words.

Concetto chiave: Ora possiamo distinguere D2 da D1. D2 sembra molto più focalizzato sui gatti. Tuttavia, la frequenza grezza ha un problema: un documento che ripete una parola 10 volte non è necessariamente 10 volte più rilevante .
##### <font color="#5f497a">3. Term-Document Frequency Matrix (Log-Weighted)</font>

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
##### <font color="#5f497a">4. TF-IDF: Caratteristiche e Calcolo</font>

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
#### Conclusione Teorica
In questo esempio, D1 vince su D2 (0.62 vs 0.33).
Anche se D2 parla molto di "gatti", D1 menziona entrambi i termini della query. Inoltre, il termine "cane" (più raro nella collezione) ha spinto in alto il punteggio di D1 grazie all'IDF. Questo dimostra come il TF-IDF bilanci la frequenza locale con la rarità globale.
### 2. **How can we go beyond binary relevance? Discuss the metrics** 

I sistemi di Information Retrieval classici valutano spesso le prestazioni basandosi sul concetto di **Binary Relevance**, ovvero l'idea che un documento sia semplicemente rilevante o non rilevante rispetto a una query. Questo approccio binario è alla base delle metriche di Precision e Recall. Tuttavia, in scenari reali, la rilevanza non è quasi mai bianca o nera; essa presenta sfumature. Un documento può essere parzialmente pertinente, molto pertinente o totalmente inutile. Per superare i limiti della valutazione binaria, si introduce il concetto di **Graded Relevance**, dove la rilevanza è misurata su una scala ordinale (ad esempio da 0 a 4, dove 0 è non rilevante e 4 è altamente rilevante).

L'adozione della rilevanza graduata richiede nuove metriche che non si limitino a contare i documenti, ma che considerino il valore di utilità (gain) che l'utente ottiene da ciascun risultato e, soprattutto, la posizione in cui tali risultati vengono presentati. L'assunto fondamentale è che un documento altamente rilevante è molto più utile se appare nelle prime posizioni della lista dei risultati piuttosto che in fondo. Le metriche principali che operano su questo principio sono il Cumulative Gain (CG), il Discounted Cumulative Gain (DCG) e il Normalized Discounted Cumulative Gain (NDCG).
##### Cumulative Gain (CG)

La metrica più basilare per gestire la rilevanza graduata è il **Cumulative Gain**. Questa metrica misura il guadagno totale accumulato dall'utente scorrendo la lista dei risultati fino a una certa posizione $p$. Il calcolo consiste nella semplice somma dei punteggi di rilevanza ($rel_i$) dei documenti recuperati fino al rango $p$.
$$CG_p = \sum_{i=1}^{p} rel_i$$
Sebbene il CG superi la logica binaria integrando i punteggi di rilevanza, esso soffre di una limitazione critica per un motore di ricerca: non tiene conto dell'ordinamento interno dei risultati. Scambiare un documento altamente rilevante alla posizione 1 con un documento poco rilevante alla posizione $p$ non altera il valore finale del $CG_p$. Questo contraddice l'esperienza utente, dove l'attenzione è massima sui primi risultati e decresce rapidamente scorrendo la lista.
##### Discounted Cumulative Gain (DCG)

Per risolvere il problema dell'ordinamento, si introduce il **Discounted Cumulative Gain**. Il principio alla base del DCG è che l'utilità di un documento decresce logaritmicamente man mano che la sua posizione nel ranking aumenta. La rilevanza di ogni risultato viene quindi "scontata" (discounted) in base al suo rango. Se un documento rilevante appare in basso nella lista, il suo contributo al punteggio totale sarà penalizzato.

La formula standard per il calcolo del DCG alla posizione $p$ è definita come:
$$DCG_p = rel_1 + \sum_{i=2}^{p} \frac{rel_i}{\log_2(i)}$$
Esiste una formulazione alternativa, spesso utilizzata in contesti industriali o quando si vuole enfatizzare drasticamente i documenti molto rilevanti, che modifica il numeratore utilizzando una potenza di 2. Questa variante premia maggiormente il recupero di documenti con il massimo grado di rilevanza rispetto a quelli marginalmente rilevanti:
$$DCG_p = \sum_{i=1}^{p} \frac{2^{rel_i} - 1}{\log_2(i+1)}$$
Con il DCG, un sistema che posiziona i risultati migliori in cima alla lista otterrà un punteggio superiore rispetto a un sistema che restituisce gli stessi documenti ma in un ordine subottimale.
##### Normalized Discounted Cumulative Gain (NDCG)

Il DCG presenta ancora un difetto operativo: il valore calcolato non è limitato superiormente e dipende dalla lunghezza della lista dei risultati e dalla "difficoltà" della query. Una query con molti documenti rilevanti nel corpus avrà naturalmente un DCG molto più alto di una query con pochi documenti rilevanti, rendendo impossibile il confronto delle prestazioni del sistema su query diverse.

Per rendere i punteggi comparabili, è necessario normalizzare il DCG rispetto al miglior ordinamento possibile per quella specifica query. Si introduce quindi l'**Ideal DCG (IDCG)**, che rappresenta il valore di DCG ottenuto se i documenti recuperati fossero ordinati in modo perfetto (monotonicamente decrescente per rilevanza).

Il **Normalized Discounted Cumulative Gain** è quindi il rapporto tra il DCG ottenuto dal sistema e l'IDCG ideale:
$$NDCG_p = \frac{DCG_p}{IDCG_p}$$
Il risultato è un valore compreso tra 0 e 1. Un $NDCG_p$ pari a 1 indica che l'ordinamento prodotto dal sistema è perfetto fino alla posizione $p$. Grazie a questa normalizzazione, è possibile calcolare la media dell'NDCG su un intero set di query, ottenendo una misura affidabile e confrontabile della qualità del ranking del sistema di Information Retrieval.
### 3. **Encode number 14 in Elias Gamma and Delta. The importance of prefix-free encoding, and to compare them with variable-byte encoding.** 
### 4. **Seismic, describe the data structure used and the difference between the simple inverted index. What are the limit of the inverted index and how seismic resolve those problems.**

##### Seismic e la Struttura dei Dati

Seismic è un sistema di retrieval progettato specificamente per gestire le **Learned Sparse Representations** (come quelle generate da modelli tipo SPLADE), ovvero vettori che, pur essendo sparsi (molti zeri), presentano una densità e una distribuzione dei pesi molto diversa dai tradizionali vettori _Bag-of-Words_ basati sulla frequenza dei termini (TF-IDF/BM25).
##### La Struttura Dati: Block-Max Inverted Index "Geometrico"

A differenza di un indice invertito semplice, che è sostanzialmente una lista piatta di identificativi di documenti (_postings_) per ogni termine, Seismic adotta una struttura ibrida e organizzata gerarchicamente.

La struttura portante è un **indice invertito a blocchi geometricamente coesi**. Invece di memorizzare i documenti in ordine puramente crescente di ID (o di frequenza), Seismic raggruppa i documenti all'interno di una _posting list_ in **blocchi** basati sulla similarità dei loro vettori.

Ogni blocco non è solo un contenitore di documenti, ma è equipaggiato con un **Summary Vector** (un vettore riassuntivo o "schizzo"). Questo vettore sommario rappresenta l'aggregato o il centroide dei documenti contenuti in quel blocco.

Parallelamente all'indice invertito, Seismic mantiene un **Forward Index** (indice diretto) in memoria. Questo è fondamentale per la fase di rifinitura: una volta che l'indice invertito ha identificato i blocchi candidati promettenti, il sistema accede all'indice diretto per recuperare i vettori completi dei documenti e calcolare il punteggio esatto.
##### Differenze con l'Inverted Index Semplice

L'indice invertito semplice ("Simple Inverted Index") è ottimizzato per il testo tradizionale. Le sue liste di _postings_ contengono coppie `(docID, term_frequency)`, spesso compresse con tecniche che sfruttano la sequenzialità degli ID (come VByte o PForDelta). L'attraversamento avviene processando ogni documento o piccoli gruppi di documenti sequenziali per calcolare il punteggio BM25.

La differenza chiave in Seismic risiede nell'**organizzazione interna delle liste**. In un indice semplice, la prossimità di due documenti nella lista è data solo dal loro ID numerico, che è casuale rispetto al contenuto semantico. In Seismic, i documenti all'interno di un blocco sono raggruppati perché sono semanticamente simili (o meglio, hanno valori simili per quella specifica dimensione del vettore sparso). Inoltre, Seismic memorizza pesi quantizzati (spesso float o interi scalati) derivanti da reti neurali, non semplici conteggi interi.
##### Limiti dell'Inverted Index Tradizionale

L'applicazione di un indice invertito semplice alle _Learned Sparse Representations_ incontra due limiti critici che ne degradano le prestazioni:

1. **Inefficienza del Dynamic Pruning (WAND):** Algoritmi come WAND (Weighted AND) sono essenziali per evitare di scansionare tutti i documenti. Funzionano stimando un "massimo punteggio possibile" per i documenti non ancora visitati. Con i vettori appresi (Learned Vectors), i pesi non seguono la distribuzione Zipfiana tipica del linguaggio naturale, ma sono più "piatti" e densi. Questo rende le stime di WAND poco efficaci (i bounds sono troppo larghi), costringendo l'algoritmo a valutare quasi tutti i documenti e annullando i benefici dell'indice.
    
2. **Overhead di Decompressione e Accesso alla Memoria:** I vettori appresi tendono ad avere molte più dimensioni attive rispetto a una query testuale classica. Questo comporta _posting lists_ molto più lunghe. Decomprimere milioni di interi e processarli sequenzialmente satura la banda di memoria e la CPU, creando un collo di bottiglia insostenibile per la latenza in tempo reale.
##### Come Seismic Risolve i Problemi

Seismic supera questi limiti spostando il focus dal "documento singolo" al "blocco di documenti":

- **Pruning Aggressivo basato sui Blocchi:** Grazie ai _Summary Vectors_, Seismic può decidere se scartare un intero blocco di documenti senza doverli decomprimere o analizzare uno per uno. Prima di accedere al blocco, il sistema calcola il prodotto scalare tra la query e il vettore sommario del blocco. Se questo punteggio stimato non supera una certa soglia (determinata dinamicamente rispetto ai migliori risultati trovati finora), l'intero blocco viene saltato (Skipping).
    
- **Località dei Dati:** Poiché i documenti nel blocco sono raggruppati per similarità ("geometricamente coesi"), è molto probabile che se il _Summary Vector_ è poco rilevante, tutti i documenti nel blocco siano irrilevanti. Questo rende il pruning molto più robusto e preciso rispetto al pruning basato su semplici ID.
    
- **Utilizzo del Forward Index:** Delegando il calcolo finale al Forward Index solo per i documenti sopravvissuti al pruning, Seismic evita i costosi accessi casuali e le decompressione inutili nelle liste invertite per i documenti di bassa rilevanza, ottimizzando l'uso della cache e riducendo drasticamente il numero di operazioni di prodotto scalare necessarie.