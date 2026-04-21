# Bitcoin Security

Il sistema Bitcoin si articola su quattro livelli — **Blockchain Design**, **P2P Architecture**, **Consensus** e **Transactions** — ognuno soggetto ad attacchi specifici. La rete P2P affronta l'Eclipse Attack e l'Approx. Bitcoin Mining; il consenso subisce il 51% Attack e il Selfish Mining; le transazioni fronteggiano il Double Spending e il Malleability Attack. La sicurezza è analizzata tramite modelli formali come Random Oracle, Nakamoto's Model e Bitcoin Backbone.

---

## Double Spending e Attacco del 51%

Il **Double Spending Attack** consiste nel tentare di spendere lo stesso bitcoin due volte verso due destinatari diversi. L'approccio ingenuo — trasmettere entrambe le transazioni alla rete — fallisce: finiscono entrambe nella MemPool, ma un minatore onesto ne includerà solo una nel blocco successivo, scartando la seconda. Se due minatori le validano contemporaneamente generando un fork, la Longest Chain Rule ne renderà valida solo una — motivo per cui si attendono solitamente 6 conferme.

L'attacco diventa pericoloso in modalità **stealth**. Il minatore malevolo:
1. Spende bitcoin sulla catena pubblica per acquistare un bene (es. una barca)
2. Mina in segreto una catena privata omettendo quella transazione — mantenendo il controllo dei bitcoin
3. Quando la catena privata supera quella pubblica, la trasmette alla rete
4. Per la Longest Chain Rule, la rete adotta la nuova catena: l'attaccante riottiene i fondi e mantiene il bene

![[Pasted image 20260319164820.png]]

Perché l'attacco riesca con regolarità, il minatore deve detenere oltre il **50% dell'hashing power** — da qui il nome "51% Attack".

> [!note] Il caso GHash.IO (2014)
>
> La mining pool GHash.IO raggiunse quasi il 50% della potenza di calcolo (38,24%), scatenando il panico nella community. Non subì alcun attacco: per chi detiene tanto potere è economicamente più conveniente continuare a minare e incassare i block reward che distruggere il network per annullare una singola transazione.

---

## Transaction Malleability e il Caso Mt. Gox

La **Transaction Malleability** permetteva di alterare il **TXID** (identificativo) di una transazione senza modificarne gli effetti reali — mittente, destinatario e importo restavano invariati, ma l'ID cambiava. Come se il numero di tracking di un pacco venisse sostituito in transito.

La vulnerabilità era nell'algoritmo **ECDSA**: una firma è una coppia $(r, s)$, ma se $(r, s)$ è valida lo è anche $(r, n-s)$, dove $n$ è una costante della curva. Poiché il TXID è l'hash SHA-256 dell'intera transazione (inclusa la firma nello ScriptSig), cambiare la rappresentazione in $(r, n-s)$ modifica il TXID senza invalidare la firma logica.

Lo schema di attacco funziona così: Alice invia 50 BTC a Bob, generando una transazione con TXID = 1234. Bob intercetta la transazione, ne crea una copia con la firma modificata in $(r, n-s)$ — stessi input, stesso output, stessa firma logicamente valida, ma TXID diverso (es. 4567). Ora c'è una gara: entrambe le transazioni finiscono nella MemPool. Il momento in cui una viene confermata, l'altra viene scartata come duplicato. Se viene confermata quella di Bob (TXID 4567), Alice non trova mai il suo TXID originale (1234) sulla blockchain — e Bob sostiene di non aver ricevuto nulla, costringendola a un secondo pagamento.

> [!note] Il fallimento di Mt. Gox (2013–2014)
>
> Mt. Gox gestiva il 72% delle transazioni Bitcoin mondiali. Ad aprile 2014 dichiarò bancarotta, avendo perso ~744.408 BTC (il 6% dell'offerta totale, ~450 milioni di dollari dell'epoca). Mt. Gox non usava codifiche di firma standard: alcuni utenti le "standardizzavano" applicando la reverse malleability, ispirando gli hacker. Gli attaccanti prelevavano fondi, alteravano il TXID in transito, ricevevano i fondi e — poiché l'exchange non vedeva l'ID originale — rieffettuavano il prelievo.

La vulnerabilità è stata eliminata con il soft fork **Segregated Witness (SegWit)**: i dati della firma vengono spostati in "witness data" separati, e il TXID viene calcolato senza includerli — la modifica della firma non può più cambiare l'ID.

---

## Attacco Denial of Service

Se un minatore rifiuta di processare le transazioni di un utente sgradito, quest'ultimo subisce solo un ritardo. La transazione resta nella MemPool finché qualsiasi altro nodo onesto non propone un blocco che la include. Non esiste un meccanismo efficace per censurare permanentemente una transazione in una rete con sufficienti nodi onesti.

---

## Attori del Network

| Tipo di nodo | Wallet | Mining | Blockchain completa | Routing P2P |
|---|---|---|---|---|
| **Reference Client (Bitcoin Core)** | Sì | Sì | Sì | Sì |
| **Full Node** | No | No | Sì | Sì |
| **Solo Miner** | No | Sì | Sì | Sì |
| **Lightweight / SPV Wallet** | Sì | No | No | Sì |

---

## Evoluzione Hardware per il Mining

Il mining consiste nell'eseguire SHA-256 in loop variando il nonce fino a ottenere un hash inferiore al target. Il ciclo centrale è concettualmente semplice:

```
while (1)
    HDR[kNoncePos]++;
    if (SHA256(SHA256(HDR)) < (65535 << 208) / DIFFICULTY)
        return;
```

L'hardware su cui gira questo loop si è evoluto in quattro generazioni, con guadagni di efficienza enormi ad ogni salto:

| Generazione | Periodo | Caratteristiche | Tempo medio per un blocco |
|---|---|---|---|
| **CPU** | 2009 | Elaborazione sequenziale su core generici | ~139.461 anni (2015, singolo PC) |
| **GPU** | 2010+ | Alto parallelismo via OpenCL; overclocking diffuso | ~300 anni (100 GPU) |
| **FPGA** | 2011+ | Schede programmabili via Verilog; ottime per operazioni bitwise | ~25 anni |
| **ASIC** | 2013+ | Chip dedicati esclusivamente a SHA-256 (es. TerraMiner 4: 2 TH/s a $3.500) | Secondi/minuti |

---

## Solo Mining vs Mining Pool

Il **solo mining** segue una distribuzione di Poisson con alta varianza: nel 2014, con 1700 GH/s, l'attesa media per un blocco era oltre 3 anni. Si accumula stress e spese senza entrate garantite.

La deviazione standard è alta per costruzione: se in un mese ci si aspetta di trovare 4 blocchi, la deviazione standard è $\sqrt{4} = 2$. Significa che alcuni mesi se ne trovano 6, altri 2, altri 0. Oltre all'incertezza economica, il solo mining introduce un problema di fiducia: il miner non può verificare in modo indipendente se il proprio hardware funzioni correttamente, né se gli altri miner stiano barando per ottenere una quota sproporzionata dei premi — e in effetti i miner *possono* imbrogliarsi a vicenda. Questo è uno dei motivi principali che ha spinto verso le mining pool.

Le **Mining Pool** aggregano i minatori riducendo la varianza in cambio di entrate più piccole ma costanti. Un *Pool Manager* centrale distribuisce il lavoro, raccoglie le soluzioni e redistribuisce i premi trattenendo una fee. Per verificare che i minatori stiano davvero lavorando, questi inviano **shares**: blocchi "quasi validi" con una difficoltà ridotta rispetto alla rete, che fungono da prova probabilistica del lavoro svolto.

### Metodi di Pagamento

La scelta del metodo di pagamento è il cuore del rapporto economico tra pool e miner: determina chi si fa carico del rischio della varianza e come viene scoraggiato il comportamento opportunistico. Esistono tre schemi principali, con varianti ibride usate dalle pool moderne.

#### Pay Per Share (PPS) e FPPS

Nel modello **PPS** (*Pay Per Share*), il miner viene pagato per ogni share valida inviata, indipendentemente dal fatto che la pool trovi effettivamente un blocco. Il pagamento è deterministico: se la difficoltà della rete è $D$ e la difficoltà delle share è $d$, ogni share vale esattamente $\frac{d}{D} \cdot \text{block\_reward}$. La pool si assume interamente il rischio della varianza — in settimane sfortuna, pagherà i miner anche senza ricavare premi.

**FPPS** (*Fully Pay Per Share*) è la variante estesa: oltre al block reward, include nella quota per share anche le **transaction fees** del blocco (che in PPS puro vengono spesso trattenute dall'operatore). FPPS è quindi più generoso per il miner ma richiede una fee operativa più alta.

> [!warning] Incentivo perverso del PPS
>
> Poiché il miner viene pagato a prescindere, non ha alcun incentivo a trasmettere immediatamente un blocco valido trovato — potrebbe teoricamente scartarlo per massimizzare le proprie share senza contribuire alla catena. Nella pratica questo comportamento è raro perché degenera nella loss di reputazione e nella rimozione dalla pool.

#### Pay Per Last N Shares (PPLNS)

Nel modello **PPLNS**, la ricompensa viene distribuita solo quando la pool trova un blocco, e viene ripartita in proporzione alle share che ciascun miner ha inviato nell'ultima finestra di $N$ share. Il parametro $N$ è scelto dall'operatore: una finestra piccola premia i miner più recenti; una finestra grande diluisce il contributo nel tempo.

L'effetto principale è che il miner si espone alla stessa varianza della pool: se la pool trova molti blocchi in rapida successione, guadagna molto; se è sfortunata, guadagna poco. In compenso, la fee è bassa perché l'operatore non anticipa pagamenti.

> [!tip] PPLNS scoraggia il pool hopping
>
> Il **pool hopping** è la strategia di un miner opportunista che si unisce a una pool all'inizio di un round (quando le share accumulate sono poche e la sua quota relativa è alta) per poi passare a un'altra pool verso la fine. Con PPLNS la finestra scorrente penalizza chi entra tardi o è intermittente: le sue share recenti hanno peso minore rispetto a chi contribuisce stabilmente. Questo rende PPLNS resistente al pool hopping.

#### Pay Proportional

Nel modello **proporzionale**, la ricompensa di ogni blocco trovato viene divisa tra i miner in proporzione alle share inviate *durante quel round* (cioè dall'ultimo blocco trovato dalla pool al blocco corrente). A differenza di PPLNS non c'è finestra scorrevole: si azzera ad ogni blocco.

Questo schema è vulnerabile al pool hopping in modo ancora più diretto: un miner che entra a inizio round (quando poche share sono state accumulate) ha una quota percentuale molto alta. Man mano che il round si allunga, nuovi miner entrano e la quota si diluisce — conviene quindi abbandonare i round lunghi. Per questo motivo il Pay Proportional puro è stato quasi completamente abbandonato in favore di PPLNS.

#### Confronto riassuntivo

| Metodo | Chi porta il rischio varianza | Vulnerabile al pool hopping | Fee tipica | Transaction fees incluse |
|---|---|---|---|---|
| **PPS** | Operatore della pool | No | Alta | No |
| **FPPS** | Operatore della pool | No | Alta | Sì |
| **PPLNS** | Il miner | Parzialmente (finestra scorrevole lo riduce) | Bassa | Dipende dalla pool |
| **Pay Proportional** | Il miner | Sì (molto vulnerabile) | Bassa | Dipende dalla pool |

> [!warning] Mining Pool Decentralizzate (es. P2Pool, dal 2011)
>
> Non richiedono un operatore fidato. I miner costruiscono in parallelo una **sharechain** — una blockchain privata con difficoltà ridotta (~un blocco ogni 30 secondi) — agganciata all'ultimo blocco Bitcoin. Ogni share è scritta sulla sharechain e registra la quota di ricompensa spettante. Quando si trova un blocco Bitcoin valido, i pagamenti vengono processati sulla rete principale tramite *merge mining*. L'auditability totale impedisce truffe interne.

### Top Mining Pool (2025)

| Pool | Fee | Hashrate | Metodo |
|---|---|---|---|
| Foundry USA | 2% PPLNS / 4% PPS | 231,5 EH/s | FPPS |
| BTC.com | 1,38% | 161,44 EH/s | Advanced FPPS |
| Antpool | 0% PPLNS / 4% PPS+ | 30,5 EH/s | PPLNS, PPS+ |
| F2Pool | 2,5% | 25,81 EH/s | PPS+ |
| Binance | 2,5% | 23,86 EH/s | FPPS, PPS+, PPS |
| Poolin | 2,5% | 23,59 EH/s | FPPS |
| ViaBTC | 2% PPLNS / 4% PPS | 20,32 EH/s | PPLNS, PPS |

Le pool sono nate nel 2010 (era GPU) e già nel 2014 raccoglievano il 90% dell'hashrate globale. I protocolli standardizzati odierni facilitano lo spostamento dei miner tra pool diverse.

> [!question] Possibili domande d'esame
>
> - Descrivi il **Double Spending Attack** nella modalità "stealth": quali passi compie l'attaccante e perché la Longest Chain Rule non è sufficiente a proteggersi?
> - Cos'è il **51% Attack**? Quale risorsa deve controllare l'attaccante? È economicamente razionale eseguirlo?
> - Cos'è la **Transaction Malleability**? Perché era possibile in Bitcoin e come è stata corretta con SegWit?
> - Descrivi l'attacco a Mt. Gox: come veniva sfruttata la malleability e perché l'exchange non rilevava la frode?
> - Perché un attacco **Denial of Service** contro un singolo utente non è efficace in Bitcoin?
> - Confronta i quattro tipi di nodo Bitcoin (Reference Client, Full Node, Solo Miner, SPV Wallet) in termini di funzionalità.
> - Descrivi i tre schemi di pagamento delle mining pool: **PPS**, **PPLNS** e **Pay Proportional**. Quali sono i loro incentivi perversi?
> - Cos'è il **pool hopping** e perché il modello Pay Proportional vi è particolarmente vulnerabile? Come PPLNS lo mitiga?
> - Cos'è una **P2Pool** (mining pool decentralizzata)? Come funziona la sharechain e quale problema risolve rispetto alle pool tradizionali?
> - Perché si è passati dal solo mining alle mining pool? Qual è il trade-off tra varianza e dimensione del premio atteso?
