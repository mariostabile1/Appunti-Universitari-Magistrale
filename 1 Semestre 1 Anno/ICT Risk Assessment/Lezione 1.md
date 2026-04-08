## Introduzione: Dalla sicurezza dei computer alla sicurezza della società

Il paradigma fondamentale della cybersecurity è mutato radicalmente. Come osservato da Marc Andreessen, "Software Is Eating The World", il che implica che non ci stiamo più limitando a proteggere dei calcolatori, ma stiamo proteggendo la società stessa. I direttori e i consigli di amministrazione spesso mancano delle conoscenze necessarie per svolgere efficacemente il loro ruolo di supervisione in questo ambito, nonostante la pervasività della tecnologia.

È essenziale comprendere che l'apprendimento di come "crackare" un sistema è utile solo come primo passo per capire come difenderlo. La vera sfida risiede nell'imparare a costruire sistemi robusti. Nel dark web, infatti, le bande criminali offrono somme di denaro maggiori a chi è in grado di costruire sistemi robusti rispetto a chi sa solo attaccarli, a testimonianza del valore economico della difesa.

## La natura degli oggetti intelligenti e la vulnerabilità pervasiva

La pervasività della cybersecurity deriva dal fatto che qualsiasi oggetto nel mondo è stato trasformato in uno _smart object_ aggiungendo un computer e una connessione internet. Per ragioni di scalabilità ed economia, questi computer sono _general purpose_, in grado di eseguire qualsiasi software, anche se inizialmente offrono solo un set limitato di funzioni. Mikko Hypponen sintetizza questa condizione con l'assioma: "If it's smart, it's vulnerable".

Poiché il software su qualsiasi computer deve essere aggiornato per migliorare le funzionalità o per ragioni di sicurezza (aggiornamenti "on the air"), un agente remoto ha la capacità di modificare il software sull'oggetto. In casi malevoli, questa funzionalità viene sfruttata per alterare il ruolo originale dell'oggetto; ad esempio, un termostato intelligente potrebbe essere usato per il mining di bitcoin o un assistente vocale per intercettare conversazioni. Non bisogna pensare a una smart TV come a un televisore, ma come a un computer che trasmette film; analogamente, un frigorifero intelligente è un computer che mantiene il cibo freddo.

## Intelligenza Artificiale e Cybersecurity

Per comprendere l'applicazione dell'Intelligenza Artificiale (AI) nella sicurezza, è necessario distinguere formalmente tra _attack_ e _intrusion_. Un **attack** è un'azione singola volta ad acquisire un diritto di accesso o un privilegio non posseduto. Un'**intrusion** è invece una sequenza di azioni, alcune delle quali sono attacchi, volta a scoprire la struttura di un sistema e acquisire diritti di accesso per raggiungere un obiettivo di alto livello (come rubare informazioni o distruggere equipaggiamento). L'intrusione richiede pianificazione e scoperta di informazioni sul target.

Attualmente, l'uso dell'AI nelle intrusioni reali è concentrato nelle fasi di _reconnaissance_, _weaponization_ ed esecuzione finale, con un uso limitato nei passaggi intermedi. Nonostante i progressi, le capacità offensive pratiche dell'AI nelle intrusioni _end-to-end_ rimangono limitate, specialmente quando la lunghezza della catena di attacco (PoC) aumenta, rendendo difficile per gli agenti AI manipolare input complessi. Al contrario, negli attacchi diretti agli esseri umani, come il _social engineering_ e il _phishing_, l'AI ha portato a un incremento significativo degli incidenti (135% per il social engineering, 260% per il voice phishing).

## Proprietà di Sicurezza nei Sistemi ICT

Un sistema ICT/OT è costituito da moduli che definiscono servizi invocati da altri moduli o utenti. La sicurezza di tali sistemi è regolata da una **security policy**, ovvero l'insieme di regole che definiscono chi può invocare operazioni per leggere o aggiornare valori. Le tre proprietà fondamentali che una policy deve preservare, note come triade CIA, sono:

La **Confidentiality** garantisce che un'informazione sia leggibile esclusivamente da chi ne possiede il diritto. L'**Integrity** assicura che le informazioni possano essere aggiornate solo da chi ha il diritto di farlo. Infine, l'**Availability** impone che il sistema garantisca a chi ha il diritto di eseguire un'operazione la possibilità di farlo entro un tempo finito. Mentre le prime due proprietà specificano il comportamento del sistema rispetto alla policy, l'availability introduce un vincolo temporale complesso, dipendente dalle risorse fisiche e logiche disponibili.

Queste proprietà sono definite **non-functional properties**, ovvero vincoli su _come_ il sistema implementa le sue funzionalità, distinte dai requisiti funzionali. Esse sono anche dette proprietà emergenti, poiché possono manifestarsi solo in determinate condizioni operative o contesti.

### Robustezza, Resilienza e Proprietà Derivate

Per preservare le proprietà CIA contro avversari intelligenti e malevoli, si valutano due caratteristiche sistemiche. La **Robustness** misura quanto bene il sistema resiste e non viene violato da un attacco. La **Resilience** valuta come un sistema, pur non essendo pienamente robusto, possa essere violato ma ritornare successivamente a un comportamento normale. La resilienza è spesso più _cost effective_ della robustezza, poiché una difesa completa è eccessivamente costosa.

Le vulnerabilità sono difetti (bug) che riducono la robustezza; sebbene ogni vulnerabilità sia un difetto, non tutti i difetti sono vulnerabilità. La _root cause_ è la vulnerabilità originale che permette il successo di un attacco.

Oltre alla triade CIA, esistono proprietà di sicurezza derivate:

- **Traceability**: la capacità di scoprire chi ha invocato una determinata operazione.
    
- **Accountability**: il principio per cui chi utilizza una risorsa deve pagarne il costo o renderne conto.
    
- **Auditability**: la possibilità di verificare se la security policy è applicata e soddisfatta.
    
- **Forensics**: la capacità di provare in un quadro legale che certe azioni sono avvenute e identificare l'esecutore.
    
- **Privacy**: la regolamentazione su chi può leggere e aggiornare informazioni personali.

## Cause Radice e Complessità della Difesa

Analizzando le cause radice (_root causes_) degli incidenti, i dati recenti (IRIS 2025) mostrano che l'uso di **Valid Accounts** rimane la tecnica di accesso iniziale predominante (circa il 46-57% negli anni), seguita dallo sfruttamento di applicazioni _public-facing_ e dal _phishing_.

La difesa è intrinsecamente più complessa dell'attacco a causa di diverse asimmetrie:

1. **Asimmetria nel costo del fallimento**: Gli attaccanti necessitano di un solo exploit di successo, mentre i difensori devono proteggersi contro ogni attacco. Un singolo falso negativo può compromettere l'intero sistema.
    
2. **Asimmetria nel dispiegamento**: Anche con correzioni note, la _remediation_ è lenta e richiede risorse ingenti per test e deployment globale, mentre gli attaccanti operano con risorse minime e tempi compressi.
    
3. **Scalabilità vs Affidabilità**: I difensori mirano all'affidabilità completa; gli attaccanti ottimizzano per scalabilità e impatto. L'AI abbassa le barriere per gli attaccanti, permettendo operazioni su larga scala senza competenze profonde.

## Aspetti Economici: Esternalità e Free Riding

La sicurezza non dipende solo dalle scelte individuali, ma dall'ecosistema, configurandosi come un bene pubblico. Questo concetto si spiega attraverso le **esternalità**, ovvero costi o benefici subiti da terze parti. Una "*security externality*" negativa si verifica quando computer non protetti vengono usati per attaccarne altri (ad esempio tramite botnet), disincentivando il singolo a proteggersi poiché il costo dell'attacco ricade su altri.

Questa situazione porta al problema del **Free Riding**, dove gli individui scelgono un livello di sicurezza inferiore all'ottimo sociale, sperando di beneficiare degli sforzi altrui. Esistono tre modelli prototipici di come lo sforzo individuale influenza la sicurezza globale:

- **Total Effort**: La sicurezza dipende dalla somma degli sforzi individuali. Il sistema diventa più sicuro all'aumentare degli individui che non fanno _free riding_.
    
- **Weakest Link**: La sicurezza dipende dallo sforzo minimo. Il sistema è determinato dall'individuo con l'investimento più basso; la sicurezza decresce all'aumentare degli individui con bassa protezione.
    
- **Best Shot**: La sicurezza dipende dallo sforzo massimo, determinato dall'individuo con il miglior rapporto costi-benefici, mentre gli altri fanno _free riding_ su di esso.

Un ulteriore problema è l'**asimmetria informativa**: gli individui non conoscono l'investimento di sicurezza degli altri agenti né hanno informazioni certe sugli attaccanti, rendendo difficile valutare il proprio livello di rischio e complicando il mercato delle assicurazioni cyber.
