## <font color="#3f3151">Lezione 2: Valutazione dei Sistemi di IR</font>

### <font color="#5f497a">Metodologia Cranfield e Benchmark</font>

La felicità dell'utente è un concetto elusivo da misurare direttamente, pertanto nell'Information Retrieval si utilizza comunemente la rilevanza dei risultati di ricerca come proxy principale per valutarla. Questo approccio sperimentale trova le sue radici negli esperimenti di Cranfield, pionieristici studi condotti da Cyril Cleverdon che hanno stabilito il paradigma standard per la valutazione dei sistemi. Per misurare la rilevanza in modo scientifico e replicabile, è necessario stabilire un benchmark standardizzato composto da tre elementi fondamentali: una collezione di documenti di riferimento, una suite di query (che rappresentano i bisogni informativi) e un insieme di giudizi di rilevanza (relevance judgments), ovvero una valutazione esplicita che determina se uno specifico documento è rilevante o meno ("Relevant" o "Nonrelevant") per una data query.
### <font color="#5f497a">Text REtrieval Conference (TREC) e la tecnica del Pooling</font>

Un ruolo centrale nella standardizzazione della valutazione è svolto dalla Text REtrieval Conference (TREC), sponsorizzata dal NIST, che fornisce l'infrastruttura necessaria per test su larga scala delle tecnologie di IR. Le attività del TREC sono organizzate in "track", aree di interesse specifiche che astraggono compiti dell'utente reali (user tasks). Quando un sistema di retrieval esegue un task su una collezione di test, il risultato prodotto viene definito "run".

Poiché valutare manualmente la rilevanza di ogni documento per ogni query in collezioni vaste (come quelle da 5 milioni di documenti) è impraticabile per limiti di tempo e costi, si utilizza la tecnica del "Pooling". Questa consiste nel raccogliere i primi risultati (top results) restituiti da diverse run e combinarli per formare un pool; solo i documenti inclusi in questo pool vengono sottoposti al giudizio degli assessori umani. I documenti non inclusi nel pool (unpooled) e quindi non giudicati, vengono assunti come non rilevanti durante la fase di valutazione.

### <font color="#5f497a">Metriche non ponderate (Unranked Measures)</font>

Quando si opera con giudizi di rilevanza binari, si utilizzano metriche non ponderate che valutano l'insieme dei documenti recuperati senza considerare il loro ordinamento.

La Precision (Precisione) è definita come la frazione di documenti recuperati che sono effettivamente rilevanti ($P = \frac{tp}{tp + fp}$).

Il Recall (Richiamo) rappresenta la frazione di documenti rilevanti totali presenti nella collezione che il sistema è riuscito a recuperare ($R = \frac{tp}{tp + fn}$).
![[Pasted image 20260104161353.png]]
Poiché esiste spesso un compromesso tra queste due misure, si utilizza la **F-Measure** (o F-Score), che rappresenta la media armonica tra precisione e recall. La media armonica è preferita a quella aritmetica perché penalizza fortemente i valori molto bassi di una delle due componenti, restando più vicina al minimo tra i due valori. Esiste anche una versione ponderata della F-Measure che, attraverso un parametro $\beta$, permette di attribuire maggiore peso alla precisione (con $\beta < 1$) o al recall (con $\beta > 1$).
### <font color="#5f497a">Metriche basate sul Rank</font>

Nei sistemi di ricerca reali, l'ordine di presentazione è cruciale e le metriche unranked non sono sufficienti.

La Precision@K (P@K) calcola la percentuale di documenti rilevanti considerando solo i primi $K$ risultati restituiti (ignora tutto ciò che è oltre il rank $K$); questa metrica è fondamentale nella ricerca web dove l'utente esamina solo i primi risultati.

Per una valutazione che consideri tutte le posizioni dei documenti rilevanti, si utilizza la Mean Average Precision (MAP). La MAP è la media aritmetica (macro-averaging) dei valori di Average Precision (AP) calcolati per un insieme di query. L'Average Precision per una singola query è la media dei valori di precisione calcolati in corrispondenza di ogni documento rilevante recuperato nella lista ordinata.

In scenari specifici come il "known-item search" o le query navigazionali, dove l'utente cerca un'unica risposta corretta o un fatto specifico, si utilizza il Mean Reciprocal Rank (MRR). L'MRR calcola il reciproco della posizione (rank) del primo documento rilevante trovato; questa misura riflette lo sforzo dell'utente nel trovare la risposta corretta.
### <font color="#5f497a">Rilevanza Graduata: DCG e NDCG</font>

Le metriche binarie non catturano le sfumature di utilità tra documenti diversi. Per questo si adottano metriche che supportano una rilevanza graduata (es. scala 0-3), basate su due assunzioni: i documenti molto rilevanti sono più utili di quelli marginalmente rilevanti e la loro utilità decresce se appaiono in posizioni basse della classifica.

Il Discounted Cumulative Gain (DCG) accumula il guadagno (gain) fornito dalla rilevanza dei documenti, applicando uno sconto basato sulla posizione logaritmica del risultato (tipicamente $1/\log_2(\text{rank})$). Una formulazione alternativa, spesso usata dai motori di ricerca, enfatizza ulteriormente i documenti altamente rilevanti utilizzando $2^{rel_i} - 1$ al numeratore.

Per confrontare le performance su query diverse, che potrebbero avere un numero variabile di risultati rilevanti disponibili, si utilizza il Normalized Discounted Cumulative Gain (NDCG). L'NDCG normalizza il DCG dividendolo per l'Ideal DCG (IDCG), ovvero il valore massimo di DCG ottenibile se i risultati fossero ordinati in modo perfetto per rilevanza decrescente.