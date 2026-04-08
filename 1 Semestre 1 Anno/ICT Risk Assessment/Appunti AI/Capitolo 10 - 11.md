# 10 Deception with Honeypots

La sicurezza informatica non si limita alla difesa perimetrale statica; include anche tecniche di inganno (Deception) mirate a confondere, rallentare o analizzare l'attaccante. Un **Honeypot** è una risorsa di sicurezza il cui valore risiede nell'essere sondata, attaccata o compromessa. Concettualmente, è una trappola: un sistema che non ha alcun valore produttivo e nessuna attività legittima associata. Di conseguenza, qualsiasi interazione con un honeypot è per definizione sospetta o malevola (anomaly detection per definizione).
### 10.0.1 Classification

Gli honeypot vengono classificati principalmente in base al livello di interazione che permettono all'attaccante e allo scopo del loro utilizzo.

In base all'**Interaction Level**, distinguiamo:

- **Low-interaction honeypots**: Simulano servizi e protocolli (es. FTP, HTTP, Telnet) senza implementare un intero sistema operativo. L'attaccante interagisce con degli script che emulano le risposte attese. Sono facili da mantenere e meno rischiosi, ma raccolgono informazioni limitate e sono più facili da identificare come falsi (fingerprinting).
    
- **High-interaction honeypots**: Sono sistemi reali (spesso macchine virtuali) con un OS completo e applicazioni vulnerabili. Offrono all'attaccante un ambiente verosimile in cui operare, permettendo di catturare zero-day, rootkit e tattiche complesse. Tuttavia, comportano un rischio elevato: se l'attaccante esce dalla "gabbia" (breakout), può usare l'honeypot per attaccare altri sistemi.

In base allo **scopo**, si dividono in:

- **Production honeypots**: Posizionati all'interno della rete aziendale per rilevare intrusioni interne o distrarre l'attaccante dalle risorse reali.
    
- **Research honeypots**: Utilizzati da università e centri di ricerca per studiare le tendenze delle minacce, le motivazioni degli attaccanti e le nuove varianti di malware.

## 10.1 Honeyd

**Honeyd** è uno dei software open-source più noti per la creazione di honeypot a bassa interazione. È un demone che permette di creare host virtuali sulla rete. Un singolo computer fisico che esegue Honeyd può simulare centinaia o migliaia di indirizzi IP, ognuno con una configurazione diversa, popolando lo spazio di indirizzamento non utilizzato della rete (darkspace).
### 10.1.1 Architecture

L'architettura di Honeyd si basa sulla capacità di intercettare il traffico destinato a indirizzi IP che non corrispondono a macchine fisiche reali (tramite ARP spoofing o instradamento router). Una volta ricevuto il pacchetto, Honeyd utilizza un "Personality Engine". Questo motore è in grado di modificare le caratteristiche dei pacchetti in uscita per imitare lo stack TCP/IP di specifici sistemi operativi (ingannando strumenti di fingerprinting come Nmap). Se la configurazione prevede un servizio attivo (es. un server web IIS su Windows), Honeyd esegue script specifici che simulano la risposta del servizio alla richiesta dell'attaccante, registrando ogni passo dell'interazione.
### 10.1.2 Research results

L'uso estensivo di strumenti come Honeyd nella ricerca ha permesso di comprendere meglio le dinamiche di propagazione dei worm e delle botnet. I dati raccolti mostrano come il "rumore di fondo" di Internet (scansioni automatiche) sia costante e come il tempo medio prima che un dispositivo non protetto connesso alla rete venga attaccato sia estremamente ridotto (spesso questione di minuti). Inoltre, ha evidenziato come gli attaccanti utilizzino spesso script automatizzati che non verificano la veridicità del bersaglio, cadendo facilmente nella trappola dei sistemi a bassa interazione.
# 11 Countermeasures

Le contromisure tecniche non si limitano all'aggiunta di dispositivi di sicurezza, ma devono essere integrate nel ciclo di sviluppo del software e nell'architettura di rete.
## 11.1 Robust Programming

La **Robust Programming** (o Secure Coding) mira a sviluppare software in grado di gestire input anomali o malformati senza crashare o permettere comportamenti imprevisti. La maggior parte delle vulnerabilità software deriva da un'assunzione errata di fiducia nei dati provenienti dall'esterno.
### 11.1.1 Input validation

La regola d'oro della programmazione sicura è "Mai fidarsi dell'input utente". L'**Input Validation** è il processo di verifica che i dati ricevuti soddisfino i criteri attesi (tipo, lunghezza, formato, range) prima di elaborarli.

Esistono due approcci:

- **Blacklisting**: Rifiutare input noti come malevoli (es. bloccare la stringa `<script>`). È un approccio debole perché è impossibile elencare tutte le possibili varianti di attacco.
    
- **Whitelisting** (o Positive Validation): Accettare solo input che corrispondono rigorosamente a un modello sicuro noto (es. accettare solo caratteri alfanumerici per un username). Questo approccio è molto più robusto.

Oltre alla validazione, è necessaria la **Sanitization** (o escaping), che trasforma i caratteri pericolosi in formati sicuri (es. convertire `<` in `&lt;`) per neutralizzare il loro significato sintattico in contesti come HTML o SQL.
### 11.1.2 CWE - Vulnerabilities Ranking

La comunità di sicurezza mantiene il **Common Weakness Enumeration** (CWE), un elenco formale dei tipi di debolezze software. A differenza del CVE (che elenca specifiche istanze di vulnerabilità in specifici prodotti), il CWE classifica le categorie di errori (es. "CWE-89: SQL Injection"). Il "CWE Top 25 Most Dangerous Software Weaknesses" è una classifica aggiornata periodicamente che elenca gli errori di programmazione più diffusi e critici, guidando gli sviluppatori su cosa prioritizzare durante la code review.
## 11.2 Firewall

Il **Firewall** è un dispositivo di sicurezza di rete (hardware o software) che monitora e controlla il traffico in entrata e in uscita basandosi su regole di sicurezza predefinite. Agisce come una barriera tra una rete fidata (interna) e una rete non fidata (Internet).
### 11.2.1 Segmenting

La funzione primaria di un firewall in un'architettura moderna non è solo proteggere il perimetro esterno, ma implementare la segmentazione interna. Il firewall funge da punto di strozzatura (**Choke Point**) obbligatorio attraverso cui deve passare il traffico tra diverse zone di sicurezza (es. tra la rete uffici e la rete server). Questo permette di applicare controlli granulari e impedire che la compromissione di un segmento si propaghi automaticamente agli altri.
### 11.2.2 Classification

I firewall si sono evoluti nel tempo, offrendo livelli crescenti di ispezione:

1. **Packet Filtering (Stateless)**: Opera a livello di rete (L3) e trasporto (L4). Esamina le intestazioni dei pacchetti (IP sorgente/destinazione, porte, protocollo) in modo isolato. È veloce ma poco sicuro, poiché non conosce il contesto della connessione.
    
2. **Stateful Inspection**: Mantiene una tabella di stato delle connessioni (**State Table**). Capisce se un pacchetto fa parte di una connessione esistente, ne inizia una nuova o è non valido. Permette il traffico di ritorno solo se richiesto dall'interno.
    
3. **Application Level Gateway (Proxy)**: Opera a livello applicazione (L7). Interrompe la connessione tra client e server, agendo da intermediario. Ispeziona il payload del pacchetto, permettendo di bloccare comandi specifici (es. "GET" HTTP permesso, "POST" negato). È molto sicuro ma introduce latenza.
    
4. **Next-Generation Firewall (NGFW)**: Integra le funzionalità stateful con Deep Packet Inspection (DPI), Intrusion Prevention System (IPS), controllo delle applicazioni e utilizzo di threat intelligence esterna.
### 11.2.3 Pros & Cons analysis

I vantaggi dell'uso dei firewall includono la centralizzazione del controllo degli accessi, la capacità di generare log dettagliati per l'auditing e la protezione contro minacce esterne basate su scansioni e exploit di rete.

Tuttavia, presentano limitazioni significative: possono diventare un collo di bottiglia per le prestazioni (**Single Point of Failure** se non ridondati); non proteggono contro minacce interne (insider threat) che non attraversano il firewall; e hanno visibilità limitata o nulla sul traffico cifrato (VPN, HTTPS) a meno che non si applichino tecniche di ispezione SSL/TLS (Man-in-the-Middle autorizzato).
### 11.2.4 Wrapping Up

L'efficacia di un firewall dipende interamente dalla qualità della sua configurazione. Un firewall con regole "Any-Any Allow" è inutile. La gestione delle regole (Rule Management) è un processo critico: regole obsolete o conflittuali (Shadowing) possono creare buchi di sicurezza inavvertiti.
### 11.2.5 Takeaway guidelines

Per una configurazione sicura:

- **Default Deny**: La regola finale (implicita o esplicita) deve essere "Blocca tutto ciò che non è esplicitamente permesso".
    
- **Principio del privilegio minimo**: Aprire solo le porte strettamente necessarie verso gli IP strettamente necessari.
    
- **Logging**: Registrare i pacchetti scartati (Dropped packets) per analizzare tentativi di scansione.
## 11.3 Segmentation

La **Segmentation** è la pratica di dividere la rete in sottoreti logiche o fisiche per migliorare le prestazioni e la sicurezza. L'esempio più comune è la creazione di una **DMZ** (Demilitarized Zone).

La DMZ è una sottorete fisica o logica che espone i servizi esterni dell'organizzazione (Web server, Mail server, DNS) a Internet. La DMZ è separata sia da Internet che dalla rete interna (LAN) tramite firewall.

La regola di flusso tipica è:

- Internet può accedere alla DMZ.
    
- La DMZ non può accedere alla LAN (o ha accesso estremamente limitato).
    
- La LAN può accedere a Internet e alla DMZ.
    
    In caso di compromissione di un server pubblico nella DMZ, l'attaccante si trova isolato e non ha accesso diretto ai dati sensibili nella rete interna.
## 11.4 VPN

Una **Virtual Private Network** (VPN) estende una rete privata attraverso una rete pubblica (Internet), permettendo agli utenti di inviare e ricevere dati come se i loro dispositivi fossero direttamente connessi alla rete privata.

La sicurezza della VPN si basa sul **Tunneling** (incapsulamento dei pacchetti) e sulla **Encryption** (cifratura del contenuto del tunnel).

Esistono due tipologie principali:

- **Site-to-Site**: Collega intere reti (es. la sede centrale con una filiale). È trasparente per gli utenti finali.
    
- **Remote Access**: Collega singoli host alla rete aziendale (es. dipendenti in smart working). Richiede un client software sull'host.
    
    I protocolli più diffusi sono **IPsec** (nativo, opera a livello 3) e **SSL/TLS** (usato da OpenVPN e soluzioni browser-based, opera a livello applicativo/sessione), quest'ultimo spesso preferito per la facilità di attraversamento dei firewall (NAT traversal).

### 11.4.1 IPSec

All'interno del framework IPsec, la protezione del traffico dati è affidata principalmente a due protocolli distinti, che possono essere usati singolarmente o in combinazione: **Authentication Header (AH)** ed **Encapsulating Security Payload (ESP)**.
### 11.4.2 Authentication Header

**Authentication Header (AH)** è progettato per garantire l'integrità dei dati, l'autenticazione dell'origine dei dati e la protezione contro gli attacchi di replay. AH calcola una firma digitale (ICV - Integrity Check Value) sull'intero pacchetto IP, inclusi i dati e la maggior parte dei campi dell'intestazione IP originale (esclusi quelli mutabili come il TTL). È fondamentale notare che AH **non offre confidenzialità**: il contenuto del pacchetto rimane in chiaro. Questo protocollo è utile in scenari dove è necessario assicurarsi che il pacchetto provenga da una fonte fidata e non sia stato manipolato, ma dove la segretezza del contenuto non è richiesta o è gestita altrove.
### 11.4.3 Encapsulating Security Payload

**Encapsulating Security Payload (ESP)** è il protocollo più completo e diffuso, in quanto fornisce confidenzialità attraverso la crittografia, oltre a integrità, autenticazione dell'origine e protezione anti-replay. A differenza di AH, ESP non protegge l'intestazione IP esterna originale nella stessa misura, ma incapsula il payload (che può includere i protocolli di livello superiore o l'intero pacchetto IP originale, a seconda della modalità) e lo cifra. ESP aggiunge un'intestazione e una coda (trailer) al pacchetto.

Sia AH che ESP possono operare in due modalità distinte, che determinano come il pacchetto IP originale viene trattato: **Transport Mode** e **Tunnel Mode**.
### 11.4.4 Transport Mode

Nel **Transport Mode**, la protezione viene applicata solo al payload del pacchetto IP (solitamente un segmento TCP o UDP), mentre l'intestazione IP originale viene mantenuta e utilizzata per il routing. Questa modalità è tipicamente impiegata per comunicazioni end-to-end tra due host specifici (Host-to-Host). L'intestazione IPsec viene inserita tra l'intestazione IP originale e il payload. Il vantaggio è un minore overhead, ma l'infrastruttura di rete intermedia può vedere gli indirizzi IP originali di sorgente e destinazione.
### 11.4.5 Tunnel Mode

Nel **Tunnel Mode**, l'intero pacchetto IP originale viene incapsulato all'interno di un nuovo pacchetto IPsec. Il pacchetto originale viene cifrato (nel caso di ESP) e diventa il payload del nuovo pacchetto. Viene aggiunta una nuova intestazione IP esterna che instrada il pacchetto tra i gateway di sicurezza (ad esempio due firewall o router VPN). Questa modalità è lo standard per le VPN Site-to-Site o Client-to-Site, poiché nasconde completamente la struttura della rete interna e gli indirizzi IP originali, esponendo solo gli indirizzi pubblici dei gateway.