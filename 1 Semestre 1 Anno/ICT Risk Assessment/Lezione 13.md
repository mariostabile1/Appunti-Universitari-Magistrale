## Zero Trust Architecture: Piani e Microsegmentazione

L'architettura **Zero Trust** si fonda sulla netta separazione tra due piani operativi: il **Control Plane** e il **Data Plane**. Il Control Plane è il dominio decisionale dove risiede il **Policy Decision Point (PDP)**, che scambia messaggi di controllo con i **Policy Enforcement Points (PEP)** per configurare dinamicamente gli accessi. Il Data Plane è il dominio dove avviene l'effettivo trasferimento dei dati tra il soggetto (Subject) e la risorsa (Resource), sempre attraverso la mediazione di un PEP. In questo modello, i dati viaggiano sempre cifrati e il flusso è abilitato solo dopo un'esplicita autorizzazione del PDP.

Un'applicazione avanzata di questo modello è la **Microsegmentazione**, che colloca i PEP il più vicino possibile alle singole risorse o gruppi di risorse (Resource Enclave). Questo riduce drasticamente la superficie di attacco, creando zone di fiducia implicita ("Implicit Trust Zone") minime o nulle. In uno scenario microsegmentato, il movimento laterale di un attaccante è fortemente limitato poiché ogni tentativo di accesso a un nuovo segmento richiede una nuova validazione da parte del PDP.

## Minacce alla Zero Trust Architecture

Nonostante la robustezza teorica, il NIST identifica diverse minacce specifiche per la ZTA. La più critica è la **Subversion of ZTA decision process**, dove un attaccante compromette il Control Plane (il PDP o l'amministratore delle policy) per autorizzare accessi illegittimi. Altre minacce includono il **Denial-of-Service (DoS)** o la distruzione della rete, che impedirebbe la comunicazione tra PEP e PDP bloccando ogni accesso legittimo, e l'uso di **Stolen Credentials** o minacce interne (_Insider Threat_), che rimangono efficaci se l'autenticazione non è supportata da analisi comportamentale.

Esiste inoltre il problema della **Visibility on the network**: poiché tutto il traffico è cifrato end-to-end, gli strumenti di analisi di rete tradizionali (NIDS) diventano ciechi ("opaque traffic"), rendendo difficile rilevare malware o esfiltrazioni senza l'uso di analisi dei metadati o machine learning. Infine, la dipendenza da formati dati proprietari può creare _vendor lock-in_ e problemi di interoperabilità tra soluzioni di sicurezza diverse.

## Firewalling e Packet Filtering

Il **Firewall** è un componente hardware o software che separa due reti caratterizzate da livelli di fiducia differenti (es. rete interna vs Internet), analizzando e filtrando il traffico che le attraversa. La forma più basilare di firewalling è il **Packet Filtering**, implementato tipicamente sui router.

Un router instrada i messaggi basandosi su una tabella di routing che mappa i messaggi in ingresso verso le linee di uscita. Nel packet filtering, ogni linea del router è associata a un insieme di regole che specificano come trattare i messaggi in arrivo o in uscita. Queste regole analizzano l'intestazione (header) del pacchetto IP, verificando attributi come indirizzo sorgente, indirizzo destinazione e protocollo, senza ispezionare il contenuto (payload) del messaggio.

## Access Control Lists (ACL)

Le regole di filtraggio su un router sono organizzate in **Access Control Lists (ACL)**. Una ACL è una lista ordinata di regole, dove ogni regola definisce un pattern (solitamente un range di indirizzi IP) e un'azione associata: **route** (instrada il pacchetto) o **drop** (scarta il pacchetto).

L'ordine delle regole è fondamentale: il router scorre la lista sequenzialmente e applica la prima regola che soddisfa il match con il pacchetto ("first match wins"). Ad esempio, se una regola specifica "route" per un indirizzo specifico e una regola successiva impone "drop" per l'intera sottorete a cui quell'indirizzo appartiene, l'ordine determinerà se il pacchetto specifico passerà o meno.

La struttura dell'ACL definisce implicitamente la politica di sicurezza:

- **Default Deny**: Se l'ultima regola della lista è `* -> drop` (scarta tutto il resto), la policy elenca solo ciò che è permesso. È l'approccio più sicuro.
    
- **Default Allow**: Se l'ultima regola è `* -> route` (permetti tutto il resto), la policy elenca solo ciò che è vietato. Questo approccio è meno sicuro poiché lascia passare traffico imprevisto o non esplicitamente bloccato.