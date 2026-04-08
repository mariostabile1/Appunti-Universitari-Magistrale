## Question #1: Describe the different approaches used in literature to evaluate the performance of a search system. The answer should cover at least the definitions of Precision, Recall, DCG, and NDCG. Why these metrics belong to different lies of literature? What are the main differences between them?

La valutazione delle prestazioni di un sistema di ricerca si basa sulla misurazione della capacità del sistema di restituire documenti rilevanti per l'utente. Storicamente, le metriche si sono evolute da una visione binaria della rilevanza a una graduata. La **Precision** è definita come la frazione di documenti recuperati che sono effettivamente rilevanti rispetto alla query, ovvero $P = \frac{|Rel \cap Ret|}{|Ret|}$. Al contrario, la **Recall** misura la capacità del sistema di trovare tutti i documenti rilevanti presenti nella collezione, calcolata come $R = \frac{|Rel \cap Ret|}{|Rel|}$. Queste metriche classiche trattano la rilevanza come un valore booleano (0 o 1) e non tengono conto della posizione dei documenti nella lista dei risultati.

Con l'avvento del Web search, l'enfasi si è spostata sull'ordine dei risultati, portando alla definizione del **Discounted Cumulative Gain** (DCG). Il DCG introduce due concetti fondamentali: la rilevanza graduata (un documento può essere "molto rilevante", "parzialmente rilevante" o "non rilevante") e il decadimento logaritmico della posizione. L'idea è che l'utilità (gain) di un documento per l'utente diminuisce man mano che la sua posizione nella classifica aumenta. La formula tipica è $DCG_p = \sum_{i=1}^{p} \frac{rel_i}{\log_2(i+1)}$. Tuttavia, il DCG non è confrontabile tra query diverse poiché il suo valore dipende dal numero di documenti rilevanti disponibili. Per ovviare a ciò, si utilizza il **Normalized Discounted Cumulative Gain** (NDCG), calcolato come il rapporto tra il DCG ottenuto dal sistema e l'Ideal DCG (IDCG), ovvero il punteggio massimo teorico ottenibile ordinando i documenti in modo perfetto: $NDCG = \frac{DCG}{IDCG}$.

Queste metriche appartengono a "filoni" diversi della letteratura perché rispondono a esigenze evolutive. Precision e Recall derivano dal recupero di informazioni classico (biblioteche, basi dati legali) dove l'obiettivo è spesso la completezza o l'accuratezza pura su insiemi non ordinati. DCG e NDCG appartengono alla letteratura moderna del Web Search e del Learning-to-rank, dove l'attenzione è focalizzata sull'esperienza dell'utente che interagisce quasi esclusivamente con i primi risultati della lista (top-k). La differenza principale risiede dunque nella sensibilità alla posizione (ranking-awareness) e nella granularità del giudizio di rilevanza.

---

## Question #2: Please detail all the fundamental concepts behind TF-IDF and BM25. Why the second one is considered an important evolution of the first one?

Il modello **TF-IDF** (Term Frequency-Inverse Document Frequency) è uno schema di pesatura che valuta l'importanza di un termine in un documento all'interno di un corpus. Si compone di due parti: la **Term Frequency** (TF), che assume che un termine che appare più spesso in un documento sia più rappresentativo del suo contenuto, e l'**Inverse Document Frequency** (IDF), che penalizza i termini troppo comuni nel dataset (come le stop-words), premiando quelli rari che hanno maggior potere discriminante. Sebbene efficace, il TF-IDF soffre di limitazioni dovute alla crescita lineare del punteggio con la frequenza del termine e alla mancanza di una gestione robusta della lunghezza dei documenti.

Il **BM25** (Best Matching 25) è considerato l'evoluzione fondamentale del TF-IDF poiché introduce una funzione di ranking probabilistica più sofisticata. La sua importanza risiede nell'introduzione di due meccanismi critici: la saturazione della frequenza del termine e la normalizzazione della lunghezza del documento. A differenza del TF-IDF, nel BM25 l'aumento della frequenza di un termine apporta un contributo marginale decrescente al punteggio totale; dopo un certo numero di occorrenze, il "guadagno" di rilevanza si stabilizza, evitando che documenti eccessivamente ripetitivi dominino la classifica. Inoltre, il BM25 penalizza i documenti molto lunghi (che potrebbero contenere molti termini solo per caso) e premia i documenti brevi che contengono i termini della query, utilizzando un parametro $b$ per calibrare l'intensità di questa normalizzazione rispetto alla lunghezza media dei documenti nel corpus.

---

## Question #3: Simulate the evaluation of the query Pluto AND NOT Pippo using the Term-At-A-Time (TAAT) strategy, given the following inverted lists: Pluto: 2, 3, 5, 9; Pippo: 3, 4, 5, 11, 12. Show the main steps of the query processing and determine the set of documents that satisfy the query.

La strategia **Term-At-A-Time** (TAAT) prevede il processamento sequenziale di intere liste invertite, termine per termine, accumulando i risultati in una struttura dati temporanea (accumulatori). Per la query booleana `Pluto AND NOT Pippo`, il processo segue questi passaggi:

1. **Inizializzazione**: Si crea un set di accumulatori vuoto.
    
2. **Processamento di "Pluto"**: Si scorre l'intera lista invertita di Pluto. Gli ID dei documenti incontrati (2, 3, 5, 9) vengono inseriti negli accumulatori. In una query booleana, questi rappresentano i candidati iniziali.
    
    - _Stato accumulatori_: {2, 3, 5, 9}
        
3. **Processamento di "NOT Pippo"**: Si scorre la lista invertita di Pippo (3, 4, 5, 11, 12). Poiché l'operatore è `NOT`, per ogni documento presente nella lista di Pippo, se esso è presente negli accumulatori, deve essere rimosso.
    
    - Documento 3: Presente negli accumulatori $\rightarrow$ rimosso.
        
    - Documento 4: Non presente $\rightarrow$ ignorato.
        
    - Documento 5: Presente negli accumulatori $\rightarrow$ rimosso.
        
    - Documento 11 e 12: Non presenti $\rightarrow$ ignorati.
        
4. **Risultato Finale**: Gli accumulatori rimasti dopo il processamento di tutti i termini costituiscono il set di risposta.
    
    - _Documenti rimanenti_: {2, 9}
        

I documenti che soddisfano la query sono **2** e **9**.

---

## Question #4: Encode the number 11 using both Elias Gamma and Elias Delta coding schemes.

Per codificare un numero intero $x=11$ con i sistemi di Elias:

- **Elias Gamma**:
    
    1. Si rappresenta $x$ in binario: $11 = 1011_2$.
        
    2. Si conta il numero di bit meno uno: $N = 4-1 = 3$.
        
    3. Si scrivono $N$ zeri seguiti da un uno (prefisso unario): `0001`.
        
    4. Si appendono i bit del numero binario escludendo il bit più significativo (suffisso): `011`.
        
    
    - **Risultato Elias Gamma (11)**: `0001011`
        
- **Elias Delta**:
    
    1. Si calcola $L = 1 + \lfloor \log_2(11) \rfloor = 4$.
        
    2. Si codifica $L=4$ in Elias Gamma:
        
        - $4 = 100_2$.
            
        - Bit meno uno: $2 \rightarrow$ Unario: `001`.
            
        - Suffisso (senza il primo bit): `00`.
            
        - Gamma(4) = `00100`.
            
    3. Si appendono i bit del numero originale $11$ escludendo il primo bit: `011`.
        
    
    - **Risultato Elias Delta (11)**: `00100011`
        

### Prefix-free e Variable Byte

Un codice per interi deve essere **prefix-free** (senza prefissi) per garantire la decodificabilità univoca durante la lettura di un flusso continuo di bit. Se un codice fosse il prefisso di un altro, il decodificatore non saprebbe se fermarsi o continuare a leggere per identificare un numero più grande, rendendo impossibile separare i valori in una lista invertita compressa.

I vantaggi principali del **Variable Byte** (VByte) rispetto ai codici di Elias riguardano l'efficienza computazionale: essendo allineato ai byte, il VByte è molto più veloce da decodificare poiché sfrutta le operazioni native della CPU, evitando manipolazioni di singoli bit costose in termini di cicli di clock. Tuttavia, il principale svantaggio è lo spreco di spazio (compressione meno densa) per numeri molto piccoli, poiché il VByte utilizza sempre almeno 8 bit (di cui uno di controllo), mentre Elias Gamma può rappresentare piccoli interi con pochissimi bit.