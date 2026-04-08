## Stealth Mode Scanning e Tipologie di Scansione

Sia i proprietari dei sistemi che gli attaccanti utilizzano il vulnerability scanning, ma con finalità e modalità differenti. Il proprietario è interessato a simulare le scansioni per comprendere cosa accade durante un'intrusione. L'attaccante, invece, deve configurare la frequenza dei messaggi e il numero di nodi scansionati in parallelo per evitare il rilevamento.

Lo **Stealth mode scanning** consiste nella selezione di parametri di scansione che minimizzano la probabilità di essere individuati. Minore è la frequenza dei pacchetti e il numero di nodi coinvolti, minore è la probabilità di detection. Un attaccante _state-sponsored_, ad esempio, potrebbe inviare pochissimi pacchetti al giorno per operare in modalità nascosta, specialmente durante la fase di costruzione dell'infrastruttura di attacco.

Per acquisire una visione completa del sistema target, il difensore deve eseguire scansioni da diversi punti di vista:

- **External Vulnerability Scans:** Eseguiti dall'esterno per controllare le aree dell'ecosistema IT esposte a internet e capire cosa un attaccante può scoprire prima di iniziare l'intrusione.
    
- **Internal Vulnerability Scans:** Testano ogni dispositivo nella rete per identificare le vulnerabilità che lasciano il business suscettibile a danni dopo un _initial access_. Vengono eseguiti dal proprietario o dall'attaccante dopo l'accesso iniziale.
    
- **Intrusive Scans (Breach and Simulation):** A differenza delle scansioni di base tipicamente non intrusive, queste tentano di sfruttare una vulnerabilità quando viene trovata per scoprire i falsi positivi. È la scansione più rigorosa, ma può interrompere sistemi e processi operativi.

## Vulnerability Ranking e CVSS

Una volta identificate le vulnerabilità, è necessario classificarle per prioritizzare la rimozione. Lo standard industriale aperto per valutare la severità è il **CVSS (Common Vulnerability Scoring System)**.

Il CVSS assegna un punteggio di severità alle vulnerabilità, permettendo ai risponditori di prioritizzare le risposte e le risorse in base alla minaccia. Il punteggio varia da 0 a 10, con i seguenti rating qualitativi:

- None: 0.0
    
- Low: 0.1 - 3.9
    
- Medium: 4.0 - 6.9
    
- High: 7.0 - 8.9
    
- Critical: 9.0 - 10.0

È fondamentale notare che il CVSS misura la **severity** (gravità tecnica), non il **risk** (rischio). Il rischio è calcolato come $Threat \times Vulnerability \times Impact$, mentre il CVSS si concentra sulla severità tecnica della vulnerabilità stessa.

### Gruppi di Metriche CVSS

Il punteggio CVSS si compone di tre gruppi di metriche:

1. **Base Score Metrics:** Rappresentano le caratteristiche intrinseche di una vulnerabilità, costanti nel tempo e negli ambienti utente (es. complessità dell'attacco, privilegi richiesti).
    
2. **Temporal Score Metrics:** Riflettono le caratteristiche che cambiano nel tempo (es. disponibilità di un exploit).
    
3. **Environmental Score Metrics:** Rappresentano le caratteristiche uniche per l'ambiente dell'utente (es. importanza dell'asset).

### Evoluzione verso CVSS v4.0

La versione 4.0 del CVSS è stata introdotta per rispondere alle esigenze dei settori OT (Operational Technology), ICS (Industrial Control Systems) e IoT. Tra le modifiche principali vi è una granularità più fine nelle metriche base e l'inclusione della metrica **Safety** nelle metriche ambientali e di impatto, essenziale per i sistemi cyber-fisici dove un guasto può causare danni fisici alle persone.

Inoltre, il gruppo di metriche "Temporal" è stato rinominato in **Threat** per sottolineare che le metriche non sono solo temporali ma riguardano l'attualità della minaccia. La nuova nomenclatura per i punteggi è la seguente:

- **CVSS-B:** Base metrics
    
- **CVSS-BT:** Base + Threat metrics
    
- **CVSS-BE:** Base + Environmental metrics
    
- **CVSS-BTE:** Base + Threat + Environmental metrics

## Prioritizzazione: EPSS e KEV

Poiché il CVSS misura solo la severità, per una corretta prioritizzazione è necessario considerare anche la probabilità di sfruttamento.

L'**EPSS (Exploit Prediction Scoring System)** è un modello data-driven che stima la probabilità che una vulnerabilità software venga sfruttata in natura (in the wild). Produce un punteggio di probabilità tra 0 e 1 (0% e 100%), dove più alto è il punteggio, maggiore è la probabilità che la vulnerabilità venga sfruttata entro i prossimi 30 giorni.

Parallelamente, la CISA gestisce il catalogo **KEV (Known Exploited Vulnerabilities)**. Su milioni di vulnerabilità note, solo una piccola frazione (circa il 4%) è stata sfruttata attivamente. Il KEV aiuta a ridurre il rumore concentrandosi su quelle vulnerabilità che sono state effettivamente utilizzate dagli attaccanti, rendendo la difesa più efficace ed efficiente.

## Standard e Database di Vulnerabilità

Per gestire le informazioni sulle vulnerabilità, esistono diversi standard e identificatori:

- **CVE (Common Vulnerabilities and Exposures):** Un elenco di record per vulnerabilità di sicurezza note pubblicamente. Ogni record ha un ID univoco. Le **CNA (CVE Numbering Authorities)** sono le organizzazioni autorizzate ad assegnare gli ID CVE.
    
- **CWE (Common Weakness Enumeration):** Un elenco comunitario dei tipi di debolezze software e hardware. È importante distinguere tra _vulnerability_ (un'istanza specifica di un errore, es. CVE-2023-XYZ) e _weakness_ (il tipo di errore sottostante, es. Buffer Overflow). La lista "CWE Top 25" elenca le debolezze più pericolose.
    
- **NVD (National Vulnerability Database):** Il repository del governo USA per i dati di gestione delle vulnerabilità standardizzati, che include i database delle checklist di sicurezza, i difetti software e le misconfigurazioni.
    
- **CPE (Common Platform Enumeration):** Schema di denominazione strutturato per sistemi informatici, software e pacchetti.
    
- **OVAL (Open Vulnerability and Assessment Language):** Standard internazionale per promuovere contenuti di sicurezza aperti e disponibili pubblicamente.

## Vulnerability Disclosure

Il processo di divulgazione delle vulnerabilità è critico e segue diverse filosofie:

- **VEP (Vulnerability Equities Process):** Utilizzato dai governi (es. USA) per decidere se divulgare una vulnerabilità zero-day scoperta (per permettere la difesa) o mantenerla segreta per operazioni di intelligence/offensive. La decisione si basa su considerazioni di equità tra difesa e offesa.
    
- **CVD (Coordinated Vulnerability Disclosure):** È il modello standard industriale in cui un ricercatore segnala una vulnerabilità al fornitore e attende un periodo di grazia (solitamente 90 giorni) prima di renderla pubblica, dando al vendor il tempo di sviluppare una patch.

## Fuzzing

Il **Fuzzing** è una tecnica di test del software automatizzata che comporta la fornitura di dati non validi, imprevisti o casuali come input a un programma. Il programma viene monitorato per eccezioni come crash, asserzioni di codice fallite o potenziali memory leaks.

Questa tecnica è particolarmente efficace per scoprire vulnerabilità di corruzione della memoria in linguaggi come C e C++, o errori di logica e condizioni di gara in linguaggi memory-safe. Sebbene non garantisca l'assenza di bug (poiché trova solo quelli che causano un comportamento anomalo osservabile), è altamente scalabile ed è diventata lo standard de facto per la sicurezza del software.

### Input Generation: Mutation vs Generation

Esistono due approcci principali per creare gli input nel fuzzing:

1. **Mutation Based (Dumb Fuzzing):** Modifica campioni di input esistenti (seed) per crearne di nuovi. È semplice da implementare e non richiede conoscenza del formato dell'input, ma può generare molti input non validi che vengono scartati subito dal parser.
    
2. **Generation Based (Smart Fuzzing):** Crea input da zero basandosi su una grammatica o un modello del formato di input. Richiede una conoscenza preliminare del formato, ma produce input sintatticamente corretti che raggiungono parti più profonde del codice ("Deep reach").

### Architetture di Fuzzing: Black, White e Grey Box

- **Black Box:** Il fuzzer non ha accesso al codice sorgente e osserva solo l'input/output. È facile da applicare ma esplora il codice "alla cieca".
    
- **White Box:** Analizza la struttura interna e la logica del codice (es. Symbolic Execution) per generare input mirati. È computazionalmente costoso ma raggiunge una copertura elevata.
    
- **Grey Box (Coverage-guided):** Utilizza un'instrumentazione leggera (compile-time o binary) per raccogliere informazioni sulla copertura del codice (es. quali rami vengono eseguiti) e guida la generazione degli input per massimizzare tale copertura. È il compromesso più efficace ed è utilizzato da strumenti come AFL.
    

### Coverage-Guided Fuzzing e Sanitizers

Nel **Coverage-guided fuzzing**, si parte da un corpus di seed. Il fuzzer muta un seed e lo esegue. Se il nuovo input copre un percorso di esecuzione precedentemente inesplorato (nuovi branch), viene aggiunto al corpus. Questo approccio evolutivo (simile agli algoritmi genetici) permette di esplorare profondamente il programma.

Poiché il fuzzing rileva solo crash visibili, si utilizzano i **Sanitizers** (come ASAN - AddressSanitizer, MSAN, UBSAN) per instrumentare il codice e rilevare violazioni che non causerebbero un crash immediato ma che rappresentano vulnerabilità, come buffer overflow non fatali o use-after-free.

## Symbolic Execution e Hybrid Fuzzing

La **Symbolic Execution** considera gli input come simboli anziché valori concreti. Esegue il programma costruendo formule logiche che rappresentano i vincoli sui percorsi di esecuzione. Utilizzando un _SMT Solver_, è possibile calcolare i valori concreti di input necessari per percorrere uno specifico ramo del codice (es. per soddisfare un `if (x == 0xDEADBEEF)`).

Il limite principale è la **Path Explosion**: il numero di percorsi possibili cresce esponenzialmente con la dimensione del programma, rendendo la symbolic execution impraticabile per software complessi.

L'**Hybrid Fuzzing** combina la velocità del fuzzing con la precisione della symbolic execution. Il fuzzer esplora rapidamente il codice facile; quando si blocca su controlli complessi (come "magic numbers"), interviene la symbolic execution per risolvere il vincolo e sbloccare nuovi percorsi per il fuzzer. Esempi di questo approccio includono sistemi come _Driller_ e _QSYM_.

## Web Vulnerability Scanners (DAST)

I Web Vulnerability Scanners operano secondo il paradigma **DAST (Dynamic Application Security Testing)**, interagendo con l'applicazione web in esecuzione (Black Box).

Il processo avviene in due fasi:

1. **Crawling:** Lo scanner esplora l'applicazione (link, form, script) per costruirne una mappa.
    
2. **Fuzzing:** Lo scanner inietta input malevoli (payload) nei punti di input scoperti per rilevare vulnerabilità come SQL Injection o XSS, analizzando le risposte HTTP.

Questi strumenti affrontano sfide specifiche come la gestione dell'autenticazione, il mantenimento dello stato della sessione e la necessità di evitare effetti collaterali distruttivi (es. "nuke" del database) durante i test.

## American Fuzzy Lop (AFL)

**AFL** è uno dei fuzzer coverage-guided più popolari. Funziona instrumentando il binario (durante la compilazione o tramite QEMU) per tracciare i percorsi eseguiti. Utilizza una coda di input che muta deterministicamente e casualmente. Associa un contatore a ciascun arco del grafo di controllo del programma (registrato in una bitmap condivisa) per rilevare se un input ha raggiunto una nuova transizione di stato o aumentato la frequenza di esecuzione di un arco.