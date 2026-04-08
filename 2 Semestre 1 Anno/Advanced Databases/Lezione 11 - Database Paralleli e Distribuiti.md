---
tags:
  - università/advanced-databases
  - parallel-databases
  - distributed-databases
  - two-phase-commit
data: 2024-01-01
lezione: "11 - Database Paralleli e Distribuiti"
professore: ""
---

# Database Paralleli e Distribuiti

Un **sistema parallelo** è un sistema che consente l'elaborazione parallela delle operazioni. Un **sistema distribuito** è composto da più macchine che lavorano insieme per raggiungere lo stesso obiettivo. I database possono essere ospitati su singole macchine (che possono anche essere sistemi paralleli) o distribuiti su diversi componenti di un'architettura distribuita. Un caso estremo di architettura distribuita è rappresentato dalle reti **peer-to-peer**: una collezione di macchine indipendenti senza alcun indice centralizzato che indica quale porzione dei dati si trova dove.

---

## Sistemi Paralleli

### Modelli di Parallelismo

Le architetture parallele si suddividono in tre gruppi:

- **Shared-memory**: tutti i processori condividono la stessa memoria principale, ma ognuno ha la propria cache;
- **Shared-disk**: ogni processore ha la propria memoria, non accessibile dagli altri. Tuttavia, tutti condividono la stessa memoria permanente;
- **Shared-nothing**: tutti i processori hanno la propria memoria e disco/i. Tutta la comunicazione avviene tramite la rete, da processore a processore. Questa è l'architettura più comunemente usata per i sistemi di database.

> [!note] Nota del corso
>
> Questo corso usa le macchine **shared-nothing**. Gli algoritmi progettati per queste macchine devono essere consapevoli del costo di invio dei messaggi tra processori: tipicamente, il costo è composto da un overhead fisso elevato più un piccolo costo per byte trasmesso. Conviene quindi inviare grandi quantità di dati contemporaneamente.

### Partizionamento dei Dati

Diverse tecniche possono essere usate per distribuire i dati tra i dischi. Assumendo $p$ processori:

- **Range partitioning**: il nodo $i$-esimo riceve le tuple con $k(i) \leq A \leq k(i+1)$;
- **Hash partitioning**: si applica una funzione hash a un attributo della relazione (es. modulo $p$), distribuendo i record su $p$ "bucket";
- **Random partitioning con round-robin**: i record vengono assegnati casualmente ai processori, uno alla volta, in modo circolare;
- **Block partitioning con round-robin**: come sopra, ma vengono assegnati interi blocchi invece di un record alla volta;
- **Co-located partitioning**: dopo aver partizionato una relazione $R$ in un insieme di $R_i$, il semijoin $(S, R_i)$ tra un'altra tabella $S$ e una singola partizione $R_i$ viene memorizzato nel nodo $i$-esimo. Particolarmente utile per eseguire join in parallelo.

Il numero di frammenti può essere fisso in anticipo e poi mappato ai nodi, oppure può essere dinamico. I frammenti vengono tipicamente duplicati per resilienza. Le relazioni possono anche essere partizionate verticalmente.

> [!tip] Utilità del data partitioning
>
> Il partizionamento dei dati è fondamentale per l'esecuzione parallela, ma anche per accedere solo a certe parti dei dati quando i dati sono partizionati, e per allocare i frammenti cruciali ai processori più veloci.

### Algoritmi Paralleli

- **Distinct**: le tuple vengono distribuite tramite una funzione hash e il Distinct viene eseguito localmente in parallelo su tutti i nodi;

- **Union, Intersection, Difference**: se le due relazioni sono hashate con la stessa funzione, queste operazioni possono essere eseguite localmente. Altrimenti, entrambe le relazioni vengono ri-hashate con una funzione in $[0, p-1]$;

- **Join**: le tuple delle due relazioni $R$ e $S$ vengono distribuite usando la stessa funzione hash dipendente dall'attributo di join. Il join effettivo viene poi eseguito localmente. Esistono quattro algoritmi:
  - **Co-located join**: usato quando $R$ e $S$ sono partizionati allo stesso modo e i frammenti sono co-localizzati;
  - **Directed join**: se le relazioni sono partizionate allo stesso modo ma non co-localizzate, si invia una delle due ai nodi corrispondenti dell'altra;
  - **Repartitioned join**: se non sono partizionate allo stesso modo, si ripartiziona una o entrambe e si usa il directed join;
  - **Broadcast join**: usato quando una delle due tabelle è molto piccola; una copia dell'intera tabella viene inviata a ogni nodo.

- **GroupBy**: le tuple vengono distribuite usando una funzione hash su un sottoinsieme degli attributi di raggruppamento, e il GroupBy viene eseguito localmente;

- **Projection e Filter**: entrambe le operazioni possono essere eseguite localmente.

Quando si usano algoritmi paralleli, il numero totale di accessi e il tempo CPU aumentano, ma il tempo elapsed (si spera) sarà inferiore rispetto a un'esecuzione sequenziale. Un operatore unario richiederà $\frac{1}{p}$ del tempo elapsed con $p$ processori.

Il costo del join in caso peggiore (repartitioning necessario) si suddivide in:
$$\frac{N_{pag}(R) + N_{pag}(S)}{p} + \frac{(N_{pag}(R) + N_{pag}(S))(p-1)}{p} + \frac{2 \cdot N_{pag}}{p}$$

Questo significa che il tempo elapsed è quasi uguale al tempo sequenziale diviso per $p$, con un costo extra per il passaggio di messaggi. Un problema importante è la **distribuzione non uniforme**: se un nodo ha molte più tuple degli altri, gli altri devono aspettare.

---

## Sistemi Distribuiti

I sistemi distribuiti sono molto simili ai sistemi shared-nothing. La differenza è nel costo della comunicazione: nei sistemi paralleli il costo del passaggio di messaggi è tipicamente piccolo rispetto agli accessi al disco (i nodi sono collegati in una rete ad alta capacità); nei sistemi distribuiti, i processori sono fisicamente distanti, la rete potrebbe non avere alta capacità, e il costo dello scambio di messaggi può essere molto alto. Inoltre, la comunicazione non è altrettanto affidabile: i siti potrebbero non essere raggiungibili per problemi di rete o di sito, causando **partizioni** nel sistema.

Un grande vantaggio dei sistemi distribuiti è la **resilienza ai guasti** tramite la duplicazione dei dati su più siti.

### Distribuzione dei Dati

Un motivo importante per distribuire i dati è che l'organizzazione stessa è distribuita su molti siti. La distribuzione avviene tramite **partizionamento** e **replicazione**:

- **Partizionamento**: orizzontale (es. dividere i dati per nazione) o verticale (es. tenere solo certi attributi su certi nodi);
- **Replicazione**: copiare i frammenti di relazioni per garantire la resilienza. La replicazione rende le letture più veloci e gli aggiornamenti più lenti.

La fase di progettazione della distribuzione dei dati si divide in due passi: prima si dividono le relazioni in frammenti; poi ogni frammento viene associato a $n$ siti (si può scegliere una **copia primaria** con priorità più alta). Decidere come assegnare i frammenti è un problema di ottimizzazione difficile.

### Distributed Query Processing

Poiché la comunicazione tra siti ha un costo significativo, alcuni piani di query sono più efficienti. La sfida principale è la computazione dei join. Esistono due possibilità:

1. Una delle tabelle è molto piccola e viene inviata ai siti contenenti i frammenti dell'altra;
2. Si usa la **riduzione semijoin**.

Il piano con semijoin per calcolare $\text{Join}(R(X, Y), S(Y, Z))$ è:

1. Inviare $\pi_Y(R)$ al sito contenente $S$, chiamato $s$;
2. $s$ calcola $S_1 = \text{Semijoin}(\pi_Y(R), S(Y, Z))$, che restituisce tutti i record di $S$ che soddisfano la condizione di join;
3. $s$ invia $S_1$ al sito $r$ contenente $R$;
4. $r$ calcola il join tra $R$ e il semijoin, equivalente al join tra le tabelle complete.

Questo approccio conviene quando $Y$ è ragionevolmente più piccolo di $X$ e $Z$.

---

## Consistenza Distribuita

Le transazioni sono ora un processo distribuito. Una transazione non è più eseguita da un singolo processore; è composta da diverse componenti, ognuna su un sito diverso.

### Two-Phase Commit (2PC)

Un protocollo comune per i commit distribuiti è il **two-phase commit** (commit a due fasi). Si assume un sito coordinatore $C$ e un insieme di partecipanti $P_i$.

**Prima fase**:
- $C$ scrive `⟨Prepare, T⟩` sul suo log e invia a ogni partecipante: `send(Pi, prepare T)`;
- Ogni partecipante deve rispondere, comunicando se può o non può fare commit. Se vuole fare commit, scrive `⟨ready, T⟩` sul suo log ed entra nello stato **pre-committed** (da questo momento solo $C$ può abortire la transazione).

**Seconda fase**:
- Se tutti i partecipanti hanno inviato un messaggio "ready": $C$ scrive `⟨Commit, T⟩` sul suo log e invia un messaggio di commit a ogni $P_i$;
- Se anche solo un partecipante ha risposto "no" o non ha risposto: $C$ scrive `⟨Abort, T⟩` sul suo log e invia un messaggio di abort a ogni $P_i$.

Dopo questa fase, tutti i partecipanti escono dallo stato pre-committed e riprendono l'esecuzione normale.

> [!tip] Proprietà del protocollo 2PC
>
> La proprietà principale è che offre una semplice procedura di restart che mantiene la consistenza:
> - Se c'è un guasto in qualsiasi momento, si può sempre recuperare;
> - Se ogni sito è garantito di ripartire eventualmente, il protocollo è garantito di terminare.

Quando un attore scrive una decisione nel log e invia un messaggio, se poi subisce un guasto, al riavvio rilegge il log e re-invia il messaggio. I messaggi duplicati non sono un problema poiché l'invio è idempotente. I messaggi persi sono gestiti con timeout e reinvio. Il riavvio è **log-guided**: ogni messaggio viene prima scritto nel log.

**Riavvio di un partecipante**:
- Se l'ultimo record è un commit o un abort: si gestisce come il caso non distribuito;
- Se è un "don't commit" o un write: si esegue un abort locale;
- Se è un "ready": si contatta il coordinatore e gli altri siti per scoprire la decisione; fino alla risposta, la transazione rimane in pre-committed. Se il coordinatore non risponde, è il **caso critico** (il partecipante è bloccato).

**Riavvio del coordinatore**:
- Se l'ultimo record è "prepare": può decidere di abortire o non fare nulla;
- Se è un abort o un commit: può reinviare o non fare nulla (i partecipanti richiederanno informazioni se necessario).

### Locking Distribuito

In sistemi distribuiti, i dati sono replicati. Per garantire la consistenza, tutte le copie dello stesso oggetto devono essere modificate nello stesso modo. Esistono due approcci per implementare il locking:

**Soluzione centralizzata** (lock fisici): un sito centralizzato gestisce l'accesso agli elementi logici. Quando una transazione vuole un lock su $X$, invia una richiesta al sito di lock, che concede o nega il lock. Il costo usuale è tre messaggi per lock: richiesta, concessione, rilascio. Problemi: collo di bottiglia con molti siti e transazioni, punto singolo di guasto.

**Soluzione distribuita** (lock logici): un lock logico viene preso sulle copie dei dati. Due approcci:
- **Copia primaria**: una copia è dichiarata primaria e gli aggiornamenti vengono propagati alle altre copie in un secondo momento. Crea un collo di bottiglia e un punto singolo di guasto;
- **Lock su qualsiasi copia**: ogni copia ha il proprio lock; le transazioni prendono lock sulle copie locali.

Per garantire la consistenza con lock su qualsiasi copia, si usano due politiche:

- **Write-locks-all**: per scrivere, una transazione deve ottenere un lock esclusivo su tutte le copie; per leggere, basta un lock;
- **Majority locking**: per leggere o scrivere, una transazione deve acquisire $\lceil(n+1)/2\rceil$ lock corrispondenti.

Possono esistere soluzioni intermedie con quorum diversi. Le condizioni da soddisfare sono:
$$x + x > n \quad \text{(write-write)}$$
$$s + x > n \quad \text{(read-write)}$$

dove $x$ e $s$ sono i quorum per lock esclusivi e condivisi. Casi tipici:
- $x = s = (n+1)/2$ (majority locking);
- $x = n, s = 1$ (write-locks-all);
- $x = n-1, s = 2$.

Per gestire i deadlock, si usano le stesse soluzioni viste nei sistemi centralizati (wait-for graph, timeout, wait-die, wound-wait). In pratica, i **timeout** sono preferiti.

> [!question] Possibili domande d'esame
>
> - Quali sono i tre modelli di parallelismo? Quale usa questo corso?
> - Descrivi le tecniche di data partitioning (range, hash, round-robin, co-located).
> - Come vengono parallelizzati i quattro algoritmi di join distribuito?
> - Differenza tra sistema parallelo e sistema distribuito.
> - Come funziona la riduzione semijoin? Quando conviene?
> - Descrivi il protocollo two-phase commit (entrambe le fasi).
> - Cos'è il caso critico nel 2PC?
> - Differenza tra locking centralizzato e distribuito. Cos'è il majority locking?
> - Quali sono le condizioni per garantire la consistenza con quorum variabili?
