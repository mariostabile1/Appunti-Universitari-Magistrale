## Intrusion Detection Systems (IDS): Tassonomia e Metodologie

Il rilevamento delle intrusioni e dei malware si basa sull'analisi di eventi generati dal sistema per identificare attività malevole. Le metodologie di rilevamento si classificano in tre macro-categorie principali: basate su firme (_Signature-based_), basate su anomalie (_Anomaly-based_) e basate su specifiche (_Specification-based_). Ciascuna di queste può essere implementata utilizzando approcci statici, dinamici o ibridi .

### Sorgenti Dati: Endpoint e Network

Gli IDS necessitano di sensori per raccogliere informazioni. Se le firme o le analisi sono definite su eventi locali di un nodo (es. invocazioni al sistema operativo, analisi della memoria, file scaricati), si parla di **Host IDS** o sistemi di protezione _Endpoint_. Questi sensori spesso si "agganciano" (_hook_) alle chiamate di sistema per monitorare il comportamento dei programmi .

Se invece l'analisi si concentra sul traffico di rete (contenuto dei pacchetti, flussi di comunicazione), si parla di **Network IDS**. Questi sistemi si basano su moduli _sniffer_ che monitorano l'infrastruttura. Tuttavia, l'approccio di rete deve affrontare sfide come la perdita di messaggi (se lo sniffer non tiene il passo con la velocità della rete) e gli attacchi di _evasione_, dove l'attaccante manipola i pacchetti (es. frammentazione sovrapposta) per eludere il rilevamento senza alterare il payload finale consegnato al target .

Esistono anche approcci che analizzano i **Log** di sistema per scoprire attacchi _a posteriori_, utili per identificare intrusioni che potrebbero essere sfuggite ai controlli in tempo reale .

## Metodologie di Rilevamento

### Rilevamento basato su Firme (Signature-based)

Questo approccio si fonda sull'idea che il malware o l'attacco possieda caratteristiche univoche identificabili, dette **firme**. Queste possono essere sequenze di byte statiche (es. in un file binario), stringhe specifiche nel traffico di rete (es. URL malevoli), o pattern comportamentali . Il sistema confronta gli eventi osservati con un database di firme note. Questo metodo garantisce una **precisione molto elevata** (pochi falsi positivi) per le minacce conosciute (_Known-Known_), ma è inefficace contro attacchi nuovi (**0-day**) o varianti non ancora catalogate del malware.

L'approccio può essere:

- **Statico:** Analizza il codice o il file senza eseguirlo. È il metodo tradizionale degli antivirus .
    
- **Dinamico:** Esegue il programma in un ambiente protetto (sandbox o macchina virtuale) e confronta il comportamento osservato con le firme .
    
- **Ibrido:** Combina l'analisi statica per filtrare i programmi sospetti e quella dinamica per la conferma .

### Rilevamento basato su Anomalie (Anomaly-based)

A differenza dell'approccio basato su firme, il rilevamento di anomalie costruisce un modello del **comportamento normale** del sistema durante una fase di apprendimento (_learning_). I parametri appresi possono includere l'orario di utilizzo dei servizi, la durata delle sessioni, le funzioni del SO invocate o la larghezza di banda utilizzata .

Una volta costruito il modello, qualsiasi comportamento che si discosta significativamente da esso (superando una certa soglia di distanza) viene segnalato come anomalia e potenziale intrusione. Questo metodo è in grado di rilevare attacchi sconosciuti (_Unknown-Unknown_), inclusi gli 0-day, ma tende a generare un numero elevato di falsi positivi se il modello di normalità non è accurato o se il comportamento legittimo cambia nel tempo (_drift_) .

### Rilevamento basato su Specifiche (Specification-based)

In questo modello, il comportamento "normale" non viene appreso statisticamente, ma dedotto dalle **specifiche** del programma o del protocollo. Un approccio statico può utilizzare l'output di un'analisi del codice per determinare cosa il programma _dovrebbe_ fare, mentre un approccio dinamico confronta l'esecuzione in tempo reale con le specifiche previste. È un metodo potente per ridurre i falsi positivi tipici dell'anomaly detection, ma richiede la disponibilità e la formalizzazione delle specifiche .

## Metriche di Valutazione e ROC Curve

Per valutare l'efficacia di un IDS, si utilizzano metriche derivate dalla matrice di confusione, che incrocia la realtà (attacco presente/assente) con il risultato del test (allarme/non allarme).

Le metriche fondamentali sono:

- **Accuracy:** La percentuale complessiva di classificazioni corrette. $Accuracy = \frac{TN+TP}{TN+FP+TP+FN}$.
    
- **Precision:** La capacità di non generare falsi allarmi. Risponde alla domanda: "Quando il sistema segnala un'intrusione, quanto spesso ha ragione?". È cruciale quando il costo di investigare un falso positivo è alto. $Precision = \frac{TP}{TP+FP}$ .
    
- **Recall (o Sensitivity):** La capacità di individuare tutte le intrusioni reali. Risponde alla domanda: "Quando c'è un'intrusione, quanto spesso il sistema la rileva?". È fondamentale quando mancare un attacco (falso negativo) ha conseguenze disastrose. $Recall = \frac{TP}{TP+FN}$ .
    
- **F-score:** Una media armonica che bilancia Precision e Recall. $F\text{-}score = \frac{2 \cdot Recall \cdot Precision}{Recall + Precision}$.


La relazione tra il tasso di veri positivi (_True Positive Rate_) e il tasso di falsi positivi (_False Positive Rate_) al variare della soglia di rilevamento è descritta dalla **curva ROC (Receiver Operating Characteristic)**. L'area sotto la curva (**AUC**) rappresenta la probabilità che il classificatore valuti un'istanza positiva casuale più alta di una negativa casuale; un AUC maggiore indica un sistema migliore .

## Alert Fatigue e Gestione degli Eventi

Un problema critico nei moderni Security Operations Center (SOC) è l'**Alert Fatigue**. I team di sicurezza sono sommersi da un volume ingestibile di avvisi: il 70% degli analisti si sente sopraffatto e il 55% ammette di perdere allarmi critici a causa del rumore di fondo. Si stima che un SOC gestisca migliaia di allarmi al giorno, con un alto tasso di falsi positivi .

Per mitigare questo problema, non basta la semplice aggregazione. Le strategie efficaci includono:

- **Correlazione:** Collegare logicamente gruppi di avvisi per identificare un evento complesso e assegnare una priorità .
    
- **Filtering e Tuning:** Sopprimere gli avvisi noti come falsi positivi o non necessari (tuning) e filtrare la telemetria in ingresso .
    
- **Enrichment:** Arricchire automaticamente gli avvisi con contesto aggiuntivo (es. dati di threat intelligence) per facilitare l'indagine e ridurre il carico cognitivo sugli analisti .
    
- **Prioritizzazione:** Utilizzare algoritmi (anche basati su Machine Learning) per classificare il rischio degli avvisi, permettendo agli analisti di concentrarsi sulle minacce reali, sebbene questo comporti il rischio di trascurare eventi a bassa priorità che potrebbero essere precursori di attacchi gravi .