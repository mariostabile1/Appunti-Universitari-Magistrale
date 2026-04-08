## Analisi delle Gang Ransomware tramite MITRE ATT&CK

L'utilizzo della matrice MITRE ATT&CK si rivela fondamentale per l'investigazione e la profilazione dei gruppi criminali moderni, in particolare quelli legati al _Ransomware-as-a-Service_ (RaaS). Analizzando i report di crimeware (come quelli di Kaspersky), è possibile mappare le TTP (_Tactics, Techniques, and Procedures_) utilizzate dalle principali gang, come Conti/Ryuk, Pysa, Clop, Hive, Lockbit 2.0 e altri. Questa mappatura evidenzia le "Common TTPs", ovvero le tecniche ricorrenti che costituiscono il _modus operandi_ condiviso o distintivo di questi attori.

Dall'analisi comparativa emerge come diversi gruppi sfruttino vettori d'attacco simili. Per l'accesso iniziale (_Initial Access_), tecniche come i **Remote Services** esterni (T1133), lo sfruttamento di applicazioni _public-facing_ (T1190) e il **Phishing** (T1566) sono onnipresenti. Una volta ottenuto l'accesso, l'esecuzione malevola (T1204.002) e l'uso di interpreti di comando e scripting (T1059) sono standard. Per la persistenza e il movimento laterale, si osserva un uso massiccio di **Windows Management Instrumentation (WMI)** (T1047) e la manipolazione dei token di accesso (T1134). Tecniche di evasione come la deobfuscazione di file (T1140) e il _masquerading_ (T1036.003) sono impiegate per nascondere le tracce.

Approfondendo l'analisi, si nota che la scoperta degli account (T1087) e la manipolazione degli stessi (T1098) sono preludi comuni all'escalation dei privilegi. L'uso di binari firmati per l'esecuzione proxy (T1218) e l'iniezione di processi (T1055) dimostrano un livello di sofisticazione volto a eludere i controlli di sicurezza tradizionali. Queste osservazioni confermano il principio che "non c'è insegnante migliore del nemico": studiare le TTP reali permette di anticipare le mosse degli attaccanti.

## Adversary Emulation

L'**Adversary Emulation** è l'applicazione pratica della _Threat Intelligence_ nelle operazioni di sicurezza. Consiste nel simulare le tattiche e le tecniche di uno specifico avversario in un ambiente controllato per valutare l'efficacia delle difese. A differenza dei _penetration test_ generici, l'emulazione si concentra su minacce reali e pertinenti per l'organizzazione, replicando scenari di attacco end-to-end.

### Strumenti di Emulazione: CALDERA

**CALDERA** è un framework di emulazione automatica sviluppato da MITRE. Costruito sulla base del framework ATT&CK, opera attaccando una rete Windows (tramite un agente installato) ed eseguendo le stesse tecniche che utilizzerebbe un avversario umano. Il sistema è progettato per ridurre le risorse necessarie ai team di sicurezza ("red teams") per le valutazioni di routine, permettendo loro di concentrarsi su problemi più complessi.

Il funzionamento di CALDERA si basa su un sistema di pianificazione intelligente. Il planner, dotato di un modello pre-configurato del comportamento avversario, interroga il suo modello di conoscenza per scegliere l'azione successiva più appropriata, rispettando i vincoli operativi. Una volta selezionata l'azione, il sistema genera il comando specifico e lo invia all'agente bersaglio per l'esecuzione. Questo ciclo permette di testare la resilienza della rete in modo dinamico e adattivo.

### Cascade e Threat Hunting Automatizzato

Un'evoluzione nell'uso del framework ATT&CK per la difesa è rappresentata da **Cascade**. Questa piattaforma è progettata per assistere gli analisti nel _Threat Hunting_, ovvero la ricerca proattiva di minacce non rilevate. Utilizzando la knowledge base di ATT&CK, Cascade permette di identificare e cercare comportamenti avversari noti all'interno della rete.

Il valore aggiunto di Cascade risiede nell'automazione. Tipicamente, un analista, dopo aver trovato un'attività sospetta, deve cercare manualmente processi correlati e collegare gli allarmi. Cascade automatizza porzioni significative di questo processo: raccoglie automaticamente il contesto, costruisce grafi di attività e li etichetta con i dati di ATT&CK. Questo permette agli analisti di focalizzarsi sul processo decisionale e sulla risposta (_remediation_), piuttosto che sulla raccolta dati manuale.