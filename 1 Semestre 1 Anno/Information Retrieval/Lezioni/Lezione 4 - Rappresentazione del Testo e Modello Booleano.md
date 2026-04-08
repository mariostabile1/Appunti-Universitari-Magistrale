## <font color="#3f3151">Lezione 4: Rappresentazione del Testo e Modello Booleano</font>

### <font color="#5f497a">Modelli di rappresentazione: Bag of Words e N-grammi</font>

La trasformazione del testo non strutturato in una struttura dati elaborabile da un algoritmo richiede un processo di modellazione che inevitabilmente comporta una perdita di informazione. Il modello più comune e fondamentale è il **Bag of Words** (BOW). In questo approccio, un documento viene immaginato come un "sacco" in cui vengono inserite le parole che lo compongono, ignorando completamente la struttura grammaticale, sintattica e l'ordine di apparizione dei termini. L'unica informazione preservata è la molteplicità, ovvero quante volte un termine appare nel documento. Sebbene sembri una semplificazione eccessiva, il BOW è spesso sufficiente per compiti di classificazione e retrieval di base, poiché il lessico utilizzato è spesso un forte indicatore dell'argomento trattato.

Per mitigare la perdita di contesto locale causata dal BOW, si utilizza il modello degli N-grammi. Un N-gramma è una sequenza contigua di $N$ elementi estratta da un dato testo o discorso.

Quando gli elementi sono parole, gli N-grammi permettono di catturare frasi o espressioni idiomatiche. Ad esempio, nel caso dei bi-grammi ($N=2$), la frase "il cielo è blu" genera i token "il cielo", "cielo è", "è blu".

Esistono anche gli N-grammi di caratteri, dove la sequenza mobile considera le singole lettere. Questi sono particolarmente utili per gestire errori di battitura (spelling correction) o lingue morfologicamente ricche, poiché permettono di identificare radici comuni o suffissi indipendentemente dalla parola intera esatta. Tuttavia, all'aumentare di $N$, la dimensione del vocabolario tende a esplodere, aumentando la sparsità dei dati.

### <font color="#5f497a">Leggi statistiche: Legge di Zipf e impatto delle Stopwords</font>

L'analisi della distribuzione delle parole nei testi naturali rivela che non tutti i termini hanno la stessa importanza o frequenza. Questa distribuzione è descritta empiricamente dalla Legge di Zipf.

La legge afferma che, dato un corpus di linguaggio naturale, la frequenza $f$ di una parola è inversamente proporzionale al suo rango $r$ (la sua posizione nella classifica delle parole più frequenti). Formalmente:
## $$f(r) \propto \frac{1}{r^\alpha}$$
dove $\alpha$ è un parametro vicino a 1.

Ciò implica che ci sono pochissime parole estremamente frequenti e una lunghissima coda di parole molto rare.

Le parole ad altissima frequenza, che occupano le prime posizioni del rango, sono generalmente termini funzionali come articoli, preposizioni e congiunzioni (in inglese the, a, to, of). Questi termini sono definiti Stopwords.

Poiché appaiono in quasi tutti i documenti, le stopwords hanno un potere discriminante quasi nullo per il retrieval: non aiutano a distinguere un documento rilevante da uno non rilevante. Pertanto, è pratica comune rimuoverle durante la fase di indicizzazione per ridurre la dimensione dell'indice e il rumore semantico, sebbene in alcuni contesti moderni (come la ricerca di frasi esatte) la loro presenza possa essere necessaria.
### <font color="#5f497a">Vector Space Model (VSM)</font>

Per applicare l'algebra lineare all'Information Retrieval, si adotta il Vector Space Model (VSM). In questo modello, sia i documenti che le query sono rappresentati come vettori in uno spazio euclideo ad alta dimensionalità.

Ogni dimensione dello spazio corrisponde a un termine distinto del vocabolario $V$. Se il vocabolario contiene $|V|$ termini, lo spazio avrà $|V|$ dimensioni.

Un documento $d_j$ è rappresentato dal vettore $\vec{d_j} = (w_{1,j}, w_{2,j}, ..., w_{|V|,j})$, dove ogni componente $w_{i,j}$ rappresenta il peso del termine $i$ nel documento $j$. Nel caso più semplice, il peso può essere binario (1 se il termine è presente, 0 altrimenti) o basato sulla frequenza di occorrenza.

Una caratteristica cruciale di questi vettori è la loro sparsità. Dato che un singolo documento contiene solo una piccolissima frazione delle parole totali del vocabolario, la stragrande maggioranza delle componenti del vettore sarà pari a zero. Questa proprietà viene sfruttata computazionalmente utilizzando strutture dati efficienti come le Inverted Lists.
### <font color="#5f497a">Retrieval Booleano</font>

Il modello di retrieval più antico e, per certi versi, più intuitivo è il **Retrieval Booleano**. Questo approccio si basa sulla teoria degli insiemi e sulla logica booleana esatta. Le query sono formulate combinando i termini con gli operatori logici **AND**, **OR** e **NOT**.

- **AND** (intersezione): Restituisce i documenti che contengono _tutti_ i termini specificati (riduce il set di risultati).
    
- **OR** (unione): Restituisce i documenti che contengono _almeno uno_ dei termini (espande il set di risultati).
    
- **NOT** (differenza): Esclude i documenti che contengono il termine specificato.

![Immagine di Venn diagram boolean operators AND OR NOT](https://encrypted-tbn1.gstatic.com/licensed-image?q=tbn:ANd9GcQyR-gYK-OcxEcBdQEKNOwpUZarMd6UNFj2X9D4H8VhVPoa-skkOP3qpAw35XrmFCta40-c94uyH8eUakcZAW0wPdLEyjsInZOCR-0LVueZRK5qj4w)

Il limite principale di questo modello è noto come il problema del "**feast or famine**" (banchetto o carestia). Poiché il modello booleano non prevede un ranking (ordinamento) dei risultati, ma solo una decisione binaria (rilevante/non rilevante), l'utente si trova spesso di fronte a due estremi:

1. Query troppo generiche (spesso con OR) restituiscono un numero enorme di documenti ("feast"), rendendo impossibile l'analisi manuale.
    
2. Query troppo specifiche (spesso con molti AND) non restituiscono alcun risultato ("famine"), poiché nessun documento soddisfa contemporaneamente tutte le condizioni rigorose.

### <font color="#5f497a">Scoring iniziale: Coefficiente di Jaccard</font>

Per superare la limitazione binaria del modello booleano e introdurre una forma di ordinamento basata sulla similarità, si può utilizzare il Coefficiente di Jaccard.

Questo coefficiente misura la sovrapposizione tra due insiemi, tipicamente l'insieme dei termini della query $A$ e l'insieme dei termini del documento $B$.

Matematicamente è definito come il rapporto tra la cardinalità dell'intersezione e la cardinalità dell'unione degli insiemi:
## $$J(A, B) = \frac{|A \cap B|}{|A \cup B|}$$
Il valore varia tra 0 (nessun termine in comune) e 1 (insiemi identici).

Sebbene offra un primo metodo di scoring, il coefficiente di Jaccard presenta limiti significativi per l'Information Retrieval avanzato.

In primo luogo, non considera la frequenza dei termini: una parola che appare una volta conta quanto una che appare dieci volte. In secondo luogo, non penalizza adeguatamente i documenti molto lunghi o molto brevi in modo sofisticato, né tiene conto della rarità dei termini (una stopword in comune aumenta lo score tanto quanto un termine tecnico raro). Tratta semplicemente i documenti come "sacchi di parole" binari senza pesi.