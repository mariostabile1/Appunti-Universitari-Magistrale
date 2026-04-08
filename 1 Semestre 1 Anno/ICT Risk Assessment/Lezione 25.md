## MITRE ATT&CK Framework

Il **MITRE ATT&CK** (Adversary Tactics, Techniques, and Common Knowledge) è una knowledge base accessibile globalmente che raccoglie le tattiche e le tecniche avversarie basate su osservazioni del mondo reale. Costituisce uno standard _de facto_ per descrivere i possibili comportamenti degli attaccanti, strutturando i passi di un'intrusione in modo granulare. A differenza di modelli teorici, ATT&CK funge da fondamento per lo sviluppo di threat models specifici e metodologie di difesa sia nel settore privato che governativo.

La struttura del framework si articola su tre livelli gerarchici di astrazione. Al livello più alto troviamo le **Tactics**, che rappresentano l'obiettivo tattico dell'avversario, ovvero il "perché" di un'azione (es. ottenere l'accesso iniziale, eseguire codice, esfiltrare dati). Ogni tattica include diverse **Techniques**, che descrivono il "come" un avversario raggiunge quell'obiettivo tattico (es. dumping delle credenziali per ottenere accesso). A un livello ancora più specifico ci sono le **Procedures**, che dettagliano l'implementazione esatta usata da un avversario o malware specifico (es. il gruppo APT28 che utilizza PowerShell per iniettare codice lsass.exe). Esistono anche le **Sub-techniques**, che offrono una descrizione ancora più specifica del comportamento tecnico.

### Differenze con la Cyber Kill Chain

Sebbene entrambi i modelli descrivano le intrusioni, presentano differenze sostanziali. La **Cyber Kill Chain** (sviluppata da Lockheed Martin) è un modello di alto livello composto da sette fasi sequenziali (Reconnaissance, Weaponization, Delivery, Exploitation, Installation, C2, Actions on Objectives). È utile per comprendere la progressione lineare di un attacco tradizionale basato su malware, ma risulta rigida: se l'attaccante salta una fase o reitera, il modello fatica a rappresentarlo.

Al contrario, **ATT&CK** non assume una sequenzialità rigida e offre un dettaglio molto maggiore sul _come_ le azioni vengono eseguite. Si focalizza sulle TTP (Tactics, Techniques, and Procedures), permettendo di mappare scenari complessi dove gli attaccanti cambiano vettore o tornano sui propri passi. Mentre la Kill Chain è "malware-focused", ATT&CK è "technique-focused", correlando le azioni agli obiettivi operativi dell'avversario.

## The Pyramid of Pain

Il concetto della **Pyramid of Pain**, introdotto da David Bianco, illustra la relazione tra i tipi di indicatori di compromissione (IoC) che un difensore può rilevare e il "dolore" (costo/sforzo) che causa all'attaccante quando tali indicatori vengono negati o bloccati.

Alla base della piramide troviamo gli indicatori banali come i valori **Hash** (MD5, SHA1) e gli indirizzi **IP**. Bloccarli è facile per il difensore, ma è altrettanto facile per l'attaccante modificarli (basta ricompilare il malware o usare un proxy) causando un dolore insignificante. Salendo, troviamo i **Domain Names** e i **Network/Host Artifacts** (es. stringhe user-agent, chiavi di registro), che richiedono un po' più di sforzo per essere cambiati.

Verso la cima, il dolore per l'attaccante aumenta drasticamente. Bloccare i **Tools** (strumenti software) costringe l'avversario a riscrivere o riconfigurare il proprio arsenale. Al vertice della piramide ci sono le **TTPs** (Tactics, Techniques, and Procedures). Rilevare e bloccare un comportamento o una tecnica intera (es. Pass-the-Hash) costringe l'attaccante a riaddestrarsi e a inventare un metodo completamente nuovo per raggiungere l'obiettivo. Questo è il livello di difesa più efficace, poiché colpisce il _modus operandi_ dell'avversario e non solo i suoi artefatti effimeri.

## Living off the Land (LotL)

La strategia **Living off the Land (LotL)** descrive un approccio in cui gli attaccanti utilizzano strumenti legittimi, già presenti nel sistema operativo ("native tools") o amministrativi, per condurre le fasi dell'attacco invece di introdurre malware personalizzato. Questi binari sono spesso chiamati **LOLBins** (Living Off The Land Binaries).

L'uso di LotL offre vantaggi strategici significativi all'attaccante. Primo, permette di evadere le difese basate su liste di programmi consentiti (_allowlist_), poiché strumenti come PowerShell, WMI, Bash o PsExec sono firmati e legittimi. Secondo, l'attività malevola si confonde con il normale traffico amministrativo, rendendo difficile il rilevamento tramite anomalie. Infine, complica enormemente l'attribuzione dell'attacco, poiché non ci sono binari unici (come un malware compilato ad hoc) che possano essere collegati a un gruppo specifico tramite firma o analisi del codice.

Per contrastare gli attacchi LotL, non è sufficiente cercare file malevoli, ma è necessario utilizzare soluzioni basate sull'**analisi comportamentale** (Behavioral Analysis) che monitorano _come_ vengono utilizzati gli strumenti legittimi (es. rilevare se PowerShell sta eseguendo script codificati in Base64 o scaricando file da domini sospetti).

### Esempi di Comportamento APT

L'analisi comportamentale tramite framework come ATT&CK permette di profilare gruppi avanzati.

Ad esempio, il gruppo **APT3** è noto per utilizzare strumenti legittimi in modo specifico: identifica documenti locali (T1005), utilizza la versione a riga di comando di **WinRAR** per comprimerli e cifrarli (T1002), e poi esfiltra l'archivio spostandolo nel cestino (T1074) prima di inviarlo via HTTPS sulla porta 443 (T1043). Sebbene il traffico sembri normale HTTPS e il tool sia un comune WinRAR, la _sequenza_ di azioni e il contesto rivelano l'intento malevolo.

Analogamente, il gruppo **FIN6**, specializzato in crimini finanziari, utilizza una combinazione di strumenti per muoversi lateralmente e compromettere i sistemi POS (Point of Sale), dimostrando come la comprensione delle TTP sia fondamentale per rilevare campagne complesse che attraversano molteplici fasi tattiche.