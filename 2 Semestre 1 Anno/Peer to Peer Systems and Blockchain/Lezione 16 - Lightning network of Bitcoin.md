## Il Trilemma della Blockchain

Prima di capire perché esiste la Lightning Network, occorre capire il problema che intende risolvere. Vitalik Buterin ha formalizzato l'osservazione che i sistemi blockchain tendono a soddisfare al massimo due delle tre proprietà desiderabili simultaneamente.

> [!definition] Trilemma della Blockchain (Buterin)
> 
> Un sistema blockchain può soddisfare al massimo due delle seguenti tre proprietà: **decentralizzazione** (nessun punto di controllo centrale, resistenza alla censura), **scalabilità** (capacità di gestire un numero crescente di transazioni per unità di tempo), **sicurezza** (capacità di operare correttamente e difendersi dagli attacchi).

Concretamente, i sistemi esistenti si posizionano in modo diverso su questo triangolo. Bitcoin ed Ethereum privilegiano sicurezza e decentralizzazione, ma non scalano — Bitcoin elabora circa 7 transazioni al secondo a livello globale, contro le 65.000 di Visa. Hyperledger e Ripple sono sicuri e scalabili, ma centralizzati: un numero ristretto di nodi controlla la rete, con minima resistenza alla censura. IOTA era scalabile e decentralizzato, ma usava un Proof-of-Work leggero che ne comprometteva la sicurezza.

Il problema è quantitativamente serio. Con blocchi da 1 MB (4 MB dal 2017 con SegWit) e transazioni medie da 250 byte, Bitcoin può contenere circa 400 transazioni per blocco, che a un blocco ogni 10 minuti danno 7 TPS. La conferma richiede 6 blocchi per considerarsi definitiva, quindi circa un'ora di attesa. Non c'è confronto con i sistemi di pagamento tradizionali.

### Perché le soluzioni on-chain non bastano

La prima risposta intuitiva è: aumentiamo la dimensione del blocco. Bitcoin Cash ha fatto esattamente questo, portando prima a 8 MB e poi a 32 MB in un hard fork. Il problema è che per raggiungere la stessa capacità di Visa servirebbero blocchi da 8 GB — non è un errore di battitura. I nodi dovrebbero archiviare circa 400 TB di dati generati ogni anno e disporre di 120 megabit/sec di banda. Il risultato inevitabile è che solo un piccolo numero di nodi con risorse molto elevate potrebbe partecipare, aumentando la centralizzazione e riducendo la sicurezza.

Aumentare il tasso di produzione dei blocchi introduce un altro problema: più fork concorrenti, con conseguente riduzione della sicurezza complessiva. Il Proof-of-Stake e i protocolli di consenso leggeri risolvono il problema energetico e scalano meglio, ma spesso non sono davvero decentralizzati in pratica, e tendono a favorire chi è già ricco.

---

## L'Idea dei Canali di Pagamento Off-Chain

L'intuizione chiave che sblocca il problema è questa: non è necessario che ogni transazione finisca sulla blockchain. La blockchain è lenta e cara da usare perché richiede consenso globale tra tutti i nodi — ma il consenso globale è necessario solo per il regolamento finale dei fondi, non per ogni singolo pagamento intermedio.

> [!tip] Intuizione chiave
> 
> Spostare la maggior parte delle transazioni _fuori_ dalla blockchain, usando la catena solo per aprire e chiudere i canali di pagamento. In mezzo, le parti si scambiano "cambiali" (promissory notes) off-chain, a velocità di rete.

L'analogia della slide è illuminante: immaginate un cliente al bar che dà la carta di credito al barista all'inizio della serata. Il barista segna ogni drink su un conto ma non addebita la carta ad ogni giro, evitando le commissioni. Alla fine della serata regola tutto con un'unica transazione. Il barista non rischia niente perché ha la carta in mano come garanzia; se il cliente sparisce, può addebitarla. Nella Lightning Network, la "carta di credito" è il deposito in un indirizzo multifirma sulla blockchain, e le "cambiali" sono transazioni Bitcoin firmate ma non ancora trasmesse in rete.

I canali di pagamento off-chain sono quindi:

- **trustless**: non richiedono fiducia reciproca tra le parti, perché la blockchain funge da arbitro
- **decentralizzati**: costruiti sopra l'infrastruttura di Bitcoin, senza hard fork
- **istantanei**: le transazioni avvengono alla velocità della rete peer-to-peer, non ai ritmi della blockchain
- **ad alto volume**: potenzialmente illimitati in numero di transazioni per canale

La tipologia base è **unidirezionale** (una parte paga sempre l'altra), ma le estensioni più importanti sono i **canali bidirezionali** e la **composizione di canali** — che insieme costituiscono la Lightning Network vera e propria.

---

## Il Protocollo Lightning Network

La Lightning Network è un protocollo di livello 2 proposto da Joseph Poon e Thaddeus Dryja nel 2015. Tecnicamente si basa su tre operazioni fondamentali per ogni canale: apertura, impegni off-chain e chiusura.

### Apertura del Canale: la Funding Transaction

A livello tecnico, un canale di pagamento è un **indirizzo multifirma 2-di-2** — per spendere i fondi in quell'indirizzo servono le firme di entrambe le parti (Alice e Bob), esattamente come un conto bancario cointestato che richiede due firme per i prelievi.

Per aprire il canale, Alice crea una **funding transaction** che invia i suoi bitcoin all'indirizzo multifirma. Questa è l'unica transazione che deve comparire sulla blockchain durante l'intera vita del canale. La struttura è:

```
Funding Transaction:
  Input:  Indirizzo di Alice
  Output: Indirizzo multifirma Alice+Bob
  Importo: 100K satoshi
```

I fondi restano "in escrow" nell'indirizzo condiviso finché il canale non viene chiuso.

### Impegni Off-Chain: le Commitment Transactions

Una volta aperto il canale, Alice e Bob si scambiano **commitment transactions** — transazioni Bitcoin valide che ridistribuiscono i fondi del multifirma tra i due, ma che _non vengono trasmesse_ sulla blockchain. Rimangono conservate localmente da ciascuna delle parti.

Ogni commitment transaction ha questa struttura:

```
Commitment Transaction N:
  Input:  Indirizzo multifirma Alice+Bob
  Output: Alice → importo_A satoshi
  Output: Bob  → importo_B satoshi
```

La transazione deve essere firmata da entrambi. La sequenza tipica è: Alice firma la transazione e la invia a Bob, che la controfirma e la conserva (e viceversa). Così entrambi hanno in mano un documento valido che, se trasmesso, si traduce in un regolamento on-chain corrispondente al loro saldo attuale.

La cosa cruciale è che ogni nuova transazione _sostituisce_ la precedente — non la annulla tecnicamente, ma la rende obsoleta. Se Alice invia 80 BTC a Bob, entrambi conservano una commitment transaction che dice "Alice: 20, Bob: 80". Se poi Bob ne rimanda 10 ad Alice, entrambi creano e conservano una nuova transazione che dice "Alice: 30, Bob: 70". L'ultima transazione valida rappresenta il saldo corrente del canale.

### Chiusura del Canale: il Settlement On-Chain

Quando le parti vogliono chiudere il canale, una delle due trasmette l'ultima commitment transaction alla rete Bitcoin. La blockchain la registra, i fondi vengono distribuiti secondo i saldi finali, e il canale è chiuso. Solo questa seconda transazione, sommata alla funding transaction d'apertura, finisce sulla blockchain — indipendentemente da quante migliaia di transazioni siano state scambiate nel mezzo.

> [!abstract] Sintesi del ciclo di vita di un canale
> 
> 1. **Apertura** (on-chain): Alice deposita fondi nel multifirma — 1 transazione blockchain
> 2. **Operatività** (off-chain): N transazioni scambiate direttamente tra le parti, nessuna sulla chain
> 3. **Chiusura** (on-chain): l'ultima commitment viene trasmessa, i saldi finali vengono regolati — 1 transazione blockchain

Questo schema aumenta anche la **privacy**: la blockchain registra solo apertura e chiusura, senza alcun dettaglio sui singoli pagamenti intermedi.

---

## Meccanismi di Sicurezza Anti-Frode

Il protocollo introduce un problema serio: le commitment transaction precedenti sono ancora firme valide, perché sono state controfirmate in passato da entrambe le parti. Alice potrebbe voler trasmettere una vecchia transazione in cui aveva un saldo più favorevole. Come si impedisce?

### Double Spending Protection

Il primo livello di protezione è semplice: Bitcoin già previene il double spending. Quando viene trasmessa la commitment transaction finale, l'UTXO del multifirma viene "consumato". Se Alice prova a trasmettere anche una vecchia transazione che spende lo stesso multifirma, la rete la rifiuterà come tentativo di doppia spesa.

Questo funziona bene nel caso normale di chiusura cooperativa, ma non nel caso in cui sia Alice a trasmettere _per prima_ una vecchia transazione prima che Bob trasmetta quella corretta.

### Il Problema della "Cambiale Stracciata"

L'analogia delle cambiali chiarisce il nodo: ogni volta che Alice e Bob si accordano su un nuovo saldo, idealmente "straccerebbero" la cambiale precedente. Ma in Bitcoin non esiste un meccanismo per "stracciare" una transazione off-chain — non c'è garanzia che Alice non ne abbia conservato una copia e la trasmetta quando gli fa comodo.

### La Soluzione: Revocation Secrets e Punishment Mechanism

La Lightning Network risolve il problema con un meccanismo di **punizione basato su segreti di revoca** (_revocation secrets_). L'idea è: ogni commitment transaction è "pericolosa" da usare se sei disonesto, perché l'altra parte ha gli strumenti per punirti.

> [!definition] Meccanismo di Revoca
> 
> Ogni commitment transaction contiene un output condizionale sulla sua share: Alice può riscuotere i suoi fondi solo dopo un ritardo (es. 24 ore), oppure immediatamente se viene fornito il **revocation secret** della transazione. Prima di emettere una nuova commitment transaction, Alice deve rivelare a Bob il segreto di revoca della transazione _precedente_. Se Alice pubblica una vecchia transazione, Bob ha il segreto per "punirla" e prendere tutti i fondi del canale.

Il flusso concreto è:

1. Stato 1: Alice ha 700 sat, Bob 300 sat. Entrambi conservano la commitment T1.
2. Si aggiorna a Stato 2: Alice 400 sat, Bob 600 sat. Alice rivela a Bob il **segreto di revoca di T1** e riceve la nuova T2.
3. Se Alice ora trasmette T1 (il vecchio stato più favorevole), Bob ha il segreto per "punirla": presenta il segreto, e un apposito script (hash lock script) gli permette di riscuotere _tutti_ i fondi del canale.

Il ritardo nell'output di Alice (es. 24 ore) è fondamentale: dà a Bob il tempo di rilevare il tentativo di frode e reagire prima che Alice possa incassare i suoi fondi dall'old commitment.

```
Commitment Transaction con revoca:
  Input:  Indirizzo multifirma
  Output1: 90K sat
      → IF revocation_secret THEN paga a Bob
      → ELSE after 24 hours paga ad Alice
  Output2: 10K sat → paga a Bob
```

### Watchtower: Delega del Monitoraggio

Un nodo che partecipa a canali Lightning deve monitorare la blockchain almeno una volta a settimana, per poter reagire a eventuali commit disonesti entro la finestra di 1000 blocchi (~7 giorni). Se un nodo è spesso offline, può delegare questo compito a una **watchtower** — un servizio terzo che monitora la blockchain per conto dell'utente. La watchtower è progettata in modo che non possa tradire l'utente: conosce solo le informazioni necessarie per rilevare e punire la frode, non per rubare i fondi.

---

## Protezione dai Fondi Bloccati: Time Lock

C'è un'altra trappola potenziale: cosa succede se Bob sparisce subito dopo che Alice ha depositato nel multifirma? I fondi di Alice sarebbero bloccati a tempo indeterminato, perché per liberarli serve la firma di entrambi.

La soluzione prevede che **prima** di trasmettere la funding transaction, Alice si faccia firmare da Bob una transazione di rimborso con time lock:

> "Paga 100 BTC ad Alice dall'indirizzo multifirma dopo 30 giorni"

Alice conserva questa transazione off-chain. Solo dopo aver ricevuto questa garanzia, trasmette la funding transaction. Se Bob sparisce, Alice aspetta 30 giorni e poi trasmette la transazione di rimborso firmata da Bob — e recupera i suoi fondi.

---

## La Rete Lightning: Routing Multi-Hop

Finora abbiamo parlato di canali bilaterali. Il salto concettuale che trasforma i canali in una _rete_ è la **composizione di canali**: Alice può pagare Dave anche se non ha un canale diretto con lui, purché esista un percorso di canali intermedi.
![[Pasted image 20260407113700.png]]
Se Alice ha un canale con Bob, e Bob ha un canale con Carol, e Carol ha un canale con Dave, Alice può instradare il pagamento attraverso Bob e Carol. I nodi intermedi **non si fidano l'uno dell'altro** — ognuno impegna i propri fondi solo a condizione che il nodo successivo faccia altrettanto. Questo si ottiene tramite gli **HTLC**.

### HTLC: Hashed Timelock Contract

L'HTLC è il cuore tecnico del routing sicuro. Combina due meccanismi già visti — hash lock e time lock — in un unico contratto che rende i pagamenti multi-hop **atomici**: o tutti i nodi vengono pagati, o nessuno.

> [!definition] HTLC (Hashed Timelock Contract)
> 
> Un contratto che condiziona il pagamento alla rivelazione di un **segreto preimage** $R$ tale che $H(R) = H$ (hash lock), con un limite di tempo entro il quale il segreto deve essere rivelato (time lock). Se il segreto non viene rivelato in tempo, i fondi tornano al mittente.

Il protocollo di pagamento multi-hop funziona così:

1. **Dave** (destinatario) genera un numero casuale segreto $R$ e calcola il suo hash $H = H(R)$. Invia $H$ ad Alice fuori banda (es. nell'invoice di pagamento).
2. **Alice** crea un HTLC verso Bob: "ti pago 1 BTC se riesci a darmi l'$R$ tale che $H(R) = H$, entro 20 giorni."
3. **Bob**, sapendo che deve ottenere $R$ da Dave tramite Carol, crea un HTLC verso Carol: "ti pago 1 BTC se mi dai $R$, entro 15 giorni."
4. **Carol** crea un HTLC verso Dave: "ti pago 1 BTC se mi dai $R$, entro 10 giorni."
5. **Dave** rivela $R$ a Carol e incassa il suo BTC.
6. Il segreto $R$ si propaga all'indietro: Carol lo presenta a Bob, Bob lo presenta ad Alice, tutti vengono pagati in cascata.

I timelock sono **decrescenti** lungo il percorso (20→15→10 giorni): questo garantisce che ogni nodo abbia abbastanza tempo per riscuotere il proprio pagamento prima del timeout del nodo precedente. Se Dave non rivela $R$ entro il termine, tutti i pagamenti vengono automaticamente rimborsati.

> [!tip] Atomicità degli HTLC
> 
> Poiché ogni nodo può riscuotere solo presentando $R$, e $R$ diventa noto solo quando Dave lo rivela, l'intera catena di pagamenti è atomica: o Dave rivela il segreto e tutti vengono pagati, oppure nessuno lo rivela e i fondi tornano indietro. Un nodo intermedio non può rubare i fondi.

### Capacità del Canale e Liquidità

Il routing introduce due concetti fondamentali che spesso vengono confusi:

> [!definition] Channel Capacity vs Channel Balance
> 
> La **channel capacity** è la somma totale dei fondi depositati nel canale alla sua apertura. È fissa per tutta la vita del canale. Il **channel balance** è come quei fondi sono distribuiti tra i due nodi in un dato momento. Varia dinamicamente ad ogni transazione.

Un nodo che vuole fare routing deve avere **liquidità in uscita** (outbound liquidity) verso il nodo successivo. Il routing calcola percorsi basandosi sulla channel capacity (pubblica), ma non conosce il channel balance (privato). Questo genera fallimenti di routing: un canale con capacity 3 BTC potrebbe avere tutto il balance dalla parte sbagliata, rendendo impossibile instradare 2 BTC in quella direzione. La conseguenza pratica è che l'algoritmo attuale usa **brute force path probing**: prova un percorso, se fallisce ne prova un altro, e così via — il che porta a latenze elevate per circa il 5% dei pagamenti (oltre 3 minuti).

### Onion Routing per la Privacy

Per garantire la privacy, il routing usa una tecnica ispirata a [[Tor]] denominata **onion routing**: il mittente costruisce un "cipolla" a strati crittografati. Ogni nodo intermedio può decriptare solo il proprio strato, scoprendo esclusivamente l'identità del nodo precedente e del successivo — mai l'intera traiettoria del pagamento. Questo impedisce ai nodi intermedi di sapere chi sta pagando chi.

### Rebalancing dei Canali

Con l'uso, un canale tende a sbilanciarsi: se Alice paga sempre Bob, il balance si sposta tutto dal lato di Bob, esaurendo la liquidità in uscita di Alice. Per ribilanciare, un nodo può eseguire un **circular payment** — instrada un pagamento a se stesso attraverso un percorso che ripristina i balance desiderati senza costi di apertura/chiusura di nuovi canali.

---

## Stato Attuale e Problemi Aperti

La Lightning Network ha rilasciato la versione alpha a gennaio 2017. Il primo acquisto noto tramite Lightning è avvenuto a gennaio 2018. Il 20 marzo 2018 è stato sferrato il primo attacco DDoS, portando offline 200 nodi. Da allora la rete è cresciuta significativamente, con migliaia di nodi e canali attivi.

Esistono diverse implementazioni open source indipendenti e interoperabili: C-Lightning, Eclair (Scala), LND (Go), Ptarmigan (C++), Rust-Lightning, LIT (Python), Electrum. Le specifiche sono pubblicate come **BOLT** (Basis Of Lightning Technology), da BOLT #1 (protocollo base) a BOLT #11 (protocollo invoice per pagamenti Lightning).

Per Ethereum esiste la **Raiden Network/uRaiden**, lanciata sulla mainnet a novembre 2017, ma non ha avuto lo stesso successo, soppiantata da altre soluzioni Layer-2 come i rollup.

### Limiti della Lightning Network

La Lightning Network risolve brillantemente il problema della scalabilità, ma introduce nuovi trade-off:

**Fund locking**: i fondi depositati nei canali sono immobilizzati per tutta la durata del canale. Un nodo che vuole fare routing deve impegnare capitali significativi. Comportamenti disonesti della controparte possono bloccare i fondi per settimane.

**Always-on requirement**: senza watchtower, un nodo deve monitorare la blockchain regolarmente per proteggersi da chiusure fraudolente. Questo è problematico per i wallet mobile.

**Centralizzazione strisciante**: ci sono indizi che la rete stia sviluppando una topologia hub-and-spoke, con pochi nodi hub molto connessi che gestiscono la maggior parte del routing. Se confermato, ridurrebbe la decentralizzazione effettiva.

**Routing inefficiente**: il brute force probing è lento e produce molti fallimenti. I nuovi algoritmi proposti (gossip-based, ant algorithms) non sono ancora maturi.

**Apertura del canale costosa**: si ha bisogno di almeno una transazione on-chain per aprire ogni canale, il che non è economico durante periodi di fee elevate.

> [!question] Possibili domande d'esame
> 
> - Cos'è il trilemma della blockchain di Buterin? Quali sistemi reali si posizionano su ciascun lato del triangolo?
> - Descrivi il ciclo di vita completo di un canale Lightning (apertura, commitment, chiusura). Quante transazioni finiscono on-chain?
> - Cos'è una commitment transaction? Perché non viene trasmessa immediatamente?
> - Come funziona il meccanismo di punizione basato sui revocation secrets? Perché è necessario un time delay sull'output?
> - Cos'è un HTLC? Spiega come garantisce l'atomicità in un pagamento multi-hop.
> - Qual è la differenza tra channel capacity e channel balance? Perché questa distinzione crea problemi di routing?
> - Come funziona l'onion routing in Lightning e perché è importante per la privacy?
> - I canali off-chain risolvono il trilemma della blockchain? Discuti vantaggi e limiti.

---

## Conclusioni

La Lightning Network è la risposta più matura al problema della scalabilità di Bitcoin. Sfruttando le primitive crittografiche già disponibili in Bitcoin — multifirma, timelock, hashlock — costruisce un layer-2 che sposta quasi tutto il carico transazionale fuori dalla blockchain, riducendo drasticamente i tempi di conferma (da minuti/ore a millisecondi) e le commissioni.

La sicurezza del sistema si basa su tre pilastri: la blockchain come arbitro di ultima istanza, il meccanismo di punizione tramite revocation secrets per scoraggiare la trasmissione di vecchie commitment transactions, e gli HTLC per garantire l'atomicità dei pagamenti multi-hop. Non richiede un hard fork di Bitcoin.

Resta aperta la domanda centrale posta alla fine della lezione: i canali off-chain sono una soluzione al trilemma? La risposta è: _non è ancora dimostrato formalmente_. Migliorano enormemente la scalabilità senza sacrificare la sicurezza, ma la questione della decentralizzazione — specialmente nella topologia del grafo che si sta formando — rimane un punto di ricerca attivo.