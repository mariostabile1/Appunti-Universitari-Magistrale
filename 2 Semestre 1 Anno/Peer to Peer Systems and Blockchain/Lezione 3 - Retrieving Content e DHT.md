# Retrieving Content e DHT

## Il Problema del Recupero Distribuito

Il problema fondamentale delle reti P2P è: il nodo A possiede un contenuto I, il nodo B lo vuole ma non ne conosce la posizione. Come decidere dove memorizzare I e come trovarlo senza un server centralizzato? Qualunque soluzione deve soddisfare due requisiti: **scalabilità** (overhead di comunicazione e memoria in funzione del numero di nodi N) e **adattabilità** (tolleranza ai guasti e al churn continuo).

## Searching vs Addressing

In un sistema P2P puro, il recupero dei contenuti segue due paradigmi opposti.

Il **searching** guida la ricerca tramite attributi del contenuto, come un motore di ricerca. Permette query complesse senza strutture ausiliarie, ma scala male: in reti non strutturate con flooding, il costo nel caso peggiore è $O(N^2)$ perché ogni nodo contatta tutti i propri vicini (ottimizzabile a $O(N)$ con TTL e ID per evitare cicli).

L'**addressing** assegna un ID univoco a ogni contenuto — tipicamente il suo hash — e usa quella chiave per recuperarlo. È il fondamento delle **Distributed Hash Tables (DHT)**, che offrono garanzie teoriche certe: overhead di comunicazione $O(\log N)$ e memoria per nodo $O(\log N)$. Il trade-off è la rinuncia alle query complesse e il costo del mantenimento della struttura di indirizzamento.

> [!tip] Perché le DHT
>
> Le DHT sono il compromesso ottimale tra server centralizzato (lookup $O(1)$, spazio $O(N)$, ma single point of failure) e overlay non strutturato (spazio $O(1)$, robusto, ma lookup $O(N^2)$).

---

## Funzioni Hash

Una **funzione hash** mappa un dato di lunghezza arbitraria in un valore di lunghezza fissa (tipicamente un intero). L'insieme degli input è più grande dell'insieme degli output, quindi le **collisioni** sono inevitabili. In una **hash table**, la chiave viene hashata per trovare direttamente il bucket; ogni bucket contiene in media $\frac{\#items}{\#buckets}$ elementi.

Le **funzioni hash crittografiche** devono soddisfare proprietà aggiuntive di sicurezza. Il caso d'uso principale è **SHA** (Secure Hash Algorithm):

- Input di lunghezza variabile → output di lunghezza fissa
- Una piccola variazione nell'input produce output completamente diversi (*effetto valanga*)
- La funzione è deterministica
- SHA-1 produce un digest di 40 cifre esadecimali = 160 bit → $2^{160}$ valori possibili
- Famiglie: SHA-1, SHA-224, SHA-256, SHA-384, SHA-512 (le ultime quattro = **SHA-2**, il suffisso indica la lunghezza del digest)
- Ethereum usa **Keccak** (variante SHA-3)

> [!example] Output SHA-1 in Java
>
> `SHA1("") = DA39A3EE5E6B4B0D3255BFEF95601890AFD80709`
> `SHA1("abc") = A9993E364706816ABA3E25717850C26C9CD0D89D`
> `SHA1("abd") = CB4CC28DF0FDBE0ECF9D9662E294B118092A5735`
>
> "abc" e "abd" differiscono di un solo carattere ma producono hash completamente diversi.

Le stesse funzioni SHA vengono usate come mattone base per il **consistent hashing** delle DHT e per i **puzzle crittografici** del Proof of Work di Bitcoin.

---

## Da Memcached alle DHT

**Memcached** è un sistema di caching distribuito per il web: mantiene un pool di server che forniscono accesso rapido alle informazioni, riducendo il carico sul database (accesso al DB solo in caso di cache miss). L'idea è distribuire una hash table su più server per superare i limiti di memoria di una singola macchina.

Il meccanismo di base: l'hash dell'URL di una risorsa determina in quale server di cache è memorizzata; ogni macchina può calcolare localmente quale cache contiene la risorsa cercata, senza comunicazione tra le cache. Questo schema viene esteso alle DHT per i sistemi P2P — ma sorge immediatamente il **problema del rehashing**.

---

## Il Problema del Rehashing

Le hash table distribuite classiche calcolano il nodo target come $h(key) \bmod N$, dove $N$ è il numero di server. Questo funziona bene con un numero fisso di nodi, ma nei sistemi P2P il **churn** è continuo.

Quando $N$ cambia, la quasi totalità delle chiavi non soddisfa più $h(key) \bmod N = h(key) \bmod (N+1)$: tutti gli oggetti devono essere riassegnati, saturando la rete.

> [!example] Entità del problema
>
> Con 4 server di caching che memorizzano risorse tramite `SHA(URL) mod 4`, se aggiungiamo 2 server (tot. 6), le uniche URL che rimangono nello stesso server sono quelle in cui `SHA(URL) mod 4 == SHA(URL) mod 6`. Con 10 bucket e 1000 chiavi, circa il **99% delle chiavi deve essere rimappato** — un traffico enorme.

---

## Consistent Hashing

> [!definition] Consistent Hashing
>
> Tecnica di hashing in cui sia i contenuti che i nodi vengono mappati nello **stesso spazio di indirizzamento**, visualizzato come un **anello circolare** di dimensione $2^M$. Ogni nodo gestisce un **intervallo contiguo** di chiavi dell'anello, non un insieme sparso. Lo schema non dipende direttamente dal numero di server: aggiungere o rimuovere nodi richiede di spostare solo una minoranza di elementi.

### Costruzione dell'Anello

La costruzione di una DHT basata su consistent hashing segue tre passi:

1. **Spazio di chiavi comune**: si definisce un identifier space condiviso tra nodi e valori, tipicamente $\{0, 1, \ldots, 2^M - 1\}$ organizzato come anello modulo $2^M$.
2. **Connessione dei nodi**: ogni nodo è collegato a un numero piccolo e limitato di vicini in modo tale che il massimo numero di hop sia limitato (diverse topologie possibili: anello con chord, albero, ecc.).
3. **Assegnazione dei dati**: sia i nodi che i dati vengono mappati dalla stessa funzione hash nello stesso spazio; si definisce una relazione tra hash dei contenuti e hash dei nodi.

### Esempio: Anello con N=16

Spazio $\{0, \ldots, 15\}$, cinque nodi con: $H(a)=6$, $H(b)=5$, $H(c)=0$, $H(d)=11$, $H(e)=2$.

Il **successore** `succ(x)` è il primo nodo nell'anello con ID $\geq x$ procedendo in senso orario. Esempi:
- `Succ(12) = 0` (wrap-around)
- `Succ(1) = 2`
- `Succ(6) = 6`

Ogni nodo punta al proprio successore: $0 \to 2 \to 5 \to 6 \to 11 \to 0$.
![[Pasted image 20260407110807.png]]
Per archiviare i dati si usa la stessa funzione hash: `<key, value>` dove ad es. `key="crown"`, `value=JPEG`. Ogni dato viene archiviato nel suo nodo successore.

> [!example] Peer leave
>
> Se il nodo 11 si disconnette, solo le chiavi che gli erano assegnate devono essere rimappate al suo successore (il nodo 0). Il resto dell'anello è invariato.

### Proprietà del Consistent Hashing

Quando la tabella viene ridimensionata, in media solo $K/n$ chiavi devono essere rimappate (dove $K$ = numero totale di chiavi, $n$ = numero di server) — purché la funzione hash sia uniforme.

- **Rimozione di un nodo**: solo le chiavi associate a quel nodo vengono riassegnate al suo successore.
- **Aggiunta di un nodo**: le chiavi tra il nuovo nodo e il nodo precedente nell'anello vengono riassegnate al nuovo nodo; le altre rimangono invariate.

### Node Leave e Node Failure

Una **disconnessione volontaria** richiede: suddividere l'intervallo di indirizzi tra i nodi vicini, copiare le coppie chiave/valore ai nodi corrispondenti, e rimuovere il nodo dalle routing table degli altri.

Un **guasto improvviso** è più problematico: tutti i dati memorizzati sul nodo vanno persi se non sono replicati. Le soluzioni sono:
- **Replicazione dei dati** su più nodi per ridondanza
- **Refresh periodico** delle informazioni
- **Percorsi di routing alternativi**: probing periodico dei vicini per rilevarne l'attività; quando si rileva un guasto, si aggiornano le routing table.

---

## Data Lookup e Routing

### Lookup Sequenziale (inefficiente)

Seguendo solo il puntatore al successore diretto, un lookup richiede di attraversare l'anello sequenzialmente: $O(N)$ nel caso peggiore. Il lookup è implementato come algoritmo distribuito tramite **RPC** (Remote Procedure Calls): `n.foo()` indica una RPC verso il nodo n.

> [!example] Lookup "Crown"
>
> Nodo 2 vuole trovare "Crown". Calcola $H(\text{"Crown"}) = 9$. Segue i puntatori: $2 \to 5 \to 6 \to 11$. Il nodo 11 ha la chiave 9 nel proprio intervallo (9 ≤ 11) e restituisce il valore al nodo iniziatore.

### Finger Table (Chord)

Chord risolve l'inefficienza con la **Finger Table**: ogni nodo mantiene $M$ puntatori verso nodi distanti a salti esponenziali:

$$
\text{finger}[i] = \text{succ}(n + 2^{i-1}) \quad \text{per } i = 1, \ldots, M
$$

La tabella ha dimensione $M$, dove $N = 2^M$. Ogni nodo n conosce: $\text{succ}(n+1)$, $\text{succ}(n+2)$, $\text{succ}(n+4)$, $\text{succ}(n+8)$, ..., $\text{succ}(n+2^{M-1})$.

Ad ogni passo dell'algoritmo di lookup, il nodo che riceve la query la inoltra al finger più vicino alla chiave cercata senza superarla. Questo dimezza l'intervallo residuo a ogni salto, portando il costo a $O(\log N)$ hop.

> [!example] Scala del miglioramento
>
> Con $N = 10^6$ nodi: routing sequenziale richiede fino a 500.000 hop; con la finger table bastano $\log_2(10^6) \approx 20$ hop.

**Chord** è il DHT di riferimento costruito su questi principi: sviluppato nel 2001 da ricercatori del MIT e dell'Università della California (Ion Stoica, Robert Morris, David Karger et al.), pubblicato su IEEE/ACM Transactions on Networking.

> [!note] Varianti di DHT
>
> Diverse proposte "consistent-hashing compliant" differiscono nel modo in cui i dati vengono associati ai peer e nell'operatore usato per trovare il peer responsabile di una chiave (`FindPeer` o `FindSuccessor`). Esempi: CAN, Chord, Pastry, Tapestry, Kademlia.

---

## Location Addressing vs Content Addressing

Il **location addressing** è il metodo classico di Internet: un link HTTP punta a una specifica posizione su uno o più server. Chi controlla quella posizione controlla il contenuto. Anche se migliaia di persone hanno una copia di un dato, HTTP punta a una sola posizione — il web tradizionale ci costringe a fingere che i dati esistano in un unico posto.

Il **content addressing** rovescia il paradigma: il contenuto viene identificato dalla propria "impronta" crittografica (il suo hash) anziché dalla sua posizione. Avere l'hash di un contenuto permette di ottenerlo da chiunque ne abbia una copia. L'hash non cambia, quindi i link restano sempre validi indipendentemente da dove il contenuto viene recuperato, da chi lo ha aggiunto, e da quando è stato aggiunto.

Questo approccio è alla base di **IPFS** (InterPlanetary File System), la rete P2P che implementa Web3, il web distribuito. I dati non sono più delegati a server centrali (Facebook, Instagram, Dropbox), ma memorizzati in un ambiente P2P distribuito.

---

## API della DHT

L'API esposta da una DHT è volutamente minimale:

- `PUT(key, value)` — inserisce un valore
- `GET(key)` → value — recupera un valore

Non esiste generalmente una funzione per spostare le chiavi. Il valore associato a una chiave può essere un file, un indirizzo IP, o qualsiasi altro dato, a seconda dell'applicazione.

### Applicazioni delle DHT

Le DHT offrono un servizio generico di memorizzazione e indicizzazione distribuita. Applicazioni concrete:
- **IPFS** (Internet Planetary File System)
- **BitTorrent**: memorizzazione dei riferimenti ai peer in uno swarm
- **Ethereum**: memorizzazione dei riferimenti ai peer nella rete P2P
- Supporto a servizi di livello superiore

### Virtual Servers e Load Balancing

Il load imbalance in una DHT ha tre cause possibili:
1. Un nodo gestisce una porzione di spazio di indirizzi più grande degli altri (risolvibile con una funzione hash uniforme)
2. Lo spazio è distribuito uniformemente, ma il nodo gestisce molta più *quantità* di dati
3. Lo spazio è distribuito uniformemente, ma il nodo gestisce molte più *query* (i dati assegnatigli sono molto popolari)

Il load imbalance produce: minore robustezza del sistema, minore scalabilità, e garanzie $O(\log N)$ non più rispettate. La soluzione è usare **virtual server**: ogni nodo fisico mantiene più identità sull'anello, distribuendo la responsabilità delle chiavi in modo più uniforme.

---

## Sfide Aperte delle DHT

> [!warning] DHT Challenges
>
> - **Hotspot**: distribuire le responsabilità in modo uniforme per evitare che alcuni nodi siano molto più carichi di altri
> - **Churn**: redistribuire le responsabilità quando i nodi si connettono o disconnettono
> - **Trade-off**: dimensione delle routing table vs traffico nell'overlay vs stretch rispetto all'underlay (i percorsi logici possono essere molto più lunghi di quelli fisici ottimali)

---

## Confronto tra Architetture

| Approccio | Memoria per nodo | Overhead comunicazione | Query complesse | Falsi negativi | Robustezza |
|---|---|---|---|---|---|
| **Server Centralizzato** | $O(N)$ | $O(1)$ | Sì | No | No (SPOF) |
| **P2P Non Strutturato** | $O(1)$ | $O(N^2)$ | Sì | Sì | Sì |
| **DHT** | $O(\log N)$ | $O(\log N)$ | No | No | Sì |

---

## Proprietà delle DHT

> [!abstract] Sintesi
>
> Le DHT sono sistemi auto-organizzanti, semplici ed efficienti. Il routing è basato su chiave; le chiavi sono distribuite uniformemente tra i nodi → bottleneck avoidance, inserimento incrementale, fault tolerance. I termini "Structured Peer-to-Peer" e "DHT" sono spesso usati come sinonimi.

> [!question] Possibili domande d'esame
>
> - Qual è la differenza tra **searching** e **addressing** nel recupero distribuito dei contenuti? Quali sono le complessità nei due approcci?
> - Perché il **consistent hashing** risolve il problema del rehashing? Quante chiavi devono essere rimappate in media quando si aggiunge o rimuove un nodo?
> - Descrivi la costruzione di un anello di consistent hashing con $N=16$ e cinque nodi. Come si calcola il successore di una chiave?
> - Spiega il meccanismo della **Finger Table** di Chord: come è costruita, come viene usata nel lookup e quale complessità garantisce?
> - Qual è la differenza tra **location addressing** e **content addressing**? Quali sistemi reali adottano il content addressing?
> - Cosa sono i **virtual server** nelle DHT e perché vengono usati? Quali dei tre tipi di load imbalance risolvono?
> - Descrivi le tre cause di load imbalance in una DHT e come possono essere mitigate.
> - Confronta server centralizzato, overlay non strutturato e DHT in termini di memoria per nodo, overhead di comunicazione, query complesse e robustezza.
> - Spiega cosa succede in un **node leave** volontario e in un **node failure** improvviso in una DHT. Come si garantisce la disponibilità dei dati?
> - Quali sono le sfide aperte delle DHT (hotspot, churn, trade-off)? Come si correlano tra loro?
