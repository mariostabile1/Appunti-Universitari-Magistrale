# 5 Discovering Vulnerabilities

La scoperta delle vulnerabilità è una fase preliminare essenziale sia per chi difende un sistema (per correggerle) sia per chi attacca (per sfruttarle). Le vulnerabilità non sono distribuite uniformemente e la loro identificazione richiede un approccio sistematico che analizza diverse superfici di attacco.
## 5.1 Classification

Le vulnerabilità possono essere classificate secondo diversi criteri che aiutano a determinare la gravità e la priorità di intervento. Una classificazione comune si basa sulla **locazione** (software applicativo, sistema operativo, hardware, configurazione di rete) o sul **ciclo di vita** (note, zero-day). Un'altra tassonomia fondamentale è quella basata sull'**impatto**:

- **Design Vulnerabilities**: errori nella progettazione logica del sistema o del protocollo. Sono le più costose da correggere poiché richiedono spesso una ristrutturazione dell'architettura.
    
- **Implementation Vulnerabilities**: errori introdotti durante la scrittura del codice (es. buffer overflow non gestiti).
    
- **Configuration Vulnerabilities**: errori nel setup di sistemi altrimenti sicuri (es. password di default, servizi non necessari attivi).
## 5.2 Vulnerability Life-Cycle

Il ciclo di vita di una vulnerabilità descrive l'evoluzione temporale di una falla di sicurezza, dal momento in cui viene introdotta fino alla sua risoluzione.

Le fasi principali sono:

1. **Creation**: L'errore viene introdotto durante lo sviluppo.
    
2. **Discovery**: La vulnerabilità viene scoperta (da ricercatori, vendor o attaccanti).
    
3. **Exploitation**: Viene creato un codice (exploit) capace di sfruttare la falla.
    
4. **Disclosure**: La vulnerabilità viene resa nota pubblicamente o al vendor.
    
5. **Patching**: Il vendor rilascia una correzione.
    
6. **Deployment**: La patch viene applicata sui sistemi.

Il periodo più critico è la finestra temporale tra la scoperta e l'applicazione della patch. Se un attaccante scopre e sfrutta la vulnerabilità prima che il vendor ne sia a conoscenza, si parla di **Zero-Day Exploit**. La gestione del rischio mira a ridurre al minimo il tempo che intercorre tra la _Disclosure_ e il _Deployment_ della patch (**Time-to-Remediate**).
## 5.3 Attacker vs Owner POV

Esiste una asimmetria fondamentale tra l'attaccante e il proprietario del sistema (Owner).

L'**Owner** ha una visione difensiva e deve proteggere l'intero perimetro; un solo punto debole ignorato può compromettere la sicurezza globale. Il difensore deve conoscere il proprio sistema meglio di chiunque altro, ma spesso è sovraccaricato dalla complessità e dal volume di log e alert.

L'**Attaccante**, al contrario, deve trovare anche solo una singola vulnerabilità sfruttabile per avere successo. Questo squilibrio è spesso riassunto nel "Dilemma del Difensore": il difensore deve aver ragione il 100% delle volte, l'attaccante solo una volta. Tuttavia, l'attaccante ha il vincolo di dover operare spesso senza essere rilevato (stealth), il che aggiunge complessità operativa alle sue azioni.
## 5.4 Scanning

Il **Scanning** è il processo di esplorazione sistematica di una rete o di un sistema per identificare host attivi, porte aperte e servizi in ascolto. È una tecnica utilizzata sia per l'audit di sicurezza che per la fase di ricognizione di un attacco. Lo scanner invia pacchetti specifici a un range di indirizzi IP e analizza le risposte per dedurre la topologia della rete e i potenziali punti di ingresso.
### 5.4.1 Fingerprinting

Il **Fingerprinting** (o OS detection) è una tecnica avanzata di scanning che mira a determinare l'identità precisa del sistema operativo e le versioni dei servizi in esecuzione su un host remoto.

Poiché ogni sistema operativo implementa lo stack TCP/IP con lievi differenze (ad esempio nel valore iniziale del TTL, nella gestione dei flag TCP o nella dimensione della finestra), analizzando queste sottili variazioni nelle risposte ai pacchetti probe è possibile identificare l'OS con alta precisione. Il **Banner Grabbing** è una forma più semplice di fingerprinting che consiste nel leggere i messaggi di benvenuto (banner) forniti dai servizi (come SSH, FTP o HTTP) che spesso dichiarano esplicitamente nome e versione del software.
### 5.4.2 Stealth scanning

Per evitare di essere rilevati dai sistemi di difesa come IDS (Intrusion Detection System) o Firewall, gli attaccanti utilizzano tecniche di **Stealth Scanning**. Invece di stabilire connessioni complete (che verrebbero loggate), si utilizzano scansioni "half-open" come il **TCP SYN Scan**, dove la connessione viene interrotta prima del completamento del three-way handshake. Altre tecniche includono la frammentazione dei pacchetti IP per eludere i filtri dei firewall o l'uso di scansioni molto lente e distribuite nel tempo per non superare le soglie di allarme degli IDS.
### 5.4.3 More on scanning

Lo scanning non è infallibile. I risultati possono contenere **False Positives** (segnalazione di vulnerabilità inesistenti) o **False Negatives** (mancata rilevazione di vulnerabilità reali). Inoltre, la presenza di firewall stateful o sistemi IPS (Intrusion Prevention System) può alterare i risultati della scansione, bloccando i pacchetti probe o fornendo risposte ingannevoli.
## 5.5 Searching in a Module

Quando l'analisi si sposta dalla rete al singolo componente software (modulo), si utilizzano tecniche di ricerca delle vulnerabilità nel codice. Questo può avvenire tramite **Static Analysis** (analisi del codice sorgente o binario senza esecuzione) o **Dynamic Analysis** (osservazione del comportamento del software durante l'esecuzione).
### 5.5.1 Fuzzing

Il **Fuzzing** è una tecnica di test dinamico che consiste nell'inviare dati di input casuali, malformati o inaspettati a un programma per causarne il crash, leak di memoria o comportamenti anomali. L'ipotesi alla base è che i crash siano spesso sintomo di vulnerabilità sfruttabili, come buffer overflow o format string bugs.

Esistono due approcci principali:

- **Mutation-based**: modifica casualmente input validi esistenti.
    
- **Generation-based**: crea nuovi input da zero seguendo le specifiche del protocollo o del formato file, ma violando vincoli specifici.
## 5.6 Web Vulnerability Scanner

I **Web Vulnerability Scanners** sono strumenti specializzati per l'analisi delle applicazioni web. A differenza degli scanner di rete che operano a livello 3 e 4 dello stack ISO/OSI, questi operano a livello 7 (Applicazione). Esplorano la struttura del sito (crawling) e testano i campi di input, i cookie e le intestazioni HTTP iniettando payload specifici per rilevare vulnerabilità comuni come **SQL Injection**, **Cross-Site Scripting (XSS)** e **Cross-Site Request Forgery (CSRF)**.
# 6 Attacks

Un attacco informatico è l'applicazione pratica di una strategia offensiva che mira a compromettere gli asset di un'organizzazione.
## 6.1 Attacks and Vulnerabilities

Esiste una relazione causale diretta tra attacchi e vulnerabilità: un attacco, per avere successo tecnico, deve sfruttare una o più vulnerabilità. L'**Exploit** è il mezzo (spesso un pezzo di codice) attraverso cui l'attacco fa leva sulla vulnerabilità. Tuttavia, esistono attacchi che non sfruttano vulnerabilità software in senso stretto, ma debolezze umane (Social Engineering) o limiti fisici dei canali di comunicazione (DoS volumetrico). La comprensione di questa relazione è cruciale per il Risk Assessment: rimuovere la vulnerabilità elimina la possibilità dell'attacco corrispondente.
## 6.2 Attack Classification

Gli attacchi possono essere classificati in base al comportamento dell'attaccante e all'impatto sul sistema:

- **Passive Attacks**: L'attaccante monitora o ascolta la comunicazione senza alterarla. L'obiettivo è ottenere informazioni (confidenzialità). Esempi: Sniffing, Traffic Analysis. Sono difficili da rilevare perché non lasciano tracce evidenti sui dati.
    
- **Active Attacks**: L'attaccante modifica il flusso di dati o lo stato del sistema. L'obiettivo è colpire l'integrità o la disponibilità. Esempi: Masquerade (spoofing), Replay attack, Modification of messages, Denial of Service (DoS).

Un'altra classificazione distingue l'origine:

- **Insider Attack**: Proviene da un utente autorizzato (dipendente, partner) che abusa dei propri privilegi.
    
- **Outsider Attack**: Proviene da un'entità esterna al perimetro di sicurezza.
## 6.3 Examining attacks

Per analizzare e comprendere attacchi complessi si utilizzano framework come la **Cyber Kill Chain** o il framework **MITRE ATT&CK**. Questi modelli scompongono l'attacco in fasi sequenziali, permettendo di studiare le Tattiche, Tecniche e Procedure (TTPs) dell'avversario.

Le fasi tipiche includono:

1. **Reconnaissance**: Raccolta informazioni sul bersaglio.
    
2. **Weaponization**: Creazione dell'exploit accoppiato a un payload (es. trojan).
    
3. **Delivery**: Trasmissione dell'arma al bersaglio (es. via email, USB, web).
    
4. **Exploitation**: Esecuzione del codice malevolo sfruttando la vulnerabilità.
    
5. **Installation**: Installazione di malware o backdoor per mantenere la persistenza.
    
6. **Command and Control (C2)**: Stabilire un canale di comunicazione remoto con l'attaccante.
    
7. **Actions on Objectives**: Esecuzione dell'obiettivo finale (esfiltrazione dati, cifratura ransomware, distruzione).
    

Interrompere la catena in una qualsiasi di queste fasi significa spesso neutralizzare l'intero attacco.