## Approccio Modulare alla Security Policy

La definizione di una **Security Policy** completa non è monolitica, ma risulta dall'aggregazione di diversi moduli specifici che coprono vari aspetti dell'organizzazione. Questo approccio presuppone che sia disponibile un inventario aggiornato del sistema e che gli asset siano stati selezionati. Tra le componenti fondamentali troviamo:

- **Acceptable Use Policy (AUP):** definisce le norme di utilizzo accettabile per gli utenti.
    
- **Access Control Policies:** regolano l'accesso alle risorse e agli asset, implementando controlli di _robustness_.
    
- **Incident Response Policy:** stabilisce cosa accade dopo un'intrusione riuscita.
    
- **Change Management Policy:** detta le regole sull'evoluzione e modifica del sistema.
    
- **Remote Access Policy:** gestisce il lavoro remoto e le politiche BYOD (Bring Your Own Device).
    
- **Business Continuity Plan (BCP):** definisce i servizi minimi da mantenere in caso di crisi.
    
- **Disaster Recovery Policy:** specifica come garantire il BCP e la resilienza del sistema.

## Information Security Policy: Paradigmi e Scelte Fondamentali

La **Information Security Policy** determina quali utenti o applicazioni possono invocare operazioni sugli oggetti per leggere e manipolare le informazioni di sistema. La definizione di questa politica deriva dalla soluzione di due scelte ortogonali.

La prima scelta riguarda la modalità di espressione della policy. Si può optare per il **Default Allow**, dove la policy definisce solo le operazioni vietate, oppure per il **Default Deny**, dove vengono definite solo le operazioni legali . Secondo Marcus Ranum, il _Default Allow_ è una delle "Six Dumbest Ideas in Computer Security" perché costringe a "Enumerating Badness" (elencare il male) . Questo approccio crea il cosiddetto "Badness Gap": mentre le applicazioni legittime crescono linearmente, le applicazioni ostili crescono esponenzialmente, rendendo impossibile elencarle tutte.

La seconda scelta riguarda il grado di libertà del proprietario del sistema. Nel **Discretionary Access Control (DAC)**, ogni oggetto ha un proprietario (owner) che decide i diritti degli altri utenti senza vincoli, tipico del mondo commerciale . Nel **Mandatory Access Control (MAC)**, esistono vincoli di sistema che il proprietario non può violare; gli oggetti e i soggetti sono partizionati in classi parzialmente ordinate e le operazioni sono permesse solo se soddisfano condizioni predefinite indipendenti dalla volontà dei soggetti .

### Modelli MAC: Confidenzialità e Integrità

I modelli MAC si basano su regole fisse per controllare il flusso di informazioni.

Il modello **Bell-LaPadula** (Multilevel Security) è progettato per proteggere la **confidenzialità**. Le regole fisse sono "no read up, no write down". Un soggetto in una classe $C$ può leggere file di classe inferiore o uguale (per non acquisire informazioni segrete) e scrivere su file di classe uguale . È permesso l'_append_ (aggiunta) a file di classe superiore, in quanto non implica la lettura del contenuto. Questo previene il _leakage_ di informazioni dai livelli alti ai bassi. Tuttavia, ciò comporta che il livello di classificazione delle informazioni possa solo crescere, richiedendo operazioni periodiche di desegretazione che diventano bersagli per gli attaccanti .

Il modello **Biba Integrity Model** è speculare e mira a proteggere l'**integrità**. Le regole sono "no write up, no read down". Un soggetto può scrivere solo su oggetti di livello inferiore o uguale (_no write up_) per evitare che un soggetto a bassa integrità corrompa dati ad alta integrità. Può leggere oggetti di livello superiore (_no read down_) per acquisire informazioni corrette, ma non può modificarle. In questo contesto, l'integrità è privilegiata a spese della confidenzialità.

Esistono varianti dinamiche di questi modelli. Il **Watermark** prevede che il livello di un soggetto non sia fisso ma aumenti monotonicamente in base al livello massimo degli oggetti su cui ha lavorato, minimizzando il flusso verso l'alto . Il **Chinese Wall** è utilizzato per evitare conflitti di interesse: l'accesso a un oggetto di una classe inibisce l'accesso a oggetti di classi in conflitto . Infine, la proprietà di **No interference** stabilisce che le azioni di soggetti a un livello (alto o basso) non devono causare alcun effetto osservabile su oggetti di un altro livello, garantendo un isolamento totale .

## Regolamentazioni, Standard e Best Practices

Mentre la distinzione MAC/DAC è teorica, nella pratica le regolamentazioni impongono vincoli simili al MAC sopra sistemi DAC.

- **Privacy Regulation (GDPR):** Richiede la minimizzazione dei dati e robustezza per prevenire data leaks .
    
- **Critical Infrastructure (NIS/NIS2):** Impone la robustezza delle infrastrutture e la condivisione delle informazioni sulle intrusioni .
    
- **Entità Finanziarie (DORA):** Si concentra sulla **resilienza**, ovvero la capacità di ripristinare il comportamento normale dopo un incidente.

Per dimostrare la conformità, le organizzazioni adottano **Standard** (come ISO 27001 o PCI DSS per le carte di pagamento) che prevedono una certificazione di terza parte . Quando la certificazione è troppo complessa, si seguono le **Best Practices**, regole pubbliche non certificate come l'aggiornamento software, l'uso di antivirus e password sicure .

## Trusted Computing Base (TCB) e Access Control

La **Trusted Computing Base (TCB)** comprende tutti i moduli coinvolti nell'implementazione della security policy. Questi moduli sono critici perché ogni bug al loro interno costituisce quasi sempre una vulnerabilità. L'_assurance_ (fiducia) nel sistema aumenta al diminuire della dimensione del TCB, poiché un TCB piccolo è più facile da verificare formalmente .

A livello implementativo, il controllo degli accessi è descritto da una **Access Control Matrix (ACM)**, dove le righe sono i soggetti e le colonne gli oggetti . Poiché tale matrice è spesso sparsa, la sua rappresentazione interna avviene solitamente tramite **Access Control List (ACL)**, ovvero una lista associata a ciascun oggetto che definisce i diritti per ogni soggetto (come i permessi rwx nel file system Linux) .

## Vulnerabilità, Attacchi e Intrusioni

Una **Vulnerability** è un difetto (errore o bug) in una persona, componente o set di regole che permette a un _threat agent_ di eseguire un attacco. Sebbene tutte le vulnerabilità siano bug, non tutti i bug sono vulnerabilità. Il successo di un attacco basato su una vulnerabilità è spesso un evento stocastico.

Un **Attack** (o attacco elementare) è un'azione o esecuzione di codice (exploit) che garantisce diritti di accesso illegali, ovvero non previsti dalla security policy . Un attacco è caratterizzato da:

1. **Precondition:** le informazioni e i diritti di accesso necessari per eseguire l'attacco.
    
2. **Postcondition:** le informazioni e i diritti acquisiti dopo il successo dell'attacco.
    
3. **Exploitability:** determinata dalla dimensione della precondition; maggiore è la precondition, minore è l'exploitability.

Lo stato dell'attaccante è definito dall'insieme di informazioni e diritti posseduti. L'intrusione ha successo se lo stato dell'attaccante raggiunge l'obiettivo desiderato; in caso contrario, la strategia dell'attaccante seleziona l'azione successiva basandosi sullo stato corrente .