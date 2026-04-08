---
tags:
  - università/advanced-databases
  - storage-engine
  - access-methods
  - physical-plans
data: 2024-01-01
lezione: "3 - Gestione dei Metodi di Accesso"
professore: ""
---
# Gestione dei Metodi di Accesso

L'**Access Methods Manager** fornisce un'interfaccia con diverse operazioni per interagire con le organizzazioni e gli indici implementati dallo Storage Structure Manager, così che i dati possano essere trasferiti tra la memoria principale e quella permanente. Il linguaggio usato per implementare queste operazioni trasforma la macchina in una **macchina database astratta** (*abstract database machine*), il cuore del sistema di gestione di database.

Le macchine database astratte sono divise in due parti:

- **Relational Engine**: macchina astratta per il modello di dati logico; include i moduli per supportare l'esecuzione di comandi SQL;
- **Storage Engine**: macchina astratta per il modello di dati fisico; include i moduli per eseguire operazioni sui dati in memoria permanente.

---

## Storage Engine

L'interfaccia dello Storage Engine dipende dalle strutture dati usate in memoria permanente. Normalmente non è direttamente disponibile all'utente, che interagirà invece con il Relational Engine, il quale a sua volta comunica con lo Storage Engine. Useremo un'interfaccia ispirata a quella del sistema relazionale JRS, che memorizza le relazioni in heap file e fornisce indici B+-tree.

### Dati e Transazioni

```
beginTransaction : null → TransactionId
commit           : TransactionId → null
abort            : TransactionId → null
createDB         : Path × DBName × TransactionId → DB
createHF         : DB × Path × HFName × TransactionId → HF
createIdx        : DB × Path × IdxName × HFName × Attr × Ord × Unique × TransactionId → Idx
dropDB           : DBName × TransactionId → null
dropHF           : HFName × TransactionId → null
dropIdx          : IdxName × TransactionId → null
```

### Heap File

```
HFopen         : DB × HFName × TransactionId → HF
HFclose        : HF → null
HFgetRecord    : HF × RID → Record
HFdeleteRecord : HF × RID → null
HFupdateRecord : HF × RID × FieldNum × NewField → null
HFinsertRecord : HF × Record → RID
HFgetNPage     : HF → int
HFgetNRec      : HF → int
```

### Indici

```
Iopen        : DB × IdxName × TransactionId → Idx
Iclose       : Idx → null
IdeleteEntry : Idx × Entry → null   (Entry = Value × RID)
IinsertEntry : Idx × Entry → null
IgetNKey     : Idx → int
IgetNLeaf    : Idx → int
IgetMin      : Idx → Value
IgetMax      : Idx → Value
```

---

## Operatori dei Metodi di Accesso

Le seguenti operazioni trasferiscono dati tra la memoria principale e quella permanente. I record di un heap file o di un indice vengono acceduti tramite **scan** (scansioni): un operatore di scansione su heap file legge i record uno dopo l'altro, mentre un operatore di scansione su indice fornisce un modo efficiente per recuperare i RID dei record.

Gli operatori di scansione sono implementati usando un **cursore** (*cursor* o *iterator*), un oggetto con metodi che restituisce un record alla volta e si sposta tra i record. La struttura tipica di un programma che usa gli operatori di scansione è:

```
while !C.isDone() do
    Val = C.getCurrent()
    ...
    C.next()
end while
```

dove `C` è l'oggetto cursore.

### Heap File Scan

```
HFSopen       : HF → HFS
HFSisDone     : HFS → bool
HFSgetCurrent : HFS → RID
HFSnext       : HFS → null
HFSreset      : HFS → null
HFSclose      : HFS → null
```

### Index Scan

```
ISopen       : Idx × fstKey × lstKey → IS
ISisDone     : IS → bool
ISgetCurrent : IS → null
ISreset      : IS → null
ISclose      : IS → null
```

---

## Piani Fisici

Quando una query SQL deve essere eseguita, viene prima rappresentata come un **piano logico** (*logical plan*), una rappresentazione ad albero della query, che viene trasformata in una forma più efficiente. Questo piano logico trasformato viene poi tradotto in un **piano fisico** (*physical plan*), i cui nodi sono gli operatori fisici effettivi che implementano quella query.

> [!definition] Operatore come Iterator
>
> Ogni operatore in un piano fisico è un iteratore che usa un'interfaccia "pull": quando un operatore riceve una richiesta dall'alto, "tira" (*pulls*) i suoi nodi di input e calcola il risultato, restituendolo all'operatore padre.

Un'interfaccia operatore fornisce i metodi necessari: `open`, `next`, `isDone`, e `close`, implementati usando l'interfaccia dello Storage Engine.

> [!note] Relazione con i capitoli successivi
>
> I piani fisici e gli operatori fisici sono approfonditi nel [[Lezione 4 - Operatori Fisici Relazionali]], dove si vedono nel dettaglio gli algoritmi per proiezione, selezione, raggruppamento e join.

> [!question] Possibili domande d'esame
>
> - Qual è la differenza tra Relational Engine e Storage Engine?
> - Cosa si intende per cursore/iteratore nel contesto degli operatori di scansione?
> - Come viene tradotta una query SQL in un piano fisico?
