# Language Models I

### Statistical Language Models

Un **Statistical Language Model** è definito formalmente come una distribuzione di probabilità $P$ su sequenze di termini. Dato un documento $d$ composto dalla sequenza di parole $w_1, w_2, w_3$, la probabilità del documento è data dalla regola della catena:

$$P(d) = P(w_1w_2w_3) = P(w_1)P(w_2|w_1)P(w_3|w_1w_2)$$

Questa formulazione generale non fa assunzioni ma è impraticabile poiché richiederebbe di apprendere la probabilità di qualsiasi sequenza possibile nella lingua.
#### Unigram Model

Il modello **Unigram** assume la **total independence** tra le parole. La probabilità del documento è semplicemente il prodotto delle probabilità delle singole parole:

$$P(d) \approx \prod_{i} P(w_i)$$

- Questa assunzione è alla base del classificatore **Naïve Bayes**.
    
- Per stabilità numerica, si lavora spesso nello spazio logaritmico: $\log P(d) = \sum \log P(w_i)$.
    
- È necessario applicare tecniche di **Smoothing** (es. "add one") per evitare probabilità zero per parole mai viste.
#### N-gram Model

I modelli **N-gram** (es. Bigram) introducono una dipendenza limitata. Un Bigram assume che la probabilità di una parola dipenda solo dalla precedente (assunzione di Markov):

$$P(d) \approx \prod_{i} P(w_i | w_{i-1})$$

Allungare la dimensione di $n$ (la finestra di contesto) permette di catturare più semantica, ma rende il modello statisticamente meno significativo (problema di memorizzazione vs generalizzazione).

---
### Neural Language Models: Word2Vec

**Word2Vec** rappresenta una svolta nell'apprendimento di rappresentazioni dense (**Embeddings**) delle parole. L'idea chiave è che gli embedding sono un "sottoprodotto" di un task di predizione (Prediction Task).

Esistono due architetture principali:

1. **Skip-gram**: Dato una parola centrale ($w_t$), predice le parole del contesto ($w_{t-2}, w_{t-1}, w_{t+1}, w_{t+2}$).
    
2. **CBOW (Continuous Bag of Words)**: Dato il contesto, predice la parola centrale ($w_t$).

L'architettura è una rete neurale lineare a due livelli:

1. **Input**: Rappresentazione One-hot delle parole.
    
2. **Hidden Layer**: Matrice $W_I$ che proietta l'input in uno spazio denso (Embedding).
    
3. **Output**: Matrice $W_O$ che converte la rappresentazione densa in probabilità tramite Softmax.
    

- **Non richiede dati etichettati**: Si addestra su puro testo (Unsupervised/Self-supervised).
    
- **Window Size**: Finestre più ampie (6-10 parole) catturano più relazioni semantiche (topicality), finestre strette catturano più sintassi.
    
- **Dimensione tipica**: I vettori hanno dimensione 200-300.
#### Proprietà degli Embeddings

Le parole usate in contesti simili finiscono per avere vettori simili. Questo spazio vettoriale cattura proprietà sintattiche e semantiche verificabili tramite **analogie vettoriali**:

$$e(\text{King}) - e(\text{Man}) + e(\text{Woman}) \approx e(\text{Queen})$$

La semantica catturata dipende fortemente dai dati di training (es. "Chianti" è associato al vino nei libri, ma a volte a località geografiche in Wikipedia).

---
### Estensioni di Word2Vec

#### FastText

**FastText** estende Word2Vec rappresentando ogni parola come un insieme di **n-grammi di caratteri** (subword information).

- Esempio: "goodbye" $\rightarrow$ `<go`, `goo`, `ood`, `...`, `bye`, `ye>`.
    
- L'embedding finale è la somma degli embedding degli n-grammi più quello della parola intera.
    
- **Vantaggio**: Gestisce parole **Out-Of-Vocabulary (OOV)** e errori di ortografia, poiché può costruire rappresentazioni basandosi sulle sottostringhe note.
#### Doc2Vec

Estende il concetto ai documenti. Inserisce un vettore "Document ID" come input addizionale nella rete neurale, che viene addestrato insieme alle parole per predire il target. Questo vettore cattura il "tema" del documento.

---
### Integrazione in Reti Neurali (Deep Learning)

Quando si usano gli embedding come primo livello di una rete neurale profonda:

- **Matrice di Embedding**: È una matrice $|V| \times d$ (Vocabolario $\times$ Dimensione). Può essere inizializzata casualmente o con pesi pre-addestrati (Word2Vec/GloVe).
    
- **Fine-tuning**: I pesi possono essere aggiornati durante il training del task specifico o tenuti fissi.
    
- **Gestione OOV e Padding**:
    
    - Si usa un token speciale `[UNK]` per parole sconosciute.
        
    - Si usa un token `[PAD]` per uniformare la lunghezza delle frasi nei batch (necessario per il calcolo parallelo su GPU).

---

### Modelli di Ultima Generazione: Transformers e Self-Attention

I modelli recenti (BERT, GPT, ELMo) superano i limiti delle rappresentazioni statiche (come Word2Vec dove una parola ha sempre lo stesso vettore) utilizzando architetture profonde basate su **Self-Attention**.

**Self-Attention**: Meccanismo che apprende il contributo degli elementi contestuali alla definizione semantica di una parola. È simile a ciò che fanno i filtri nelle CNN per le immagini. Nelle visualizzazioni, le "Attention Heads" mostrano linee che collegano una parola alle altre parole della frase che la "influenzano" (es. un pronome che punta al soggetto a cui si riferisce).

---

### Dense Retrieval e Siamese Networks

L'applicazione di questi modelli all'Information Retrieval porta al paradigma del **Dense Retrieval** (o Representation-based Retrieval).

Si utilizza un'architettura **Siamese Network** (o Dual Encoder):

1. **Offline**: Tutti i documenti $d$ vengono passati nella rete neurale e trasformati in vettori densi (indicizzati).
    
2. **Online**: La query $q$ viene trasformata in un vettore dalla stessa rete.
    
3. **Matching**: La rilevanza $S_{q,d}$ è calcolata come **Dot Product** o **Cosine Similarity** tra i due vettori.


$$S_{q,d} = \text{Dense}(q) \cdot \text{Dense}(d)$$

Questo approccio abilita la ricerca efficiente tramite **k-Nearest Neighbour (k-NN)** approssimato (ANN), permettendo di scalare su grandi collezioni con complessità $O(n \log k)$ o migliore usando strutture dati come alberi, grafi o quantizzazione.