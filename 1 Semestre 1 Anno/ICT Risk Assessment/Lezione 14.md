## Access Control: Capabilities vs ACL

L'implementazione della matrice di controllo degli accessi può avvenire per colonne o per righe.

L'approccio per colonne corrisponde alle **Access Control Lists (ACL)**, dove ogni oggetto mantiene una lista di soggetti e i relativi permessi.

L'approccio per righe è definito dalle **Capabilities**. Una capability è un token infalsificabile che un soggetto possiede e che gli conferisce il diritto di accedere a un oggetto. È analogo a un biglietto per un evento: chi lo possiede entra, senza necessità di controllare un'identità su una lista.

La gestione delle capabilities presenta sfide specifiche:

- **Protection:** devono essere protette dalla falsificazione (tampering) e dal furto.
    
- **Generation:** solo il server o il gestore delle risorse può crearle.
    
- **Delegation:** un soggetto può trasmettere la capability a un altro per parallelizzare il lavoro, ma ciò aumenta il rischio di furto.
    
- **Revocation:** è estremamente difficile revocare un diritto concesso tramite capability (problema della revoca), poiché una volta delegata non se ne ha più il controllo diretto.

## Evoluzione dei Firewall

### Stateful Firewall

A differenza del _packet filtering_ (o firewall stateless) che valuta ogni pacchetto isolatamente, lo **Stateful Firewall** mantiene una **State Table** per tracciare lo stato delle connessioni di rete.

Questo firewall ispeziona non solo l'header IP, ma verifica se un pacchetto appartiene a una connessione attiva e legittima (es. verifica il _three-way handshake_ TCP). Una regola tipica permette il traffico in ingresso solo se è una risposta a una connessione iniziata dall'interno (_Established_). Se la tabella di stato si riempie (ad esempio sotto attacco DoS), il firewall degrada a semplice packet filter o blocca il traffico.

### Application Gateway (Proxy)

Un **Application Gateway** o **Proxy** agisce come un intermediario a livello applicativo. Interrompe la connessione diretta tra client e server esterno: il client si connette al proxy, e il proxy si connette alla destinazione.

Questo permette un'ispezione profonda del protocollo applicativo (es. HTTP, FTP, SMTP), filtrando comandi specifici (es. bloccare `PUT` in HTTP) o contenuti. Offre vantaggi come il _caching_ delle risposte e il mascheramento della struttura interna della rete (NAT), ma introduce un collo di bottiglia nelle prestazioni e richiede configurazioni specifiche per ogni applicazione.

### Next Generation Firewall (NGFW) e Deep Packet Inspection

I **Next Generation Firewall (NGFW)** superano il concetto di porta e protocollo standard. Utilizzano la **Deep Packet Inspection (DPI)** per analizzare il payload del pacchetto e identificare l'applicazione reale indipendentemente dalla porta usata (es. rilevare traffico Skype o BitTorrent su porta 80). Possono decifrare il traffico SSL/TLS per ispezionare contenuti cifrati, prevenendo l'uso di tunnel cifrati per nascondere malware o esfiltrazione dati.

## Architettura di Rete e DMZ

La **Demilitarized Zone (DMZ)** o _Screened Subnet_ è un segmento di rete isolato che funge da zona cuscinetto tra la rete interna sicura (Trusted) e la rete esterna insicura (Untrusted/Internet).

I servizi che devono essere accessibili dall'esterno (es. Web Server, Mail Server, DNS pubblico) vengono collocati nella DMZ. In questo modo, se un attaccante compromette un server pubblico ("Bastion Host"), si trova isolato nella DMZ e non ha accesso diretto alla rete interna, dove risiedono gli asset critici come i Database Server.

## Sandboxing e Isolamento

Il **Sandboxing** è una tecnica di isolamento che permette di eseguire codice non fidato o sospetto in un ambiente ristretto, limitando l'accesso alle risorse del sistema ospite (file system, rete, altri processi). Se il codice è malevolo, i danni restano confinati nella sandbox.

Esistono diverse tecniche di implementazione:

- **Virtualizzazione:** Esecuzione in una VM separata (es. Windows Sandbox), offrendo un isolamento forte basato su hardware.
    
- **Containerizzazione:** Isolamento a livello OS (es. Docker), condividendo il kernel ma isolando lo user space.
    
- **Restrizione dei privilegi:** Uso di meccanismi OS per ridurre i diritti del processo.

### Caso di Studio: Google Chrome Sandbox

Chrome utilizza un'architettura multiprocesso per stabilità e sicurezza.

- **Broker Process:** È il processo principale (Browser Kernel), gira con i privilegi dell'utente e gestisce l'accesso alle risorse (file, rete).
    
- **Target Process:** Sono i processi che renderizzano le pagine (Renderer), eseguono JavaScript o decodificano video. Questi girano all'interno della sandbox con privilegi minimi.

I processi Target non possono accedere direttamente al file system o creare nuove finestre; devono richiedere queste azioni al Broker tramite **IPC (Inter-Process Communication)**. Il Broker applica le policy di sicurezza e decide se soddisfare la richiesta.

## Meccanismi di Sicurezza del Sistema Operativo

Per implementare sandboxing e isolamento, i sistemi operativi moderni offrono meccanismi granulari.

### Linux Capabilities

Linux scompone i privilegi del superutente (root) in unità distinte chiamate **Capabilities**. Invece di concedere pieni poteri (UID 0), si assegnano solo i permessi necessari (es. `CAP_NET_BIND_SERVICE` per aprire porte < 1024, `CAP_SYS_TIME` per cambiare l'orario). Le _Ambient Capabilities_ permettono di ereditare questi permessi dai processi figli, utile per servizi che non girano come root.

### Windows Integrity Levels e Restricted Tokens

Windows implementa il **Mandatory Integrity Control (MIC)**, che assegna un livello di integrità (Low, Medium, High, System) a ogni processo e oggetto.

Un processo può scrivere solo su oggetti con livello di integrità inferiore o uguale (_No Write Up_) e leggere da oggetti con livello superiore o uguale. Questo previene attacchi di tipo _Shatter_ dove un processo meno privilegiato invia messaggi a una finestra di un processo privilegiato per eseguirvi codice.

Inoltre, i **Restricted Tokens** (o Low Privilege Tokens) permettono di creare una copia del token di accesso di un processo rimuovendo privilegi o gruppi specifici. Chrome usa questa tecnica per creare i token dei processi renderer, limitando drasticamente ciò che possono toccare nel sistema.