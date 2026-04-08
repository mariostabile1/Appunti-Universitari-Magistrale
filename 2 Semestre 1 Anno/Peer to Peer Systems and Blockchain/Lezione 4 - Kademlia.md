# Kademlia

## Panoramica

Kademlia è un protocollo DHT progettato da Maymounkov e Mazières nel 2002. Il nome viene dal bulgaro: è il nome di una vetta montuosa in Bulgaria, nonché una parola turca per "uomo fortunato". È considerato lo standard de facto per le reti P2P su Internet ed è adottato da Ethereum, IPFS (nelle varianti S/Kademlia e Sloppy Kademlia), BitTorrent (Mainline DHT) ed eMule (rete KAD).

Kademlia presenta caratteristiche che non sono offerte da nessun'altra DHT esistente:
- Le informazioni di routing si diffondono **automaticamente** come effetto collaterale dei lookup, senza messaggi dedicati
- Supporta il **parallel routing** (richieste multiple in parallelo) per ridurre latenza e timeout
- Usa un'unica metrica — lo **XOR** — per calcolare distanze, instradare messaggi e organizzare la routing table
- È **fault tolerant**: se uno o più peer abbandonano la rete, i dati — replicati su più peer — restano recuperabili
- Non richiede motori di database complessi: i dati sono coppie chiave-valore, quindi anche dispositivi IoT con storage limitato possono partecipare

---

## Chord: Riepilogo

In Chord i nodi scelgono un ID quasi uniformemente casuale nell'anello. Gli identificatori sull'anello vengono chiamati **chiavi** (per distinguerli dagli ID dei nodi). Ogni blocco contiguo dell'anello è assegnato a un nodo. Un nodo con identificatore ID mantiene puntatori verso i nodi `successor(ID + 2^i)` per potenze di 2 fino a $2^m$ (finger table), dove $m$ è il numero di bit degli identificatori.

---

## Spazio degli Identificatori

Sia i nodi che i contenuti sono identificati da hash a $M$ bit. Lo spazio di tutti i possibili identificatori è visualizzato come un **trie binario completo**: ogni livello rappresenta un bit, ogni nodo interno ha al più due figli (figlio sinistro → bit 0, figlio destro → bit 1), e ogni foglia rappresenta un identificatore completo.
![[Pasted image 20260407111031.png]]
In pratica, lo spazio degli identificatori è enorme e i nodi che partecipano alla DHT sono molto meno degli identificatori possibili. Il **node tree** è una vista alternativa al trie completo: un albero binario non bilanciato che mostra solo le foglie corrispondenti ai nodi effettivamente presenti nella rete. Una foglia del node tree corrisponde a un **prefisso identificatore** che identifica univocamente quel peer — ovvero il prefisso più corto che non è condiviso da nessun altro peer nell'overlay.

### Mappatura delle Chiavi ai Nodi

Per assegnare una chiave (identificatore di un dato) a un nodo, Kademlia usa il **Lowest Common Ancestor (LCA)**: il punto più profondo dell'albero in cui il percorso della chiave e quello del nodo coincidono prima di biforcarsi. La chiave viene assegnata al nodo con LCA più profondo con essa.

In caso di parità tra due nodi che hanno lo stesso LCA rispetto a una chiave, si spezza il pareggio guardando il bit più significativo in cui i due nodi differiscono (il suo indice è $b$): la chiave viene assegnata al nodo il cui $b$-esimo bit coincide con il $b$-esimo bit della chiave.

> [!example] Tie-breaking
>
> Peer 110 e 111 hanno lo stesso LCA rispetto alla chiave 101. I due peer differiscono al terzo bit. Poiché il terzo bit di 101 è 1, la chiave 101 viene assegnata al nodo 111.

---

## La Metrica XOR

### Derivazione

Per trovare il nodo più vicino a una chiave, si può usare un algoritmo brute-force: si esaminano tutti gli ID dei nodi bit per bit dal più significativo al meno significativo; ad ogni posizione, se almeno un candidato ha lo stesso bit della chiave, si eliminano quelli che differiscono. La complessità è $O(n \cdot m)$.

Un approccio più elegante: si definisce un valore scalare di distanza che penalizza le differenze ai bit più significativi più di tutte le differenze ai bit meno significativi messe insieme. Poiché $2^i > \sum_{j=0}^{i-1} 2^j$, la penalità giusta per una differenza al $i$-esimo bit è $2^i$. Questo è esattamente ciò che fa l'operazione **XOR**.

> [!definition] Distanza XOR
>
> La distanza tra due identificatori $x$ e $y$ è definita come $d(x, y) = x \oplus y$. Un contenuto viene memorizzato sul nodo con distanza XOR minima rispetto alla sua chiave.

> [!warning] XOR ≠ distanza numerica
>
> Due identificatori possono essere vicini numericamente ma lontani secondo la metrica XOR:
> $1000 \oplus 0111 = 1111 = 15$ (distanza XOR massima), mentre la differenza numerica è solo 1.
> La metrica XOR si basa sull'albero binario, non sulla linea numerica.

### Proprietà della Metrica XOR

La metrica XOR soddisfa le proprietà di una metrica formale:

- $d(x, x) = 0$
- $d(x, y) > 0$ se $x \neq y$
- **Simmetria** — $d(x,y) = d(y,x)$. Se $A$ vede $B$ come vicino, anche $B$ vede $A$ come vicino. Questo permette a entrambi di apprendere informazioni di routing dall'altro gratuitamente. In Chord questo non vale: se $x$ riceve una query da $y$, $y$ ha $x$ nella sua finger table, ma $x$ potrebbe non avere $y$ come finger → le informazioni nelle query ricevute non possono arricchire la finger table.
- **Disuguaglianza triangolare** — $d(x,z) \le d(x,y) + d(y,z)$. Andare direttamente da $x$ a $z$ è sempre almeno comodo quanto passare per $y$. Segue dalla transitività: $d(x,y) \oplus d(y,z) = d(x,z)$.
- **Unidirezionalità** — Dato un nodo $x$ e una distanza $\Delta$, esiste un **unico** nodo $y$ tale che $d(x,y) = \Delta$. Esempio: $x = 1001$, $\Delta = 0001$, l'unico punto a distanza $\Delta$ da $x$ è $y = 1000$. Le ricerche per la stessa chiave convergono sempre sullo stesso percorso, permettendo di usare **cache lungo la rotta** per evitare hotspot.

### Relazione tra Prefisso Comune e Distanza

La metrica XOR è legata al prefisso identificatore: più lungo è il prefisso comune tra due nodi, minore è la loro distanza XOR. Due identificatori che condividono un prefisso di lunghezza $p$ e differiscono negli ultimi $i = L - p$ bit hanno distanza XOR:

$$2^{i-1} \le d(x,y) < 2^i$$

> [!example] Subtree distances
>
> $X = 010110$, $Y = 011110$, $X \oplus Y = 001000$, $d(x,y) = 2^3 = 8$ (minima per quel sottalbero).
>
> Due foglie nei due sottoalberi opposti (prefisso comune = 0): $0111 \oplus 1000 = 1111 = 15$ (massima), $0111 \oplus 1111 = 1000 = 8$ (minima).

---

## Routing Table e K-Buckets

### Struttura

> [!definition] K-Bucket
>
> Lista di al massimo $k$ contatti, ognuno memorizzato come tripla `(Node ID, IP address, UDP port)`. I contatti sono ordinati per recency: il meno recente in testa, il più recente in coda. Il bucket $i$ contiene contatti con distanza XOR $d \in [2^{i-1}, 2^i)$ dal nodo proprietario.

Ogni nodo mantiene una routing table di $M$ bucket (uno per ogni sottoalbero dal punto di vista del nodo). I bucket per distanze piccole (lunga prefix comune) tendono ad avere pochi contatti — ci sono pochi nodi in quel vicinato. I bucket per distanze grandi coprono porzioni enormi dello spazio e sono generalmente pieni. Ad ogni passo di routing, la query si avvicina di almeno un bit al target: costo totale $O(\log N)$ — è il **prefix match routing**.

![[Pasted image 20260407111152.png]]
Il parametro $k$ è scelto in modo che la probabilità di $k$ guasti simultanei sia trascurabile.

### Aggiornamento dei K-Buckets

Quando arriva un messaggio da un nodo:

```
if sender già nel bucket:
    sposta sender in coda (più recente)
else if bucket non pieno:
    inserisci sender in coda
else:
    ping al nodo in testa (il meno recente)
    if non risponde:
        rimuovi il nodo in testa, inserisci sender in coda
    else:
        sposta il nodo in testa in coda, scarta sender
```

Questa politica **favorisce i nodi vecchi**: l'analisi di tracce Gnutella mostra che più a lungo un nodo è rimasto online, più è probabile che continui a farlo (*least recently seen eviction*). Ne derivano due vantaggi:
- La routing table è stabile e punta a nodi affidabili
- **Resistenza ai DoS**: un attaccante non può svuotare la routing table inviando messaggi da nodi fasulli, perché i nuovi contatti vengono inseriti solo se i vecchi scompaiono

### Refresh Periodico

I k-bucket vengono aggiornati automaticamente dal traffico dei messaggi. Tuttavia, se un bucket non riceve messaggi per un certo periodo, Kademlia esegue un **refresh** orario: sceglie un ID casuale nel range del bucket e vi effettua una ricerca. Se il nodo con quell'ID risponde, viene inserito nel bucket.

---

## Primitive del Protocollo

Kademlia espone quattro RPC su **UDP**:

| Primitiva | Descrizione |
|---|---|
| `PING` | Verifica se un nodo è online |
| `STORE(key, value)` | Istruisce un nodo a memorizzare una coppia chiave-valore |
| `FIND_NODE(target)` | Restituisce $k$ triple `(ID, IP, UDP port)` per i $k$ nodi più vicini al target. Può attingere da un singolo bucket o da più bucket se il più vicino non è pieno; restituisce sempre $k$ elementi, salvo che il nodo conosca meno di $k$ nodi in totale |
| `FIND_VALUE(key)` | Se il nodo possiede il valore corrispondente a `key`, lo restituisce direttamente. Altrimenti si comporta come `FIND_NODE` e restituisce $k$ triple; spetta al richiedente continuare la ricerca |

---

## Lookup Iterativo e Parallelo

### Tipi di Lookup

Un lookup in Kademlia è una procedura di ricerca distribuita che si avvicina progressivamente a un ID target nello spazio XOR. Il target può essere:
- Un **node ID** (node lookup): usato per mantenere le routing table e trovare il nodo in cui memorizzare un dato
- Un **key ID** (value lookup): usato per recuperare il valore associato a una chiave

### Procedura a 3 step

1. Il nodo iniziatore trova il nodo più vicino al target nella propria routing table (tramite distanza XOR)
2. Invia la query a quel nodo chiedendo nodi più vicini al target
3. Ripete il processo con i nodi più vicini ricevuti

Il lookup si ferma quando: nessuna risposta restituisce nodi più vicini di quelli già noti, oppure il nodo più vicino noto è già stato interrogato. Ad ogni iterazione, la metrica XOR si riduce della metà — il numero di hop è $O(\log N)$.

### Routing Parallelo (α)

Il parametro $\alpha$ controlla la concorrenza: invece di aspettare un nodo alla volta, si interrogano $\alpha$ nodi simultaneamente. Questo riduce la latenza totale e rende il sistema robusto ai nodi lenti o offline.

> [!example] Utilità del parallel routing
>
> Il nodo blu 0011 cerca il nodo rosso 1101. Conosce i nodi verdi 1001 e 1110.
> $d(1101, 1001) = 4$, $d(1101, 1110) = 3$.
> Il nodo 1001 è più distante, ma potrebbe conoscere il target mentre 1110 non lo conosce.
> Con routing parallelo, 0011 invia la query a entrambi simultaneamente.

> [!warning] Il nodo più vicino non è necessariamente il percorso più breve
>
> Dati $x$, $y$, $z$ con $d(y,x) < d(z,x)$: è possibile che $z$ conosca $x$ mentre $y$ non lo conosce. L'unidirezionalità della metrica XOR garantisce che tutti i percorsi convergano verso il target, quindi inviare la query a $\geq 1$ nodi più vicini è sufficiente.

Quando $\alpha = 1$ il lookup è simile a Chord (un passo alla volta); con $\alpha > 1$ Kademlia ha la flessibilità di scegliere tra $k$ nodi a ogni passo.

### Algoritmo Completo

```
k-closest ← α contatti dal k-bucket non vuoto più vicino alla chiave
             (se meno di α, si aggiungono contatti da bucket adiacenti)
closestNode ← il nodo più vicino in k-closest

repeat:
    seleziona α contatti da k-closest non ancora interrogati
    invia FIND_NODE in parallelo e in modo asincrono
    ogni contatto vivo risponde con k nodi
    aggiungi i nuovi nodi a k-closest, aggiorna closestNode
until nessun nodo più vicino di closestNode viene restituito

invia FIND_NODE in parallelo ai k nodi più vicini non ancora interrogati
return k nodi più vicini
```

---

## Join, Store e Leave

### Ingresso nella rete (Join)

1. Il nuovo nodo contatta un **bootstrap node** noto; la routing table iniziale contiene solo la coppia `(nuovo nodo, bootstrap node)` in un unico bucket
2. Esegue `FIND_NODE(proprio ID)` → scopre nodi vicini a sé, riempie i bucket ad alto indice, e si fa conoscere dagli altri nodi
3. Esegue `FIND_NODE(ID casuale)` per bucket sempre più distanti dal proprio, arricchendo progressivamente tutti i bucket

La flessibilità di questa procedura è superiore rispetto a Chord, che richiede passi di join più rigidi.

### Archiviazione di un dato (Store)

1. Si esegue un lookup per trovare i $k$ nodi con ID più vicini alla chiave
2. Si invia `STORE` a tutti i $k$ nodi → il dato è replicato $k$ volte

**Pubblicazione periodica**: i dati sono *soft-state* e devono essere rinfrescati. Il nodo che ha pubblicato un dato deve **ripubblicare periodicamente** per contrastare il churn. Nella versione per file sharing, la ripubblicazione avviene ogni 24 ore. Ottimizzazione: se un nodo riceve un `STORE`, suppone che sia stato inviato anche ai vicini stretti e non ripubblica nell'ora successiva, riducendo i messaggi scambiati.

**Caching lungo il percorso**: i valori vengono **cached** nel primo nodo sul percorso di lookup che non li conosceva. Questo aiuta a distribuire il carico per dati molto popolari.

### Disconnessione (Leave)

La disconnessione non richiede operazioni esplicite: se un nodo non risponde, viene silenziosamente rimosso dai k-bucket degli altri nodi.

---

## Kademlia vs Chord

| Aspetto | Chord | Kademlia |
|---|---|---|
| **Metrica** | Distanza numerica (anello) | XOR |
| **Simmetria** | No — finger table asimmetriche | Sì — apprendimento mutuale gratuito |
| **Routing** | Ricorsivo | Iterativo + parallelo ($\alpha$ nodi) |
| **Lookup** | $O(\log N)$ hop | $O(\log N)$ hop |
| **Aggiornamento routing table** | Messaggi dedicati | Effetto collaterale dei lookup |
| **Località** | Non considerata | RTT memorizzato per ogni contatto; preferisce il contatto con RTT minore |
| **Tolleranza ai guasti** | Limitata | Alta (parallel routing + bucket policy) |

In Chord, se $x$ riceve una query da $y$, $y$ ha $x$ nella sua finger table ma $x$ potrebbe non avere $y$ come finger — le informazioni nelle query ricevute non si possono usare per arricchire la finger table. In Kademlia la simmetria della metrica permette a ogni nodo di arricchire la propria routing table attraverso le query che riceve.

---

## Punti di Forza e Debolezze

> [!abstract] Strengths
>
> - Basso overhead di messaggi di controllo
> - Tolleranza a guasti e disconnessioni dei nodi
> - Selezione di percorsi a bassa latenza (RTT-aware)
> - Limiti di prestazione dimostrabili formalmente

> [!warning] Weaknesses
>
> - Distribuzione non uniforme dei nodi nello spazio degli ID → routing table sbilanciata e routing inefficiente
> - Bilanciamento del carico di memorizzazione non completamente risolto
> - Protocollo originariamente sottospecificato → proliferazione di implementazioni diverse
> - Difficile produrre risultati analitici precisi
> - Risultati di routing non deterministici (tempo, vicinato)

> [!question] Possibili domande d'esame
>
> - Definisci la **metrica XOR** in Kademlia. Perché soddisfa le proprietà di una metrica formale? Quali sono le sue quattro proprietà e perché la **unidirezionalità** è particolarmente importante?
> - Qual è la differenza tra distanza numerica e distanza XOR? Porta un esempio in cui due nodi sono vicini numericamente ma lontani secondo XOR.
> - Cos'è un **k-bucket**? Descrivine la struttura e la politica di aggiornamento. Perché si favoriscono i nodi vecchi?
> - Spiega il meccanismo di **mappatura delle chiavi ai nodi** in Kademlia tramite LCA (Lowest Common Ancestor). Come si gestisce il tie-breaking?
> - Descrivi le quattro primitive RPC di Kademlia (`PING`, `STORE`, `FIND_NODE`, `FIND_VALUE`). Cosa restituisce `FIND_VALUE` se il nodo non ha il valore?
> - Descrivi l'algoritmo di **lookup iterativo e parallelo** in Kademlia. Qual è il ruolo del parametro $\alpha$?
> - Come avviene il **join** in Kademlia? Quali operazioni esegue il nuovo nodo per riempire i propri k-bucket?
> - Perché in Kademlia i dati vengono **ripubblicati periodicamente**? Cosa succederebbe senza questa operazione?
> - Confronta Kademlia e Chord su: metrica, simmetria, routing, aggiornamento della routing table e tolleranza ai guasti.
> - Cos'è il **caching lungo il percorso** in Kademlia? Quale problema risolve e come si correla con l'unidirezionalità della metrica?
