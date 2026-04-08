# Efficient Training and Inference of Ensembles of Decision Trees

### Gradient Boosting Machine (GBM)

Il **Gradient Boosting Machine (GBM)** è una tecnica di apprendimento supervisionato che costruisce un modello predittivo forte combinando molteplici "weak learners" (modelli deboli), tipicamente alberi di decisione. L'obiettivo è apprendere una funzione $F(x)$ come somma di $M$ modelli parziali:

$$F(x) = \sum_{i=0}^{M} F_i(x)$$

Questo approccio è utilizzato sia per compiti di regressione che di classificazione e trova implementazioni celebri in librerie come **XGBoost** e **LightGBM**.

Il processo di addestramento è iterativo. Si parte da una predizione iniziale $F_0(x)$ semplice (es. la media dei valori target per la regressione) e si procede aggiungendo nuovi alberi che correggono gli errori dei precedenti.

Ad ogni passo $m$, si calcolano i **pseudo-residui** $r_{i,m}$ (il gradiente negativo della funzione di costo rispetto alla predizione corrente) e si addestra un nuovo albero $h_m(x)$ per predire questi residui. Il modello viene quindi aggiornato:

$$F_m(x) = F_{m-1}(x) + \alpha \cdot y_j,_m$$

dove $\alpha$ è il _learning rate_. Questo processo equivale a muoversi nella direzione che minimizza la funzione di costo (es. Mean Squared Error o Cross Entropy) nello spazio delle funzioni.
### Ottimizzazioni in XGBoost

**XGBoost** (eXtreme Gradient Boosting) introduce diverse ottimizzazioni per scalare il GBM su grandi dataset:

- **Approximate Split Finding**: Invece di valutare tutti i possibili punti di split (che sarebbe computazionalmente oneroso), utilizza istogrammi e quantili per trovare split approssimati ma efficaci.
    
- **Parallelismo**: Sebbene i singoli alberi debbano essere costruiti sequenzialmente (il successivo dipende dal precedente), la costruzione di ogni singolo albero (finding dei nodi) viene parallelizzata.
    
- **Regularization**: Include termini di regolarizzazione nella funzione obiettivo per controllare la complessità degli alberi e prevenire l'overfitting.
    
- **Sparsity Awareness**: Gestisce efficientemente dati sparsi (molti zeri) imparando direzioni di default per i valori mancanti.
### Ottimizzazioni in LightGBM

**LightGBM** è progettato per essere ancora più veloce ed efficiente, introducendo due tecniche principali per ridurre la quantità di dati e feature da processare senza sacrificare significativamente l'accuratezza:

**1. Gradient-based One-Side Sampling (GOSS)**

GOSS campiona i dati di training basandosi sui gradienti. L'idea è che i punti dati con gradienti grandi (errori elevati) sono più importanti per l'apprendimento rispetto a quelli con gradienti piccoli (già ben modellati). Invece di un campionamento casuale uniforme, GOSS mantiene tutti i dati con gradienti grandi e campiona casualmente solo una frazione di quelli con gradienti piccoli, ri-pesandoli per mantenere corretta la distribuzione originale.

**2. Exclusive Feature Bundling (EFB)**

EFB riduce il numero di feature effettive raggruppando feature che sono "mutuamente esclusive" (cioè, che non assumono quasi mai valori non-nulli contemporaneamente). Questo è comune nei dati sparsi (es. one-hot encoding). Le feature vengono "impacchettate" in una singola feature densa, riducendo drasticamente la dimensionalità e velocizzando la costruzione degli istogrammi.

### Efficient Inference: Il problema dello Scoring

L'inferenza (o _scoring_) in un ensemble di foreste richiede di valutare migliaia di alberi per ogni documento. L'approccio ingenuo ("Naive") attraversa ogni albero nodo per nodo usando strutture `if-then-else`.

Questo metodo è inefficiente sulle moderne CPU a causa dei **Branch Mispredictions**. Poiché le decisioni negli alberi sono spesso imprevedibili e dipendono dai dati, la CPU non riesce a sfruttare efficacemente la pipeline e l'esecuzione speculativa, causando frequenti svuotamenti della pipeline e stalli. Inoltre, l'accesso alle feature in memoria è spesso casuale, riducendo l'efficacia della cache (**Low Cache Hit Ratio**).

### QuickScorer: Un approccio Bitwise

**QuickScorer** è un algoritmo innovativo che trasforma il problema dell'attraversamento degli alberi in operazioni bitwise (sui bit), eliminando completamente i salti condizionali (`if-then-else`) durante la valutazione dell'albero.

Il concetto chiave si basa sui **False Nodes**. In un albero di decisione binario completo, ogni foglia è identificata univocamente dal percorso preso. QuickScorer inverte la logica: invece di cercare la foglia corretta, elimina quelle irraggiungibili.

- A ogni nodo interno è associata una "condition check" (es. $feature < threshold$).
    
- Se la condizione è **Falsa** (si dovrebbe andare a destra), significa che tutte le foglie nel sottoalbero sinistro non possono essere la destinazione finale.
    
- Ogni nodo possiede una **Bitmask** che ha i bit settati a 0 corrispondenti alle foglie del sottoalbero sinistro (e 1 altrove).

**Algoritmo di Scoring:**

1. Si mantiene un vettore di bit `v` (inizialmente tutti 1), dove ogni bit rappresenta una possibile foglia di uscita.
    
2. Si valutano tutti i nodi dell'albero (o un sottoinsieme rilevante). Se la condizione di un nodo è falsa, si aggiorna `v` con un'operazione **AND** bit a bit: `v = v & mask_nodo`.
    
3. Questa operazione "spegne" i bit delle foglie che diventano irraggiungibili.
    
4. Alla fine, il bit più a sinistra rimasto a 1 indica la foglia di uscita corretta.

$$\text{Result} \leftarrow \text{Result} \ \text{AND} \ \text{Mask}_{\text{FalseNode}}$$

Questo approccio permette di processare l'albero (o persino intere foreste con V-QuickScorer e BlockWise-QS) sfruttando il parallelismo dei registri CPU e mantenendo un flusso di istruzioni lineare, risolvendo i problemi di branch misprediction.