La **Precision ($P$)** indica la frazione di documenti recuperati che sono rilevanti. Risponde alla domanda: "Quanto sono validi i risultati che ho trovato?".

$$P = \frac{TP}{TP + FP} = \frac{|Relevant \cap Retrieved|}{|Retrieved|}$$

La **Recall ($R$)** misura la capacità del sistema di trovare tutti i documenti rilevanti presenti nella collezione. Risponde alla domanda: "Ho trovato tutto quello che c'era da trovare?".

$$R = \frac{TP}{TP + FN} = \frac{|Relevant \cap Retrieved|}{|Relevant|}$$
#### F-Measure

La F-Measure (o F1-Score) è la media armonica tra Precision e Recall. Si utilizza per avere un'unica metrica che bilanci le due precedenti, penalizzando fortemente i valori molto bassi di una delle due componenti.

$$F_1 = 2 \cdot \frac{P \cdot R}{P + R}$$

In generale, la $F_\beta$ permette di pesare diversamente la Precision rispetto alla Recall. Se $\beta > 1$ si dà più peso alla Recall, se $\beta < 1$ si dà più peso alla Precision.

$$F_\beta = (1 + \beta^2) \cdot \frac{P \cdot R}{\beta^2 \cdot P + R}$$
La formula della **Precision at k (P@k)** può essere espressa in modo più formale utilizzando una funzione indicatrice per denotare la rilevanza dei documenti nelle singole posizioni del ranking:
### $$P@k = \frac{1}{k} \sum_{i=1}^{k} rel_i$$
La **Mean Average Precision (MAP)** è semplicemente la media aritmetica delle AP calcolate su un insieme di query $Q$. È una delle metriche più robuste per valutare la qualità globale del ranking.
$$MAP = \frac{1}{|Q|} \sum_{q \in Q} AP(q)$$
#### Mean Reciprocal Rank (MRR)

L'MRR è una metrica focalizzata sulla capacità del sistema di restituire il _primo_ risultato rilevante il più in alto possibile. Per ogni query, si considera il "Reciprocal Rank", ovvero l'inverso della posizione ($rank$) del primo documento rilevante.

$$MRR = \frac{1}{|Q|} \sum_{i=1}^{|Q|} \frac{1}{rank_i}$$
Se non viene trovato nessun documento rilevante, il reciprocal rank è 0.
#### Discounted Cumulative Gain (DCG e NDCG)

A differenza delle metriche precedenti che sono binarie (rilevante/non rilevante), il DCG gestisce livelli di rilevanza graduata (es. 0=irrilevante, 1=parziale, 2=ottimo). L'idea è che i documenti molto rilevanti devono apparire prima di quelli meno rilevanti. Il "gain" viene scontato (diviso) in base alla posizione logaritmica, perché un risultato utile in posizione 1 vale molto più dello stesso risultato in posizione 10.

Il **Discounted Cumulative Gain (DCG)** alla posizione $p$ è:

$$DCG_p = rel_1 + \sum_{i=2}^{p} \frac{rel_i}{\log_2(i)}$$

Oppure, nella versione che enfatizza maggiormente la rilevanza alta:

$$DCG_p = \sum_{i=1}^{p} \frac{2^{rel_i} - 1}{\log_2(i+1)}$$

Il **Normalized DCG (NDCG)** normalizza il DCG dividendolo per l'**Ideal DCG (IDCG)**, che è il massimo DCG ottenibile per quella query (calcolato ordinando i documenti per rilevanza decrescente ideale). Questo rende la metrica confrontabile tra query diverse, con valori tra 0 e 1.

$$NDCG_p = \frac{DCG_p}{IDCG_p}$$
#### TF-IDF

Il modello TF-IDF pesa i termini bilanciando la loro frequenza nel documento con la loro rarità nella collezione.

La **Term Frequency ($tf_{t,d}$)** misura quante volte il termine $t$ appare nel documento $d$. Spesso si usa la variante logaritmica per attenuare l'impatto di ripetizioni eccessive:
### $$wf_{t,d} = 1 + \log(tf_{t,d}) \quad \text{se } tf_{t,d} > 0, \text{ altrimenti } 0$$
L'**Inverse Document Frequency ($idf_t$)** misura quanto il termine è raro nell'intera collezione. Se $N$ è il numero totale di documenti e $df_t$ è il numero di documenti che contengono $t$:
### $$idf_t = \log \left( \frac{N}{df_t} \right)$$
#### BM25 (Okapi BM25)
Dato un termine $t$ e un documento $d$:

$$Score(d, q) = \sum_{t \in q} IDF(t) \cdot \frac{tf_{t,d} \cdot (k_1 + 1)}{tf_{t,d} + k_1 \cdot (1 - b + b \cdot \frac{|d|}{avgdl})}$$

- $|d|$: lunghezza del documento corrente.
    
- $avgdl$: lunghezza media dei documenti nella collezione.
    
- $k_1$: solitamente impostato tra 1.2 e 2.0. Se $k_1=0$, la TF viene ignorata.
    
- $b$: solitamente impostato a 0.75. Se $b=1$, si applica la normalizzazione completa; se $b=0$, nessuna normalizzazione.
