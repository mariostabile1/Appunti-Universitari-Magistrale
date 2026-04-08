## Controlli di Sicurezza (Security Controls)

I controlli di sicurezza, spesso definiti come salvaguardie o contromisure, sono misure tecniche o amministrative implementate per evitare, rilevare, contrastare o minimizzare i rischi di sicurezza che gravano su proprietà fisiche, informazioni, sistemi informatici o altri asset. Questi controlli proteggono la triade CIA (Confidenzialità, Integrità, Disponibilità).

### Classificazione Funzionale dei Controlli

I controlli possono essere classificati in base al momento in cui agiscono rispetto a un incidente di sicurezza:

- **Controlli Preventivi (Preventive):** Hanno l'obiettivo di evitare che un'azione malevola avvenga. Esempi tipici includono firewall, sistemi IPS e recinzioni fisiche.
    
- **Controlli Investigativi (Detective):** Operano durante o dopo un attacco per rilevare l'attività malevola. Rientrano in questa categoria i sistemi IDS (Intrusion Detection Systems), i CCTV e i log di sistema.
    
- **Controlli Correttivi (Corrective):** Intervengono dopo il rilevamento di un incidente per correggere la situazione e riportare il sistema allo stato normale. Un esempio è l'aggiornamento delle firme antivirus dopo un'infezione.
    
- **Controlli Deterrenti (Deterrent):** Mirano a scoraggiare l'attaccante dal tentare l'intrusione. Includono segnali di avvertimento ("Beware of Dog") o banner di login che avvisano delle conseguenze legali.
    
- **Controlli di Ripristino (Recovery):** Servono a ripristinare le funzionalità del sistema dopo un incidente, come le procedure di disaster recovery o il ripristino da backup.
    
- **Controlli Compensativi (Compensating):** Sono controlli alternativi implementati quando un controllo primario non è fattibile o è troppo costoso, fornendo un livello di sicurezza equivalente. Ad esempio, l'uso di una smart card quando non è possibile implementare una guardia di sicurezza.

### Classificazione Implementativa

Un'altra categorizzazione riguarda la natura dell'implementazione del controllo:

- **Controlli Amministrativi (o Manageriali):** Riguardano le politiche, le procedure e le linee guida definite dal management (es. policy sulle password, screening dei dipendenti).
    
- **Controlli Tecnici (o Logici):** Utilizzano la tecnologia per applicare le policy di sicurezza, come firewall, sistemi di autenticazione e crittografia.
    
- **Controlli Fisici:** Misure tangibili per prevenire o rilevare l'accesso fisico non autorizzato, come guardie, recinzioni e sistemi di spegnimento incendi.

## Vulnerability Management

Il Vulnerability Management è la pratica ciclica di identificare, classificare, prioritizzare, rimediare e mitigare le vulnerabilità del software. Questo processo è fondamentale per ridurre la "Window of Exposure", ovvero il periodo di tempo durante il quale un sistema rimane vulnerabile prima di essere corretto.

Il processo si avvale spesso di **Vulnerability Scanner**, applicazioni di rete che interrogano un inventario di sistemi per rilevare vulnerabilità note (come porte aperte o software non patchato). Questi scanner utilizzano database di vulnerabilità note, come il NVD, per identificare i problemi.

## CVSS: Common Vulnerability Scoring System

Il CVSS è uno standard aperto industriale per valutare la gravità (_severity_) di una vulnerabilità, fornendo un punteggio numerico (da 0 a 10) che riflette l'urgenza della risposta necessaria. È importante notare che il CVSS misura la severità tecnica, non il rischio complessivo per un'organizzazione specifica.

Il punteggio CVSS è calcolato basandosi su tre gruppi di metriche:

1. **Metriche Base:** Rappresentano le caratteristiche intrinseche e immutabili della vulnerabilità (es. complessità dell'attacco, privilegi richiesti).
    
2. **Metriche Temporali (o Threat in v4.0):** Riflettono le caratteristiche che cambiano nel tempo, come la disponibilità di un exploit o di una patch.
    
3. **Metriche Ambientali:** Considerano il contesto specifico dell'utente finale, come l'importanza dell'asset colpito all'interno dell'organizzazione.

Il risultato è spesso rappresentato da una _Vector String_, una stringa di testo che condensa tutti i valori delle metriche (es. `CVSS:3.1/AV:N/AC:L/PR:H/UI:N/S:U/C:L/I:L/A:N`).

### Evoluzione: CVSS v4.0

La versione 4.0 del CVSS ha introdotto cambiamenti significativi rispetto alla v3.1 per indirizzare meglio i settori OT (Operational Technology), ICS (Industrial Control Systems) e IoT. Le novità principali includono:

- Una granularità più fine nelle metriche Base.
    
- La rinominazione del gruppo "Temporal" in **"Threat"** per enfatizzare l'attualità della minaccia.
    
- L'inclusione esplicita della **Safety** (sicurezza fisica delle persone) nelle metriche ambientali e d'impatto, cruciale per i sistemi cyber-fisici.
    
- Una nuova nomenclatura per chiarire quali metriche sono state usate nel calcolo: CVSS-B (solo Base), CVSS-BT (Base + Threat), CVSS-BE (Base + Environmental), CVSS-BTE (tutte le metriche).

## Standard di Identificazione e Classificazione

Per gestire le vulnerabilità in modo efficace, l'industria si affida a diversi standard per l'enumerazione e la classificazione.

### CVE (Common Vulnerabilities and Exposures)

Il sistema CVE fornisce un metodo di riferimento per le vulnerabilità di sicurezza note pubblicamente. Ogni vulnerabilità riceve un identificativo univoco (es. CVE-2023-12345). La gestione di questi identificativi è affidata alle **CNA (CVE Numbering Authorities)**, organizzazioni autorizzate (come MITRE, vendor software, o ricercatori) ad assegnare i codici CVE.

### CWE (Common Weakness Enumeration)

È fondamentale distinguere tra vulnerabilità e debolezza (_weakness_). Una **Vulnerabilità** è un'istanza specifica di un errore in un prodotto (es. un buffer overflow in un software specifico). Una **Weakness** è il tipo di errore sottostante o la causa radice che porta alla vulnerabilità (es. "Improper Input Validation").

Il CWE è un elenco comunitario dei tipi di debolezze software e hardware. La lista "CWE Top 25" elenca le debolezze più pericolose e diffuse, utile per sviluppatori e architetti.

### NVD e CPE

Il **NVD (National Vulnerability Database)** è il repository del governo USA che arricchisce i dati CVE con analisi aggiuntive, punteggi CVSS e associazioni ai prodotti colpiti.

Per identificare i prodotti (sistemi operativi, applicazioni, hardware), si utilizza il **CPE (Common Platform Enumeration)**, uno schema di denominazione strutturato (es. `cpe:/o:microsoft:windows_10`).

### OVAL (Open Vulnerability and Assessment Language)

OVAL è uno standard internazionale per promuovere contenuti di sicurezza aperti e disponibili pubblicamente, utilizzato per standardizzare il trasferimento di informazioni di sicurezza attraverso l'intero spettro degli strumenti e servizi di sicurezza. Definisce come controllare la presenza di una vulnerabilità o di una patch su un sistema.

## Exploitability e Prioritizzazione

Poiché il CVSS misura la severità e non la probabilità di sfruttamento, sono nati strumenti complementari per aiutare nella prioritizzazione, dato che non tutte le vulnerabilità vengono sfruttate attivamente.

- **EPSS (Exploit Prediction Scoring System):** Stima la probabilità che una vulnerabilità software venga sfruttata in natura entro i prossimi 30 giorni, con un punteggio da 0 a 1 (0% - 100%).
    
- **KEV (Known Exploited Vulnerabilities):** È un catalogo gestito dalla CISA che elenca le vulnerabilità che sono state effettivamente sfruttate "in the wild". Questo è fondamentale perché, su milioni di vulnerabilità, solo una piccola percentuale (circa il 4%) viene realmente attaccata. L'uso del KEV permette ai difensori di concentrarsi sui problemi reali piuttosto che su quelli teorici.

## Disclosure delle Vulnerabilità

La gestione della scoperta di una vulnerabilità comporta decisioni etiche e strategiche complesse.

- **VEP (Vulnerability Equities Process):** È il processo utilizzato dai governi (in particolare USA) per decidere se divulgare una vulnerabilità 0-day scoperta ai vendor (perché venga corretta) o mantenerla segreta per utilizzarla in operazioni di intelligence o militari. La decisione si basa su un bilanciamento tra l'interesse difensivo (proteggere i propri sistemi) e quello offensivo.
    
- **CVD (Coordinated Vulnerability Disclosure):** È il modello standard per la divulgazione responsabile. In questo processo, i ricercatori che scoprono una vulnerabilità la segnalano privatamente al fornitore e attendono un periodo concordato (spesso 90 giorni) per permettere il rilascio di una patch prima di rendere pubblici i dettagli. Questo riduce il rischio che gli attaccanti sfruttino la vulnerabilità prima che esista una difesa.