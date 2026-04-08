## <font color="#244061">Lezione 9: Learning to Rank (LtR) I - Fondamenti</font>

### <font color="#244061">Framework Generale</font>

Il **Learning to Rank** (LtR) rappresenta l'applicazione delle tecniche di Machine Learning (ML) alla costruzione di modelli di ranking. A differenza dei modelli di recupero classici (come il Modello Vettoriale o il BM25), dove la funzione di punteggio è determinata da formule analitiche con pochi parametri liberi, nel LtR la funzione di ranking $f(q, d)$ viene appresa automaticamente dai dati.

Il sistema viene addestrato su un _Training Set_ composto da query, documenti e, crucialmente, etichette di rilevanza (relevance judgments). Queste etichette rappresentano il _ground truth_ e possono essere binarie (rilevante/non rilevante) o graduate (scala da 0 a 4). L'obiettivo dell'algoritmo di apprendimento è minimizzare una funzione di costo (Loss Function) calcolata sulle predizioni rispetto alle etichette reali, generando un modello capace di ordinare documenti mai visti per nuove query in modo ottimale.

### <font color="#244061">Feature Engineering</font>

L'efficacia di un modello LtR dipende fortemente dalla qualità delle caratteristiche (features) estratte per rappresentare la coppia query-documento. Le feature non sono il testo grezzo, ma segnali numerici che sintetizzano diverse proprietà. Si dividono in tre macro-categorie.

Le **Feature Query-Dependent** catturano la relazione tra la query specifica e il documento. Queste includono i punteggi standard dei modelli classici (score BM25, score TF-IDF), la frequenza dei termini della query nel titolo o nel corpo del documento, e la posizione della prima occorrenza. Sono le feature più determinanti per stabilire la rilevanza tematica.

Le **Feature Query-Independent** descrivono la qualità intrinseca del documento a prescindere dalla ricerca effettuata. Esempi tipici sono il PageRank (importanza del nodo nel grafo web), il punteggio di affidabilità del dominio (Spam Score), la lunghezza del documento, la freschezza (data di aggiornamento) e la leggibilità. Queste feature aiutano a promuovere pagine autorevoli rispetto a pagine spazzatura, anche a parità di corrispondenza testuale.

Le **Feature Query-Level** (o Query-only) dipendono esclusivamente dalla query e non dai documenti. Possono includere la lunghezza della query, la sua popolarità storica (click-through rate aggregato) o la classificazione dell'intento (informazionale vs transazionale). Queste caratteristiche sono utili per modulare il peso delle altre feature; ad esempio, per una query di navigazione, le feature di URL match potrebbero essere ponderate maggiormente.

### <font color="#244061">Approcci Principali: Pointwise, Pairwise e Listwise</font>

Gli algoritmi di LtR vengono classificati in base a come la Loss Function considera gli esempi di training e come viene modellato il problema di ranking.

L'approccio **Pointwise** riduce il ranking a un problema classico di regressione o classificazione. Il modello prende in input una singola coppia query-documento e predice un punteggio assoluto (es. la probabilità di rilevanza). La Loss Function (spesso Mean Squared Error) cerca di avvicinare il punteggio predetto all'etichetta di rilevanza esatta. Il limite principale è che non considera l'ordine relativo tra i documenti: predire 4.1 invece di 4.0 per il primo documento è considerato un errore tanto quanto per il centesimo, sebbene l'impatto sul ranking visibile sia nullo nel secondo caso.

L'approccio **Pairwise** trasforma il problema in una classificazione binaria su coppie di documenti. Per una data query, il modello considera coppie $(d_i, d_j)$ e cerca di predire quale dei due è più rilevante. L'obiettivo è minimizzare il numero di **inversioni** (casi in cui un documento meno rilevante è classificato sopra uno più rilevante). Esempi famosi sono _RankNet_ e _LambdaRank_. Sebbene più efficace del Pointwise, questo metodo può soffrire se non pesato correttamente, poiché correggere un'inversione in fondo alla lista vale quanto correggerne una in cima (top-k).

L'approccio **Listwise** affronta il problema del ranking nella sua interezza. La Loss Function è definita direttamente sulla lista ordinata di documenti o su una metrica di valutazione IR (come NDCG o MAP). L'algoritmo cerca di ottimizzare direttamente la qualità dell'ordinamento finale. Poiché le metriche IR sono spesso non differenziabili (sono a gradini), si utilizzano approssimazioni o bounding (es. _LambdaMART_, _ListNet_, _SoftRank_). Questo è attualmente l'approccio stato dell'arte.

### <font color="#244061">Tuning e Ottimizzazione dei Parametri</font>

Prima di applicare modelli complessi di ML, il primo passo nel miglioramento del ranking è l'ottimizzazione dei parametri dei modelli esistenti, come il BM25F (BM25 field-weighted). Nel BM25F, il documento è visto come un insieme di campi (Titolo, Corpo, Anchor text), ciascuno con un peso diverso.

La frequenza del termine non è calcolata sul documento piatto, ma come somma pesata delle frequenze nei campi:
## $$\text{tf}_{t,d} = \sum_{c \in fields} w_c \cdot \text{tf}_{t,d,c}$$
I pesi $w_c$ (boost factors) e i parametri di saturazione $b_c$ per ogni campo sono iperparametri che devono essere sintonizzati. Poiché la funzione di valutazione (es. NDCG) non è convessa e non differenziabile rispetto a questi parametri, non si può usare la discesa del gradiente semplice. Si utilizzano invece tecniche di ricerca diretta come la Grid Search (esaustiva ma lenta) o la Coordinate Ascent (ottimizza un parametro alla volta tenendo fissi gli altri, iterando fino a convergenza). Questo tuning è una forma primitiva ma essenziale di Learning to Rank.