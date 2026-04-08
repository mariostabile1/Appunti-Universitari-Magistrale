## Controllo degli Accessi e Implementazione

Il controllo degli accessi (_Access Control_) rappresenta l'implementazione pratica della _information policy_ per ogni risorsa del sistema a un determinato livello. Per ogni risorsa (oggetto) $R$, un modulo della _Trusted Computing Base_ (TCB) decide se la richiesta di un soggetto $S$ per una specifica operazione debba essere soddisfatta o proibita.

Formalmente, questo meccanismo è descritto dalla **Access Control Matrix (ACM)**, una matrice dove le righe rappresentano i soggetti e le colonne gli oggetti. La cella $ACM[i,j]$ definisce quali operazioni il soggetto $i$ può invocare sull'oggetto $j$. L'esistenza di una rappresentazione interna di questa matrice è una condizione necessaria, ma non sufficiente, per verificare l'applicazione della policy; se tale rappresentazione manca, la policy non è applicata.

Poiché la matrice è spesso sparsa, una comune implementazione è la **Access Control List (ACL)**. In questo modello, ogni oggetto (es. un file) possiede una lista che specifica i diritti per ogni soggetto o gruppo di soggetti. Un esempio classico è il _Linux File System_, dove per ogni file vengono definiti i permessi di lettura (r), scrittura (w) ed esecuzione (x) per tre categorie di soggetti: il proprietario (_user_), il gruppo (_group_) e gli altri (_others_).

## Struttura Formale dell'Attacco

Un attacco (_cyber attack_) è un'azione o l'esecuzione di un codice (_exploit_) che garantisce a chi lo esegue diritti di accesso illegali, ovvero non previsti nella posizione corretta della matrice di controllo degli accessi legale. L'esistenza di un attacco è legata a una vulnerabilità che lo abilita; solitamente, una singola vulnerabilità può essere mappata su diversi attacchi.

Un attacco è caratterizzato da una struttura precisa:

- **Precondition**: l'insieme di informazioni e diritti di accesso necessari per poter eseguire l'attacco.
    
- **Postcondition**: l'insieme di informazioni e diritti che l'esecuzione con successo dell'attacco garantisce.

Lo stato dell'attaccante (_Attacker State_) è definito dall'insieme di informazioni e diritti posseduti in un dato momento. Questo stato viene esteso da ogni attacco riuscito e dalle attività di raccolta informazioni. L'**exploitability** (sfruttabilità) di un attacco è inversamente proporzionale alla dimensione della sua precondizione: più grande è la precondizione (ovvero più requisiti sono necessari), minore è la sfruttabilità.

## Threat Agent e Dinamiche dell'Intrusione

Il **Threat Agent** (o avversario) è il terzo concetto fondamentale per la sicurezza: senza agenti di minaccia, sicurezza e safety sarebbero garantite. Un agente può essere naturale (es. disastri) o umano (malevolo o accidentale). Un agente malevolo ha solitamente un obiettivo e per raggiungerlo implementa un'**Intrusion**.

Un'intrusione non è un singolo evento, ma una sequenza di azioni e attacchi volta a raggiungere un obiettivo, che tipicamente consiste nel controllare illegalmente un sottoinsieme di risorse del sistema ICT/OT. Chi controlla un sottosistema può esfiltrare, manipolare o impedire l'accesso alle informazioni.

Il processo di intrusione è ciclico e segue un loop operativo:

1. **Initial Access**: L'ingresso nel sistema tramite tecniche come compromissione di asset OT, servizi remoti esterni, o attacchi alla supply chain.
    
2. **Information Discovery & Collection**: Raccolta di informazioni sui moduli del sistema.
    
3. **Vulnerability Discovery**: Individuazione di difetti nei moduli scoperti.
    
4. **Selection**: Scelta dell'azione o dell'attacco da eseguire.
    
5. **Execution**: Esecuzione dell'exploit o dell'azione umana.
    
6. **Management**: Gestione dell'output (successo o fallimento) e ripetizione del ciclo.

Questo ciclo implementa quella che viene spesso definita _privilege escalation_, termine talvolta fuorviante perché si concentra sugli attacchi trascurando l'importanza cruciale della raccolta di informazioni.

### Automazione delle Intrusioni

Le piattaforme di attacco (_attack platforms_) sono strumenti che automatizzano sequenze di attacchi. Tuttavia, le soluzioni attuali hanno limiti: non riescono ancora a intrecciare in modo ottimizzato la fase di raccolta informazioni con quella di sfruttamento (_exploitation_). Spesso operano in due fasi distinte: prima raccolgono i dati necessari e poi programmano la piattaforma per l'esecuzione.

## Attack Paths, Esposizioni e Mentalità Difensiva

Esiste una differenza sostanziale tra la mentalità del difensore e quella dell'attaccante. Il difensore tende a pensare in termini di singole vulnerabilità e della loro pericolosità. L'attaccante, invece, pensa in termini di **Attack Paths** (percorsi di attacco): scompone i diritti di accesso desiderati in una sequenza di attacchi e azioni propedeutiche.

Se un'intrusione permette a un attaccante di controllare un set di risorse, tale set rappresenta le **Exposures** (esposizioni) del sistema target. Le statistiche mostrano una densità di esposizioni molto elevata nelle organizzazioni, rendendo spesso inutili i semplici _penetration tests_ che non coprono la totalità dei percorsi possibili. Il compito più arduo per il difensore è scoprire e fermare questi percorsi.

## Case Study: Akira Ransomware

L'analisi del ransomware Akira (osservazioni 2023-2025) fornisce un esempio concreto di queste dinamiche.

- **Initial Access**: Avviene prevalentemente tramite l'abuso di VPN (Cisco ASA, SonicWall) o sfruttando vulnerabilità note (CVE).
    
- **Discovery**: Utilizzo massiccio di strumenti come Netscan e Advanced Port Scanner.
    
- **Collection**: Uso di utility di compressione (WinRAR, 7-Zip) prima dell'esfiltrazione.
    
- **Impact**: Distruzione dei backup, cifratura di sistemi ESXi/NAS ed esfiltrazione dati.
    
    I tempi di rilevamento sono spesso legati solo alla fase finale di distruzione (ransomware visibile), mentre l'intrusione può essere iniziata settimane o mesi prima.


## Contromisure e Choke Points

Una **Countermeasure** è una modifica statica o dinamica al sistema volta a fermare un'intrusione rimuovendo vulnerabilità o impedendone lo sfruttamento.

Un concetto chiave per l'efficienza difensiva è quello dei **Choke Points** (punti di strozzatura). Poiché diverse intrusioni (percorsi di attacco) possono condividere le stesse vulnerabilità o passare per gli stessi nodi critici, rimuovendo una singola vulnerabilità in un "choke point" è possibile bloccare molteplici percorsi di attacco simultaneamente. Questo dimostra che non è necessario rimuovere _tutte_ le vulnerabilità per garantire la sicurezza, ma è sufficiente concentrarsi su quelle che abilitano le azioni utili all'interno delle catene di attacco (_Attack Chains_). L'investimento più efficace in sicurezza è quello focalizzato sull'interruzione dei percorsi di intrusione.