## <font color="#244061">Lezione 12: Neural Information Retrieval</font>

### Modelli di Linguaggio Neurali: Word2Vec e FastText

I modelli di linguaggio neurali superano i limiti delle rappresentazioni "one-hot" (dove le parole sono simboli atomici ortogonali) introducendo il concetto di **word embedding**. Un embedding è una rappresentazione vettoriale densa e continua in cui parole semanticamente simili sono mappate in punti vicini nello spazio geometrico.

**Word2Vec** è una famiglia di architetture predittive basata sull'ipotesi distribuzionale: il significato di una parola è determinato dal contesto in cui appare. Esistono due architetture principali:

- **CBOW (Continuous Bag-of-Words):** Il modello predice la parola target basandosi sulla media dei vettori delle parole del contesto (finestra scorrevole). È veloce e si comporta bene con termini frequenti.
    
- **Skip-gram:** Inverte il processo, utilizzando la parola target per predire le parole del contesto circostante. Questa architettura è computazionalmente più costosa ma gestisce meglio i termini rari.

Un limite di Word2Vec è l'incapacità di gestire parole fuori vocabolario (OOV - Out Of Vocabulary) e la mancanza di comprensione della morfologia. **FastText** risolve questo problema rappresentando ogni parola come un sacco di n-grammi di caratteri (subword information). Il vettore della parola è la somma dei vettori dei suoi n-grammi. Ad esempio, "apple" potrebbe essere rappresentato dai vettori di `<ap`, `app`, `ppl`, `ple`, `le>`, permettendo di generare embedding anche per parole mai viste prima o con errori di battitura, basandosi sulle loro radici e suffissi.

### Document Embeddings: Doc2Vec e Pooling

Per l'Information Retrieval è necessario rappresentare intere sequenze di testo (documenti o query), non solo singole parole. Un approccio semplice è il **pooling** (media o massimo) dei vettori Word2Vec delle parole che compongono il documento. Sebbene efficace come baseline, il pooling perde completamente l'informazione sull'ordine delle parole e sulla sintassi.

**Doc2Vec** (o Paragraph Vector) estende l'idea di Word2Vec inserendo un vettore aggiuntivo che rappresenta l'ID del paragrafo o del documento. Nell'architettura **PV-DM** (Distributed Memory), il vettore del documento agisce come una memoria globale che contribuisce, insieme ai vettori delle parole di contesto, a predire la parola successiva. Questo permette al modello di catturare la semantica complessiva del documento in un vettore di dimensione fissa, utile per compiti di classificazione o recupero.

### Transformer e BERT

L'introduzione dei **Transformer** ha rivoluzionato il NLP permettendo di generare embedding contestuali, dove la rappresentazione di una parola cambia in base al suo utilizzo nella frase (gestione della polisemia). Il cuore del Transformer è il meccanismo di **Self-Attention**, che calcola quanto ogni parola della sequenza dovrebbe "prestare attenzione" alle altre parole per costruire la propria rappresentazione.

Data una matrice di query $Q$, chiavi $K$ e valori $V$, l'attenzione è calcolata come:

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

Il fattore $\sqrt{d_k}$ serve a scalare i prodotti scalari per evitare gradienti troppo piccoli. La Multi-headed Attention applica questo meccanismo in parallelo più volte (teste), permettendo al modello di focalizzarsi simultaneamente su diversi sottospazi di rappresentazione (es. relazioni sintattiche in una testa, semantiche in un'altra).

**BERT** (Bidirectional Encoder Representations from Transformers) utilizza una pila di encoder Transformer addestrati con un obiettivo di **Masked Language Modeling** (MLM), dove alcune parole vengono oscurate e il modello deve predirle usando il contesto sia sinistro che destro. Questo produce rappresentazioni profonde e bidirezionali, ideali per il ranking.

### Metodi basati sulla Rappresentazione: Retrieval Denso vs Sparso Appreso

Questi metodi, spesso chiamati **Dual Encoders** o Bi-Encoders, codificano query e documento separatamente in vettori indipendenti. Il punteggio di rilevanza è calcolato solitamente tramite prodotto scalare o similarità coseno. Questo permette di pre-calcolare i vettori dei documenti e utilizzare librerie di ricerca approssimata (ANN) per l'efficienza.

Il Retrieval Denso mappa il testo in vettori di bassa dimensione (es. 768 float). Modelli come ANCE (Approximate Nearest Neighbor Negative Contrastive Estimation) migliorano l'addestramento utilizzando negativi recuperati dinamicamente dall'indice durante il training, correggendo la discrepanza tra fase di training e fase di inferenza.

ColBERT (Contextualized Late Interaction over BERT) è un ibrido che, pur essendo un dual encoder, mantiene i vettori di tutti i token (multi-vector) invece di comprimerli in uno solo. La similarità è calcolata tramite un'operazione di "Late Interaction" (somma di massimi), che preserva dettagli fini persi nel single-vector encoding.

Il **Retrieval Sparso Appreso** (Learned Sparse Retrieval) proietta i documenti in vettori sparsi di altissima dimensione (la dimensione del vocabolario), dove i pesi non sono frequenze (TF-IDF) ma valori appresi. **SPLADE** (Sparse Lexical and Expansion Model) utilizza l'output del MLM di BERT per espandere il documento con termini semanticamente correlati anche se non presenti nel testo originale, assegnando pesi di importanza tramite un meccanismo di scarsità (L1 regularization).

### Interaction-based Methods: MonoBERT

I metodi basati sull'interazione, noti come **Cross-Encoders**, non codificano query e documento separatamente. Al contrario, concatenano la query e il documento in un'unica sequenza di input per il modello (es. `[CLS] Query [SEP] Document [SEP]`).

**MonoBERT** è l'archetipo di questo approccio per il re-ranking. Poiché l'attenzione del Transformer è "full self-attention", ogni token della query può interagire direttamente con ogni token del documento a tutti i livelli della rete. Questo garantisce un'accuratezza superiore rispetto ai Dual Encoders, ma rende impossibile l'indicizzazione vettoriale pre-calcolata. Pertanto, i Cross-Encoders sono utilizzati tipicamente solo nell'ultimo stadio di una pipeline di re-ranking, lavorando su un piccolo set di candidati (es. top-100 o top-1000) recuperati da un modello più efficiente.

### Negative Sampling e Hard Negatives

L'addestramento dei modelli di Neural IR richiede triplette composte da: una Query $q$, un Documento Positivo $d^+$ e uno o più Documenti Negativi $d^-$. La scelta dei negativi è critica per la convergenza e la qualità del modello.

Utilizzare negativi casuali è spesso inefficace perché sono troppo facili da distinguere dal positivo, portando a gradienti nulli (vanishing gradients) che non aggiornano i pesi del modello. È necessario utilizzare **Hard Negatives**: documenti che non sono rilevanti per la query ma che le assomigliano molto (es. condividono molte parole o sono vicini nello spazio vettoriale corrente).

Le strategie di campionamento includono approcci statici, come l'uso dei top-k documenti recuperati da BM25 che non sono etichettati come rilevanti, e approcci dinamici, dove il modello stesso (o un modello "teacher") viene usato durante il training per identificare i documenti che il modello classifica erroneamente come rilevanti con alta confidenza.