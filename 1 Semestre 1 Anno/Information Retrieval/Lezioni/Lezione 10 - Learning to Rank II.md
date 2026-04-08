## <font color="#244061">Lezione 10: Learning to Rank (LtR) II</font>

### Pipeline Multi-stage

L'architettura di un moderno motore di ricerca raramente si affida a un singolo modello per classificare l'intero corpus di documenti. Si adotta invece un approccio a Cascade, strutturato come una pipeline sequenziale di ranker, progettata per filtrare progressivamente i candidati.

Il primo stadio della cascata opera sull'intero indice ed è caratterizzato da algoritmi ad alta efficienza computazionale ma bassa complessità semantica. Questi modelli sono definiti Recall-oriented, poiché il loro obiettivo primario è recuperare il maggior numero possibile di documenti rilevanti, accettando anche una quantità significativa di falsi positivi pur di non scartare prematuramente informazioni utili.

I documenti sopravvissuti al primo taglio vengono passati agli stadi successivi, dove il numero di candidati diminuisce drasticamente (da milioni a centinaia o decine). In queste fasi finali si impiegano modelli **Precision-oriented**, come complessi algoritmi di Learning to Rank o reti neurali profonde. Questi ranker sono computazionalmente costosi ma estremamente accurati nel riordinare la lista finale (re-ranking), massimizzando metriche come la NDCG sulle prime posizioni.
### Trade-off Efficienza/Efficacia

La progettazione di sistemi di Information Retrieval è governata dal compromesso tra la qualità dei risultati (Efficacia) e il tempo di risposta (Efficienza). Il costo computazionale totale di un sistema di ranking può essere approssimato considerando tre fattori principali: il costo di estrazione delle feature, il numero di documenti candidati e la complessità del modello di scoring.

Le feature non hanno tutte lo stesso peso computazionale. Le feature statiche, come il PageRank o la lunghezza del documento, sono pre-calcolate e hanno un costo di accesso irrisorio. Al contrario, le feature query-dependent, che richiedono l'analisi in tempo reale della corrispondenza tra termini della query e contenuto del documento, o peggio le feature basate su modelli linguistici contestuali, impongono un carico elevato sulla CPU.

Per mantenere la latenza sotto soglie accettabili (tipicamente nell'ordine dei millisecondi), si agisce riducendo il numero di candidati $N$ attraverso la pipeline multi-stage o semplificando la complessità del modello $M$. Un modello lineare è estremamente veloce ma incapace di catturare relazioni non lineari tra le feature, mentre un ensemble di alberi o una rete neurale offrono prestazioni superiori al prezzo di un costo di inferenza molto più alto.
### Ottimizzazioni: Early Exit e modelli EET

Nei modelli basati su ensemble, come le foreste di alberi, è possibile ottimizzare il tempo di inferenza introducendo meccanismi di **Early Exit**. L'intuizione alla base è che non tutti i documenti richiedono la valutazione dell'intero insieme di alberi per essere classificati con certezza. Se, dopo aver valutato un sottoinsieme dei primi $k$ alberi, il punteggio cumulativo di un documento è sufficientemente basso da renderlo irrilevante, o sufficientemente alto da garantirgli una posizione in top-k, l'algoritmo può interrompere il calcolo per quel documento specifico.

Questa strategia riduce il costo medio di valutazione senza degradare significativamente le metriche di ranking. Si parla spesso di modelli o strutture dati ottimizzate per l'**Efficient Ensemble Trees** (EET), dove l'ordine di valutazione degli alberi o la struttura stessa dell'ensemble sono progettati per massimizzare la probabilità di uscita anticipata (pruning dinamico durante il scoring), risparmiando cicli di calcolo su documenti che non hanno speranza di apparire nei primi risultati.
### Alberi di Decisione: Regression Trees e Gradient Boosting

Nel contesto del Learning to Rank, gli algoritmi più performanti su dati tabulari e feature ingegnerizzate sono basati sugli alberi di decisione, specificamente i **Regression Trees**. Un albero di regressione suddivide lo spazio delle feature in regioni rettangolari disgiunte. Ogni nodo interno dell'albero rappresenta una condizione su una feature (ad esempio, "BM25 score > 1.5"), e ogni foglia contiene un valore numerico che rappresenta lo score parziale assegnato al documento che "cade" in quella regione.

La tecnica dominante per combinare questi alberi è il Gradient Boosting Machine (GBM), noto anche come Gradient Boosted Regression Trees (GBRT). A differenza del Bagging (usato nelle Random Forest) che costruisce alberi indipendenti in parallelo, il Gradient Boosting costruisce l'ensemble in modo sequenziale e additivo.

Ogni nuovo albero viene addestrato per correggere gli errori residui commessi dalla combinazione degli alberi precedenti. Matematicamente, ogni iterazione cerca di approssimare il gradiente negativo della funzione di perdita (Loss function) rispetto allo score di ranking corrente. Il punteggio finale di un documento è dato dalla somma pesata degli output di tutti gli alberi nell'ensemble. Questo approccio permette di modellare relazioni estremamente complesse e non lineari tra le feature, risultando spesso superiore ai modelli lineari e alle singole reti neurali su dataset di ranking strutturati.