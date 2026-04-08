## Web Vulnerability Scanner

A differenza dei network scanner, i **Web Vulnerability Scanner** si concentrano sulle vulnerabilità delle applicazioni web, determinate dalle pagine dinamiche sviluppate dai programmatori e non solo dal web server o browser sottostanti . Questi strumenti effettuano il _crawling_ del sito e analizzano le pagine dinamiche, eseguendo test che possono richiedere autenticazione tramite password o cookie . Il loro comportamento è simile a una simulazione di violazione (_breach and simulation_).

Le vulnerabilità tipiche scoperte includono:

- **SQL Injection:** inserimento o cancellazione illegale di dati nel database.
    
- **XSS (Cross Site Scripting):** inserimento di malware che viene eseguito dall'utente finale .
    
- **CSRF (Cross Site Request Forgery):** forzatura dell'utente a eseguire azioni indesiderate su un'applicazione in cui è autenticato .
    
- **Watering Hole:** strategia che infetta siti frequentati abitualmente da un target.

## Cross Site Scripting (XSS)

L'XSS si verifica quando un'applicazione include dati non fidati (input utente) nelle pagine web senza validazione o _escaping_. Esistono tre tipi principali:

- **Stored XSS:** lo script malevolo è salvato permanentemente sul server (es. database) ed eseguito quando l'utente carica la pagina.
    
- **Reflected XSS:** lo script è riflesso dal server web ed eseguito immediatamente (es. tramite link malevolo).
    
- **DOM-based XSS:** lo script è eseguito modificando il DOM nel browser, senza coinvolgere difetti lato server.

La scoperta avviene iniettando payload (come `<script>alert('XSS')</script>`) in campi di input, URL o ispezionando il codice HTML per vedere se l'input è incorporato direttamente .

## Limiti degli Scanner e SEO Poisoning

Alcune vulnerabilità sfuggono agli scanner web, come il **Search Engine Optimization (SEO) Poisoning**, tecnica usata per aumentare la visibilità di siti malevoli. Include il _Typosquatting_ (registrare domini simili a quelli legittimi sfruttando errori di battitura) e il _Blackhat SEO_ . Tecniche di Blackhat SEO comprendono il _Keyword stuffing_, il _Cloaking_ (mostrare contenuti diversi a crawler e utenti) e la manipolazione del ranking tramite click falsi o reti di link privati .

## Scansione di Immagini e Vulnerabilità Strutturali

Per container e macchine virtuali, esistono strumenti che scansionano le immagini per identificare pacchetti vulnerabili (tramite indicizzazione e matching con database) prima del deployment .

Le **vulnerabilità strutturali** sono più complesse, emergenti dall'interazione di molti moduli. Non esiste un metodo consolidato per cercarle, ma si può estendere il fuzzing monitorando la propagazione di input malformati tra moduli (_concurrent taint analysis_) . Queste vulnerabilità possono derivare da protocolli gestiti male (messaggi ripetuti, fuori ordine) o flussi informativi non protetti tra livelli gerarchici (es. iniezione di codice in un web server che manipola un database) . Il fuzzing tradizionale non copre problemi legati alla cifratura dei messaggi.

## Analisi degli Attacchi e Remediations

La decisione di rimuovere una vulnerabilità dipende dalle intrusioni che essa abilita. Un attacco è descritto da attributi come precondizione (diritti necessari), postcondizione (diritti ottenuti), probabilità di successo, _know-how_ richiesto e rumore (probabilità di rilevamento) .

Una **Attack Chain** è una sottosequenza di attacchi all'interno di un'intrusione che realizza una _privilege escalation_, dove la postcondizione di un attacco soddisfa la precondizione del successivo . Un'intrusione include anche azioni di esplorazione e persistenza .

Gli **Automated Attacks** sono particolarmente pericolosi perché non richiedono intervento umano, possono essere eseguiti da chiunque abbia accesso a un exploit database, e avvengono in tempi elettronici . La relazione tra sofisticazione dell'attacco e conoscenza tecnica dell'intruso mostra che, con l'aumento degli strumenti automatizzati, anche attaccanti con basse competenze possono lanciare attacchi sofisticati .

Gli attacchi si distinguono anche in **Locali** (richiedono un account sul nodo) e **Remoti** (eseguibili senza account, potenzialmente _wormable_), questi ultimi molto pericolosi perché lanciabili da chiunque e ovunque .

## Tassonomia e Analisi di Attacchi Specifici

Una tassonomia degli attacchi include: Buffer overflow, Sniffing, Replay Attack, Interface attack, Man-in-the-Middle (MITM), Race condition, XSS, SQL Injection, Covert channel e Masquerading (IP spoofing, DNS spoofing) .

- **Replay Attack:** Un attaccante intercetta un messaggio valido (es. un ordine di bonifico) e lo reinvia più volte. Si contrasta includendo valori nel messaggio per rilevare duplicati (es. timestamp o nonce) .
    
- **Man-in-the-Middle (MITM):** Un attaccante B si interpone tra A e C, intercettando e manipolando i messaggi. È critico quando i canali protetti non sono mutuamente autenticati .

## SQL Injection (SQLi)

La SQL Injection consiste nell'inserire istruzioni SQL malevole all'interno di input utente che vengono concatenati dinamicamente per formare una query. Ad esempio, inserendo `' OR '1'='1` o comandi come `DROP TABLE`, l'attaccante può alterare la logica della query, bypassare login o distruggere dati .

Le contromisure principali sono:

- **Allowlist:** definire un set rigoroso di caratteri validi (es. solo alfanumerici), preferibile rispetto al _denylisting_ che spesso dimentica caratteri pericolosi .
    
- **Prepared Statements (Bind Variables):** l'uso di query precompilate con placeholder (es. `?`) dove i parametri sono associati in modo tipizzato e non interpretati come comandi SQL. Questa è la difesa più efficace .

Le statistiche mostrano che SQLi rimane uno dei vettori di attacco web predominanti, rappresentando in passato quasi i due terzi degli attacchi .

## Attacchi Crittografici e Side-Channel

Oltre agli attacchi alla crittoanalisi classica (brute force, known-plaintext, ecc.) , esistono i **Side-channel attacks**. Questi non sfruttano debolezze dell'algoritmo, ma misurano valori fisici durante l'esecuzione, come emissioni elettromagnetiche (TEMPEST), consumo energetico (_differential power analysis_) o tempi di esecuzione, per dedurre lo stato interno o le chiavi .

## Il Modello a Cipolla e l'Attacco Blue Pill

Ogni sistema cyber è strutturato come una "cipolla" di macchine virtuali stratificate. Le vulnerabilità nei livelli inferiori (hardware, firmware, OS) compromettono tutte le macchine virtuali sovrastanti e non possono essere astratte .

L'attacco **Blue Pill**, concettualizzato da Joanna Rutkowska, sfrutta la tecnologia di virtualizzazione per inserire surrettiziamente un _hypervisor_ malevolo (una "nuova macchina virtuale") sotto il sistema operativo in esecuzione. Questo malware agisce come una "macchina nel mezzo", intercettando comandi e falsificando lo stato del sistema per rimanere invisibile ai protocolli di sicurezza tradizionali che girano al livello superiore. Un esempio pratico di questo concetto è Stuxnet, che manipolava le centrifughe industriali inviando dati falsi ai sistemi di monitoraggio .