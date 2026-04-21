# Bitcoin Mining

## Consenso Tradizionale vs Consenso di Nakamoto

Gli algoritmi di consenso classici — Paxos, Raft, PBFT — sono progettati per ambienti **chiusi**: i nodi sono noti in anticipo, ciascuno conosce l'identità degli altri e i canali di comunicazione sono autenticati. Funzionano bene per database distribuiti, state machine replication, sincronizzazione di orologi.

Le blockchain permissionless come Bitcoin operano in un contesto radicalmente diverso: nodi anonimi, churn elevato, nessun canale autenticato, nessuna sincronizzazione degli orologi, nessuna lista di partecipanti. I protocolli tradizionali semplicemente non si applicano.

> [!definition] Consenso di Nakamoto
>
> Un approccio "implicito" al consenso: nessuna votazione, nessuno scambio di messaggi collettivo. Garantisce *eventual consistency* — i nodi possono avere visioni temporaneamente divergenti del registro, ma alla fine convergeranno tutti sulla stessa cronologia, a patto che la maggioranza della potenza computazionale sia onesta.

Come osservato in lezione: la decentralizzazione di Bitcoin non è raggiunta con metodi puramente tecnici, ma con una combinazione di crittografia e *clever incentive engineering*.

---

## Il Problema del Double Spending

Senza consenso, il sistema crollerebbe immediatamente. Immaginiamo che Bob invii bitcoin ad Alice (transazione "verde"), propagata nel network. Subito dopo Bob prova a spendere gli stessi bitcoin con July (transazione "rossa"), propagata da un altro nodo. Nodi diversi riceverebbero le due transazioni in ordine diverso e i loro registri divergerebbero — impossibile capire quale sia quella valida.

La **MemPool** (*Memory Pool*) è la "sala d'attesa" in RAM di ogni nodo: contiene le transazioni ricevute ma non ancora incluse in un blocco. È importante sottolineare che la MemPool **non è l'UTXO set**: quest'ultimo contiene le transazioni già confermate nella blockchain, mentre la MemPool è un buffer temporaneo di transazioni in attesa di conferma. Funziona come una *clearing house*: quando un nodo riceve una transazione in conflitto con una già presente nella sua MemPool, la scarta immediatamente. Ma questo risolve solo il problema locale — il consenso globale richiede qualcosa di più.

---

## Mining: la Lotteria della Blockchain

I nodi competono per estrarre transazioni dalla propria MemPool e aggiungerle al registro. Questa competizione è il **mining**: una lotteria in cui il vincitore ha il diritto di proporre il blocco successivo e lo trasmette in broadcast alla rete.

Quando un nodo riceve un nuovo blocco valido, elimina dalla propria MemPool tutte le transazioni ora confermate e quelle in conflitto — garantendo che il double spending tentato da Bob venga neutralizzato.

---

## Anatomia di un Blocco

Il miner riempie un **blocco candidato** con transazioni dalla MemPool, poi costruisce il **Block Header**: un riassunto compatto dei metadati (circa 1000 volte più piccolo della lista delle transazioni).

| Campo | Dimensione | Descrizione |
|---|---|---|
| **Magic Number** | 4 byte | Identificatore costante della rete Bitcoin |
| **Block Size** | 4 byte | Dimensione totale del blocco |
| **Version** | 4 byte | Versione del protocollo; gestisce soft/hard fork |
| **Previous Block Hash** | 32 byte | Hash SHA-256 dell'header del blocco precedente — il "link" crittografico della catena |
| **Merkle Root** | 32 byte | Radice del Merkle Tree delle transazioni; qualsiasi modifica a una transazione altera questo valore |
| **Timestamp** | 4 byte | Secondi Unix dalla mezzanotte del 1 gen 1970; usato per calibrare la difficoltà |
| **Difficulty Target** | 4 byte | Soglia al di sotto della quale deve cadere l'hash del blocco |
| **Nonce** | 4 byte | Il valore che il miner modifica iterativamente per trovare la soluzione |
| **Transaction Counter** | 1–9 byte | Numero di transazioni nel blocco |
| **Transactions List** | Variabile (fino a 1 MB) | L'elenco effettivo delle transazioni |

Perché raggruppare in blocchi e non minare singole transazioni? Una catena di hash su blocchi è molto più corta di una su milioni di transazioni, rendendo la verifica esponenzialmente più rapida. Blocchi più grandi significano anche trasmissioni di rete più efficienti.

---

## Proof of Work: Meccanica e Complessità

> [!definition] Proof of Work
>
> Funzione $F_d(c, x) \to \{\text{true, false}\}$ dove $d$ è la difficoltà, $c$ è la challenge (l'header del blocco senza il nonce) e $x$ è il **nonce** da trovare. Calcolare $F_d$ con $x$ noto è rapido; trovare $x$ tale che il risultato sia "true" è computazionalmente difficile.

Il procedimento del miner:

1. Imposta il nonce a 0
2. Calcola `SHA256(SHA256(Block-Header + Nonce))`
3. Se l'hash è **inferiore al Target** → blocco valido, trasmetti
4. Altrimenti, incrementa il nonce di 1 e ripeti

Ogni singolo bit dell'hash a 256 bit è indipendente dagli altri — come il lancio di una moneta. Non esistono scorciatoie: l'unico metodo è la forza bruta. La probabilità che un hash casuale cada sotto la soglia $T$ è:

$$p = \frac{T + 1}{2^{256}}$$

Il numero medio di tentativi per trovare un hash valido è $1/p$. Al 1° gennaio 2017, questo valore era circa $2^{70}$.

> [!tip] Gli "zeri iniziali" sono una semplificazione
>
> Spesso si dice che la PoW è risolta quando l'hash inizia con un certo numero di zeri. È una buona approssimazione, ma non precisa: il target può abbassarsi senza cambiare il numero di zeri (es. da `001001` a `001000` — stessi due zeri, target più basso). La soglia effettiva è numerica, non basata sui caratteri.

L'idea di usare puzzle crittografici costosi da risolvere ma facili da verificare non è nuova: sistemi simili erano stati proposti per mitigare DoS e spam email (richiedendo cicli CPU invece di denaro per "affrancare" ogni messaggio). Nakamoto l'ha adattata al consenso distribuito.

> [!tip] La metafora dei dardi
>
> La PoW può essere immaginata come il lancio di freccette verso un bersaglio con gli occhi bendati. Ogni lancio ha uguale probabilità di colpire qualsiasi punto del bersaglio. La difficoltà è inversamente proporzionale alla dimensione del cerchio verde (la zona valida): più il cerchio è piccolo, più è difficile colpirlo. Se i giocatori diventano più bravi (hardware più veloce), si restringe il cerchio — si abbassa il target. Aggiungere uno zero al prefisso richiesto raddoppia in media lo sforzo computazionale; rimuoverne uno lo dimezza.

### PoW: Applicazioni Precedenti a Bitcoin

La Proof of Work come strumento generale ha storia propria, ben prima di Nakamoto. Il suo principio è semplice: un meccanismo che consente a una parte di *dimostrare* a un'altra di aver impiegato una certa quantità di risorse computazionali, dove la verifica richiede molto meno tempo dell'esecuzione.

**Contrasto agli attacchi DoS** — Si può condizionare l'accesso a un servizio alla risoluzione di un puzzle computazionalmente costoso. Questo throttle le richieste: chi vuole inondare il server deve pagare un costo CPU per ogni tentativo, rendendo l'attacco proibitivo su larga scala.

**Contrasto allo spam email** — L'idea è affrancare ogni messaggio non con denaro ma con cicli CPU: chi invia poche email non sente quasi il costo, perché il puzzle viene eseguito poche volte. Per uno spammer che invia centinaia di migliaia di messaggi al giorno, lo stesso puzzle moltiplicato per milioni di invii diventa proibitivamente costoso. Il costo computazionale funge da "francobollo" digitale.

---

## Resistenza agli Attacchi Sybil

Ad ogni round, il miner che risolve la PoW viene eletto implicitamente come leader. L'elezione è proporzionale alla **potenza computazionale**, una risorsa fisica difficile da monopolizzare — non al numero di identità di rete. Creare migliaia di identità false non dà alcun vantaggio: conta solo quanti hash al secondo si riesce a calcolare. Per sabotare il sistema bisognerebbe controllare almeno il 51% della potenza di hashing globale.

---

## Incentivi per i Miner

Validare blocchi costa enormi quantità di energia. Perché farlo onestamente?

**Block Reward** — La prima transazione di ogni blocco è la **Coinbase Transaction**: crea bitcoin "dal nulla" e li assegna al miner. È l'unico meccanismo per immettere nuovi BTC nel sistema. La supply totale è fissa a **21.000.000 BTC** (raggiunta circa nel 2140), il che rende Bitcoin strutturalmente resistente all'inflazione — impossibile "stampare" nuova moneta per decisione politica. Questo implica una proprietà che non esiste in alcuna valuta fiat: chi possiede 1 BTC possiede sempre almeno un ventunomilionesimo dell'intera supply globale. Nelle valute tradizionali, i governi e le banche centrali possono aumentare l'offerta per decisioni politiche, diluendo il valore di chi già possiede quella valuta.

La ricompensa si dimezza ogni 210.000 blocchi (~4 anni): è il celebre **Halving**.

| Era | Periodo | Block Reward |
|---|---|---|
| 1 | 2009 | 50 BTC |
| 2 | 2012 | 25 BTC |
| 3 | 2016 | 12,5 BTC |
| 4 | 2020 | 6,25 BTC |
| 5 | 2024 (attuale) | 3,125 BTC |
| … | … fino all'Era 33 | 1 Satoshi ($10^{-8}$ BTC) |

**Transaction Fees** — La differenza tra $\sum \text{inputs}$ e $\sum \text{outputs}$ di ogni transazione va al miner. Gli utenti le impostano volontariamente per ottenere priorità di inclusione. Con il susseguirsi degli halving, le fee costituiranno una percentuale sempre maggiore dei ricavi dei miner.

La Coinbase Transaction contiene anche un input "fittizio" usato per messaggi personalizzati. Nel Genesis Block, Nakamoto vi nascose: *"The Times 03/Jan/2009: Chancellor on brink of second bailout for banks"*.

> [!note] Struttura dell'output della Coinbase
>
> L'output della Coinbase Transaction è inviato a uno o più indirizzi Bitcoin del miner stesso. Il valore corrisponde alla somma della block reward più le fee di tutte le transazioni incluse nel blocco. È il miner a decidere a quali propri indirizzi destinare la ricompensa.

### Dinamiche a Lungo Termine dei Ricavi

Storicamente la componente dominante dei ricavi dei miner è stata il block reward; le transaction fee rappresentano solo una piccola percentuale. Questa situazione è però destinata a cambiare: man mano che gli halving si succedono e la block reward si avvicina a zero, le fee diventeranno la fonte quasi esclusiva di compensazione.

C'è però una tensione strutturale: ogni nuovo miner che entra nella rete abbassa la probabilità di ricompensa degli altri. Per mantenere competitive le proprie probabilità, ogni miner è incentivato ad aumentare continuamente il proprio hash rate, alimentando una corsa agli armamenti computazionale. Questa dinamica ha spinto i miner a organizzarsi in **mining pool** — discusse nella lezione successiva.

---

## Regolazione Automatica della Difficoltà

Il sistema è calibrato per produrre **un blocco ogni 10 minuti**. Questo intervallo non è casuale: deve essere molto più lungo del tempo di propagazione del blocco sulla rete, affinché tutti i miner lavorino sulla stessa catena senza sprecare energia su rami obsoleti. Nelle parole di Nakamoto stesso: *"We want blocks to usually propagate in much less time than it takes to generate them, otherwise nodes would spend too much time working on obsolete blocks."* Se i blocchi vengono minati troppo frequentemente, i miner costruiscono catene concorrenti: solo una diventerà la più lunga, e tutti gli altri avranno sprecato energia su rami che verranno abbandonati — riducendo la sicurezza effettiva del sistema.

Ethereum ha scelto tempi più rapidi, ottenendo conferme più veloci e minore variabilità nei payout per i miner, ma al costo di più fork e un sistema di ricompensa molto più complesso.

La difficoltà si auto-regola ogni **2016 blocchi** (~2 settimane a 10 min/blocco = 20.160 minuti). Il ricalcolo avviene in modo completamente autonomo su ogni nodo:

$$\text{Nuovo Target} = \text{Vecchio Target} \times \frac{\text{Tempo effettivo}}{20.160 \text{ min}}$$

- Rete troppo veloce (es. 16.128 min) → rapporto $0{,}8$ → target si abbassa → difficoltà aumenta
- Rete troppo lenta (es. 22.176 min) → rapporto $1{,}1$ → target si alza → difficoltà diminuisce

L'aspetto più elegante: nessun coordinatore centrale. Tutti i nodi eseguono lo stesso algoritmo open-source sulle stesse informazioni e convergono autonomamente allo stesso nuovo target.

---

## Struttura della Blockchain e Fork

La blockchain non è una catena perfettamente lineare, ma un **albero** in cui solo il ramo più lungo è canonico.

I blocchi non hanno un indice interno: vengono identificati dal loro **Block Hash** (calcolato dinamicamente alla ricezione) o dalla **Block Height** (numero di blocchi dal Genesis Block, che ha altezza 0). L'ultimo blocco aggiunto è la *blockchain head*.

La tamper-freeness è garantita strutturalmente: modificare una transazione altera il Merkle Root, cambia l'hash del blocco, invalida il `hashprev` del blocco successivo, e innesca un effetto a cascata che obbliga a ricalcolare tutta la PoW da quel punto in poi.

### Fork Temporanei e Longest Chain Rule

La latenza di rete introduce i **fork temporanei**: due miner possono trovare un blocco valido quasi simultaneamente, puntando allo stesso genitore. La rete si divide — alcuni nodi vedono prima il blocco A, altri il blocco C, in base alla vicinanza fisica al miner che ha trovato il blocco — e due rami crescono in parallelo. Ogni miner continua a lavorare sul ramo che ha ricevuto per primo; il ramo alternativo viene conservato in una cache locale. I due fork possono crescere indipendentemente, con miner diversi che lavorano su rami diversi.

> [!definition] Longest Chain Rule (Regola di Nakamoto)
>
> I nodi considerano valida la catena che rappresenta il maggiore lavoro cumulativo (la più lunga). Non appena un nuovo blocco estende uno dei due rami rendendolo più lungo, tutti i miner abbandonano il ramo più corto e migrano sul ramo vincente. I blocchi del ramo perdente diventano **orphan blocks**; le loro transazioni non confermate tornano nella MemPool per essere eventualmente incluse in un blocco futuro.

Per questo motivo si raccomanda la **regola delle 6 conferme**: una transazione è considerata definitiva solo quando ha almeno altri 5 blocchi costruiti sopra di essa. Il valore 6 è il default, ma può essere configurato dal client in base al livello di sicurezza desiderato — transazioni di alto valore possono richiedere più conferme. Scopo della regola: dare alla rete il tempo di raggiungere un accordo sull'ordinamento dei blocchi.

### Algoritmo di Ricezione di un Blocco

Quando un nodo riceve un nuovo blocco $b$, esegue il seguente algoritmo per aggiornare la propria visione della blockchain:

```
Receive block b
  For this node the current head is block bmax at height hmax
  Connect block b in the tree as child of its parent p at height
    hb = hp + 1
  if hb > hmax then
    hmax = hb
    bmax = b
    compute UTXO for the path leading to bmax
    cleanup memory pool
  end if
```

Se il nuovo blocco è più alto della testa corrente, diventa la nuova testa. Si ricalcola l'UTXO set lungo il percorso fino alla nuova testa e si pulisce la MemPool rimuovendo le transazioni ora confermate e quelle in conflitto. Se invece il blocco appartiene a un fork più corto, viene conservato in cache senza diventare la testa.

### Teorema del Consenso di Nakamoto

> [!theorem] Eventual Consistency
>
> I fork vengono risolti e tutti i nodi concordano alla fine su quale sia la blockchain più lunga. Il sistema garantisce *eventual consistency*.
>
> **Proof sketch**: affinché un fork continui a esistere, devono essere trovati blocchi quasi simultaneamente su entrambi i rami, estendendoli in parallelo. La probabilità che questo accada ripetutamente decresce esponenzialmente con la lunghezza del fork. Quindi esisterà sempre un momento in cui un solo ramo viene esteso, diventando la catena più lunga e imponendosi come quella canonica.

Fork prolungati — due catene che crescono in parallelo a lungo — sono matematicamente possibili ma estremamente improbabili: la componente aleatoria del mining e i ritardi di propagazione introducono sufficiente rumore da impedire una crescita perfettamente sincrona dei due rami.

> [!question] Possibili domande d'esame
>
> - In cosa differisce il **consenso di Nakamoto** rispetto ai protocolli classici come Paxos o PBFT? Quali proprietà del contesto Bitcoin rendono inapplicabili questi ultimi?
> - Descrivi la struttura del **Block Header** di Bitcoin: quali campi contiene e qual è il ruolo di ciascuno nel meccanismo di consenso?
> - Spiega la meccanica della **Proof of Work**: qual è la formula per la probabilità che un hash sia valido? Perché non esistono scorciatoie algoritmiche?
> - Perché Bitcoin mira a produrre **un blocco ogni 10 minuti**? Cosa succederebbe se i blocchi fossero molto più frequenti?
> - Descrivi il meccanismo di **regolazione automatica della difficoltà**: ogni quanti blocchi avviene, quale formula si usa e chi lo esegue?
> - Cos'è la **Longest Chain Rule**? Come risolve i fork temporanei e cosa sono gli **orphan block**?
> - Perché si raccomandano **6 conferme** prima di considerare una transazione definitiva? Questo valore è fisso o configurabile?
> - Descrivi l'evoluzione dell'hardware per il mining (CPU → GPU → FPGA → ASIC): quali erano i vantaggi di ogni generazione?
> - Cos'è il **Block Reward** e come cambia nel tempo con i **Halving**? Qual è l'impatto a lungo termine sugli incentivi dei miner?
> - Enuncia e dimostra intuitivamente il **Teorema dell'Eventual Consistency** del consenso di Nakamoto.
