# Introduction

L'introduzione alla sicurezza ICT richiede la comprensione del contesto in cui operano i sistemi informativi moderni. La gestione del rischio non è un'attività puntuale, ma un processo ciclico che mira a identificare, valutare e mitigare le minacce che potrebbero compromettere le risorse di un'organizzazione. In questo ambito, l'obiettivo non è mai la sicurezza assoluta, concetto irraggiungibile, ma la riduzione del rischio a un livello accettabile, bilanciando costi di protezione e valore degli asset.

## 1.1 ICT Security Fundamentals

I fondamenti della sicurezza informatica si basano su tre pilastri principali, comunemente noti come **CIA Triad** (Confidentiality, Integrity, Availability).

La **Confidentiality** garantisce che le informazioni siano accessibili solo a chi è autorizzato a vederle, prevenendo la divulgazione non autorizzata. L'**Integrity** assicura che le informazioni e i sistemi non vengano modificati in modo non autorizzato o accidentale, preservando l'accuratezza e la completezza dei dati. Infine, l'**Availability** garantisce che le risorse e i servizi siano accessibili agli utenti autorizzati quando necessario. A questi tre elementi fondamentali si aggiungono spesso l'**Authenticity**, che verifica l'identità di un soggetto o l'origine di un dato, e la **Non-repudiation**, che impedisce a un soggetto di negare di aver compiuto una determinata azione.
## 1.2 Security policy - Glossary

In ambito accademico e professionale, è fondamentale distinguere tra politica e meccanismo. Una **Security Policy** è una definizione formale o informale delle regole che governano come un'organizzazione gestisce, protegge e distribuisce le informazioni sensibili. Essa definisce il "cosa" deve essere fatto. Al contrario, un **Security Mechanism** è il metodo, lo strumento o la procedura tecnica utilizzata per implementare la politica, ovvero il "come". Un sistema sicuro è quello che applica correttamente meccanismi adeguati per soddisfare la policy definita.
# 2 Security Policy

La definizione di una politica di sicurezza è il primo passo critico nella costruzione di un sistema sicuro. Senza una policy chiara, non esiste un criterio oggettivo per distinguere tra un accesso autorizzato e una violazione.
## 2.1 Definining Security Policy

Una Security Policy partiziona lo stato del sistema in due insiemi distinti: l'insieme degli stati **autorizzati** (o sicuri) e l'insieme degli stati **non autorizzati** (o insicuri). Un sistema è considerato sicuro se, partendo da uno stato autorizzato, non può entrare in uno stato non autorizzato attraverso l'esecuzione di operazioni legittime. La policy deve essere precisa, consistente e completa, coprendo tutti i possibili scenari di accesso e manipolazione dei dati. Spesso, queste policy vengono derivate da requisiti legali, normativi o aziendali.
## 2.2 Terminology

Per formalizzare le politiche di sicurezza, si utilizza una terminologia standard che astrae le entità del sistema.
### 2.2.1 Subject and Object

Il sistema viene modellato come un insieme di entità attive e passive.

Un **Subject** è un'entità attiva che richiede l'accesso a una risorsa o l'esecuzione di un'operazione. Esempi tipici sono gli utenti umani, i processi in esecuzione o i thread.

Un **Object** è un'entità passiva che contiene o riceve informazioni. Esempi includono file, directory, record di database, o dispositivi hardware come stampanti.

Va notato che la distinzione non è sempre rigida: un processo (Subject) può diventare un Object se un altro processo tenta di modificarlo o terminarlo.
### 2.2.2 Access Rights

Gli **Access Rights** (o permessi) definiscono il tipo di accesso che un Subject può esercitare su un Object. I diritti più comuni includono _Read_ (leggere il contenuto), _Write_ (modificare o cancellare il contenuto), _Execute_ (eseguire il programma) e _Append_ (aggiungere dati senza modificare quelli esistenti). In modelli più complessi, i diritti possono includere la capacità di trasferire permessi ad altri soggetti (_Grant_) o di revocarli (_Revoke_).
## 2.3 Composite policy

Raramente un sistema è governato da un'unica regola monolitica. Si parla di **Composite Policy** quando diverse politiche devono coesistere sullo stesso sistema. Ad esempio, una politica di confidenzialità potrebbe impedire la lettura di un file, mentre una politica di disponibilità potrebbe richiederne l'accesso per il backup. La sfida principale nella definizione di policy composite è la gestione dei conflitti: è necessario stabilire regole di precedenza o meccanismi di risoluzione per determinare quale policy prevale quando due regole sono in contraddizione.
## 2.4 ISP - Information Security Policy

La **Information Security Policy** (ISP) è il documento di alto livello che riflette la strategia di sicurezza dell'organizzazione. Essa stabilisce le linee guida, le procedure e gli standard che devono essere seguiti da tutti i dipendenti e gli stakeholder.
### 2.4.1 Six Dumbest Ideas in Computer Security

Nell'analisi delle policy fallimentari, si fa spesso riferimento alle critiche mosse da esperti come Marcus Ranum, che identificano approcci concettualmente errati alla sicurezza. Tra questi errori sistemici figurano il **Default Allow**, ovvero permettere tutto ciò che non è esplicitamente vietato, invece di negare tutto per default; l'idea che il **Penetration Testing** da solo costituisca una prova di sicurezza, mentre in realtà trova solo bug noti; e l'affidarsi esclusivamente agli utenti per prendere decisioni di sicurezza (**Educating Users**), ignorando che l'errore umano è inevitabile. Altri errori includono il ritenere che l'azione reattiva sia efficace quanto la prevenzione (**Action is better than inaction**) e il confondere l'autenticazione con l'autorizzazione.
### 2.4.2 Discretionary Access Control

Il **Discretionary Access Control** (DAC) è un modello di controllo degli accessi basato sull'identità del soggetto e su regole di accesso che il proprietario della risorsa può stabilire a sua discrezione. In un sistema DAC, se un utente crea un file, ne diventa il proprietario e può decidere chi altro può leggerlo o scriverlo. Questo modello è flessibile e comune nei sistemi operativi commerciali (come Windows o UNIX standard), ma soffre di una vulnerabilità intrinseca: non protegge contro il flusso di informazioni. Un soggetto autorizzato a leggere un dato può copiarlo e renderlo accessibile a soggetti non autorizzati, aggirando l'intento originale del proprietario.
### 2.4.3 Mandatory Access Control

Il **Mandatory Access Control** (MAC) è un modello in cui le decisioni di accesso sono prese dal sistema basandosi su etichette di sicurezza (security labels) associate sia ai soggetti che agli oggetti. L'utente non può alterare queste etichette né concedere l'accesso a terzi a sua discrezione. Tipicamente utilizzato in ambiti militari o ad alta sicurezza, il MAC confronta la classificazione dell'oggetto (es. "Top Secret") con il livello di autorizzazione del soggetto (es. "Secret"). L'accesso è negato se le etichette non soddisfano le regole stringenti della policy, impedendo efficacemente il flusso di informazioni non autorizzato.
### 2.4.4 Overall Policy

L'**Overall Policy** rappresenta l'aggregazione di tutte le politiche specifiche (DAC, MAC, policy applicative) in un quadro coerente. Questa deve garantire che l'interazione tra i vari modelli non apra falle di sicurezza. Ad esempio, un sistema potrebbe utilizzare il MAC per proteggere i dati sensibili a livello di kernel, ma utilizzare il DAC per la gestione dei file utente meno critici, a patto che il DAC non possa mai violare i vincoli imposti dal MAC.
## 2.5 Trusted Computing Base

Il **Trusted Computing Base** (TCB) è l'insieme di tutti i componenti hardware, firmware e software di un sistema che sono critici per la sua sicurezza. Se una qualsiasi parte del TCB ha un difetto o viene compromessa, l'intera sicurezza del sistema è a rischio. Il TCB è responsabile dell'applicazione della Security Policy. Per essere efficace, il TCB deve soddisfare tre requisiti fondamentali: deve essere a prova di manomissione (**Tamper-proof**), deve essere sempre invocato a ogni accesso (**Always invoked**), e deve essere abbastanza piccolo da poter essere verificato e analizzato in modo esaustivo (**Small enough to be checked**).
## 2.6 Representing Security Policy

Per analizzare e implementare le policy, è necessario un modello formale di rappresentazione. Lo strumento concettuale più utilizzato è l'**Access Control Matrix** (ACM).

L'ACM è una matrice in cui le righe corrispondono ai soggetti ($S$) e le colonne agli oggetti ($O$). Ogni cella $M[s, o]$ contiene l'insieme dei diritti di accesso ($R$) che il soggetto $s$ possiede sull'oggetto $o$.

Poiché una matrice completa sarebbe estremamente grande e sparsa (la maggior parte delle celle sarebbe vuota), nella pratica si utilizzano due implementazioni principali derivate dall'ACM:

Le **Access Control Lists** (ACL), che corrispondono alle colonne della matrice: ogni oggetto possiede una lista dei soggetti autorizzati e dei relativi permessi.

Le **Capabilities**, che corrispondono alle righe della matrice: ogni soggetto possiede un "ticket" o un token infalsificabile che elenca tutti gli oggetti a cui può accedere e con quali permessi.

Ecco gli appunti sui modelli formali di controllo degli accessi Bell-LaPadula e Biba.

## 2.7 Bell-LaPadula Model

Il modello **Bell-LaPadula (BLP)** è un modello formale a stati finiti focalizzato esclusivamente sulla **Confidentiality**. Sviluppato negli anni '70 per il Dipartimento della Difesa degli Stati Uniti, è lo standard di riferimento per i sistemi che implementano il **Mandatory Access Control (MAC)** in ambienti militari e governativi. L'obiettivo primario è prevenire la divulgazione non autorizzata di informazioni, garantendo che i segreti non vengano trasferiti a livelli di sicurezza inferiori.

Il modello organizza le entità in un reticolo di livelli di sicurezza gerarchici (es. Top Secret > Secret > Confidential > Unclassified) e categorie (Need-to-know). A ogni **Soggetto** ($S$) è assegnata una _Clearance_ (nulla osta di sicurezza), mentre a ogni **Oggetto** ($O$) è assegnata una _Classification_. L'accesso è determinato dal confronto tra la clearance del soggetto e la classificazione dell'oggetto.

Il funzionamento del modello si basa su due regole fondamentali che governano il flusso delle informazioni:

#### 2.7.1 Simple Security Property (No Read Up)

Questa proprietà regola l'accesso in lettura. Un soggetto a un determinato livello di sicurezza non può leggere dati a un livello di sicurezza superiore. Formalmente, un soggetto $S$ può leggere un oggetto $O$ solo se il livello di sicurezza di $S$ domina o è uguale al livello di sicurezza di $O$:

$$L(S) \ge L(O)$$

Questo impedisce l'accesso diretto ai segreti da parte di personale non autorizzato.

#### 2.7.2 The *-Property (Star Property) (No Write Down)

Questa proprietà, specifica del modello BLP, regola l'accesso in scrittura ed è essenziale per prevenire la fuga di informazioni (data leakage). Un soggetto a un determinato livello di sicurezza non può scrivere informazioni a un livello di sicurezza inferiore. Formalmente, un soggetto $S$ può scrivere su un oggetto $O$ solo se il livello di sicurezza di $O$ domina o è uguale al livello di sicurezza di $S$:

$$L(O) \ge L(S)$$

Senza questa regola, un utente (o un malware/Trojan horse che agisce per conto dell'utente) con accesso "Top Secret" potrebbe copiare deliberatamente o accidentalmente informazioni sensibili in un file "Unclassified", rendendole accessibili a chiunque. La Star Property confina l'informazione al suo livello o a livelli superiori.

## 2.8 Biba Integrity Model

Il modello **Biba** è il duale del modello Bell-LaPadula. Mentre BLP si concentra sulla segretezza, Biba si concentra esclusivamente sull'**Integrity**. Sviluppato successivamente al BLP, questo modello è progettato per ambienti commerciali dove la prevenzione della modifica non autorizzata dei dati è più critica della loro segretezza (es. sistemi bancari, record medici).

In Biba, ai soggetti e agli oggetti vengono assegnati livelli di integrità. Un livello di integrità alto indica affidabilità e accuratezza (trustworthiness), mentre un livello basso indica dati potenzialmente corrotti o sconosciuti. L'obiettivo è prevenire che dati di bassa integrità contaminino dati o processi di alta integrità.

Anche il modello Biba si basa su due assiomi principali, che sono l'inverso delle regole di Bell-LaPadula:

#### 2.8.1 Simple Integrity Property (No Read Down)

Questa regola protegge i soggetti dall'essere contaminati da informazioni inaffidabili. Un soggetto a un alto livello di integrità non può leggere dati da un livello di integrità inferiore. Formalmente, un soggetto $S$ può leggere un oggetto $O$ solo se il livello di integrità di $S$ è minore o uguale a quello di $O$:

$$I(S) \le I(O)$$

Leggere dati da una fonte meno affidabile comprometterebbe l'integrità del soggetto stesso.
#### 2.8.2 The *-Integrity Property (No Write Up)

Questa regola protegge gli oggetti di alto valore dalla modifica da parte di soggetti non fidati. Un soggetto a un livello di integrità basso non può scrivere o modificare dati a un livello di integrità superiore. Formalmente, un soggetto $S$ può scrivere su un oggetto $O$ solo se il livello di integrità di $S$ è maggiore o uguale a quello di $O$:

$$I(S) \ge I(O)$$

Questo impedisce, ad esempio, che un utente standard possa sovrascrivere file di sistema critici o che dati non verificati vengano inseriti in un database master di riferimento.
## 2.9 Clark-Wilson

Il modello **Clark-Wilson** è un modello di integrità progettato specificamente per l'ambiente commerciale, differenziandosi dai modelli militari come Biba. Mentre Biba si basa su livelli gerarchici di integrità (lattice-based), Clark-Wilson si concentra sulla validità delle transazioni e sulla separazione dei compiti (**Separation of Duty**). Il modello riconosce che nei sistemi reali l'integrità dei dati non dipende solo da chi scrive, ma da _come_ i dati vengono scritti.

Il modello definisce rigorosamente i dati e le procedure ammesse:

- **Constrained Data Items (CDI)**: Dati critici all'interno del sistema la cui integrità deve essere protetta (es. saldi bancari).
    
- **Unconstrained Data Items (UDI)**: Dati esterni al controllo del sistema di sicurezza (es. input utente non ancora validato).
    
- **Transformation Procedures (TP)**: Procedure programmate (transazioni) che modificano i CDI. Una TP deve garantire che, partendo da uno stato valido dei CDI, il sistema transisca a un altro stato valido (**Well-formed transaction**).
    
- **Integrity Verification Procedures (IVP)**: Procedure che verificano che i CDI siano in uno stato valido in un dato momento.
    

Il meccanismo di controllo degli accessi in Clark-Wilson non collega direttamente l'utente al dato, ma impone l'uso di una tripla $(User, TP, \{CDI\})$. Un utente può manipolare un CDI solo attraverso una specifica TP per la quale è autorizzato. Questo impedisce modifiche arbitrarie ai dati sensibili e garantisce che tutte le operazioni rispettino la logica di business definita nelle procedure di trasformazione.

## 2.10 Muraglia cinese

Il modello **Brewer and Nash**, meglio noto come modello **Chinese Wall** (Muraglia Cinese), è una politica di sicurezza progettata per prevenire i conflitti di interesse (**Conflict of Interest - COI**) nel settore commerciale e della consulenza. A differenza dei modelli statici come Bell-LaPadula, il Chinese Wall è un modello ibrido con controlli di accesso che cambiano dinamicamente in base alla storia delle azioni dell'utente.

Il modello organizza i dati in tre livelli gerarchici:

1. **Objects**: Le singole unità di informazione (file).
    
2. **Company Datasets (CD)**: L'insieme di tutti gli oggetti relativi a una singola azienda.
    
3. **Conflict of Interest Classes (COI Class)**: Gruppi di Company Datasets appartenenti ad aziende in competizione tra loro (es. una classe "Banche" che contiene i dataset di Banca A e Banca B).
    

La regola di accesso fondamentale afferma che un soggetto può accedere a un oggetto solo se l'oggetto appartiene allo stesso Company Dataset di un oggetto già acceduto dal soggetto, oppure se appartiene a un Company Dataset completamente diverso che non è in conflitto con quelli già acceduti.

In pratica, inizialmente l'utente ha accesso a tutti i dataset (il muro è basso). Nel momento in cui l'utente legge un file della "Azienda A", il sistema erige istantaneamente una barriera invalicabile (il muro si alza) attorno a tutti i dataset delle aziende concorrenti nella stessa COI Class (es. "Azienda B"). L'utente mantiene l'accesso all'Azienda A e a tutte le altre aziende non in conflitto, ma perde irrevocabilmente l'accesso ai concorrenti diretti dell'Azienda A per tutta la durata della sessione o dell'incarico.