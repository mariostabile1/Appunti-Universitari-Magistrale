## Robust Programming e Input Validation

Il **Robust Programming** è uno stile di programmazione volto a minimizzare la presenza di vulnerabilità e a limitare l'impatto di quelle che non sono state ancora scoperte o corrette (0-day). Questo approccio si fonda su regole rigorose, la prima delle quali è il principio "Input is Evil": ogni input esterno deve essere considerato potenzialmente malevolo fino a prova contraria.

Un'implementazione robusta richiede la validazione di qualsiasi informazione scambiata tra moduli, il controllo dei valori trasmessi ad altre funzioni (_egress filtering_) e la verifica dei risultati restituiti. Inoltre, è fondamentale minimizzare la fuga di informazioni ("information leakage") all'esterno di un modulo o funzione, preferendo ad esempio l'uso di puntatori logici rispetto a quelli fisici.

La **Input Validation** deve definire a priori la struttura legale dell'input, agendo secondo una logica di _default deny_ (o whitelist): si definisce una grammatica o un insieme di caratteri permessi e si rifiuta tutto ciò che non vi aderisce. Questo approccio è superiore al _default allow_ (blacklist), che tenta di elencare e bloccare gli input noti come pericolosi, rischiando sempre di ometterne qualcuno. I controlli dovrebbero essere specificati in fase di design, non aggiunti come ripensamento dopo un attacco.

## Buffer Overflow

Il **Buffer Overflow** si verifica quando un programma scrive in un'area di memoria una quantità di dati superiore alla sua capacità, sovrascrivendo le locazioni di memoria adiacenti. Questo accade tipicamente in linguaggi come C o C++ che non effettuano controlli automatici sui limiti degli array.

### Stack Based Buffer Overflow

Nello **Stack Overflow**, la vulnerabilità coinvolge variabili locali allocate nello stack. La struttura dello stack frame include, oltre alle variabili locali, i parametri della funzione e l'indirizzo di ritorno (_return address_), che indica all'istruzione `RET` dove saltare al termine della funzione.

Un attaccante può inviare un input appositamente craftato che riempie il buffer locale e prosegue fino a sovrascrivere il _return address_. Modificando questo indirizzo, l'attaccante può deviare il flusso di esecuzione del programma verso del codice arbitrario (spesso definito _shellcode_) che ha inserito nello stack stesso o in altra memoria.

### Heap Based Buffer Overflow

L'**Heap Overflow** colpisce la memoria allocata dinamicamente (tramite `malloc`). Nell'heap, i blocchi di memoria contengono sia i dati utente che metadati di gestione (header), utilizzati per tenere traccia dei blocchi liberi e occupati (spesso tramite liste doppiamente linkate).

Sovrascrivendo i metadati di un blocco adiacente, un attaccante può corrompere la struttura della lista. Quando il gestore della memoria (allocator) tenta di unire (_coalesce_) o liberare blocchi manipolando questi puntatori corrotti, può essere indotto a scrivere un valore arbitrario in un indirizzo di memoria arbitrario (primitiva _Write-What-Where_). Questo permette di sovrascrivere puntatori a funzione o voci nella _Global Offset Table_ (GOT) per prendere il controllo dell'esecuzione.

### Contromisure per Buffer Overflow

Per mitigare questi attacchi, i moderni sistemi operativi implementano diverse difese:

- **Canary (Stack Guard):** Il compilatore inserisce un valore segreto e casuale (il "canarino") nello stack prima dell'indirizzo di ritorno. Prima che la funzione ritorni, il programma verifica l'integrità del canarino. Se un overflow ha sovrascritto l'indirizzo di ritorno, deve aver necessariamente alterato anche il canarino (poiché l'overflow è sequenziale), innescando un errore che termina il processo.
    
- **ASLR (Address Space Layout Randomization):** Questa tecnica casualizza la posizione dello stack, dell'heap e delle librerie condivise ad ogni esecuzione del programma. Ciò rende difficile per l'attaccante predire gli indirizzi di memoria necessari per saltare al proprio shellcode o per eseguire tecniche come ROP (_Return Oriented Programming_).
    
- **NX (No-Execute) / DEP:** Si marca lo stack (e altre aree dati) come non eseguibile tramite bit hardware (NX bit). Se il processore tenta di eseguire istruzioni che risiedono nello stack, viene sollevata un'eccezione, impedendo l'esecuzione di shellcode iniettati.

## Integer Overflow

L'**Integer Overflow** avviene quando un'operazione aritmetica produce un risultato che eccede l'intervallo rappresentabile dal tipo di dato destinazione. Questo può portare a risultati inattesi, come un numero positivo che diventa negativo (wrap-around) o un valore molto grande che diventa piccolo.

Questo difetto diventa una vulnerabilità di sicurezza quando il risultato dell'operazione viene usato per allocare memoria o controllare loop. Ad esempio, se si calcola la dimensione di un buffer moltiplicando il numero di elementi per la loro dimensione e il risultato va in overflow diventando un numero piccolo, verrà allocato un buffer insufficiente. La successiva operazione di copia dei dati, basata sulle dimensioni originali, causerà un buffer overflow sull'heap.

## Race Conditions e TOCTTOU

Una **Race Condition** si verifica quando il comportamento di un sistema dipende dall'ordine o dalla tempistica di eventi incontrollabili, come l'esecuzione concorrente di processi o thread che accedono a risorse condivise. Se l'ordine di accesso non è correttamente sincronizzato, lo stato finale del sistema può essere inconsistente o non sicuro.

Una sottoclasse specifica e critica per la sicurezza è il **TOCTTOU (Time of Check to Time of Use)**. Questa vulnerabilità emerge quando esiste un intervallo di tempo tra il controllo di una condizione di sicurezza (es. "l'utente ha il permesso di scrivere su questo file?") e l'utilizzo effettivo della risorsa. Se l'attaccante riesce a modificare lo stato del sistema (es. scambiando il file con un link simbolico a un file di sistema) esattamente in quell'intervallo, il programma userà la risorsa in modo insicuro basandosi su un controllo ormai obsoleto.

Un esempio classico riguarda i programmi _SetUID_ in Unix che controllano l'accesso a un file temporaneo e poi lo aprono; un attaccante può sfruttare la finestra temporale per dirottare la scrittura su `/etc/passwd`.

## Egress Filtering e Output Validation

Mentre molta attenzione è posta sull'input, il **Robust Programming** richiede anche il controllo dei dati in uscita. L'**Egress Filtering** o _Output Validation_ consiste nel monitorare e filtrare le informazioni che lasciano un modulo o una rete.

Questo controllo serve a prevenire due scenari principali:

1. **Information Leakage:** Impedire che dati sensibili (password, chiavi crittografiche, numeri di carte di credito) vengano inviati all'esterno per errore o per azione malevola.
    
2. **Comunicazioni C2:** Rilevare e bloccare il traffico generato da malware interno che tenta di contattare i server di _Command & Control_ di una botnet. Spesso il malware utilizza protocolli non standard o porte insolite che possono essere bloccate da policy di uscita restrittive.

A livello applicativo, la validazione dell'output è essenziale anche per prevenire attacchi come il _Cross-Site Scripting_ (XSS), assicurandosi che i dati inviati al browser siano correttamente codificati.

## Indicatori di Compromissione nei Log

L'analisi dei log può rivelare tentativi di attacco in corso. Ad esempio, alcune famiglie di ransomware come Akira o LockBit 3.0 generano un gran numero di eventi con ID **10000** e **10001** provenienti dal **RestartManager** di Windows. Questi eventi indicano che il malware sta tentando di chiudere forzatamente le applicazioni attive per sbloccare i file e poterli cifrare.