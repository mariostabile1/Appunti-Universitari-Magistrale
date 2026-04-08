## Oltre la Sicurezza Perimetrale e Microsegmentazione

La sicurezza perimetrale costituisce una parte significativa dei controlli di sicurezza di rete della maggior parte delle organizzazioni. Dispositivi come i firewall di rete ispezionano il traffico "North-South" (da client a server) che attraversa il perimetro di sicurezza per bloccare il traffico malevolo . Tuttavia, gli asset all'interno del perimetro sono spesso implicitamente considerati fidati, permettendo al traffico "East-West" (da workload a workload) di fluire senza ispezione. Poiché per molte organizzazioni le comunicazioni East-West costituiscono la maggioranza dei pattern di traffico nei data center e nel cloud, le difese focalizzate sul perimetro mancano di visibilità su questo fronte, offrendo agli attori malevoli l'opportunità di muoversi lateralmente tra i workload .

Mentre la rete crea percorsi affidabili tra i workload, la **Microsegmentazione** crea isolamento e determina se due endpoint debbano effettivamente accedere l'uno all'altro. Applicare la segmentazione con un accesso basato sul principio del _least privilege_ riduce l'ambito del movimento laterale e contiene le violazioni dei dati . Questo concetto è totalmente coerente con l'architettura **Zero Trust**.

### Firewall Virtuali vs Hardware

Nell'implementazione delle difese, si distinguono firewall hardware e virtuali. I firewall hardware sono appliance fisiche individuali installate tra elementi di rete, che richiedono attività fisiche come il cablaggio e personale qualificato per l'installazione e la gestione, comportando costi iniziali elevati. Al contrario, i firewall virtuali sono software installati su server o macchine virtuali, operando su un sistema operativo di sicurezza che gira su hardware generico con un livello di virtualizzazione. Essi offrono un dispiegamento rapido tramite strumenti di automazione cloud e sono utilizzabili anche da esperti di sicurezza non specializzati in networking, fornendo spesso un ROI significativo grazie ai minori costi di dispiegamento e manutenzione.

## Virtual Private Networks (VPN)

Una **Virtual Private Network (VPN)** è una rete overlay che emula una connessione sicura al di sopra di una rete pubblica (e quindi non sicura) come Internet. Esistono principalmente due tipologie di VPN:

- **Remote Access VPN (Client-to-Site):** Permette a lavoratori remoti o utenti mobili di connettersi a una rete privata o a un server di terze parti tramite SSL/TLS, utile per accedere a file aziendali o navigare in modo sicuro su connessioni instabili.
    
- **Site-to-Site VPN:** Collega un'intera rete a un'altra rete (LAN, WAN), utilizzata da grandi organizzazioni per unire reti interne dislocate in sedi diverse .

La VPN site-to-site connette sottoreti locali (che possono includere anche una singola macchina) e, assumendo che un firewall protegga ogni rete locale, la VPN protegge il traffico tra i firewall . Tuttavia, i concentratori VPN sono vulnerabili ad attacchi **DDoS** poiché devono decifrare le comunicazioni in entrata per distinguere il traffico legittimo da quello "spazzatura", consumando ingenti risorse computazionali .

## Protocollo IPsec

**IPsec** è un'estensione per IPv4 progettata per cifrare e autenticare il flusso di informazioni (una soluzione temporanea in attesa della transizione a IPv6). A differenza di SSL/TLS che opera a livello di trasporto o applicazione, IPsec opera al livello di rete (Layer 3) .

IPsec definisce due protocolli principali:

1. **Authentication Header (AH):** Fornisce autenticazione mutua e integrità del messaggio, ma non confidenzialità (i dati sono in chiaro).
    
2. **Encapsulating Security Payload (ESP):** Garantisce confidenzialità cifrando il contenuto scambiato, oltre a poter fornire autenticazione.

Entrambi i protocolli possono operare in due modalità:

- **Transport Mode:** Vengono aggiunti nuovi campi al pacchetto originale. È utilizzato per proteggere connessioni _point-to-point_ tra due nodi, cifrando il payload ma lasciando visibile l'header IP originale, il che espone a rischi di analisi del traffico e dei metadati .
    
- **Tunnel Mode:** L'intero pacchetto IP originale diventa il contenuto informativo di un nuovo pacchetto. È utilizzato per le VPN _site-to-site_ per proteggere le comunicazioni tra due reti attraverso una rete non fidata. Nasconde i pattern di traffico poiché gli indirizzi IP interni originali sono incapsulati e invisibili all'esterno .

### Gestione delle Chiavi e Security Association

IPsec utilizza la crittografia simmetrica per proteggere ogni connessione tra nodi per motivi di efficienza, mentre utilizza la crittografia asimmetrica (come **Diffie-Hellman** o la sua variante a Curve Ellittiche **ECDH**) per lo scambio iniziale e la determinazione della chiave condivisa . Le chiavi condivise vengono aggiornate con una frequenza predefinita per minimizzare l'impatto di un'eventuale compromissione.

Il concetto fondamentale in IPsec è la **Security Association (SA)**, che descrive una connessione diretta e i servizi di sicurezza associati. Poiché le SA sono unidirezionali, per proteggere una comunicazione bidirezionale sono necessarie due SA (una per direzione) . Le SA sono gestite nel **Security Associations Database (SAD)**. Ogni SA è identificata univocamente dal **Security Parameters Index (SPI)**, un indice a 32 bit inserito nell'header del pacchetto che funge da chiave di ricerca nel SAD del destinatario per recuperare algoritmi e chiavi necessari a processare il pacchetto .

### Protocolli di Negoziazione: ISAKMP e IKE

**ISAKMP** (Internet Security Association and Key Management Protocol) gestisce il ciclo di vita delle SA, la generazione delle chiavi crittografiche e l'autenticazione dei peer . La negoziazione avviene in fasi, utilizzando messaggi per concordare gli attributi di sicurezza e scambiare valori pubblici Diffie-Hellman. ISAKMP può operare in **Main Mode** (6 messaggi, protegge l'identità dei server cifrandola) o **Aggressive Mode** (3 messaggi, più veloce ma non protegge l'identità dei server) .

**IKEv2** (Internet Key Exchange version 2) è l'evoluzione del protocollo, sviluppata per migliorare efficienza e affidabilità. Semplifica lo scambio iniziale a soli quattro messaggi, incorpora numeri di sequenza e acknowledgment per l'affidabilità, e offre resistenza nativa agli attacchi DoS verificando l'esistenza del richiedente prima di processare richieste pesanti. Inoltre, incapsulando i messaggi in UDP, IKEv2 supporta il **NAT Traversal**, permettendo alle connessioni IPsec di attraversare firewall e dispositivi NAT .

## Dettagli dei Pacchetti

Nel protocollo **AH**, il pacchetto finale contiene l'header IP originale, seguito dall'header AH (che include il _Message Integrity Code_ calcolato con MD5 o SHA-1) e infine il payload. Tutto il pacchetto è autenticato, ma il payload rimane leggibile .

Nel protocollo **ESP**, l'header ESP viene inserito dopo l'header IP (in Transport Mode) o dopo il nuovo header IP esterno (in Tunnel Mode). Il payload originale e il trailer ESP vengono cifrati, garantendo la confidenzialità. È possibile aggiungere un campo di autenticazione ESP in coda per garantire anche l'integrità .