Ecco degli appunti esaustivi e strutturati sulla lezione "Scoring, Term Weighting and The Vector Space Model".

# Scoring, Term Weighting e Vector Space Model

### Dal Boolean Retrieval al Ranked Retrieval

I modelli di recupero delle informazioni basati sulla logica Booleana, sebbene precisi ed efficaci per utenti esperti o applicazioni specifiche, presentano limiti significativi per la ricerca generica. Il problema principale è noto come **Feast or Famine** (abbondanza o carestia): una query booleana tende a restituire o troppi risultati (spesso migliaia) o nessuno . Inoltre, il modello booleano non fornisce indicazioni sulla rilevanza relativa dei documenti; un documento che menziona il termine ricercato una sola volta viene trattato allo stesso modo di uno che lo menziona ripetutamente.

Il **Ranked Retrieval** risolve questi problemi ordinando i documenti della collezione in base a un punteggio di rilevanza rispetto alla query. Questo approccio è fondamentale per le query in linguaggio naturale (free text queries), dove l'utente non specifica operatori logici ma esprime un bisogno informativo tramite parole chiave . Il sistema assegna a ogni coppia query-documento un punteggio (score), tipicamente nell'intervallo $[0,1]$, e restituisce i top-$k$ documenti ordinati per punteggio decrescente.

### Scoring e Coefficiente di Jaccard

Un primo approccio intuitivo per calcolare la similarità tra query e documento è il **Jaccard Coefficient**, comunemente usato per misurare la sovrapposizione tra due insiemi $A$ e $B$. È definito come il rapporto tra la cardinalità dell'intersezione e la cardinalità dell'unione:

$$Jaccard(A, B) = \frac{|A \cap B|}{|A \cup B|}$$

Sebbene fornisca un valore normalizzato tra 0 e 1, il coefficiente di Jaccard è inadeguato per il ranking sofisticato perché ignora la frequenza dei termini (Term Frequency). Tratta il documento come un insieme di termini unici, perdendo l'informazione su quante volte un termine appare, che è un indicatore cruciale di rilevanza . Inoltre, non normalizza adeguatamente rispetto alla lunghezza del documento in modo utile per il retrieval.

### Term Frequency e Bag of Words

Per superare i limiti degli insiemi, si adotta il modello **Bag of Words**. In questo modello, un documento è rappresentato come un _multiset_ di termini, dove l'ordine delle parole viene ignorato ma si tiene traccia del numero di occorrenze.

La **Term Frequency** ($tf_{t,d}$) è il numero di volte che il termine $t$ appare nel documento $d$. Tuttavia, la rilevanza non cresce linearmente con la frequenza: un documento che contiene un termine 10 volte non è necessariamente 10 volte più rilevante di uno che lo contiene una volta. Per modellare questo comportamento, si utilizza spesso una scala logaritmica per "smorzare" l'effetto delle frequenze elevate. Il peso logaritmico $w_{t,d}$ è definito come:

$$w_{t,d} = \begin{cases} 1 + \log_{10}(tf_{t,d}) & \text{se } tf_{t,d} > 0 \\ 0 & \text{altrimenti} \end{cases}$$

Questo approccio assicura che la presenza del termine abbia un impatto significativo (il punteggio passa da 0 a 1 alla prima occorrenza), ma le occorrenze successive contribuiscono in modo marginale decrescente .

### Document Frequency e Inverse Document Frequency (IDF)

Non tutti i termini hanno lo stesso valore discriminante. I termini rari nella collezione sono molto più informativi di quelli comuni. Ad esempio, in una collezione generica, il termine "arachnocentric" è molto più specifico e utile per restringere la ricerca rispetto al termine "high" o "increase".

Per catturare questa intuizione si utilizza la **Document Frequency** ($df_t$), ovvero il numero di documenti nella collezione che contengono il termine $t$. Poiché vogliamo assegnare un peso maggiore ai termini rari (bassa $df$), si introduce la **Inverse Document Frequency (idf)**. Dato $N$ il numero totale di documenti, l'idf è definito come:

$$idf_t = \log_{10}\left(\frac{N}{df_t}\right)$$

L'uso del logaritmo serve anche qui a smorzare l'effetto, rendendo il peso gestibile. Un termine che appare in tutti i documenti ($df_t = N$) avrà un idf pari a 0, annullando il suo contributo al ranking (comportamento desiderabile per le stop words come "the", "a", "of") . È importante notare che l'$idf$ è una misura globale, unica per ogni termine nell'intera collezione, a differenza della $tf$ che è specifica per ogni documento.

### TF-IDF Weighting

Combinando le due misure precedenti, si ottiene lo schema di pesatura **tf-idf**, il più noto in Information Retrieval. Il peso di un termine $t$ in un documento $d$ è il prodotto del suo peso locale ($tf$) e del suo peso globale ($idf$):

$$w_{t,d} = (1 + \log tf_{t,d}) \times \log\left(\frac{N}{df_t}\right)$$

Questo sistema assegna i punteggi più alti ai termini che appaiono frequentemente in un documento specifico (alta $tf$) ma raramente nella collezione intera (alta $idf$), massimizzando il potere discriminante .

### Vector Space Model

Il Vector Space Model rappresenta documenti e query come vettori in uno spazio a $V$ dimensioni, dove $V$ è la dimensione del vocabolario. Ogni asse dello spazio corrisponde a un termine. Un documento $d$ è rappresentato dal vettore $\vec{d} = (w_{1,d}, w_{2,d}, \dots, w_{V,d})$, dove ogni componente è il peso tf-idf del termine corrispondente. Poiché la maggior parte dei termini non appare in un singolo documento, questi vettori sono estremamente sparsi (molti zeri) .

La similarità tra una query $q$ e un documento $d$ viene calcolata valutando la prossimità dei loro vettori nello spazio.

### Similarità Coseno (Cosine Similarity)

La semplice distanza Euclidea tra i punti finali dei vettori non è una buona misura di similarità in IR perché è sensibile alla lunghezza dei documenti. Due documenti con lo stesso contenuto ma lunghezze diverse (es. uno è la copia duplicata dell'altro) avrebbero vettori con la stessa direzione ma modulo diverso, risultando distanti nello spazio Euclideo.

Per ovviare a questo, si utilizza l'angolo tra i vettori. Minore è l'angolo, maggiore è la similarità. In pratica, si calcola il **coseno dell'angolo** tra il vettore query $\vec{q}$ e il vettore documento $\vec{d}$. Il coseno è 1 per vettori identici (angolo 0°) e 0 per vettori ortogonali (nessun termine in comune, angolo 90°).

La formula della **Cosine Similarity** è:

$$sim(q,d) = \cos(\theta) = \frac{\vec{q} \cdot \vec{d}}{||\vec{q}|| \cdot ||\vec{d}||} = \frac{\sum_{i=1}^{V} q_i d_i}{\sqrt{\sum_{i=1}^{V} q_i^2} \sqrt{\sum_{i=1}^{V} d_i^2}}$$

Il denominatore effettua una **normalizzazione della lunghezza** (Length Normalization), proiettando i vettori sulla superficie di un'ipersfera unitaria. Questo rende comparabili documenti di lunghezze diverse .

### BM25 (Best Match 25)

Il **BM25** (o Okapi BM25) è un'evoluzione probabilistica del tf-idf ed è considerato uno degli standard di riferimento (baseline) più forti nel retrieval moderno.

A differenza del tf-idf standard, il BM25 introduce due concetti chiave per migliorare la precisione:

1. **Saturazione della Term Frequency**: Nel tf-idf, il punteggio continua a crescere al crescere della frequenza, seppur logaritmicamente. Nel BM25, l'influenza di un termine ha un limite superiore (saturazione). Oltre un certo numero di occorrenze, ulteriori ripetizioni non aumentano significativamente il punteggio.
    
2. **Normalizzazione della lunghezza del documento**: La lunghezza è trattata in modo parametrico per penalizzare i documenti lunghi, ma con un controllo fine.

La formula per il peso di un termine nel BM25 è:

$$Score(D,Q) = \sum_{i=1}^{n} IDF(q_i) \cdot \frac{f(q_i, D) \cdot (k_1 + 1)}{f(q_i, D) + k_1 \cdot (1 - b + b \cdot \frac{|D|}{avgdl})}$$

Dove:

- $f(q_i, D)$ è la frequenza del termine nel documento.
    
- $|D|$ è la lunghezza del documento e $avgdl$ è la lunghezza media dei documenti nella collezione.
    
- $k_1$ è un parametro che controlla la saturazione della frequenza (tipicamente tra 1.2 e 2.0).
    
- $b$ è un parametro che controlla l'importanza della normalizzazione della lunghezza (tipicamente $0.75$). Se $b=1$, la normalizzazione è completa; se $b=0$, non c'è normalizzazione .