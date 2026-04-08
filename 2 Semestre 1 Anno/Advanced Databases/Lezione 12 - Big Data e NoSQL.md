---
tags:
  - università/advanced-databases
  - NoSQL
  - big-data
  - map-reduce
data: 2024-01-01
lezione: "12 - Big Data e NoSQL"
professore: ""
---

# Big Data e NoSQL

Dalla loro creazione negli anni '80 fino ai primi anni 2000, i DBMS relazionali sono stati il tipo principale di sistemi usati per gestire i dati. Ma a metà degli anni 2000, l'ascesa di Internet ha causato un rapido aumento della quantità di dati generati dalle attività online. È nato un nuovo insieme di campi applicativi chiamato **Big Data**, e le capacità dei DBMS relazionali tradizionali si sono rivelate insufficienti. Le limitazioni dei database relazionali sono descritte dalle **"tre V"**:

- **Volume**: le applicazioni gestiscono quantità massive di dati e i DBMS non scalano abbastanza;
- **Velocity**: velocità computazionale e di sviluppo;
- **Variety**: i dati sono altamente eterogenei e gli schemi spesso confliggono con la varietà.

**NoSQL** si riferisce al vasto gruppo di sistemi di database non relazionali. Alcuni hanno linguaggi di query, ma la maggior parte non implementa SQL vero e proprio. Per alcune applicazioni si rinuncia ad ACID in favore di un maggior supporto alla distribuzione. Poiché i dati non sono più rappresentati da tabelle, non possono essere in prima forma normale; questo può ridurre notevolmente la necessità di join, ma a un costo (spiegato in seguito).

> [!note] Impedance Mismatch
>
> Un'importante ragione per preferire i sistemi NoSQL è il cosiddetto **impedance mismatch**: la differenza tra il modello relazionale e le strutture dati in memoria. Nel modello relazionale, le informazioni sono memorizzate in coppie nome-valore (tuple) organizzate in insiemi (relazioni). Le tuple devono essere semplici e non possono contenere strutture dati come liste o alberi. In memoria, invece, gli stessi dati sono rappresentati con strutture più ricche, richiedendo una traduzione. I sistemi NoSQL non hanno questa limitazione.

Quando SQL fu sviluppato, il ruolo di un database era quello di un **integration database**, che memorizzava dati condivisi da più applicazioni separate. Un approccio diverso è trattare il database come un **application database**, accessibile solo da una singola applicazione. In questo caso, la responsabilità per l'integrità del database può essere spostata nel codice dell'applicazione.

> [!warning] NoSQL ≠ NewSQL
>
> NoSQL è diverso da NewSQL (che include i database a colonne, i database in memoria, ecc.).

---

## Consistenza

I sistemi NoSQL non forniscono transazioni tradizionali, ma devono comunque garantire un certo livello di consistenza. I tre problemi da considerare sono: **conflitti write-write**, **consistenza delle letture** (quanto sono aggiornati i dati), e **consistenza transazionale** (scrivere solo valori basati su dati attualmente validi).

### Il Teorema CAP

Un motivo spesso citato per voler rilassare la consistenza nei sistemi NoSQL è il **teorema CAP**. Afferma che dato un sistema con le proprietà di Consistency (Consistenza), Availability (Disponibilità) e Partition tolerance (Tolleranza alla partizione), si possono avere solo due delle tre. Nei sistemi database tradizionali, si rinuncia alla disponibilità (es. two-phase commit). Nei sistemi NoSQL, si rilassa la consistenza.

Quando si scambia consistenza, però, non viene completamente abbandonata. Più nodi sono coinvolti in una richiesta, maggiore è la probabilità di evitare un'inconsistenza.

### Quorum per Letture e Scritture

Il primo problema è l'accesso coerente a più copie dello stesso valore. Nei sistemi P2P, non tutti i nodi devono confermare una scrittura per garantire una forte consistenza: è sufficiente la maggioranza. Il **write quorum** può essere impostato a qualsiasi $w > n/2$. Il **read quorum** viene poi impostato a $r > n - w$ per garantire letture fortemente consistenti.

Il secondo problema è aggiornare un valore solo se nessun altro nodo lo ha modificato nel frattempo: un approccio ottimistico usa i **version stamps** (timbri di versione), associando a ogni dato un timestamp aggiornato dopo ogni modifica. Un aggiornamento fallisce se il timestamp al momento della scrittura effettiva è diverso da quello letto (**Compare-And-Set**).

Se non si usano quorum, si ricorre alla **riconciliazione** (*reconciliation*), permettendo al sistema di divergere. La riconciliazione è fortemente dipendente dal sistema: se l'unica operazione possibile è un'inserzione, la riconciliazione può semplicemente aggiungere entrambi gli elementi; se sono possibili inserimenti e aggiornamenti, si trova e applica l'insieme minimo di modifiche compatibile con entrambe le versioni. Nei sistemi P2P si usa il **vector clock** per decidere l'ordine temporale tra versioni diverse.

---

## Map-Reduce

Il pattern **map-reduce** è un modo per organizzare l'elaborazione sfruttando la suddivisione del calcolo su più macchine di un cluster, mantenendo i dati e il calcolo sulla stessa macchina il più possibile.

Il primo passo è il **mapping**: una funzione *map* prende in input un singolo aggregato (es. un documento) e produce un insieme di coppie chiave-valore. Ogni applicazione della funzione map è indipendente dalle altre, quindi parallelizzabile. La funzione **reduce** prende più output di map con la stessa chiave e ne combina i valori.

L'implementazione **Hadoop** memorizza input e output di ogni fase in un file system distribuito che gestisce partizionamento e replicazione. L'implementazione **Spark** cerca di tenere, quando possibile, input e output in memoria principale.

---

## Tipi di Sistemi NoSQL

I sistemi NoSQL generalmente non supportano SQL, sono spesso open source, orientati ai cluster, relativamente recenti (dopo il 2000), *schema-free* e orientati a una singola applicazione. Molte applicazioni NoSQL supportano **viste materializzate**: pur non avendo viste vere, possono avere query pre-calcolate e memorizzate in cache.

I modelli di dati NoSQL si dividono in due gruppi:

### Modelli Aggregati

Memorizzano informazioni come collezioni di oggetti aggregati anziché tabelle. Questo è essenziale per lavorare senza transazioni e senza join.

**Key-value Stores** (Dynamo, Riak, Voldemort)
Memorizzano blocchi non interpretati come coppie $\langle \text{key}, \text{value} \rangle$. L'unica ricerca consentita è il recupero di un elemento tramite chiave. I dati vengono prima partizionati tra i nodi tramite **sharding**, poi replicati (con master-slave o P2P replication). Permettono al dato di diventare inconsistente, ma offrono strumenti per la riconciliazione. Usati tipicamente per: informazioni di sessione, profili utente, dati del carrello degli acquisti.

**Document Databases**
Le chiavi sono associate ad alberi (documenti). Alcuni sistemi usano XML (MarkLogic, eXist), altri JSON (CouchDB, MongoDB). Poiché memorizzano documenti, possono essere cercati per contenuto.

In **MongoDB**: una istanza, molti database, molte collezioni di documenti JSON. Ogni documento ha il proprio id, quindi MongoDB può fare query per chiave o in modi più complessi. Supporta selezione, proiezione e aggregazione, ma non join. Usa sharding e replicazione master/slave.

**CouchDB** supporta viste (sia materializzate che virtuali).

**Column-family Stores** (BigTable, HBase, Cassandra)
Rappresentano dati come insiemi di coppie $\langle \text{key}, \text{record} \rangle$, raggruppando le colonne in "famiglie di colonne" (*column families*), simili alle tabelle nei database relazionali. I record in una famiglia non sono necessariamente omogenei.

In **Cassandra**, l'amministratore fissa un numero di repliche per ogni keyspace. Il programmatore sceglie il quorum per le operazioni di lettura e scrittura. L'atomicità è garantita a basso livello. Supporta CQL (Cassandra Query Language), che implementa operazioni di selezione e proiezione (ma non join).

**Sparse Table Databases**
I dati sono rappresentati usando record, ma record diversi possono seguire schemi diversi.

### Modelli a Grafo

I database a grafo sono raramente usati nel campo del Big Data poiché non sono adatti a memorizzare grandi quantità di dati né supportano un accesso veloce; tuttavia sono molto ottimizzati per la varietà.

In un modello a grafo, ogni elemento è un nodo con un insieme di proprietà, e i nodi sono collegati tra loro con archi etichettati. Il costo di navigazione delle relazioni è principalmente sostenuto al momento dell'inserimento, non della ricerca.

I database a grafo non sono né sharded né transazionali. **Neo4J** supporta la replicazione master-slave e implementa il linguaggio di query **Cypher**, che supporta selezioni e proiezioni sia su nodi che su archi. Questi database sono particolarmente utili per modellare grafi sociali.

---

## Architetture Big Data

Un trend attuale è quello dei **data lake**. Mentre in un data warehouse c'è una lunga fase complessa di pulizia dei dati (trasformazione per adattarli alle operazioni del data warehouse), un data lake riceve tutti i dati senza alcuna pre-elaborazione. Questa informazione viene poi interrogata usando tecniche di data mining o AI che estraggono automaticamente conoscenza.

Un altro trend è quello dei **sistemi poliglotti** (*polyglot systems*): sistemi che raccolgono dati da sistemi diversi che usano linguaggi di query diversi.

> [!question] Possibili domande d'esame
>
> - Cosa sono le "tre V" del Big Data?
> - Cos'è l'impedance mismatch e perché porta all'uso di NoSQL?
> - Cosa afferma il teorema CAP? Come si applica ai sistemi NoSQL?
> - Cos'è il pattern Map-Reduce? Differenza tra Hadoop e Spark.
> - Quali sono i tipi di sistemi NoSQL? Descrivi key-value stores, document databases e column-family stores.
> - Cos'è un data lake e come si differenzia da un data warehouse?
