---
tags:
  - università/advanced-databases
  - physical-design
  - tuning
  - indici
data: 2024-01-01
lezione: "8 - Progettazione Fisica del Database"
professore: ""
---

# Progettazione Fisica del Database

La progettazione di un database relazionale si articola in quattro fasi:

1. **Analisi dei requisiti**: specifica dei dati da considerare e dei servizi da implementare;
2. **Progettazione concettuale**: definizione della rappresentazione delle entità di interesse, delle loro proprietà, associazioni e vincoli di integrità;
3. **Progettazione logica**: definizione dello schema logico;
4. **Progettazione fisica**: definizione delle appropriate strutture di archiviazione per il DBMS al fine di garantire le prestazioni desiderate.

Le informazioni necessarie per la progettazione fisica sono:

- **Statistiche**: numero di record per ogni relazione, dimensione dei record, numero di pagine usate, valori distinti per attributo, valori massimi e minimi, distribuzione (se non uniforme);
- **Descrizione del workload**: descrizione delle query e degli aggiornamenti critici con la loro frequenza e le rispettive prestazioni attese. Per ogni query critica: relazioni e attributi usati, attributi nelle condizioni di selezione e join, fattore di selettività. Per gli aggiornamenti: tipo (INSERT/DELETE/UPDATE). Questa informazione è spesso rappresentata con tabelle **ISUD** (Insert, Select, Update, Delete);
- **Conoscenza del DBMS**: organizzazioni dei dati, indici e tecniche di query processing supportate.

---

## Progettazione del Database

### Strutture di Archiviazione

Per le strutture di archiviazione, si sceglie tra le organizzazioni viste nel [[Lezione 2 - Panoramica di un DBMS]]:

- **Heap**: ideale per database piccoli dove gli inserimenti sono più frequenti delle ricerche;
- **Sequenziale**: ideale se è interessante tenere dati statici ordinati su una chiave;
- **Hash**: ideale se l'operazione più comune è la ricerca di un record per chiave;
- **Indice**: può tenere i dati ordinati su una chiave e supporta ricerche efficienti di uguaglianza e di intervallo.

### Indici

L'obiettivo principale degli indici è evitare la scansione completa di una relazione quando si cercano pochi record. I sistemi definiscono automaticamente indici su chiavi primarie e secondarie. Gli indici velocizzano le ricerche, ma occupano memoria e possono rallentare gli aggiornamenti degli attributi su cui sono definiti.

In generale, gli indici sono inutili per relazioni con pochissime pagine o per attributi scarsamente selettivi. La scelta degli indici utili non ha una soluzione banale, per cui la maggior parte dei sistemi commerciali fornisce strumenti di supporto.

> [!tip] Quando definire un indice?
>
> Considera di definire indici su:
> - Attributi usati in query che richiedono ordinamento (OrderBy, GroupBy, Distinct, operazioni insiemistiche);
> - Attributi selettivi nella clausola WHERE, scegliendo la struttura in base al tipo di ricerca (uguaglianza, intervallo, multi-attributo);
> - Per migliorare le join con IndexNestedLoop.

Un indice $I_1$ è **superfluo** se è **sussunto** da un altro $I_2$, cioè gli attributi di $I_1$ sono inclusi come prefisso di quelli di $I_2$. Ad esempio, un indice su $(A)$ è sussunto da $(A, B)$, e entrambi da $(A, B, C)$.

Due indici per due query diverse possono essere **uniti** in un unico indice che supporta entrambe. Ad esempio, indici su $(A, B)$ e $(A, C)$ possono essere uniti in $(A, C, B)$.

**Indici clustered** — Usati al meglio per ricerche di uguaglianza e di intervallo su attributi non chiave, anche se il fattore di selettività non è molto restrittivo. Utili anche per query con GroupBy.

**Indici multi-attributo** — Usabili per query con ricerche di uguaglianza-uguaglianza o uguaglianza-intervallo. L'ordine degli attributi è importante: il primo attributo dovrebbe essere quello usato principalmente per ricerche di uguaglianza.

**Piani Index-Only** — In alcuni casi, è possibile eseguire piani che accedono solo agli indici invece dei dati effettivi. Questi piani possono essere generati se è stato definito un indice sugli attributi della clausola SELECT DISTINCT o GROUP BY.

---

## Database Tuning

Il **database tuning** include tutte le attività svolte per soddisfare i requisiti di prestazione delle applicazioni di database. Per "prestazione" si intende: **throughput** (numero di operazioni completate nell'unità di tempo), **tempo di risposta** (tempo per eseguire un'applicazione) e **consumo di risorse** (memoria temporanea o permanente usata).

> [!note] Principio di Pareto nel Tuning
>
> Un'idea fondamentale nel database tuning è il **principio di Pareto**: applicando il 20% dello sforzo si ottiene l'80% degli effetti desiderati. Questo significa che un sistema completamente ottimizzato può essere non necessario; è meglio concentrarsi su modifiche piccole ma con miglioramenti significativi.

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 8.1]**: Il "quadrilemma del tuning del DBMS" — diagramma che mostra il trade-off tra query, applicazioni, parametri DBMS e hardware.

Il tuning coinvolge diverse figure professionali:
- **Progettisti di database e applicazioni**: conoscenza sulle applicazioni e il DBMS, poca su OS e hardware;
- **Amministratori di database (DBA)**: buona conoscenza su DBMS, OS e hardware, poca sulle applicazioni;
- **Esperti di database**: forte conoscenza su DBMS, OS e hardware, ma molto limitata sulle applicazioni.

### Index Tuning

La scelta iniziale degli indici può essere rivista per diversi motivi: il workload effettivo può rivelare che alcune query inizialmente considerate critiche non vengono eseguite frequentemente, oppure l'ottimizzatore non genera il piano fisico atteso. Cambiare gli indici può migliorare le prestazioni.

### Query Tuning

Se l'ottimizzatore non trasforma le query SQL, possono essere riscritte esplicitamente seguendo due insiemi di regole:

**Riscrittura semanticamente equivalente**:
- Se una query ha una condizione `OR`, può essere riscritta usando `UNION` di query diverse, o usando il predicato `IN`;
- Se una query ha `AND` di predicati diversi, può essere riscritta con `BETWEEN` se descrivono un intervallo;
- Evitare espressioni aritmetiche sugli attributi usati nelle condizioni (es. scrivere `Salary = 50` invece di `Salary*2 = 100`, perché nel secondo caso l'ottimizzatore potrebbe non usare un indice);
- Le subquery possono essere riscritte come join;
- Evitare le viste temporanee: se l'ottimizzatore non riesce a riscrivere la query senza viste, il piano fisico avrà un costo maggiore;
- Rimuovere `HAVING` e `GROUP BY` ridondanti.

**Riscrittura semanticamente non equivalente** (solo se il risultato è accettabile):
- Evitare clausole `DISTINCT` e `ORDER BY` inutili;
- Evitare prodotti cartesiani.

### Transaction Tuning

Un'altra causa comune di inefficienza è il modo in cui le transazioni sono programmate. Le transazioni dovrebbero essere il più brevi possibile per ridurre la durata dei lock. Le transazioni complesse dovrebbero essere divise in transazioni più piccole. Se una transazione deve fermarsi per attendere input dell'utente, dovrebbe fare commit e ripartire dopo che l'input è stato ricevuto.

Ogni transazione dovrebbe avere il livello di isolamento appropriato:

> [!definition] Livelli di Isolamento
>
> - **READ UNCOMMITTED**: le letture vengono eseguite senza richiedere lock; le dirty read sono possibili;
> - **READ COMMITTED**: lock sulle letture rilasciati subito dopo; lock esclusivi sulle scritture fino alla fine. Previene dirty read ma non unrepeatable read;
> - **REPEATABLE READ**: lock di lettura e scrittura sui record rilasciati alla fine. Evita unrepeatable read;
> - **SERIALIZABLE**: livello di isolamento di default; lock di lettura e scrittura di diverse dimensioni. Evita tutti i problemi ma riduce il numero di transazioni concorrenti eseguibili.

### Logical Schema Tuning

Se il design fisico non riesce a fornire prestazioni adeguate, può essere necessario modificare lo schema logico, usando due tipi di ristrutturazione: **partitioning** e **denormalizzazione**.

**Partitioning**:
- *Verticale*: riduce il volume di dati trasferiti dalla/alla memoria permanente e cache, separando gli attributi più frequentemente acceduti;
- *Orizzontale*: riduce il costo di accesso a una relazione grande separando i record frequentemente acceduti dagli altri.

**Denormalizzazione**: la normalizzazione decompone le tabelle per ridurre la ridondanza; la denormalizzazione fa il contrario, aggiungendo colonne per migliorare le prestazioni delle query di sola lettura. È utile quando due o più tabelle sono spesso lette dopo join.

---

## DBMS Tuning

Se tutto il resto fallisce, il sistema di database stesso deve essere ottimizzato. I livelli principali da considerare sono:

- **Gestione delle transazioni**: intervenire su parametri come log storage, frequenza dei checkpoint e dump. Più frequenti sono, più veloce è il riavvio dopo un guasto, ma le prestazioni normali peggiorano;
- **Dimensione del buffer del database**: un buffer più grande migliora le prestazioni, ma non dovrebbe essere così grande da non entrare in memoria principale;
- **Gestione del disco**: aggiungere dischi o usare sistemi RAID se l'I/O su disco è un collo di bottiglia;
- **Architetture parallele**: strettamente dipendente dal sistema in uso.

Un'alternativa è il **database self-tuning**.

> [!question] Possibili domande d'esame
>
> - Quante sono le fasi della progettazione di un database relazionale? Descrivile.
> - Come si sceglie la struttura di archiviazione più appropriata?
> - Quando è utile definire un indice? Cos'è un indice sussunto?
> - Quali sono i livelli di isolamento delle transazioni? Quali problemi evita ciascuno?
> - Cos'è la denormalizzazione e quando è utile?
> - Cos'è il database tuning e quali aspetti considera?
