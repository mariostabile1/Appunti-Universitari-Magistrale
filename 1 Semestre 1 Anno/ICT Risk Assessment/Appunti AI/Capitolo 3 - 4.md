# 3 Vulnerability, Attack, Intrusion

In ambito di sicurezza informatica, la comprensione delle relazioni tra vulnerabilità, attacco e intrusione è fondamentale per definire lo scenario di rischio. Questi elementi non sono isolati, ma rappresentano fasi consequenziali di una compromissione di sicurezza.
## 3.1 Vulnerability

Una **Vulnerability** è una debolezza intrinseca in un asset o in un controllo di sicurezza che può essere sfruttata da una minaccia per causare un danno. Le vulnerabilità possono risiedere nel software (bug di programmazione), nell'hardware, nelle procedure operative o nella configurazione del sistema. È importante notare che una vulnerabilità, di per sé, non costituisce un danno finché non viene attivamente sfruttata; rappresenta piuttosto un potenziale latente di compromissione. La presenza di una vulnerabilità è una condizione necessaria ma non sufficiente affinché si verifichi un incidente di sicurezza.
## 3.2 Attack

Un **Attack** è un tentativo intenzionale di aggirare i controlli di sicurezza di un sistema e violare la **Security Policy**. L'attacco è l'azione che trasforma il potenziale rischio di una vulnerabilità in un evento concreto. Gli attacchi possono essere classificati come attivi, se modificano lo stato del sistema o i dati, o passivi, se si limitano all'intercettazione e all'analisi del traffico senza alterare le risorse. Un attacco sfrutta una o più vulnerabilità specifiche per raggiungere il suo obiettivo.
## 3.3 Threat Agent

Il **Threat Agent** (o attore della minaccia) è l'entità che dà origine alla minaccia e che esegue l'attacco. L'analisi del Threat Agent si basa su tre fattori chiave: motivazione, capacità e opportunità. I Threat Agents spaziano da script kiddies con scarse competenze tecniche, a criminali informatici motivati dal profitto, fino ad arrivare a stati-nazione (Nation-State Actors) con risorse illimitate e obiettivi geopolitici. Comprendere chi è l'avversario è essenziale per calibrare le difese in modo appropriato.
## 3.4 Intrusion

L'**Intrusion** si verifica quando un attacco ha successo e l'attaccante ottiene l'accesso non autorizzato al sistema, violando la confidenzialità, l'integrità o la disponibilità delle risorse. Mentre un attacco è il tentativo, l'intrusione è il risultato del successo di tale tentativo. Una volta avvenuta l'intrusione, il sistema non è più in uno stato sicuro e si considera compromesso.
## 3.5 Initial Access

L'**Initial Access** è la fase in cui l'attaccante ottiene il primo punto di appoggio all'interno della rete o del sistema bersaglio. Le tecniche comuni includono il **Phishing** per rubare credenziali, lo sfruttamento di vulnerabilità su servizi esposti pubblicamente, o l'uso di supporti rimovibili infetti. Questa fase è critica perché trasforma una minaccia esterna in una presenza interna, permettendo all'attaccante di muoversi lateralmente e scalare i privilegi.
## 3.6 Countermeasure

Una **Countermeasure** (o controllo di sicurezza) è un'azione, un dispositivo, una procedura o una tecnica che riduce una vulnerabilità o contrasta un attacco. Le contromisure possono agire in modi diversi: prevenendo l'attacco (ad esempio tramite firewall), rilevandolo mentre è in corso (IDS - Intrusion Detection System) o recuperando il sistema dopo che il danno è avvenuto (backup e disaster recovery). L'efficacia di una contromisura si misura in base alla sua capacità di ridurre il rischio residuo a un livello accettabile.
## 3.7 Risk assessment

Il **Risk Assessment** è il processo complessivo di identificazione, analisi e valutazione del rischio. L'obiettivo è determinare la probabilità che una minaccia sfrutti una vulnerabilità e l'impatto che questo evento avrebbe sugli asset aziendali.

Formalmente, il rischio può essere espresso come funzione di tre variabili:

$$Risk = f(Threat, Vulnerability, Impact)$$

Questo processo non è statico ma deve essere ripetuto periodicamente o quando avvengono cambiamenti significativi nell'infrastruttura o nel panorama delle minacce. Il risultato del Risk Assessment guida le decisioni sugli investimenti in sicurezza e sulla prioritizzazione delle contromisure.

![Immagine di Risk Assessment matrix probability impact](https://encrypted-tbn1.gstatic.com/licensed-image?q=tbn:ANd9GcQLoQK4lrURMZkuaeLGXKnmdEd5ZiCPMhsyT-jY-HWBAwD0-vVhsbd372_zau8c6UcKzjmAYUEB4i-x3vFUjap8uPkVV6yUKaTprSJTgMbIzkeiwyg)
# 4 Vulnerabilities

Le vulnerabilità possono essere categorizzate in base alla loro natura e alla loro collocazione all'interno dell'architettura del sistema. Una distinzione fondamentale è quella tra vulnerabilità locali e strutturali.
## 4.1 Local vulnerabilities

Le **Local Vulnerabilities** riguardano difetti specifici presenti in un singolo componente, host o applicazione. Esempi classici includono errori di programmazione come il **Buffer Overflow**, dove un programma scrive dati oltre i limiti del buffer allocato, o la **Race Condition**, causata da una gestione errata della concorrenza. Rientrano in questa categoria anche le configurazioni errate (misconfiguration), come permessi di file troppo permissivi o password di default non cambiate. La mitigazione di queste vulnerabilità avviene solitamente tramite l'applicazione di patch (patch management) o il hardening della configurazione del singolo asset.

## 4.2 Structural Vulnerabilties

Le **Structural Vulnerabilities** derivano da difetti nell'architettura complessiva della rete o del sistema, piuttosto che da bug nel codice di un singolo componente. Queste vulnerabilità sono spesso più difficili da correggere perché richiedono una riprogettazione dell'infrastruttura. Esempi includono la mancanza di segmentazione della rete (che permette movimenti laterali incontrollati), l'assenza di ridondanza (Single Point of Failure), o l'utilizzo di protocolli non cifrati all'interno di zone considerate erroneamente "fidate". Una vulnerabilità strutturale compromette la resilienza dell'intero sistema, rendendo inefficaci anche host singolarmente ben protetti.
## 4.3 Security Partial Views

Un errore comune nella gestione della sicurezza è l'adozione di **Partial Views**, ovvero la tendenza a focalizzarsi esclusivamente su un aspetto tecnologico credendo che questo garantisca la sicurezza totale. La sicurezza è una proprietà olistica del sistema e non può essere ridotta a un singolo meccanismo.
### 4.3.1 Encryption

La crittografia (**Encryption**) è essenziale per garantire confidenzialità e integrità, ma affidarsi solo ad essa è una visione parziale. Sebbene protegga i dati in transito e a riposo, la crittografia è inutile se l'endpoint è compromesso (l'attaccante vedrà i dati decifrati dall'utente legittimo) o se la gestione delle chiavi è debole. Inoltre, la crittografia non protegge dalla cancellazione dei dati (disponibilità) né impedisce a un malware di operare una volta all'interno del perimetro cifrato.
### 4.3.2 Authentication

L'**Authentication** verifica l'identità di un utente, ma da sola non garantisce che le azioni compiute siano legittime. Un sistema che si basa solo sull'autenticazione forte ignora il rischio rappresentato da utenti interni malintenzionati (**Insider Threat**) o da account compromessi. Inoltre, l'autenticazione è solo il preambolo all'**Authorization**; verificare chi è l'utente non serve a nulla se non si controlla rigorosamente cosa quell'utente ha il permesso di fare.
### 4.3.3 Attack infrastructure

Analizzare l'**Attack Infrastructure** (indirizzi IP, domini, server di Command & Control) è utile per l'Intelligence e il blocco immediato, ma è una visione parziale perché l'infrastruttura degli attaccanti è volatile. Gli indirizzi IP cambiano facilmente e i domini vengono generati algoritmicamente (**DGA**). Focalizzarsi solo sugli indicatori infrastrutturali (spesso definiti "Indicatori di Compromissione" a bassa vita utile) distoglie l'attenzione dalle Tattiche, Tecniche e Procedure (**TTPs**) dell'attaccante, che sono più stabili e più difficili da modificare per l'avversario.