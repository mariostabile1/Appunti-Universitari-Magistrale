# Classificazione degli Overlay: Overlay Non Strutturati

## Dal Centralizzato al P2P

Napster (2001) ha dimostrato che si può servire una quantità di dati paragonabile a Google con molti meno server, spostando storage e trasferimento direttamente sugli utenti. All'epoca Google impiegava circa 15.000 server, Napster circa 100. Il sistema raggiunge 26,4 milioni di utenti nel 2001, con 10 TB di dati (2 milioni di canzoni, in media 220 per utente). I server si occupano solo di localizzare chi possiede la risorsa — la parte meno costosa del servizio. In questo modello ogni utente è un **servent** (server + client): partecipa "pagando" con le proprie risorse fisiche, contenuti o conoscenza.

**Punti di forza di Napster**: sistema informativo globale senza grandi investimenti, decentralizzazione dei costi e dell'amministrazione, nessun collo di bottiglia sulle risorse (storage e trasferimento distribuiti tra gli utenti). **Punti di debolezza**: il server resta un singolo punto di fallimento e di controllo, necessario per gestire l'intero sistema — esattamente questo lo ha reso vulnerabile agli attacchi legali per violazione del copyright, poiché l'analisi dell'indice centralizzato permetteva di risalire ai contenuti scambiati.

**Gnutella** rimuove anche quest'ultimo punto centrale: nessun indice, connessioni dirette tra peer usate per la ricerca (non per il download). Il risultato è un sistema senza infrastruttura né amministrazione, privo di single point of failure. I punti deboli diventano: alto traffico di rete, assenza di ricerca strutturata, e **free riding** (nodi che consumano risorse senza contribuire).

---

## Reti Overlay

> [!definition] Overlay Network
>
> Rete logica costruita sopra la rete fisica sottostante (underlay), tipicamente a livello applicativo sopra TCP/IP. I link dell'overlay sono "tunnel" che attraversano la rete fisica: un singolo collegamento logico può passare per decine di router. Più overlay possono coesistere contemporaneamente sulla stessa rete fisica, ciascuno offrendo il proprio servizio specifico non disponibile nell'underlay. I nodi dell'overlay sono spesso end host che fungono anche da nodi intermedi che inoltrano traffico.

![[Pasted image 20260407110328.png]]
Un protocollo P2P definisce formato e semantica dei messaggi tra peer. I peer sono identificati da ID univoci, generalmente calcolati tramite funzioni hash. I pacchetti P2P, analogamente ai pacchetti IP, sono caratterizzati da un **header** e un **payload**. Il protocollo definisce anche una strategia di routing a livello applicativo dello stack TCP/IP, senza dover modificare i router sottostanti.

## Classificazione degli Overlay

| Tipo | Topologia | Lookup | Garanzie |
|---|---|---|---|
| **Centralizzato** | Server centrale | $O(1)$ | Singolo punto di fallimento |
| **Non Strutturato** | Grafo casuale | $O(N)$ | Nessuna garanzia di trovare la risorsa |
| **SuperPeer (Ibrido)** | Gerarchico | $O(hops_{max})$ | Migliore scalabilità del non strutturato |
| **Strutturato (DHT)** | Topologia controllata | $O(\log N)$ | Lookup garantito, garanzie anche su join e leave |

---

## Overlay Non Strutturati

I peer si connettono arbitrariamente: la topologia forma un grafo casuale (es. Gnutella ≤ 0.4, Bitcoin). La rete è resiliente e facile da mantenere, ma la ricerca è costosa — nel caso peggiore $O(N)$. I falsi negativi sono possibili: la risorsa cercata potrebbe esistere ma non essere raggiunta entro il TTL.

### Bootstrapping e Discovery

Un nuovo nodo non conosce nessuno. Il **bootstrapping** avviene tramite due meccanismi complementari: server DNS noti che memorizzano gli indirizzi IP di un insieme di peer stabili (eseguendo script che interagiscono con i peer e aggiornano automaticamente la lista), oppure una **cache interna** in cui ogni client memorizza gli IP dei peer contattati nelle sessioni correnti e precedenti, aggiornata dinamicamente tramite *gossiping* con i vicini.
![[Pasted image 20260407110433.png]]
Il processo di partecipazione alla rete si articola in tre fasi:

- **Step 0** — join: il nodo si connette a uno o più peer noti tramite bootstrap.
- **Step 1** — peer discovery: il nodo invia messaggi **Ping** per annunciare la propria presenza; i peer rispondono con **Pong** contenenti le proprie informazioni e inoltrano il Ping ai vicini.
- **Step 2** — searching: il nodo invia una query ai vicini (es. *"do you have any content that matches the string 'Back to Black'?"*); i peer che hanno corrispondenze rispondono, gli altri inoltrano la query per TTL salti.
- **Step 3** — downloading: il trasferimento avviene via connessioni HTTP dirette usando il metodo GET.

### Flooding

L'algoritmo di ricerca base è il **Flooding**: la query viene inviata a tutti i vicini, che la inoltrano a loro volta. Per evitare loop infiniti si usa un **TTL** decrementato ad ogni salto; quando raggiunge zero il messaggio viene scartato. I duplicati si evitano con un ID univoco per ogni messaggio. Le risposte viaggiano all'indietro attraverso le connessioni non transitorie, tramite **Backward Routing**.

```
FloodForward(Query q, Source p):
  if q.id ∈ oldIdsQ: return          // duplicato, scarta
  oldIdsQ ← oldIdsQ ∪ {q.id}
  q.TTL ← q.TTL - 1
  if q.TTL == 0: return              // scaduto, scarta
  foreach s ∈ Neighbors:
    if s ≠ p: send(s, q)             // inoltra a tutti tranne il mittente
```

Il flooding equivale a una **BFS limitata dal TTL**: trova il massimo numero di risultati nel raggio TTL centrato sul nodo sorgente. Garantisce alta resilienza ma genera molto traffico e molti duplicati, scala male, e **non garantisce il ritrovamento della risorsa** (false negative). È usato anche in **Bitcoin** non solo per la ricerca, ma per **propagare le transazioni** nella rete P2P sottostante — dove mantenere la consistenza è la sfida principale.

### Tecniche di Ricerca Avanzate

Le alternative al flooding puro si dividono in due categorie: approcci **BFS-based** (iterative deepening/expanding ring, k-walker random walk, two-level k-walker, directed BFS, modified random BFS) e approcci **DFS-based** (local indices, routing indices, attenuated bloom filter).

**Expanding Ring (Iterative Deepening)** — BFS con TTL crescente. Si parte da un TTL basso (opzionalmente su un sottoinsieme casuale dei vicini); se la ricerca fallisce si ripete incrementandolo fino alla terminazione (risorsa trovata o profondità massima raggiunta). Per non riprocessare gli stessi nodi, quelli al bordo dell'anello *i-esimo* **congelano** la query per un periodo $> W$ (intervallo tra due query successive). Quando la sorgente invia un messaggio `resend` con lo stesso ID e un `NewTTL` maggiore, i nodi interni all'anello precedente lo inoltrano semplicemente; i nodi al bordo lo **scongelano** e inviano la query ai propri vicini con $TTL = NewTTL - PreviousTTL$.

**Random Walk e k-Walker** — Il random walk è modellato come una **catena di Markov** (*drunkard's walk*): il sistema è privo di memoria, la distribuzione di probabilità dello stato futuro dipende solo dallo stato presente, senza direzioni privilegiate. In pratica, il nodo invia la query a *un solo* vicino scelto a caso, che fa lo stesso; il TTL si decrementa ad ogni salto. Se la ricerca fallisce (timeout), la sorgente può riemettere la query lungo un altro cammino casuale. Riduce drasticamente il traffico ma aumenta la latenza.

La variante **k-walker** parallelizza inviando $k$ copie indipendenti, ciascuna che prende il proprio cammino casuale. La terminazione può avvenire per TTL oppure con un **checking method**: i walker periodicamente verificano con la sorgente se la condizione di stop è stata soddisfatta. Si può anche **bilanciare verso nodi ad alto grado** modulando la probabilità di scelta del vicino. Vantaggi e svantaggi: il limite superiore al traffico è $k \times TTL$ messaggi; $k$ walker dopo $T$ passi raggiungono circa lo stesso numero di nodi di 1 walker dopo $k \times T$ passi, riducendo il ritardo di un fattore $k$. Le prestazioni dipendono da $k$, $T$ e dalla popolarità $p$ della risorsa: $k$ e $T$ bassi producono alto ritardo e bassa probabilità di successo; $k$ e $T$ alti producono alto overhead. Una soluzione è impostare i parametri adattativamente in funzione della popolarità.

**Directed BFS e Routing Indices** — I vicini vengono scelti non a caso ma selezionando i "migliori". Un vicino è considerato buono se: ha prodotto risultati in passato, ha bassa latenza, ha il minor numero di hop per i risultati (segno che ha buoni vicini), ed è stabile. Dopo il primo hop, la ricerca può proseguire come un normale BFS.

I **Routing Indices (RI)** formalizzano questa selezione: ogni peer mantiene una struttura dati che, data una query, restituisce la lista dei vicini ordinata per "bontà". Ogni peer ha un indice locale per i propri documenti e un RI che stima quanti documenti sono disponibili per ciascun percorso e per ciascun argomento. Esempio: per il nodo A, tramite il vicino B e i suoi discendenti sono disponibili 100 documenti — 20 nella categoria Database, 10 in Theory, 30 in Languages. Questo permette di inviare la query solo ai vicini più rilevanti per quella specifica ricerca.

---

## Overlay Strutturati (DHT)

Negli overlay **strutturati** la scelta dei vicini segue criteri precisi, generando una topologia controllata. L'obiettivo è garantire scalabilità: il lookup è **key-based** con complessità $O(\log N)$, e le garanzie valgono anche per le operazioni di join e leave dei peer. Esempi: **CAN**, **Chord**, **Pastry**, **Kademlia**.

---

## Overlay Ibridi (SuperPeer)

> [!definition] SuperPeer
>
> Nodo con maggiore capacità (banda, CPU, disponibilità) che funge da hub locale. I peer normali si connettono ai SuperPeer e depositano l'indice delle proprie risorse. Il flooding per la ricerca avviene solo tra SuperPeer; il trasferimento dei file resta diretto tra peer.

I SuperPeer vengono scelti autonomamente dal sistema in base alle capacità (storage, banda) e alla disponibilità (tempo di connessione), definendo dinamicamente un livello gerarchico nella rete. Periodicamente si scambiano informazioni sulle risorse dei peer collegati e si fanno carico del carico dei nodi più lenti. I peer ordinari caricano la descrizione delle proprie risorse sul SuperPeer, lo interrogano per le query, e partecipano direttamente al trasferimento delle risorse.

Il vantaggio è che il traffico di ricerca è contenuto alla rete dei SuperPeer, migliorando la scalabilità rispetto al non strutturato puro — a scapito di una minore resistenza al churn dei SuperPeer stessi.

> [!note] Differenza tra implementazioni
>
> In **Gnutella v0.6** gli *ultrapeers* sono **auto-promossi** dai nodi stessi in base alle proprie capacità. In **Kazaa**, **Skype** (per il relay) e **eDonkey** (pre-Kad) gli ultrapeers sono invece **staticamente definiti**.

---

## Auto-Organizzazione

Le reti non strutturate mostrano proprietà emergenti simili a fenomeni fisici e biologici: dall'interazione locale dei peer emerge spontaneamente una struttura globale, senza alcun coordinamento centrale. In fisica, il **fenomeno di Bénard** mostra come una sostanza magnetica riscaldata da un lato e raffreddata dall'altro formi spontaneamente strutture regolari. In biologia, le colonie di insetti come le termiti producono strutture complesse senza alcun coordinamento centrale. In Gnutella emerge spontaneamente un **backbone** — una rete di nodi con alto grado di connessione (simili a server) che organizzano il traffico della rete, senza che nessuno lo abbia pianificato.

> [!question] Possibili domande d'esame
>
> - Confronta le quattro categorie di overlay (centralizzato, non strutturato, superpeer, strutturato/DHT) in termini di complessità del lookup, garanzie e robustezza.
> - Cos'è una **overlay network**? In che relazione si trova con la rete fisica (underlay)?
> - Descrivi il meccanismo di **bootstrapping** in un overlay non strutturato: quali sono i due approcci complementari?
> - Spiega l'algoritmo di **flooding** per la ricerca in reti non strutturate. Quali sono le sue proprietà (complessità, falsi negativi, robustezza)?
> - Cos'è l'**Expanding Ring** (iterative deepening)? Come funziona il meccanismo di "congelamento" dei nodi al bordo dell'anello?
> - Descrivi il **k-walker random walk**: come funziona, quali vantaggi offre rispetto al flooding puro e da quali parametri dipendono le sue prestazioni?
> - Cos'è un **SuperPeer**? Come viene scelto e quali funzioni svolge? Qual è la differenza tra l'approccio di Gnutella v0.6 e quello di Kazaa?
> - Perché la ricerca basata su flooding scala male rispetto alle DHT? Quali sono le complessità nei due casi?
> - Spiega il concetto di **Routing Indices**: cosa rappresentano e come aiutano a migliorare la ricerca in overlay non strutturati (Directed BFS)?
> - Cosa si intende per **auto-organizzazione** nelle reti P2P? Porta un esempio dal mondo fisico e uno da Gnutella.
