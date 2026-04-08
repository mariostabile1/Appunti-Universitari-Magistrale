## Tipologie di Firewall e Confronto Architetturale

L'evoluzione dei firewall ha portato allo sviluppo di diverse tecnologie che operano a livelli differenti dello stack ISO/OSI, offrendo vari gradi di sicurezza e impatto sulle prestazioni.

I **Packet-Filtering Firewalls** operano controllando gli indirizzi IP sorgente e destinazione, ma non effettuano ispezioni sui layer superiori né mantengono lo stato delle connessioni, risultando molto veloci ma limitati nella capacità di rilevamento. I **Circuit-Level Gateways** introducono il controllo del _handshake_ TCP, verificando la validità della sessione senza però ispezionare il payload.

I **Stateful Inspection Firewalls** rappresentano un'evoluzione significativa, mantenendo una tabella di stato delle connessioni attive; questo permette di associare i pacchetti in ingresso alle richieste in uscita legittime, offrendo un buon bilanciamento tra sicurezza e performance. Infine, gli **Application-Level Gateways (Proxy Firewall)** offrono il livello di ispezione più profondo, analizzando il contenuto applicativo (Deep-Layer Inspection) e terminando la connessione per conto del client (Virtualized Connection). Sebbene offrano la sicurezza maggiore, hanno un impatto moderato o alto sulle risorse di sistema.

## Network Address Translation (NAT)

Il **Network Address Translation (NAT)** funge da meccanismo di sicurezza oscurando la struttura interna della rete. Un firewall NAT intercetta il traffico in uscita e sostituisce l'indirizzo IP sorgente interno (privato) con il proprio indirizzo pubblico, mantenendo una **Translation Table** per mappare le risposte di ritorno verso l'host interno corretto. Questo impedisce connessioni dirette dall'esterno verso gli host interni, a meno che non siano esplicitamente mappate.

Nonostante i benefici di sicurezza, il NAT è considerato un ostacolo per l'adozione di IPv6, che mira a ripristinare la connettività end-to-end eliminando la necessità di traduzione degli indirizzi.

## Architetture di Rete Sicure: Screened Subnet e Split DNS

Per proteggere i servizi esposti pubblicamente, si adotta l'architettura **Screened Subnet** (spesso coincidente con la DMZ). Questa configurazione utilizza due firewall: un _Exterior Firewall_ che filtra il traffico tra Internet e la DMZ, e un _Interior Firewall_ che protegge la rete interna dal traffico proveniente dalla DMZ.

Un principio cardine di questa architettura è che non deve mai esistere una rotta diretta tra la rete interna (Trusted) e Internet (Untrusted); tutto il traffico deve essere terminato o ispezionato nella DMZ. Se un attaccante compromette un servizio nella DMZ, deve superare un secondo livello di difesa per accedere alla rete interna.

Per nascondere la topologia interna, si implementa lo **Split DNS**. Si utilizzano due server DNS distinti: un server esterno, situato nella DMZ, che risolve solo i nomi dei server pubblici accessibili da Internet, e un server interno che gestisce i nomi di tutti gli host della rete aziendale. In questo modo, le interrogazioni DNS esterne non rivelano informazioni sugli asset interni non pubblici.

## Bastion Host

Un **Bastion Host** è un sistema calcolatore che deve essere esposto al pubblico (ad esempio un Web Server o un Mail Server nella DMZ) e che, per questo motivo, è esposto a un rischio elevato di attacco. Questi sistemi richiedono una procedura di _Hardening_ rigorosa: si disabilitano tutti i servizi non essenziali, si rimuovono account e user inutili e si configurano permessi restrittivi, minimizzando la superficie di attacco disponibile.

## Evasione tramite Tunneling Crittografato

I firewall tradizionali basati su porte e protocolli possono essere aggirati tramite il tunneling. Un utente interno può stabilire un tunnel cifrato (es. **SSH**) verso un server remoto in ascolto su una porta permessa (come la 80 o la 443).

All'interno di questo tunnel cifrato, l'utente può incapsulare traffico arbitrario (es. SMTP o protocolli vietati). Poiché il firewall vede solo il traffico cifrato sulla porta permessa, non può ispezionare il contenuto né applicare le policy di filtraggio, rendendo inefficace la protezione perimetrale.

## Web Application Firewall (WAF)

Il **Web Application Firewall (WAF)** è un dispositivo di sicurezza specializzato che opera al livello 7 (Applicazione), analizzando il traffico HTTP/HTTPS per proteggere le applicazioni web da attacchi specifici come SQL Injection, XSS e CSRF. A differenza dei firewall di rete (livelli 3-4), il WAF comprende la logica del protocollo web e può terminare la connessione SSL per ispezionare il traffico cifrato.

Il WAF utilizza due modelli di sicurezza:

- **Negative Security Model (Blacklisting):** Blocca le richieste che corrispondono a firme di attacchi noti. È efficace contro minacce conosciute ma inefficace contro gli 0-day.
    
- **Positive Security Model (Whitelisting):** Definisce rigorosamente il comportamento permesso (es. tipi di dati, lunghezza dei campi) e blocca tutto ciò che devia da questo profilo. È più sicuro ma genera più falsi positivi e richiede un tuning continuo.

Il WAF abilita il **Virtual Patching**: in presenza di una vulnerabilità nell'applicazione web, invece di modificare il codice sorgente (operazione lenta e costosa), si applica una regola sul WAF che intercetta e blocca i tentativi di sfruttamento di quella specifica vulnerabilità, proteggendo l'applicazione nell'immediato.

## Runtime Application Self-Protection (RASP)

Il **RASP** rappresenta un'evoluzione della sicurezza applicativa. Invece di agire come un filtro esterno (come il WAF), il RASP è integrato direttamente nell'ambiente di esecuzione dell'applicazione (es. nella JVM o nel framework .NET).

Grazie a questa posizione privilegiata, il RASP ha visibilità completa sul flusso di controllo e sui dati processati dall'applicazione. Può distinguere tra un comportamento normale e un attacco in tempo reale, bloccando l'esecuzione di istruzioni malevole (es. una query SQL manipolata) con una precisione superiore e meno falsi positivi rispetto ai controlli perimetrali.

## Microsegmentazione e Sicurezza Oltre il Perimetro

Nelle moderne architetture data center e cloud, il traffico **East-West** (tra server all'interno della rete) supera di gran lunga il traffico **North-South** (client-server). La sicurezza perimetrale tradizionale è inefficace contro i movimenti laterali di un attaccante che ha già superato il confine esterno.

La **Microsegmentazione** applica il principio Zero Trust all'interno del data center, isolando i carichi di lavoro (workloads) a livello granulare (fino alla singola macchina virtuale o container). Utilizzando firewall virtuali distribuiti e integrati negli hypervisor, è possibile definire policy di sicurezza che seguono l'applicazione ovunque essa venga spostata, superando la rigidità delle VLAN e dei firewall hardware tradizionali.

I **Virtual Firewalls** offrono vantaggi significativi rispetto alle controparti hardware: sono più economici, facili da dispiegare e scalare tramite automazione, e permettono di ispezionare il traffico inter-VM che non lascia mai l'host fisico, coprendo i "punti ciechi" delle appliance fisiche.