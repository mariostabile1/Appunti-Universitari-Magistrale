## Ecosistemi di Sicurezza, Esternalità e Free Riding

Una delle sfide fondamentali nella sicurezza informatica risiede nel fatto che il livello di sicurezza di un'entità non dipende esclusivamente dalle proprie scelte, ma anche da quelle di altre entità. La sicurezza è quindi strettamente correlata al proprio ecosistema. Si può utilizzare l'analogia della salute pubblica: non è possibile essere sani in un mondo privo di igiene, assistenza medica e con un alto numero di persone malate, che nel contesto digitale corrispondono a nodi infetti da malware o privi di strumenti di rilevamento. Un fattore critico diventa la sicurezza dei fornitori, la cui vulnerabilità può compromettere l'intera catena.

Questo fenomeno si descrive attraverso i concetti economici di esternalità e _free riding_. Un'**esternalità** è un costo o un beneficio sostenuto o ricevuto da una terza parte che non ha controllo sulla creazione di tale effetto. Nella cybersecurity, un computer non protetto genera un'esternalità negativa (o positiva per gli attaccanti) poiché può essere utilizzato per attaccare altri sistemi. Poiché il costo di un virus o di un attacco è spesso sostenuto dagli altri e non dal proprietario del sistema infetto, manca l'incentivo individuale ad proteggersi adeguatamente. Se un individuo aumenta la _robustness_ del proprio computer, migliora anche la robustezza degli altri utenti, configurando la sicurezza come un bene pubblico.

Tuttavia, la fornitura volontaria di beni pubblici porta al problema del **Free Riding**, dove gli individui tendono a sottrarsi all'investimento necessario, risultando in un livello di bene pubblico inefficiente. Lo sforzo esercitato dipende dai benefici e costi propri, dagli sforzi altrui e dalla tecnologia che lega lo sforzo al risultato. Si distinguono tre casi prototipici di relazione tra sforzo e sicurezza risultante:

1. **Total Effort**: La sicurezza dipende dalla somma degli sforzi individuali. I sistemi diventano progressivamente più sicuri all'aumentare del numero di individui che non fanno _free riding_.
    
2. **Weakest Link**: La sicurezza dipende dallo sforzo minimo. Il sistema è determinato dall'individuo con l'investimento più basso e diventa sempre più inaffidabile all'aumentare del numero di individui con bassa sicurezza.
    
3. **Best Shot**: La sicurezza dipende dallo sforzo massimo. È determinata dall'individuo con il miglior rapporto costi-benefici, mentre gli altri fanno _free riding_ su di esso.

A complicare il quadro interviene l'asimmetria informativa: un individuo non conosce l'investimento di sicurezza degli altri agenti né possiede informazioni certe sugli attaccanti. Questa incertezza impedisce di valutare il proprio livello di sicurezza reale (ad esempio, capire se si è il _weakest link_) e rappresenta un problema significativo per il settore delle assicurazioni cyber.

## Security Policy e Analisi degli Asset

La **Security Policy** è un insieme di regole che un'organizzazione adotta per minimizzare il rischio informatico e definire gli obiettivi di sicurezza, ovvero quali asset e risorse proteggere. Essa definisce il comportamento corretto degli utenti, vieta comportamenti pericolosi e stabilisce l'architettura di sistema, l'inventario dei componenti, i ruoli di utenti e amministratori, e le conseguenze delle violazioni. Una policy efficace deve sempre definire chi ha la responsabilità di controllare che essa venga applicata.

Il primo passo per definire una policy è l'analisi degli asset, volta a costruire un inventario delle risorse hardware e software la cui compromissione comporterebbe una perdita per l'organizzazione. Questo processo implica la scoperta dei processi di business fondamentali e delle risorse ICT critiche che li supportano. L'impatto viene valutato considerando scenari come l'interruzione di un processo, la necessità di ricostruire una risorsa _ex novo_ o la perdita di confidenzialità.

Le risorse possono essere fisiche e logiche (database, applicazioni, potenza di calcolo, banda) o fisiche nel mondo reale, come nel caso di sistemi IoT e ICS (Industrial Control Systems) dove un attacco cyber può fermare una linea di produzione. L'obiettivo finale dell'analisi è calcolare la perdita potenziale per determinare l'investimento in sicurezza adeguato. La scoperta degli asset avviene solitamente tramite _inventory builders_, applicazioni che scansionano la rete per raccogliere dettagli su configurazioni, log e software installato.

## Soggetti, Oggetti e Diritti di Accesso

In una definizione astratta della policy, si identificano **Soggetti** (o _principals_) e **Oggetti**. Un soggetto è qualsiasi entità (utente, applicazione, processo) che può invocare operazioni su un oggetto. Un oggetto è un'entità (variabile, risorsa fisica, file) su cui vengono invocate operazioni; un oggetto che a sua volta invoca operazioni su altri oggetti agisce anche come soggetto.

Un soggetto possiede un **Access Right** (o diritto) su un'operazione di un oggetto se è titolato a invocarla. Questi diritti possono essere diretti o indiretti. Un diritto è indiretto quando viene dedotto: se un soggetto S può leggere un file F, allora qualsiasi programma P eseguito da S acquisisce il diritto di leggere il segmento di memoria dove F è caricato. La specifica di un oggetto e delle sue operazioni definisce un _Data Type_, controllato a tempo di compilazione, sebbene controlli dinamici (run-time) siano necessari per prevenire comportamenti divergenti dovuti a errori nel compilatore o nel supporto run-time.

## Approccio Modulare alla Policy

Una security policy completa risulta dalla composizione di diverse policy specifiche:

- **Acceptable Use Policy (AUP):** Vincoli e pratiche che i dipendenti devono accettare per accedere alla rete.
    
- **Access Control Policies (ACP):** Standard per l'accesso ai dati, gestione password e monitoraggio.
    
- **Change Management Policy:** Processo formale per gestire modifiche a software e servizi IT/OT per minimizzare impatti avversi.
    
- **Incident Response Policy:** Procedure organizzate per gestire e rimediare agli incidenti, limitando i danni.
    
- **Remote Access Policy:** Regole per connessioni remote e BYOD.
    
- **Email/Communication Policy:** Linee guida per l'uso dei mezzi di comunicazione elettronica.
    
- **Disaster Recovery Policy & Business Continuity Plan (BCP):** Piani per ripristinare l'operatività e coordinare gli sforzi dopo eventi critici.
    
- **Information Security Policy:** Determina quali utenti/applicazioni possono manipolare le informazioni di sistema.

## Paradigmi di Controllo degli Accessi

La definizione della _Information Security Policy_ si basa su due scelte ortogonali. La prima riguarda la modalità di espressione:

- **Default Allow:** La policy definisce solo le operazioni vietate (tutto il resto è permesso). Questo approccio porta all'errore di "Enumerating Badness", ovvero cercare di elencare tutte le cose cattive, creando un "badness gap" crescente poiché il traffico ostile cresce più velocemente della capacità di identificarlo.
    
- **Default Deny:** La policy definisce solo le operazioni legali (tutto il resto è vietato).

La seconda scelta riguarda il grado di libertà del proprietario del sistema:

- **Discretionary Access Control (DAC):** Per ogni oggetto esiste un proprietario (owner) che decide discrezionalmente i diritti degli altri utenti senza vincoli. È tipico del mondo commerciale.
    
- **Mandatory Access Control (MAC):** Esistono vincoli di sistema che il proprietario non può violare. È tipico del mondo della difesa.

### Modelli MAC: Confidenzialità e Integrità

Nel modello MAC, oggetti e soggetti sono partizionati in classi parzialmente ordinate (livelli di sicurezza).

Il modello **Bell-LaPadula** (Multilevel Security) mira a proteggere la **confidenzialità**. Un soggetto in una classe $C$ può:

- Leggere file con classe inferiore o uguale a $C$ (**No Read Up**).
    
- Scrivere file con classe uguale a $C$.
    
- Aggiungere (append) a file con classe superiore a $C$ (**No Write Down**). Questo previene il flusso di informazioni (leakage) da livelli alti a livelli bassi. Tuttavia, crea un accumulo di informazioni ai livelli alti, richiedendo operazioni periodiche di desegretazione.

Il modello **Biba** mira a proteggere l'**integrità**. È l'esatto opposto del Bell-LaPadula. Un soggetto può:

- Scrivere solo su oggetti di livello inferiore o uguale (**No Write Up**).
    
- Leggere solo da oggetti di livello superiore o uguale (**No Read Down**). Questo impedisce che un soggetto a bassa integrità possa corrompere (aggiornare) un oggetto ad alta integrità.

### Altri Modelli di Policy

- **Watermark Policy:** Il livello di sicurezza di un soggetto non è fisso ma aumenta monotonicamente ed è pari al massimo livello degli oggetti su cui ha lavorato. È una policy dipendente dal tempo per minimizzare il flusso di informazioni.
    
- **Chinese Wall:** Mira a prevenire conflitti di interesse. Se un soggetto opera su un oggetto di una classe, perde la capacità di operare su oggetti di classi in conflitto.
    
- **Non-Interference:** Una proprietà rigorosa dove le azioni dei soggetti a un certo livello non devono causare alcun effetto osservabile (né in scrittura né in lettura) su oggetti di livello differente, garantendo isolamento totale.

## Regolamentazioni, Standard e Best Practices

Le regolamentazioni impongono vincoli che superano la discrezionalità del DAC.

- **Privacy/GDPR:** Minimizzazione dei dati e robustezza per prevenire leak di dati personali.
    
- **Infrastrutture Critiche (NIS/NIS2):** Richiedono robustezza e condivisione delle informazioni sulle intrusioni.
    
- **Entità Finanziarie (DORA):** Focus sulla **resilienza**, ovvero la capacità di ripristinare il comportamento normale dopo un'intrusione.


L'adozione di standard come **ISO 27001** o **PCI DSS** (per i dati delle carte di pagamento) serve a certificare tramite terze parti che l'organizzazione sta adottando le misure di sicurezza adeguate. Laddove la certificazione è troppo complessa, si adottano le **Best Practices**, regole pubbliche non certificate ma raccomandate (es. aggiornamento software, uso di antivirus, gestione password).

## Trusted Computing Base (TCB) e Implementazione

La **Trusted Computing Base (TCB)** comprende tutti i moduli coinvolti nell'implementazione della security policy. Poiché ogni bug in un modulo TCB è quasi sempre una vulnerabilità, questi componenti sono critici e richiedono un'alta _assurance_ (fiducia). Minore è la dimensione del TCB, maggiore è il livello di sicurezza e la possibilità di verificarne la correttezza, anche tramite metodi formali.

A livello implementativo, il controllo degli accessi è spesso rappresentato da una **Access Control Matrix (ACM)**, dove le righe sono i soggetti, le colonne gli oggetti e le celle i diritti. Poiché la matrice è sparsa, viene spesso implementata tramite **Access Control Lists (ACL)** associate a ciascun oggetto (come nel file system Linux), che elencano chi può fare cosa (Read, Write, Execute) su quella risorsa specifica.