## Infrastrutture di Attacco e Botnet

Un attaccante strutturato non lancia mai attacchi direttamente dai propri sistemi, ma costruisce preventivamente un'infrastruttura di attacco. Elemento cardine di questa infrastruttura è la **Botnet**, definita come una _overlay network_ (rete logica) costituita da risorse di altri sistemi precedentemente compromessi . Le intrusioni necessarie per costruire una botnet sono tipicamente _stealth_, ovvero a basso rumore e lente, per evitare di essere rilevate.

Nel mercato criminale (underground economy), esistono gruppi specializzati esclusivamente nella creazione di botnet, che vengono poi affittate ad altri attaccanti per eseguire le intrusioni finali. L'infrastruttura è gestita tramite server di **Command & Control (C2)**, che coordinano le azioni dei nodi infetti. La difesa contro queste minacce può assumere una forma attiva, nota come _offensive security_ o _defense forward_, che mira a disarticolare la botnet stessa (ad esempio tramite attacchi simultanei), poiché attacchi parziali risultano inefficaci dato che la rete tende a rigenerarsi.

## Vulnerability Assessment e Penetration Testing

La gestione della sicurezza richiede la distinzione tra due attività fondamentali: il Vulnerability Assessment e il Penetration Testing.

Il **Vulnerability Assessment (VA)** è il processo di identificazione, quantificazione e prioritizzazione delle vulnerabilità in un sistema. Si avvale di strumenti automatizzati (scanner) e mira alla copertura completa (_breadth over depth_), cercando di individuare il maggior numero possibile di vulnerabilità note senza però sfruttarle . È un'attività focalizzata sull'elencazione dei difetti di sicurezza.

Il **Penetration Testing (PT)**, al contrario, è un'attività orientata all'obiettivo (_goal-oriented_). Simula il comportamento di un attaccante reale per verificare se è possibile raggiungere un determinato scopo (es. esfiltrare dati o compromettere un sistema critico) sfruttando le vulnerabilità . Il PT privilegia la profondità (_depth over breadth_) e spesso utilizza catene di vulnerabilità (_chaining_) per ottenere l'accesso. A differenza del VA, il PT è prevalentemente manuale o semi-automatizzato e verifica l'effettiva sfruttabilità dei difetti trovati.

## Funzionamento dei Vulnerability Scanner

I **Vulnerability Scanner** sono strumenti software che prendono in input un insieme di indirizzi IP o URL e restituiscono un elenco di vulnerabilità, spesso associate a un punteggio di severità (CVSS) . Il cuore dello scanner è un database di vulnerabilità note e una serie di script o plugin progettati per rilevarle.

L'efficacia di uno scanner è misurata dalla sua capacità di minimizzare gli errori di rilevamento:

- **False Positive (Falso Positivo):** Lo scanner segnala una vulnerabilità che in realtà non esiste. Questo errore consuma risorse umane per la verifica manuale, aumentando i costi operativi.
    
- **False Negative (Falso Negativo):** Lo scanner non rileva una vulnerabilità presente. Questo è l'errore più critico poiché lascia il sistema esposto a rischi ignoti.

Esiste un trade-off intrinseco: ridurre i falsi positivi tende spesso ad aumentare i falsi negativi, e viceversa.

## Architetture e Tipologie di Scansione

Le scansioni possono essere condotte con diverse modalità architetturali e livelli di privilegio, ognuna con specifici vantaggi e svantaggi.

### Scansione Esterna vs Interna

La **External Scanning** simula la prospettiva di un attaccante remoto su Internet, verificando la sicurezza del perimetro (firewall, servizi esposti). È utile per identificare le minacce che provengono dall'esterno. La **Internal Scanning** viene eseguita dall'interno della rete (dietro il firewall). Assume che l'attaccante abbia già superato il perimetro o sia un _insider_ (es. un dipendente infedele o un malware interno). Questa modalità offre una visione più profonda della sicurezza interna della rete.
### Authenticated vs Unauthenticated Scanning

La **Unauthenticated Scanning** (senza credenziali) opera analizzando i servizi di rete esposti e inferendo la presenza di vulnerabilità basandosi sulle risposte ottenute (es. banner grabbing, analisi dei pacchetti TCP/IP) . Poiché si basa su inferenze, è soggetta a un alto tasso di falsi positivi e negativi, non potendo verificare la configurazione interna o le patch installate localmente.

La **Authenticated Scanning** (o Credentialed Scanning) utilizza credenziali amministrative per accedere direttamente al sistema target. Questo approccio _White Box_ permette allo scanner di interrogare direttamente il registro di sistema, il file system e l'elenco dei pacchetti installati, garantendo una precisione molto superiore e rilevando vulnerabilità client-side non visibili dalla rete.
### Agent-Based Scanning

In contesti dinamici o distribuiti, si utilizza l'**Agent-Based Scanning**, dove un agente software viene installato direttamente su ogni host. L'agente esegue le scansioni localmente e invia i risultati a un server centrale. Questa soluzione è ideale per dispositivi mobili (laptop che si spostano tra reti diverse) e risolve problemi di credenziali e finestre di manutenzione, riducendo anche il traffico di rete necessario per la scansione .

## Modalità di Rilevamento e Rischi

### Scansione Attiva vs Passiva

La **Active Scanning** invia pacchetti di test o probe ai sistemi target e analizza le risposte. Sebbene fornisca un'istantanea precisa, può causare disservizi o essere rilevata dai sistemi di difesa. La **Passive Scanning** monitora il traffico di rete senza interagire direttamente con i sistemi. È meno intrusiva e opera in tempo reale, ma non può rilevare vulnerabilità su servizi che non stanno comunicando attivamente o vulnerabilità lato client non sollecitate.
### Scansione Intrusiva vs Non Intrusiva

La **Non-intrusive Scanning** si limita a rilevare la possibilità di una vulnerabilità senza sfruttarla (es. controllando la versione di un software). La **Intrusive Scanning** tenta di sfruttare la vulnerabilità per confermarne l'esistenza (simile a un exploit). Sebbene riduca i falsi positivi, comporta un alto rischio di danneggiare il sistema o interrompere i servizi, rendendola adatta solo ad ambienti di test .

### Client Scanning

Una variante particolare è il **Client Scanning**, dove un server scansiona il client che si connette (es. un provider internet che controlla il PC dell'utente). Questa pratica solleva significativi problemi di privacy, in quanto il server acquisisce informazioni dettagliate sulla configurazione dell'utente .

## Valutazione Continua (Continuous Assessment)

La sicurezza non è uno stato statico. Anche dopo aver rimediato a tutte le vulnerabilità rilevate, il rischio cambia nel tempo a causa del **drift**: ogni giorno vengono scoperte nuove vulnerabilità (circa 20 al giorno), emergono nuove tecniche di attacco e nuovi attaccanti . Pertanto, è necessario pianificare scansioni ricorrenti (_scheduled scanning_) con una frequenza proporzionale all'appetito per il rischio dell'organizzazione, oltre a scansioni straordinarie in caso di modifiche significative al sistema o alla pubblicazione di nuove vulnerabilità critiche.