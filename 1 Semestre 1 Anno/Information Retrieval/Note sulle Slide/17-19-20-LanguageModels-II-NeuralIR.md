
# Neural Information Retrieval e Language Models

### Il Punto di Svolta: BERT e i Transformer

Il 2018 ha segnato un punto di svolta nel Natural Language Processing (NLP) con l'introduzione di **BERT** (Bidirectional Encoder Representations from Transformers) e di altri modelli basati sull'architettura **Transformer**. Il cuore di questa architettura è il meccanismo di **Self-Attention**. A differenza delle reti ricorrenti (RNN) che elaborano il testo sequenzialmente, la Self-Attention permette al modello di pesare l'importanza di ogni parola in una frase rispetto a tutte le altre, catturando relazioni a lungo raggio e il contesto bidirezionale (sia a destra che a sinistra della parola target).

Un esempio classico per capire l'attenzione è la risoluzione della co-riferenza nella frase: _"The animal didn't cross the street because **it** was too tired"_ vs _"because **it** was too wide"_. Il meccanismo di attenzione permette al modello di legare il pronome "**it**" alla parola "**animal**" nel primo caso e a "**street**" nel secondo, basandosi sul contesto fornito dall'aggettivo ("tired" vs "wide") .

**BERT** viene pre-addestrato su enormi corpora di testo (come Wikipedia) tramite due task non supervisionati:

1. **Masked Language Model (MLM)**: Alcune parole vengono mascherate ([MASK]) e il modello deve predirle basandosi sul contesto.
    
2. **Next Sentence Prediction (NSP)**: Il modello deve predire se, date due frasi A e B, B è la logica continuazione di A.

Per gestire l'input, BERT utilizza dei token speciali:

- `[CLS]`: Inserito all'inizio della sequenza, il suo vettore finale viene spesso usato come rappresentazione dell'intera frase (es. per la classificazione).
    
- `[SEP]`: Utilizzato per separare due frasi o segmenti di testo.

### Neural Approaches for IR: Tassonomia

L'applicazione delle reti neurali all'Information Retrieval si divide in due grandi famiglie, distinte principalmente dal momento in cui avviene l'interazione tra la Query ($q$) e il Documento ($d$): **Interaction-based** e **Representation-based**.

#### 1. Interaction-based Methods (Cross-Encoders)

In questo approccio, la query e il documento vengono elaborati **congiuntamente** dalla rete neurale. L'esempio più noto è **MonoBERT**. L'input viene formattato concatenando query e documento separati dal token `[SEP]`: `[CLS] query [SEP] documento [SEP]`

Il modello calcola l'attenzione "tutti-con-tutti" (all-to-all interaction) tra i token della query e quelli del documento fin dai primi livelli. L'output finale del token `[CLS]` viene passato a un livello lineare per produrre uno **score di rilevanza** .

- **Vantaggi**: Altissima accuratezza. Il modello può catturare relazioni semantiche complesse e sfumature, poiché ogni termine della query può "vedere" ogni termine del documento.
    
- **Svantaggi**: Estremamente lento (alta latenza) e costoso computazionalmente. Non è possibile pre-calcolare le rappresentazioni dei documenti in modo indipendente, quindi l'inferenza deve essere fatta online per ogni coppia query-documento candidata. Tipicamente usato solo come _Re-ranker_ su una lista ridotta di candidati (es. i top 1000) recuperati da un primo stadio più veloce (es. BM25).
#### 2. Representation-based Methods (Bi-Encoders / Dense Retrieval)

In questo approccio, query e documenti sono rappresentati separatamente come **vettori densi** (embeddings) in uno spazio vettoriale comune. Questa architettura è spesso definita **Siamese Network** o Dual Encoder.

Il processo si divide nettamente in due fasi:

- **Offline Computation**: Tutti i documenti della collezione vengono processati dalla rete neurale e trasformati in vettori densi (es. usando l'embedding del token `[CLS]` o facendo la media degli embeddings). Questi vettori vengono indicizzati.
    
- **Online Computation**: All'arrivo della query, questa viene trasformata in un vettore dalla stessa rete (o una gemella).
    

Lo score di rilevanza è calcolato tramite una semplice funzione di similarità geometrica, come il **Dot Product** (prodotto scalare) o la **Cosine Similarity**, tra il vettore della query e il vettore del documento.

- **Vantaggi**: Molto efficiente. Permette l'utilizzo di algoritmi di **Approximate Nearest Neighbor (ANN)** (come HNSW o alberi) per recuperare i documenti più simili in tempo sub-lineare, scalando su milioni di documenti. Risolve il problema del _vocabulary mismatch_ (mancata corrispondenza esatta dei termini) tipico dei metodi sparsi.
    
- **Svantaggi**: Minore accuratezza rispetto ai Cross-Encoders, poiché la query e il documento non "interagiscono" direttamente se non alla fine tramite il prodotto scalare. Si perde parte dell'informazione strutturale fine.
### Training e Negative Sampling

L'addestramento dei modelli Representation-based (Dense Retrieval) è critico e si basa sul **Metric Learning**. L'obiettivo è addestrare la rete affinché i vettori di query e documenti rilevanti ($d^+$) siano vicini nello spazio, mentre quelli non rilevanti ($d^-$) siano lontani, rispettando un certo margine $m$ .

La funzione di costo usata è spesso la **Triplet Loss** o la Contrastive Loss. Un aspetto fondamentale è la scelta dei **Negative Examples** (documenti non rilevanti):

1. **Easy Negatives**: Documenti scelti a caso. Sono facili da distinguere per il modello e portano a gradienti nulli (il modello impara poco).
    
2. **Hard Negatives (HN)**: Documenti che sono _semanticamente simili_ alla query (o contengono le stesse parole) ma non sono rilevanti. Sono i più importanti per l'apprendimento, poiché costringono il modello a capire le sottili differenze di significato.
#### ANCE: Approximate Nearest Neighbor Negative Contrastive Learning

**ANCE** è una tecnica avanzata per gestire i negativi. Invece di usare un set statico di negativi (es. presi da BM25), ANCE seleziona i negativi **dinamicamente** durante il training. Utilizza il modello corrente per cercare nell'indice i documenti che il modello ritiene (erroneamente) molto simili alla query. Questi diventano i "Hard Negatives" per il passo successivo di training. Questo crea un curriculum di apprendimento sempre più difficile e allinea meglio la fase di training con quella di testing .

### ColBERT: Contextualized Late Interaction

**ColBERT** (Contextualized Late Interaction over BERT) propone una via di mezzo per ottenere l'efficacia dei Cross-Encoders con l'efficienza dei Bi-Encoders.

Invece di collassare l'intero documento in un _unico_ vettore (come nei Bi-Encoders), ColBERT mantiene una rappresentazione **multi-vettoriale**: ogni token del documento ha il suo embedding.

- **Query Encoder**: Produce un set di vettori $E_q$.
    
- **Document Encoder**: Produce un set di vettori $E_d$ (calcolati offline).

L'interazione è definita "**Late Interaction**" perché avviene solo alla fine tramite un'operazione chiamata **MaxSim**. Per ogni termine della query, si cerca il termine del documento con la massima similarità (prodotto scalare), e poi si sommano questi massimi.

$$S_{q,d} = \sum_{i \in E_q} \max_{j \in E_d} (E_{q_i} \cdot E_{d_j}^T)$$

Questo permette di conservare l'informazione contestuale di ogni parola (alta precisione) ma consente comunque di pre-calcolare i vettori dei documenti, rendendo il retrieval molto più veloce di un Cross-Encoder classico.

### Efficienza e Interpretability: EPIC

Per migliorare ulteriormente l'efficienza e l'interpretabilità, esistono approcci come **EPIC** (Expansion via Prediction of Importance with Contextualization). EPIC cerca di ridurre il costo computazionale calcolando uno score di importanza per ogni termine nel documento e nella query. Questo permette di:

1. **Pruning**: Rimuovere i termini meno importanti dalla rappresentazione del documento (simile a una selezione delle feature), riducendo lo spazio di archiviazione e i calcoli.
    
2. **Expansion**: Arricchire il vettore del documento con termini correlati (espansione semantica), simile a quanto fa **doc2query** (che genera query artificiali per arricchire il documento), ma lavorando direttamente nello spazio latente o lessicale.

Un esempio visivo mostrato nelle slide riguarda i "pool" (piscine): il modello impara ad associare termini come "swim", "spa", "cost" anche se non presenti esplicitamente, migliorando il recupero.