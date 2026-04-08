## Protocollo Kerberos: Principi e Architettura

Kerberos è un protocollo di autenticazione di rete progettato per fornire un'autenticazione forte per applicazioni client/server attraverso l'uso della crittografia a chiave segreta. Sviluppato inizialmente al MIT e successivamente adottato da Microsoft, si basa su un principio fondamentale: la password dell'utente non deve mai viaggiare sulla rete, né essere memorizzata sulla macchina client o in forma non crittografata nel database di autenticazione .

Il sistema centralizza la gestione delle informazioni di autenticazione su un server dedicato, garantendo che l'amministratore possa disabilitare un account agendo in un unico punto e che il cambio password si rifletta istantaneamente su tutti i servizi. L'utente inserisce la password una sola volta per sessione, accedendo poi in modo trasparente ai servizi autorizzati senza dover reinserire le credenziali .

Il cuore del sistema è il **Key Distribution Center (KDC)**, un database centralizzato che mantiene una chiave segreta condivisa con ogni entità (principal) della rete. Per gli utenti, questa chiave deriva dalla password tramite una funzione di hash one-way; per i servizi, è una chiave generata casualmente . Kerberos implementa un'autenticazione a "tre lati" (client, server, KDC) basata sul principio che se due entità condividono una chiave segreta distribuita dal KDC, allora sono autenticate.

Il KDC è composto logicamente da due servizi: l'**Authentication Server (AS)**, che gestisce l'identificazione iniziale e rilascia un _Ticket Granting Ticket_ (TGT), e il **Ticket Granting Server (TGS)**, che rilascia i ticket di servizio specifici per accedere alle risorse. La sicurezza fisica del KDC è critica, poiché comprometterlo significa compromettere l'intera rete.

## Funzionamento Dettagliato del Protocollo

Il protocollo si articola in una serie di scambi di messaggi per ottenere credenziali temporanee (ticket) che provano l'identità del client ai server.

La fase iniziale vede il client inviare una **AS_REQ** al KDC. Questa richiesta è autenticata implicitamente tramite un timestamp cifrato con la chiave derivata dalla password dell'utente. Il KDC risponde con una **AS_REP** contenente il TGT (cifrato con la chiave del TGS) e una chiave di sessione cifrata con la chiave dell'utente . Il TGT è fondamentale perché permette al client di provare la propria identità al TGS senza reinviare la password.

Quando il client necessita di accedere a un servizio specifico, invia una **TGS_REQ** al TGS. Questo pacchetto include il TGT ottenuto precedentemente e un "autenticatore" generato dal client. Il TGS verifica il TGT e risponde con una **TGS_REP**, fornendo un ticket di servizio cifrato con la chiave del server di destinazione e una nuova chiave di sessione per l'interazione client-server .

Infine, il client presenta il ticket di servizio all'Application Server tramite una **AP_REQ**. Il server decifra il ticket, verifica l'identità del client e, se richiesta la mutua autenticazione, risponde con una **AP_REP** provando a sua volta la propria identità . L'uso di timestamp (ts) e nonce è cruciale per prevenire attacchi di tipo _replay_ .

## Limitazioni e Vulnerabilità di Kerberos

Nonostante la robustezza, Kerberos presenta delle criticità. Richiede che ogni applicazione sia interfacciata con il protocollo, aumentando la complessità del codice e la dimensione della _Trusted Computing Base_. Il KDC rappresenta un _Single Point of Failure_ catastrofico: se compromesso, tutte le chiavi sono esposte. Inoltre, il sistema richiede una rigorosa sincronizzazione temporale tra i nodi per la validità dei timestamp e la gestione dei ticket, la cui durata (time window) deve essere attentamente calibrata .

### Attacchi Kerberoasting

Una delle minacce più significative in ambienti Microsoft Active Directory è il **Kerberoasting**. Si tratta di un attacco di _post-exploitation_ e _privilege escalation_ che mira a ottenere l'hash della password di un account di servizio.

L'attaccante, che ha già compromesso un account utente di dominio (anche senza privilegi elevati), richiede un ticket di servizio Kerberos (TGS) per un _Service Principal Name_ (SPN). Poiché il ticket ricevuto è cifrato con l'hash della password dell'account di servizio target, l'attaccante può estrarre questo ticket dalla memoria e tentare di crackarlo offline usando tecniche di forza bruta .

Il successo dell'attacco dipende dalla complessità della password dell'account di servizio. Mentre gli SPN basati su host sono gestiti automaticamente con password complesse e ruotate frequentemente, gli SPN basati su account utente sono spesso vulnerabili a causa di password deboli scelte dagli umani . Una volta ottenuto l'accesso, l'attaccante può impersonare il servizio o scalare i privilegi. Strumenti come **Mimikatz**, **Rubeus** e **Impacket** sono comunemente usati per eseguire questo attacco .

## Autenticazione Continua

L'evoluzione dei modelli di sicurezza ha portato al concetto di **Continuous Authentication**, spostandosi dal vecchio paradigma di "autenticazione periodica" (una tantum all'accesso) verso un modello Zero Trust dove l'utente è verificato costantemente durante l'intera sessione.

Questo approccio è vitale con l'aumento del lavoro remoto e l'esposizione dei dispositivi aziendali. Tecniche come la biometria comportamentale analizzano _come_ un utente digita (dinamica della digitazione) o interagisce con il dispositivo per creare un profilo unico. Applicazioni come _ActiveLock_ o _Focus_ apprendono lo stile di digitazione dell'utente in una fase di training e successivamente monitorano in tempo reale per rilevare anomalie o impostori, proteggendo la privacy e i dati senza impattare sull'esperienza utente .

## Zero Trust Architecture (ZTA)

La **Zero Trust Architecture** rappresenta un cambio di paradigma che sposta la difesa dai perimetri di rete statici alla protezione di utenti, asset e risorse. Il principio cardine è che non viene concessa alcuna fiducia implicita basata sulla posizione fisica o di rete (es. LAN vs Internet) o sulla proprietà del dispositivo .

In un modello Zero Trust, l'accesso a ogni risorsa deve essere esplicitamente concesso tramite policy dinamiche che valutano l'identità del soggetto, lo stato di sicurezza del dispositivo e il contesto, prima che venga stabilita qualsiasi sessione. Tutte le comunicazioni sul _data plane_ devono essere cifrate e autenticate.

### Componenti Logiche della ZTA

L'architettura Zero Trust distingue nettamente tra **Control Plane** e **Data Plane**.

Il **Control Plane** gestisce le decisioni di accesso ed è composto da:

- **Policy Engine (PE)**: responsabile della decisione finale di concedere l'accesso a una risorsa basandosi sulle policy aziendali e input da fonti esterne (CDM, Threat Intelligence).
    
- **Policy Administrator (PA)**: esegue la decisione del PE stabilendo o chiudendo il canale di comunicazione. Insieme, PE e PA formano il **Policy Decision Point (PDP)** .

Il **Data Plane** è dove avviene il transito effettivo dei dati. Qui troviamo il **Policy Enforcement Point (PEP)**, che agisce come un gatekeeper, abilitando, monitorando e terminando le connessioni tra il soggetto e la risorsa aziendale in base ai comandi del PDP.

La microsegmentazione è una caratteristica chiave, dove il PEP può essere collocato sul dispositivo, sulla rete o direttamente sull'applicazione, creando zone di fiducia implicita estremamente ridotte o nulle.

### I 7 Pilastri della Zero Trust (NSA)

Il modello della NSA definisce sette pilastri per una ZTA matura: **User**, **Devices**, **Applications & Workloads**, **Data**, **Network & Environment**, **Automation & Orchestration**, e **Visibility & Analytics**. L'obiettivo è utilizzare l'analisi avanzata e l'AI per prendere decisioni di accesso in tempo reale, automatizzare la risposta agli incidenti e garantire una visibilità totale su tutto il traffico, anche quello cifrato .

## Minacce e Mitigazioni in ZTA

Anche la ZTA è soggetta a minacce specifiche. La subversion del processo decisionale (attacco al PDP) è critica e richiede una rigorosa gestione della configurazione e monitoraggio. Il rischio di **Denial of Service (DoS)** contro il PEP o il PDP può interrompere l'accesso alle risorse. Inoltre, la mancanza di visibilità sulla rete dovuta alla crittografia diffusa (opaque traffic) necessita di tecniche di analisi dei metadati o machine learning per rilevare malware senza decifrare il traffico .

Un'altra sfida è la dipendenza da formati proprietari e la mancanza di interoperabilità tra soluzioni di diversi fornitori, che può causare un _vendor lock-in_ rischioso per la business continuity . Infine, l'amministrazione della ZTA spesso coinvolge **Non-Person Entities (NPE)**, che devono essere autenticate tramite meccanismi hardware-based (come certificati a chiave pubblica protetti in moduli TPM) per evitare compromissioni .