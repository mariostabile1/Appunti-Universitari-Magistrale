# 7 Patching

Il processo di **Patching** rappresenta una delle attività fondamentali nella gestione operativa della sicurezza e nella mitigazione delle vulnerabilità. Consiste nell'aggiornamento dei componenti software (sistema operativo, librerie, applicazioni) per correggere difetti noti, spesso scoperti dopo il rilascio. Una gestione efficace delle patch richiede un inventario accurato degli asset e una valutazione prioritaria degli aggiornamenti, poiché applicare ogni patch immediatamente potrebbe causare instabilità o disservizi (problemi di regressione). Il ritardo nell'applicazione delle patch lascia aperta una finestra di esposizione che gli attaccanti possono sfruttare, specialmente dopo che la vulnerabilità è stata resa pubblica (Disclosure).
### 7.0.1 Common Vulnerability Scoring System

Per prioritizzare le operazioni di patching, l'industria adotta standard globali come il **Common Vulnerability Scoring System** (CVSS). Questo framework fornisce un metodo per catturare le caratteristiche principali di una vulnerabilità e produrre un punteggio numerico (da 0.0 a 10.0) che ne riflette la gravità tecnica.

Il punteggio CVSS si compone di tre gruppi di metriche. Il **Base Score** rappresenta le qualità intrinseche della vulnerabilità, che non cambiano nel tempo o in base all'ambiente (es. la facilità di exploit o l'impatto su riservatezza, integrità e disponibilità). Il **Temporal Score** riflette le caratteristiche che evolvono nel tempo, come la disponibilità di un codice di exploit pubblico o il rilascio di una patch ufficiale. Infine, l'**Environmental Score** adatta il punteggio al contesto specifico dell'organizzazione, considerando la criticità dell'asset colpito e l'efficacia dei controlli di sicurezza locali.
### 7.0.2 CVSS revisions

Il CVSS è un standard in evoluzione. Le versioni precedenti (come la v2) si concentravano quasi esclusivamente sull'impatto tecnico immediato. Le revisioni successive (v3, v3.1, v4.0) hanno affinato le metriche per considerare meglio il contesto, come l'interazione utente necessaria (User Interaction) o la modifica dell'ambito di sicurezza (**Scope**), che si verifica quando una vulnerabilità in un componente impatta risorse gestite da un'autorità di sicurezza diversa (ad esempio, un exploit in una macchina virtuale che compromette l'host fisico). L'obiettivo delle revisioni è fornire una rappresentazione sempre più fedele del rischio reale, distinguendo tra vulnerabilità teoriche e quelle facilmente sfruttabili in natura.

# 8 Countermeasures

Le **Countermeasures** sono l'insieme delle tecniche e procedure adottate per proteggere il sistema. Non si limitano alla tecnologia, ma includono policy e controlli fisici. L'efficacia di una contromisura non è binaria, ma si valuta in termini di costi-benefici e di riduzione del rischio residuo.
## 8.1 Robustness and Resilience

Nel progettare sistemi sicuri è cruciale distinguere tra robustezza e resilienza.

La **Robustness** è la capacità di un sistema di resistere agli attacchi e prevenire la compromissione. Un sistema robusto è "duro" da penetrare; l'obiettivo è mantenere lo stato sicuro nonostante le sollecitazioni esterne.

La **Resilience** è la capacità di un sistema di assorbire l'impatto di un incidente, adattarsi e recuperare le funzionalità operative in tempi accettabili. Un sistema resiliente accetta che il fallimento possa avvenire, ma è progettato per degradare le prestazioni in modo controllato (**Graceful Degradation**) piuttosto che collassare, garantendo la continuità dei servizi essenziali.
### 8.1.1 Minimal system

Il principio del **Minimal System** (o _Least Functionality_) suggerisce che un sistema dovrebbe essere configurato per fornire solo i servizi e le funzioni essenziali per il suo scopo operativo. Ogni servizio aggiuntivo, porta aperta o applicazione installata aumenta la superficie di attacco (**Attack Surface**). Rimuovere o disabilitare componenti non necessari riduce le probabilità che esistano vulnerabilità sfruttabili e semplifica le attività di patching e monitoraggio.
## 8.2 Authentication outline

L'**Authentication** è il processo di verifica dell'identità di un soggetto (utente o macchina). È il prerequisito per l'autorizzazione. Generalmente si basa su uno o più fattori: qualcosa che l'utente sa (password), qualcosa che l'utente ha (smart card, token), o qualcosa che l'utente è (biometria).
### 8.2.1 Authentication Mechanisms

I meccanismi di autenticazione variano in complessità e sicurezza. L'autenticazione semplice basata su password è vulnerabile a intercettazione (se non cifrata), guessing e phishing. Meccanismi più forti utilizzano protocolli di **Challenge-Response**, dove il server invia una sfida (un numero casuale o _nonce_) e l'utente deve rispondere con un valore calcolato utilizzando la propria chiave segreta, dimostrando di conoscerla senza mai trasmetterla in rete. L'autenticazione a più fattori (**MFA**) combina diversi tipi di credenziali per elevare significativamente la barriera di sicurezza.
### 8.2.2 Kerberos

**Kerberos** è un protocollo di autenticazione di rete standard che consente a entità che comunicano su una rete non sicura di provare la propria identità reciprocamente in modo sicuro. Si basa su un modello a "Terza Parte Fidata" (**Trusted Third Party**), il Key Distribution Center (KDC).

Invece di inviare password, Kerberos utilizza **Tickets**. L'utente si autentica una volta col KDC e riceve un _Ticket Granting Ticket_ (TGT). Quando l'utente vuole accedere a un servizio specifico, presenta il TGT al KDC per ottenere un _Service Ticket_, che poi invia al server di destinazione. Questo sistema garantisce il Single Sign-On (SSO), previene attacchi di Replay (grazie all'uso di timestamp) e supporta la mutua autenticazione.
### 8.2.3 Zero Trust

Il modello **Zero Trust** rappresenta un cambio di paradigma rispetto alla sicurezza perimetrale tradizionale. Invece di fidarsi implicitamente di chiunque si trovi all'interno della rete aziendale ("Trust but Verify"), il principio è "Never Trust, Always Verify". In un'architettura Zero Trust, ogni richiesta di accesso è trattata come se provenisse da una rete aperta e ostile. L'accesso viene concesso in base a una verifica continua e dinamica dell'identità, del dispositivo, del contesto e della policy, indipendentemente dalla posizione fisica o di rete del richiedente.

![Immagine di Zero Trust Architecture diagram](https://encrypted-tbn1.gstatic.com/licensed-image?q=tbn:ANd9GcR--8lc6G4UVFmMxgk7VFTLrCa0hV05RIErmcezEgyzHhgrR4CtDkolKdGKYQYlQTcHsWjKT0m7UNP4PSHF_TvWdpt9pZay-1spckyomG10D9_kyAY)

## 8.3 Control and Management of Access Rights

Una volta autenticato il soggetto, il sistema deve gestire l'**Authorization**, ovvero determinare quali azioni il soggetto può compiere sugli oggetti.
### 8.3.1 Rows - Capabilities

Nel modello astratto della Matrice di Controllo degli Accessi, le righe rappresentano i soggetti. Implementare la sicurezza per righe significa utilizzare le **Capabilities**. Una Capability è un token infalsificabile posseduto dal soggetto che conferisce il diritto di accedere a un oggetto specifico (simile a un biglietto del cinema). Questo approccio è efficiente per delegare diritti (basta passare il token), ma rende difficile la revoca (bisogna trovare e invalidare il token) e l'auditing di chi ha accesso a una risorsa (bisogna scansionare tutti i soggetti).
### 8.3.2 Cols - Access Control List

Le colonne della matrice rappresentano gli oggetti. Implementare la sicurezza per colonne significa utilizzare le **Access Control Lists** (ACL). Ogni oggetto mantiene una lista di soggetti autorizzati e i relativi permessi. Questo è il modello più diffuso nei sistemi operativi commerciali. Vantaggi: è facile revocare l'accesso (basta rimuovere il soggetto dalla lista) e facile sapere chi può accedere a un file. Svantaggio: è computazionalmente oneroso determinare tutti i permessi posseduti da un singolo utente su tutto il sistema.
### 8.3.3 Role Based Access Control

Il **Role Based Access Control** (RBAC) semplifica la gestione dei permessi nelle grandi organizzazioni. Invece di assegnare permessi direttamente agli utenti, i permessi sono assegnati a dei **Ruoli** (che rappresentano funzioni lavorative, es. "Amministratore", "Revisore", "Impiegato"). Gli utenti vengono poi associati ai ruoli.

Questo riduce drasticamente la complessità amministrativa: quando un dipendente cambia mansione, basta cambiare il suo ruolo, senza dover riconfigurare le ACL di migliaia di file.
### 8.3.4 Attribute Based Access Control

L'**Attribute Based Access Control** (ABAC) è un'evoluzione più flessibile dell'RBAC. Le decisioni di accesso non si basano solo su ruoli statici, ma su policy che valutano attributi del soggetto (es. qualifica, età), dell'oggetto (es. classificazione, tipo), dell'azione e dell'ambiente (es. ora del giorno, geolocalizzazione). Le regole sono espressioni booleane logiche (es. "Permetti accesso SE ruolo=Manager E ora=Lavoro E ip=Interno"). Questo permette una granularità molto fine e dinamica.
# 9 Windows authentication

Windows implementa un sistema di controllo degli accessi complesso che combina autenticazione e autorizzazione attraverso l'uso di token di sicurezza e liste di controllo.
## 9.1 Access token

Quando un utente effettua il logon con successo, il sistema crea un **Access Token**. Questo oggetto del kernel identifica l'utente e viene allegato a ogni processo lanciato dall'utente. Il token contiene il **Security Identifier** (SID) dell'utente, i SID dei gruppi di appartenenza e l'elenco dei **Privileges** (diritti di sistema, come cambiare l'ora o fare shutdown). Quando un processo tenta di accedere a un oggetto, il Security Reference Monitor (SRM) confronta i SID nel token con la ACL dell'oggetto.
### 9.1.1 Mandatory Integrity Control

Per mitigare il rischio che processi a bassa affidabilità (come un browser web) compromettano componenti critici del sistema, Windows implementa il **Mandatory Integrity Control** (MIC). Ogni token e ogni oggetto ha un _Integrity Level_ (Low, Medium, High, System). Il MIC applica la policy "No-Write-Up": un processo con un livello di integrità basso non può scrivere su (o inviare messaggi a) un oggetto con un livello di integrità superiore, anche se l'ACL lo permetterebbe. Questo previene attacchi di _Shatter_ o injection da parte di malware eseguiti nel contesto utente.
### 9.1.2 (D/S) Access Control List(s)

In Windows, il Descrittore di Sicurezza di un oggetto contiene due tipi di liste.

La **Discretionary Access Control List** (DACL) controlla l'accesso effettivo: contiene una lista di _Access Control Entries_ (ACE) che permettono o negano diritti specifici a utenti o gruppi. Se non c'è una DACL, l'accesso è totale; se c'è ma è vuota, l'accesso è negato a tutti.

La **System Access Control List** (SACL) è usata per l'auditing: definisce quali tentativi di accesso (successi o fallimenti) devono generare un evento nel log di sicurezza.
### 9.1.3 Sandboxing Tokens

Il meccanismo dei token permette la creazione di **Restricted Tokens** per il sandboxing. Un processo padre può creare un token figlio "filtrato", rimuovendo specifici SID (es. rimuovendo l'appartenenza al gruppo Administrators) o disabilitando privilegi sensibili. Il processo lanciato con questo token ristretto avrà capacità limitate, riducendo il danno potenziale se venisse compromesso. Questo è il meccanismo alla base della modalità protetta di browser come Chrome o Edge.
## 9.2 Remote Hosts AC

In un ambiente di dominio, l'autenticazione verso host remoti non avviene ritrasmettendo la password (che sarebbe insicuro). Windows ha storicamente supportato NTLM (challenge-response proprietario), ma preferisce Kerberos per la maggiore sicurezza.
### 9.2.1 MS Kerberos and MIT Kerberos

Microsoft ha implementato il protocollo Kerberos standard (MIT) con alcune estensioni specifiche. La differenza principale risiede nel campo dei dati di autorizzazione del ticket. MS Kerberos include il **Privilege Attribute Certificate** (PAC) all'interno del ticket. Il PAC trasporta le informazioni sui gruppi di appartenenza dell'utente e i suoi privilegi. Questo permette al server di destinazione di costruire l'Access Token dell'utente senza dover interrogare nuovamente il Domain Controller, ottimizzando le prestazioni ma aumentando la complessità del ticket.
## 9.3 Impersonation/Delegation

L'**Impersonation** è la capacità di un thread server di eseguire azioni nel contesto di sicurezza del client connesso. Questo è fondamentale per i file server o database server che devono accedere alle risorse "come" l'utente.

La **Delegation** è un'estensione dell'impersonation che permette al server non solo di agire come l'utente localmente, ma di autenticarsi presso _altri_ server di rete per conto dell'utente (il problema del "Double Hop"). Poiché la delega è rischiosa (il server delegato ha pieni poteri), si utilizza la **Constrained Delegation**, che limita i servizi specifici verso cui il server può impersonare l'utente.