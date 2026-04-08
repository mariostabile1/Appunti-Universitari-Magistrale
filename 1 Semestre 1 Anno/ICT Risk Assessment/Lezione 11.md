## Classificazione e Strategie delle Contromisure

Una **contromisura** è definita come un'azione o un dispositivo atto a minimizzare la probabilità di successo di un attacco, ostacolando il raggiungimento dell'obiettivo da parte dell'avversario. La classificazione delle contromisure può avvenire secondo diversi criteri, il primo dei quali è temporale rispetto all'intrusione.

Le contromisure **Proactive** vengono applicate prima che l'intrusione avvenga, con lo scopo di difendere il sistema e ridurre la probabilità di successo degli attacchi (es. patching di una vulnerabilità). Le contromisure **Dynamic** intervengono durante l'intrusione per impedire all'attaccante di raggiungere il suo scopo (es. spegnere una macchina o interrompere una connessione). Infine, le contromisure **Reactive** sono applicate dopo che un'intrusione è avvenuta per prevenire il successo di tentativi futuri; sebbene possano diventare proattive per il futuro, la loro natura immediata è di risposta all'incidente.

Una classificazione più granulare, derivata dal mondo della difesa, distingue tra:

- **Prevent:** impedire l'attacco (es. patching, firewalling).
    
- **Resist:** rendere il sistema più robusto agli attacchi.
    
- **Discover:** rilevare intrusioni passate o in corso tramite strumenti di monitoraggio.
    
- **Recovery (Restore):** ripristinare le funzionalità dopo un incidente.
    
- **React:** risposte attive all'intrusione.
    
- **Attack the Attacker:** tecniche di _Forward Deception_ come gli **Honeypot**, ovvero trappole (server, dati o reti fittizie) progettate per attirare l'attaccante, studiarne il comportamento e distoglierlo dalle risorse reali.

Il framework del **NIST (National Institute of Standards and Technology)** organizza queste attività in cinque funzioni fondamentali: **Identify** (comprendere il rischio), **Protect** (implementare salvaguardie), **Detect** (identificare eventi cyber), **Respond** (agire in risposta a un evento) e **Recover** (ripristinare i servizi).

## Effetti sulle Kill Chain e Principi di Design

Le contromisure possono essere mappate sulle fasi della _Cyber Kill Chain_ per produrre effetti specifici sull'azione dell'avversario. Questi effetti includono **Detect** (rilevare la presenza), **Deny** (impedire l'attacco), **Disrupt** (interrompere il flusso dell'attacco), **Degrade** (ridurre l'efficacia dell'attacco), **Deceive** (ingannare l'attaccante con informazioni false) e **Destroy** (contrattaccare, sebbene questo sia spesso limitato a contesti militari o di law enforcement).

Per costruire un sistema robusto, si adottano principi architetturali consolidati. Il principio di **Defense in Depth** (difesa in profondità) suggerisce l'uso di molteplici linee di difesa concentriche. Poiché nessuna contromisura è perfetta, la ridondanza assicura che il fallimento di un controllo sia mitigato dagli altri, aumentando esponenzialmente il costo e lo sforzo per l'attaccante.

Il principio del **Least Privilege** impone che ogni utente o modulo debba possedere solo i diritti strettamente necessari per svolgere le proprie mansioni e solo per il tempo necessario. Questo limita drasticamente i danni in caso di compromissione delle credenziali o di _privilege escalation_. Strettamente legato è il concetto di **Segmentation**: una rete piatta ("flat network") permette a un attaccante di muoversi lateralmente senza ostacoli una volta superato il perimetro. La segmentazione divide la rete in sottoreti isolate da firewall interni, impedendo la propagazione libera delle intrusioni.

## Controllo degli Accessi: Identificazione e Autenticazione

Il controllo degli accessi si articola in tre fasi distinte. L'**Identificazione** è la dichiarazione dell'identità da parte di un soggetto (es. username). L'**Autenticazione** è la verifica della veridicità di tale dichiarazione. L'**Autorizzazione** è la decisione finale se garantire l'accesso a una risorsa basandosi sull'identità verificata.

L'autenticazione si basa su uno o più fattori:

1. **Something you know:** informazioni segrete note solo all'utente (password, PIN).
    
2. **Something you have:** possesso fisico di un oggetto (token hardware, smart card, smartphone).
    
3. **Something you are:** caratteristiche fisiche intrinseche (biometria: impronta, iride, volto).
    
4. **Something you do:** caratteristiche comportamentali (dinamica della firma, ritmo di battitura).
    
5. **Where you are:** geolocalizzazione (GPS, indirizzo IP).

### Password e Vulnerabilità

Le password rappresentano il metodo più diffuso ma anche il più vulnerabile. Gli attacchi comuni includono il **Dictionary Attack** (tentativo con parole comuni), il **Brute Force** (tentativo esaustivo di tutte le combinazioni), lo **Sniffing** (intercettazione su canali non cifrati) e il **Social Engineering/Phishing** (inganno dell'utente). Lato server, le password non devono mai essere salvate in chiaro, ma sotto forma di hash. Per contrastare gli attacchi basati su _Rainbow Tables_ (tabelle di hash precalcolate), si utilizza il **Salting**, ovvero l'aggiunta di una stringa casuale alla password prima dell'hashing, rendendo unico l'hash anche per password identiche.

### Autenticazione Biometrica

L'autenticazione biometrica digitalizza una caratteristica fisica per confrontarla con un template registrato. A differenza delle password (match esatto), la biometria è probabilistica. L'accuratezza si misura tramite due tassi di errore opposti:

- **False Acceptance Rate (FAR):** probabilità di accettare un impostore.
    
- **False Rejection Rate (FRR):** probabilità di rifiutare un utente legittimo.
    
    All'aumentare della sensibilità del sistema, il FAR diminuisce ma il FRR aumenta. Il punto di equilibrio è il **Crossover Error Rate (CER)**; un CER più basso indica un sistema biometrico più accurato.

### Multi-Factor Authentication (MFA)

La **Multi-Factor Authentication (MFA)** combina due o più fattori diversi (es. password + token OTP). Sebbene aumenti significativamente la sicurezza, non è infallibile. Tecniche come l'**MFA Fatigue** (bombardare l'utente di richieste push finché non accetta per sfinimento) o gli attacchi **Adversary-in-the-Middle (AiTM)** (dove un proxy intercetta sia la password che il token di sessione) dimostrano che anche l'MFA può essere aggirata, specialmente se basata su SMS o app push semplici. Il phishing resistente all'MFA richiede solitamente token hardware FIDO2.

## Autenticazione Continua e Zero Trust

Il modello di sicurezza sta evolvendo verso il **Zero Trust**, abbandonando l'idea della "sicurezza perimetrale" dove un'autenticazione iniziale garantisce fiducia illimitata. La **Continuous Authentication** verifica l'identità dell'utente costantemente durante l'intera sessione, non solo all'accesso. Questo approccio utilizza la biometria comportamentale, analizzando come l'utente digita (keystroke dynamics), muove il mouse o interagisce con il dispositivo, per rilevare anomalie e bloccare l'accesso se il comportamento devia dal profilo dell'utente legittimo (es. in caso di furto di sessione o dispositivo sbloccato).