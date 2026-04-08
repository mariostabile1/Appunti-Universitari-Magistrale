## <font color="#244061">Lezione 11: Training e Inferenza Efficiente di Alberi</font>
### Algoritmi di Split Finding: Exact Greedy vs Histogram-based

Il cuore dell'addestramento di un albero decisionale (o di un ensemble come nel Gradient Boosting) risiede nella ricerca del miglior punto di taglio (split) per ogni nodo. L'approccio classico è l'**Exact Greedy Algorithm**. Questo metodo enumera tutti i possibili punti di taglio per tutte le feature. Per ogni feature, i dati vengono ordinati e l'algoritmo scorre i valori per calcolare il guadagno di informazione (Information Gain) o la riduzione della varianza. Sebbene garantisca lo split matematicamente ottimale, è computazionalmente oneroso, con una complessità legata al numero di istanze $N$ e richiede un ordinamento dei dati ad ogni iterazione o in pre-processing.

Per accelerare questo processo, librerie moderne come LightGBM hanno introdotto l'approccio **Histogram-based**. Invece di valutare lo split su ogni singolo valore unico della feature, i valori continui vengono discretizzati in un numero fisso di contenitori (bin), costruendo un istogramma.

L'algoritmo cerca il miglior split solo sui confini dei bin (ad esempio 255 bin per un intero a 8 bit). Questo riduce la complessità da $O(\text{data} \times \text{features})$ a $O(\text{bins} \times \text{features})$, permettendo una riduzione drastica dell'uso della memoria e un aumento della velocità di training, con una perdita di precisione spesso trascurabile che può addirittura fungere da regolarizzazione.
## XGBoost: overview

XGBoost, acronimo di **eXtreme Gradient Boosting**, è una libreria di machine learning open-source ottimizzata che implementa algoritmi di **Gradient Boosting** sotto il framework dell'apprendimento supervisionato. L'obiettivo principale di XGBoost è fornire un sistema scalabile, portabile e accurato per problemi di regressione, classificazione e ranking. A differenza delle implementazioni tradizionali di boosting, XGBoost è ingegnerizzato per sfruttare al meglio le risorse hardware, utilizzando parallelizzazione nella costruzione degli alberi e gestione efficiente della memoria.
### Fondamenti del Gradient Boosting

Il principio di base su cui si fonda XGBoost è l'additive training. Invece di addestrare un singolo modello complesso o di addestrare modelli indipendenti in parallelo come nel Random Forest, il Gradient Boosting costruisce il modello in modo sequenziale. Si parte da un modello costante e, ad ogni iterazione, viene aggiunto un nuovo stimatore debole (weak learner), tipicamente un albero decisionale, che ha il compito di correggere gli errori residui commessi dai modelli precedenti.

Matematicamente, se $\hat{y}_i^{(t)}$ è la predizione per l'istanza $i$ all'iterazione $t$, il processo può essere descritto come l'aggiunta di una funzione $f_t$ che migliora la predizione precedente:
### $$\hat{y}_i^{(t)} = \hat{y}_i^{(t-1)} + f_t(x_i)$$
Qui $f_t(x_i)$ è il peso che il nuovo albero assegna all'istanza $x_i$. L'algoritmo non ottimizza direttamente i parametri dell'intero ensemble, ma cerca in modo greedy la funzione $f_t$ che minimizza una specifica funzione obiettivo.
### Funzione Obiettivo Regolarizzata

Una delle caratteristiche distintive di **XGBoost** rispetto al **Gradient Boosting** classico è l'inclusione esplicita di termini di regolarizzazione nella funzione obiettivo per controllare la complessità del modello e prevenire l'**overfitting**. La funzione obiettivo $\text{Obj}$ da minimizzare all'iterazione $t$ è composta da due parti: la loss function $L$, che misura la differenza tra la predizione e il valore reale, e il termine di regolarizzazione $\Omega$.
### $$\text{Obj}^{(t)} = \sum_{i=1}^n l(y_i, \hat{y}_i^{(t)}) + \sum_{k=1}^t \Omega(f_k)$$
Il termine di regolarizzazione $\Omega(f)$ penalizza la complessità dell'albero. Viene definito basandosi sul numero di foglie $T$ e sulla norma $L2$ dei punteggi (pesi) delle foglie $w$:
### $$\Omega(f) = \gamma T + \frac{1}{2}\lambda ||w||^2$$
Dove $\gamma$ rappresenta la riduzione minima della loss richiesta per effettuare un ulteriore partizionamento (split) su un nodo foglia, agendo di fatto come un meccanismo di _pruning_ (potatura) automatico durante la costruzione dell'albero, mentre $\lambda$ controlla la regolarizzazione $L2$ sui pesi, rendendo la predizione più conservativa.
### GOSS (Gradient-based One-side Sampling)

Nel Gradient Boosting, i dati non contribuiscono tutti allo stesso modo all'addestramento: le istanze con gradienti piccoli (piccolo errore) sono già ben apprese dal modello, mentre quelle con gradienti grandi (grande errore) necessitano di maggiore attenzione. Il **GOSS** è una tecnica di campionamento che sfrutta questa informazione per ridurre la dimensione del dataset di training senza perdere accuratezza.

Invece di un campionamento casuale uniforme, GOSS mantiene tutte le istanze con gradienti grandi (top $a \times 100\%$) e campiona casualmente una piccola porzione delle istanze con gradienti piccoli (top $b \times 100\%$). Per non alterare la distribuzione dei dati originale, i gradienti delle istanze campionate con errore piccolo vengono amplificati (rimoltiplicati) per un fattore costante $(1-a)/b$ durante il calcolo del guadagno di informazione. Questo permette al modello di concentrarsi sui casi difficili pur mantenendo una visione d'insieme corretta della distribuzione dei dati.

### EFB (Exclusive Feature Bundling)

Nei dataset ad alta dimensionalità, lo spazio delle feature è spesso sparso; molte feature sono mutuamente esclusive, ovvero non assumono mai valori non nulli contemporaneamente (come nel caso del One-Hot Encoding). **Exclusive Feature Bundling** (EFB) è una tecnica di riduzione della dimensionalità lossless che raggruppa (bundle) queste feature esclusive in una singola feature densa.

L'algoritmo tratta il problema come un problema di colorazione di un grafo, dove le feature sono nodi e gli archi rappresentano la co-occorrenza di valori non nulli (conflitti). Le feature che non sono connesse (o hanno pochi conflitti) possono essere unite nello stesso bundle. Per distinguere i valori originali all'interno del bundle, si aggiungono degli offset: se la feature A ha range $[0, 10)$ e la feature B ha range $[0, 20)$, nel bundle la feature B verrà mappata su $[10, 30)$. Questo riduce il numero di feature effettive su cui costruire l'istogramma senza perdere informazioni sui valori originali.
### QuickScorer e False Nodes' Masks

**QuickScorer** rappresenta lo stato dell'arte per l'inferenza di foreste di alberi su CPU. La sua innovazione principale è sostituire l'attraversamento nodo-per-nodo dell'albero con operazioni bitwise su tutto l'ensemble. QuickScorer non naviga l'albero; invece, valuta i predicati di split e determina l'uscita (la foglia) tramite maschere di bit.

Per ogni albero, ogni nodo foglia è associato a una bitmask (spesso chiamata **False Nodes' Mask**). Se una condizione di test (es. $x_i < \text{t}$) risulta falsa, significa che l'istanza non può raggiungere determinate foglie. Applicando l'AND logico tra le maschere associate ai test falliti, si ottiene un vettore di bit dove il primo bit a '1' (o l'ultimo, a seconda dell'implementazione) indica l'unica foglia valida (exit leaf) per quel documento.

Per massimizzare il throughput, QuickScorer utilizza un **attraversamento interleaved**: processa un blocco di documenti (es. 4 o 8) simultaneamente. Mentre la CPU attende il caricamento dei dati di una feature per il documento $D_1$, esegue calcoli per il documento $D_2$, nascondendo la latenza di accesso alla memoria e sfruttando i registri vettoriali (SIMD) per valutare i predicati in parallelo.