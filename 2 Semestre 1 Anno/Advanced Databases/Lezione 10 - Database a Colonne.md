---
tags:
  - università/advanced-databases
  - column-store
  - compressione
  - OLAP
data: 2024-01-01
lezione: "10 - Database a Colonne"
professore: ""
---

# Database a Colonne

I **sistemi orientati alle colonne** (*column-stores*) sono un tipo di DBMS che memorizza i dati per colonna invece che per riga. Memorizzando separatamente ogni colonna su disco, le query possono leggere solo gli attributi necessari senza dover leggere intere righe e poi scartare ciò che non serve. Questo è particolarmente vantaggioso anche perché i piani fisici possono essere eseguiti **un blocco alla volta** (*vectorized execution*) invece che una tupla alla volta, caricando più record contemporaneamente per ogni operatore. Quando si considera la compressione, le colonne sono anche più facilmente comprimibili delle righe.

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 10.1]**: Esempio di column-store — confronto tra memorizzazione per riga e per colonna degli stessi dati.

I column-store hanno guadagnato popolarità per ragioni legate a cambiamenti sia nelle applicazioni che nella tecnologia:

- La maggior parte dei task è diventata **analitica** invece che transazionale: i sistemi OLAP richiedono letture efficienti di grandi quantità di dati;
- Cache e RAM migliorano più velocemente delle prestazioni dei dischi: il focus è ridurre gli accessi al disco e tenere più informazioni in memoria principale;
- Il gap tra transfer time e seek time è aumentato: i dati vengono spesso letti sequenzialmente;
- Le CPU moderne fanno **instruction pipelining** e supportano il parallelismo **SIMD** (*Single Instruction Multiple Data*).

> [!tip] Trade-off di base dei column-store
>
> **Caso sfavorevole**: se dobbiamo leggere un record di 5 attributi, con i row-store leggiamo solo la pagina che lo contiene, con i column-store dobbiamo leggere 5 pagine.
>
> **Caso favorevole**: se dobbiamo leggere 1000 tuple consecutive di 20 attributi ma ne interessano solo 5, con un row-store leggiamo 20 pagine, con un column-store solo 5.
>
> In generale, i column-store sono vantaggiosi per sistemi in cui le letture coinvolgono molte tuple (>1000) su un insieme ristretto di attributi.

Le sfide principali dei column-store sono: la **ricostruzione delle tuple** (che richiede un join) e l'**inserimento di tuple** (che richiede molte operazioni I/O per aggiornare tutte le colonne), entrambi problemi amplificati dalla compressione.

---

## Memorizzazione delle Colonne

Esistono due scelte per memorizzare le colonne:

**In ordine per RID**: le tuple possono essere ricostruite con un algoritmo simile al sort-merge, scansionando le colonne in parallelo. Non è necessario memorizzare esplicitamente i RID, poiché possono essere inferiti dalla posizione.

**In ordine per valore**: le colonne sono mantenute ordinate per valore delle tuple. Estremamente conveniente per la compressione tramite Run Length Encoding, e permette ricerche per intervallo efficienti. Richiede di memorizzare esplicitamente i RID. La soluzione base è simile a un indice invertito, ma questo nega la possibilità di comprimere tramite RLE.

In alternativa all'indice invertito, si possono usare due strutture separate:
1. Una lista di tutti i valori, ognuno seguito dal numero totale di record con quel valore (compressa tramite RLE);
2. Un **join index**, che specifica la corrispondenza tra la prima struttura dati e una colonna ordinata per RID.

> [!abstract] Sintesi
>
> La struttura a indice invertito è migliore per ricostruire le tuple originali; il secondo metodo è migliore se i dati sono mantenuti come colonne separate.

### C-Store Projections

La soluzione usata da **C-Store** (un primo DBMS orientato alle colonne) memorizza ogni colonna in un file separato, usando un metodo di compressione specifico per colonna, ordinando i valori secondo un attributo della tabella. Gruppi di colonne ordinate sullo stesso attributo sono chiamati **proiezioni** (*projections*). Se ci sono insiemi di colonne spesso richieste insieme, possono essere raggruppate in una proiezione e mantenute ordinate su quell'attributo.

Proiezioni diverse possono avere colonne sovrapposte. Se ce ne sono molte, molte query troveranno la loro proiezione ottimale, ma gli aggiornamenti saranno più lenti. Per ricostruire l'intera tabella, si esegue un join tra le proiezioni usando join-index, oppure si definisce una grande proiezione che include tutte le colonne.

---

## Compressione

Diverse tecniche di compressione sono state studiate per i column-store:

**Run-length Encoding (RLE)** — Comprime sequenze dello stesso valore in una colonna in una rappresentazione singola compatta. Ogni sequenza è sostituita con una tripla: (valore, posizione iniziale, lunghezza della sequenza).

**Bit-vector Encoding** — Rappresenta la colonna con un insieme di vettori di bit, uno per ogni valore distinto. L'$i$-esimo bit è 1 se la $i$-esima posizione contiene quel valore. Utile solo se il numero di valori distinti è basso.

**Dictionary Encoding** — Mappa ogni valore nella colonna a un intero diverso, e rappresenta la colonna come la sequenza di quegli interi. Usato per tipi di dato che richiedono molti bit (stringhe).

**Difference Encoding** — Quando i valori di una colonna rappresentano una funzione continua, si memorizza solo il primo valore e poi le differenze successive.

**Frame of Reference** — Simile al Difference Encoding, ma usato quando i dati sono memorizzati in più pagine. Ogni pagina è compressa come il primo valore seguito da tutte le differenze.

**Frequency Partitioning** — Valori simili vengono messi nella stessa pagina; ogni pagina è compressa tramite bit-vector o dictionary encoding.

### Operare su Dati Compressi

Con la compressione RLE si può operare senza decompressione, purché ci siano informazioni sulla posizione iniziale delle sequenze. Con dictionary encoding si possono fare range query. Con bit-vector encoding si possono eseguire operazioni bit su insiemi.

Per altri algoritmi o operazioni non incluse sopra, le tuple devono essere ricostruite tramite **materializzazione anticipata** o **tardiva**:

- **Early materialization** (*materializzazione anticipata*): ricostruisce la parte di tabella su cui la query opera scansionando e unendo le colonne necessarie, poi usa l'algebra relazionale standard;

- **Late materialization** (*materializzazione tardiva*): usa l'**algebra delle colonne** per operare su una colonna alla volta, senza ricostruire la tabella. Ogni colonna ha due elementi: un insieme di object ID e una stringa o intero. Le operazioni definite per la late materialization sono:

  - `select(bat[H,T]AB, bool f(...)) : bat[H,nil]`  — seleziona le righe che soddisfano il predicato
  - `join(bat[T1,T2]AB, bat[T2,T3]CD, bool f(...)) : bat[T1,T3]`  — join tra due colonne
  - `reconstruct(bat[H,nil]AN, bat[H,T]AB) : bat[H,T]`  — ricostruzione
  - `reverse(bat[H,T]AB) : bat[T,H]`  — inversione degli ID
  - `group(bat[oID,T]AB) : bat[oID,oID]`  — raggruppamento
  - `sum(bat[oID,int]AB) : bat[oID,int]`  — aggregazione

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 10.2]**: Esempio di late materialization — mostra il flusso di operazioni su colonne separate prima della ricostruzione finale.

---

## Operazioni sulle Colonne

### Column Join

Poiché le colonne evitano l'uso di indici, gli algoritmi usati per unirle sono HashJoin, SortMerge o join in memoria principale (le colonne spesso sono abbastanza piccole da entrare in memoria). L'output del join è un insieme di coppie di posizioni nelle relazioni di input per cui il predicato è soddisfatto. Queste coppie possono essere memorizzate come **join indexes**, che sono spesso molto più piccoli delle colonne originali.

Solo le posizioni della relazione esterna saranno ordinate, mentre quelle interne non lo saranno, richiedendo molti salti nella memoria. Una soluzione è il **Jive join**, un algoritmo che:
- Opera un operatore alla volta;
- Fa una singola lettura delle relazioni di input;
- Legge solo i blocchi dove è presente almeno una tupla nel join index;
- Restituisce il risultato in forma di colonna (non ordinata).

> [!example] Esempio di Jive Join
>
> Date due tabelle:
>
> ```
> R: 1 Johnson    S: 1 Smith
>    2 Jones         2 Johnson
>    3 Johnson       3 Williams
>    4 Doe           4 Jones
>    5 Smith
> ```
>
> Il join produce il join index `[(1,2), (2,4), (3,2), (5,1)]`. Le posizioni esterne (di R) sono già ordinate. Si aggiunge una colonna con interi crescenti e si ordina per le posizioni di S, poi si estrae la colonna di S, e infine si riordina secondo la colonna aggiunta — ottenendo le tuple join in ordine originale, con lettura sequenziale di entrambe le colonne.

Il costo dell'ordinamento è di $2k$ buffer per creare $2k$ file con il join index e il join index temporaneo ordinato.

### Column GroupBy e Aggregazione

Il GroupBy è tipicamente hash-based, a meno che l'input sia già ordinato secondo gli attributi di raggruppamento. L'aggregazione sfrutta il layout a colonne in modo molto efficiente: lavora solo sulla colonna rilevante con cicli stretti e veloci.

### Column Insert, Update, Delete

L'inserimento è molto costoso nei column-store, perché deve aggiornare tutte le colonne, e se le colonne sono ordinate deve rispettare l'ordinamento. Sistemi come C-Store usano molta duplicazione, aumentando ulteriormente il costo. Gli stessi problemi valgono per aggiornamenti e cancellazioni.

Una soluzione è usare **differential files** in memoria, così che tutte le operazioni possano essere eseguite insieme. Alcuni sistemi (come C-Store) gestiscono gli aggiornamenti dividendo la loro architettura in:
- **Read-store**: gestisce la maggior parte dei dati, ottimizzato per la lettura;
- **Write-store** (in memoria principale): gestisce gli aggiornamenti recenti, compatto.

Ogni query richiede entrambi i store e unisce i risultati. Questo approccio richiede di memorizzare esplicitamente i RID. Il write-store può essere usato per implementare la SNAPSHOT isolation, e permette una facile implementazione dell'algoritmo no-undo/redo.

### Indicizzazione

Una colonna ordinata su un attributo $A$ con un join index è equivalente a un tradizionale indice su $A$. Similmente, una proiezione $\{A_1, A_2, \ldots, A_n | A_i\}$ è come un indice su $A_i$ che permette di accedere rapidamente agli altri attributi tramite un piano IndexOnly.

La creazione di indici effettivi su ogni colonna non funziona bene: la ricostruzione delle tuple richiede comunque join dell'indice con la tabella originale, e introduce overhead per operazioni di insert/delete. In alternativa, si può spezzare la tabella $R(A, B, C, \ldots)$ in colonne separate $R_1(IdA, A), R(IdA, B), R(IdA, C), \ldots$, tutte ordinate su $IdA$. Questo permette join veloci, ma non la compressione, forza l'esecuzione pipelined e introduce comunque overhead per insert/delete.

> [!question] Possibili domande d'esame
>
> - Quando conviene usare un column-store rispetto a un row-store?
> - Quali sono le sfide principali dei column-store?
> - Cos'è una C-Store projection? Quando si usa?
> - Descrivi le tecniche di compressione per column-store (RLE, bit-vector, dictionary, difference).
> - Differenza tra early materialization e late materialization.
> - Come funziona il Jive join? Perché è preferito nei column-store?
> - Come vengono gestiti gli inserimenti nei column-store? Cos'è il read-store/write-store?
