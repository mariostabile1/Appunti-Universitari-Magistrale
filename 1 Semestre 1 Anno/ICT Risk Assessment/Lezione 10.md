## Vulnerabilità Remediation e Patch Management

La **Remediation** rappresenta il processo di risoluzione delle vulnerabilità di sicurezza. Lo strumento principale di questa fase è il **Patching**, ovvero l'applicazione di aggiornamenti software che correggono i difetti del codice. Sebbene riduca la necessità di modifiche strutturali al sistema, il patching incontra due ostacoli principali: vincoli funzionali, dove una patch potrebbe alterare o interrompere le funzionalità esistenti, e vincoli di risorse, legati al numero limitato di patch che possono essere testate e applicate in un dato intervallo di tempo prima che ne emergano di nuove.

L'Australian Signals Directorate ha formalizzato le strategie di mitigazione più efficaci nel framework "The Essential Eight". Questo modello definisce tre livelli di maturità e identifica otto controlli essenziali, tra cui il patching delle applicazioni e dei sistemi operativi, l'uso della Multi-Factor Authentication (MFA), la restrizione dei privilegi amministrativi e l'hardening delle applicazioni utente.

Il **Patch Management Lifecycle** è il processo ciclico che governa l'applicazione degli aggiornamenti. Inizia con la creazione di un inventario degli asset (hardware e software) e prosegue con il monitoraggio delle fonti per identificare nuove vulnerabilità e patch disponibili. Segue la fase di assessment e prioritizzazione, in cui si decide quali patch applicare per prime. Successivamente, le patch vengono testate e dispiegate (Deploy). Infine, la fase di verifica e reporting conferma il successo dell'operazione e aggiorna lo stato di sicurezza.

## Distinzione tra Remediation e Mitigation

È fondamentale distinguere tra **Remediation** e **Mitigation**. La Remediation rimuove la vulnerabilità alla radice, solitamente tramite l'applicazione di una patch o la modifica del codice sorgente, eliminando la minaccia. La Mitigation, invece, interviene quando la remediation non è immediatamente possibile o fallisce. Essa non rimuove il difetto, ma riduce la probabilità o l'impatto di un attacco implementando controlli compensativi, come regole firewall o modifiche alla configurazione, per ridurre il rischio a un livello accettabile.

Un'ulteriore opzione è l'accettazione del rischio, praticabile solo quando la vulnerabilità è considerata a basso rischio e il costo della correzione supera il potenziale danno.

## Prioritizzazione Avanzata: Oltre il CVSS

Sebbene il CVSS fornisca una misura della severità tecnica, non è sufficiente per determinare il rischio reale, che dipende anche dal contesto di minaccia e dall'importanza dell'asset. La prioritizzazione moderna si avvale di modelli decisionali come lo **SSVC (Stakeholder-Specific Vulnerability Categorization)**.

Lo SSVC utilizza alberi decisionali per guidare le azioni di remediation basandosi su valori qualitativi. Il modello sviluppato dalla CISA (Cybersecurity and Infrastructure Security Agency) valuta le vulnerabilità secondo cinque criteri principali:

- **Exploitation Status:** indica se esiste codice di exploit pubblico o se la vulnerabilità è attivamente sfruttata (Active, PoC, None).
    
- **Technical Impact:** valuta l'effetto tecnico (Total o Partial).
    
- **Automatable:** determina se un attaccante può automatizzare l'attacco (Yes o No).
    
- **Mission Prevalence:** stima quanto il componente vulnerabile sia diffuso nei sistemi critici per la missione (Essential, Support).
    
- **Public Well-Being Impact:** valuta l'impatto fisico o sociale (High, Medium, Low).

L'output dell'albero decisionale SSVC classifica l'azione richiesta in quattro categorie:

1. **Track:** La vulnerabilità non richiede azione immediata; si continua a monitorarla.
    
2. **Monitor:** Si attende passivamente, aggiornando il software secondo le normali tempistiche.
    
3. **Attend:** Richiede attenzione interna e approvazione, spesso sollecitando un aggiornamento più rapido del normale.
    
4. **Act:** La vulnerabilità richiede una risposta immediata, compresa la mobilitazione di risorse straordinarie e la leadership.

## Sistemi Legacy e Virtual Patching

La gestione dei sistemi **Legacy** o End-of-Life (EOL) pone sfide critiche poiché il fornitore non rilascia più aggiornamenti di sicurezza. In questi scenari, dove il patching tradizionale è impossibile, si ricorre al **Virtual Patching**.

Il Virtual Patching consiste nell'implementare un livello di sicurezza a monte dell'applicazione vulnerabile, tipicamente tramite un **Web Application Firewall (WAF)** o un **Intrusion Prevention System (IPS)**. Questo livello analizza il traffico in entrata e blocca le richieste malevole mirate a sfruttare la vulnerabilità nota, impedendo che raggiungano l'applicazione target.

Questa tecnica offre una protezione immediata ("just-in-time") senza richiedere modifiche al codice sorgente o riavvii del sistema, permettendo di mantenere operativi sistemi legacy o di proteggere applicazioni in attesa di una patch ufficiale. Tuttavia, non è una soluzione permanente: non rimuove la vulnerabilità sottostante e può introdurre falsi positivi se le regole di filtro non sono calibrate correttamente.

## Sicurezza della Supply Chain e SBOM

Il software moderno è assemblato utilizzando numerose librerie di terze parti (spesso open source), creando una dipendenza complessa nota come **Software Supply Chain**. Una vulnerabilità in una singola libreria diffusa, come nel caso di Log4j, può impattare milioni di applicazioni a cascata.

Per gestire questo rischio, si utilizza la **Software Composition Analysis (SCA)**, che identifica i componenti open source utilizzati nel codice e le relative vulnerabilità note. Elemento chiave di questa strategia è lo **SBOM (Software Bill of Materials)**, un inventario formale e strutturato (una "distinta base") di tutti i componenti, librerie e moduli inclusi in un software. Lo SBOM permette alle organizzazioni di reagire rapidamente alla scoperta di nuove vulnerabilità, identificando immediatamente quali applicazioni utilizzano il componente compromesso.

## Automazione e Self-Healing Networks

L'evoluzione futura della remediation punta verso l'automazione spinta e l'uso dell'Intelligenza Artificiale. Il concetto di **Self-Healing Network** immagina organizzazioni capaci di scoprire e patchare indipendentemente le vulnerabilità nel software in esecuzione, senza attendere i tempi dei vendor. Agenti AI potrebbero generare e applicare patch per codice di terze parti, aumentando la sicurezza ma sollevando questioni su correttezza, compatibilità e responsabilità legale (right-to-repair).