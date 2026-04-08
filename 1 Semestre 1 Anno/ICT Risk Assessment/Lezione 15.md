## Mandatory Integrity Control e Struttura dei Token

Oltre al controllo discrezionale (DAC), Windows implementa il **Mandatory Integrity Control (MIC)**, un meccanismo che valuta l'accesso basandosi sui livelli di integrità e su policy obbligatorie prima ancora di consultare la DACL dell'oggetto. Il MIC assegna un livello di integrità sia ai _principals_ (utenti/processi) che agli oggetti . Esiste una gerarchia di livelli: _Untrusted_, _Low_, _Medium_, _High_, _System_ e _Installer_. Gli utenti standard operano a livello Medium, mentre quelli elevati a livello High .

La regola fondamentale del MIC è il **No Write Up**: un soggetto con un livello di integrità inferiore non può scrivere su un oggetto con integrità superiore, anche se la DACL lo permetterebbe esplicitamente. Questo protegge, ad esempio, i processi di sistema (System integrity) da codice eseguito dall'utente (Medium) e impedisce a codice non fidato (Low, come le tab di un browser) di modificare oggetti del sistema operativo .

Ogni volta che un soggetto tenta l'accesso a un oggetto, il kernel esegue un controllo incrociando tre informazioni: l'identità del richiedente (tramite il token), le sue intenzioni (tipo di accesso richiesto) e i permessi dell'oggetto (Security Descriptor) . Il **Security Descriptor** contiene due tipi di liste di controllo accessi (ACL):

- **DACL (Discretionary Access Control List):** elenca chi è autorizzato o negato all'accesso.
    
- **SACL (System Access Control List):** definisce quali eventi (successi o fallimenti) devono essere registrati nei log di audit per scopi di monitoraggio.

### Sandboxing tramite Token Ristretti

Applicazioni complesse ed esposte come i browser (es. Chrome) utilizzano il principio del **Least Privilege** tramite il _Sandboxing_. Se un processo del browser viene compromesso, l'attaccante eredita il suo token; pertanto, l'obiettivo è limitare preventivamente i danni spostando l'esecuzione in processi a basso privilegio . Per azioni che richiedono privilegi (come salvare un file), il processo sandboxato deve interrogare un processo "Broker" esterno. A livello implementativo, Windows permette di creare **Restricted Tokens** (o Low Privilege Tokens). Chrome, ad esempio, modifica la copia locale del proprio token o ne crea uno nuovo disabilitando gruppi e privilegi pericolosi, limitando drasticamente ciò che il processo _renderer_ può toccare nel sistema .

## Accesso Remoto, Impersonation e Double Hop

L'autenticazione remota introduce complessità nella gestione dei token. Un token di accesso è legato a una specifica sessione di logon su una macchina e non può essere inviato "così com'è" attraverso la rete. Quando un utente accede a una risorsa remota (es. una share SMB), il server autentica il client (via Kerberos o NTLM) e crea una **Network Logon Session**. A differenza delle sessioni interattive locali, le sessioni di rete _non_ conservano le credenziali in cache (password hash). Questo causa il problema del **Double Hop**: il server remoto non possiede le credenziali necessarie per autenticarsi a sua volta verso una terza macchina per conto dell'utente .

Per gestire le richieste concorrenti, i server utilizzano l'**Impersonation**. Mentre un processo possiede un _Primary Token_, ogni thread può adottare temporaneamente un _Impersonation Token_ diverso, permettendo al server di agire nel contesto di sicurezza di vari client simultaneamente . Tuttavia, l'abuso della delega può portare a gravi vulnerabilità. La **Unconstrained Delegation** è una configurazione (impostata da un amministratore di dominio) che forza il salvataggio del TGT (Ticket Granting Ticket) dell'utente nella memoria del server. Se un Domain Admin si collega a una macchina compromessa con questa configurazione, l'attaccante può estrarre il TGT e impersonare l'amministratore ovunque .

Un'ulteriore protezione interna di Windows sono i **Trust Labels**, una feature non documentata che restringe l'accesso a processi critici (come l'antimalware `MsMpEng.exe`) anche agli amministratori, impedendo che un attaccante riduca i privilegi del token di sistema per disabilitare le difese .

## Deception: Honeypot e Honeynet

La Deception (inganno) è una classe di contromisure preventive e investigative. Gli strumenti principali sono gli **Honeypot** (singoli sistemi) e le **Honeynet** (intere reti), risorse progettate per essere attaccate. Non avendo scopi produttivi, qualsiasi interazione con esse è per definizione un'anomalia o un attacco. I loro obiettivi sono raccogliere _Threat Intelligence_ (scoprire tattiche e vulnerabilità sfruttate) e rallentare gli attaccanti distogliendoli dagli asset reali .

Gli honeypot si classificano in base al livello di interazione:

- **Low Interaction:** Simulano servizi semplici (es. un listener su una porta). Sono sicuri ma raccolgono poche informazioni.
    
- **Medium Interaction:** Emulano il comportamento dei servizi di rete analizzando input e fornendo risposte realistiche senza usare un vero OS (es. **Honeyd**, **Kippo**, **Cowrie** per SSH/Telnet). Honeyd, ad esempio, può simulare intere topologie di rete e monitorare il "dark space" (indirizzi IP non usati) .
    
- **High Interaction:** Utilizzano sistemi operativi e servizi reali. Offrono il massimo realismo e raccolta dati, ma comportano rischi elevati se l'attaccante riesce a usarli come testa di ponte .

Un tipo particolare è il "Tarpit" o "Sticky Honeypot", come **LaBrea**. Questo strumento risponde a richieste di connessione su IP inutilizzati e "blocca" lo scanner dell'attaccante mantenendo la connessione aperta indefinitamente o rallentandola (simulando finestre TCP chiuse), consumando le risorse dell'avversario .

Esistono anche gli **Honeytokens**, che non sono macchine ma dati falsi (file, credenziali, indirizzi email) disseminati nei sistemi reali. L'accesso o l'uso di questi dati funge da allarme immediato di una violazione .

### Statistiche e Trend dagli Honeypot

Le analisi condotte tramite honeypot su cloud (AWS e Azure) rivelano che la stragrande maggioranza degli attacchi (98.5%) è automatizzata. Si notano differenze strutturali: Azure subisce prevalentemente attacchi SSH (spesso da botnet Linux/Mirai), mentre AWS è più soggetto ad attacchi sul protocollo SMB . Gli attacchi coordinati su più provider sono comuni, ma la provenienza da nodi Tor è statisticamente irrilevante.