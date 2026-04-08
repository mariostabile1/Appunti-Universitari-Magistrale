---
tags:
  - università/advanced-databases
  - transazioni
  - recovery
  - ACID
data: 2024-01-01
lezione: "6 - Transazioni"
professore: ""
---
# Transazioni

Lo Storage Engine offre funzionalità volte a risolvere i problemi di **recovery** e **concorrenza**, garantendo che ogni operazione dell'utente sia eseguita senza che quest'ultimo noti alcun guasto sottostante, e che non vi siano interferenze con altre operazioni eseguite concorrentemente. Le soluzioni a questi problemi si basano su un meccanismo chiamato **transazione**.

> [!definition] Transazione
>
> Una transazione è una sequenza di operazioni sul database e sui dati temporanei, con le seguenti proprietà:
>
> - **Atomicità** (*Atomicity*): solo le transazioni completate con successo modificano lo stato del database; se una transazione viene interrotta, il database rimane invariato come se non fosse mai stata avviata;
> - **Isolamento** (*Isolation*): quando una transazione è eseguita concorrentemente con altre, l'effetto finale deve essere lo stesso di quello che si otterrebbe se fosse eseguita da sola;
> - **Durabilità** (*Durability*): gli effetti delle transazioni committed devono sopravvivere a guasti di sistema e di supporto.

Spesso si usa l'acronimo **ACID** (Atomicity, Consistency, Isolation, Durability) per riferirsi alle proprietà delle transazioni. Il **Recovery Manager** garantisce Atomicità e Durabilità, il **Concurrency Control Manager** garantisce l'Isolamento. La Consistenza è garantita dall'implementazione di vincoli di integrità e dalla correttezza del codice.

Per il DBMS, una transazione $T$ richiede un certo numero di operazioni di lettura/scrittura sul database. Ogni transazione inizia e termina con le seguenti operazioni:

- `beginTransaction`: segnala l'inizio della transazione;
- `commit`: segnala la terminazione positiva della transazione, richiedendo al sistema di rendere permanenti i suoi aggiornamenti;
- `abort`: segnala la terminazione anomala della transazione, richiedendo al sistema di annullare i suoi aggiornamenti.

> [!warning] Attenzione sul commit
>
> L'esecuzione di un `commit` non significa automaticamente che la transazione terminerà con successo, perché i suoi aggiornamenti potrebbero non essere ancora scritti nella memoria permanente.

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 6.1]**: Diagramma di transizione degli stati per l'esecuzione di una transazione (state transition diagram). Mostra gli stati: active, partially committed, committed, failed, aborted.

Per leggere una pagina ($r_i[x]$), essa viene portata nel buffer dal disco (se non è già presente), e poi letta. Per scrivere una pagina ($w_i[x]$), una copia in memoria viene modificata, e successivamente scritta su disco quando il Buffer Manager lo ritiene opportuno. Questa scrittura ritardata deve essere compatibile con Atomicità e Durabilità.

---

## Tipi di Guasto

Un database può diventare inconsistente a causa di tre tipi di guasti:

- **Guasto di transazione** (*Transaction failure*): un'interruzione di una transazione che non danneggia il contenuto né della memoria principale né di quella permanente. Una transazione può essere interrotta perché incontra certe condizioni, viola vincoli di integrità, o il concurrency manager sceglie di abortirla per via di un deadlock;

- **Guasto di sistema** (*System failure*): un'interruzione (crash) del sistema (DBMS o OS) in cui il contenuto della memoria principale viene perso, ma il contenuto della memoria permanente rimane intatto. Dopo un crash, il sistema si riavvia automaticamente o tramite un operatore;

- **Guasto di supporto** (*Media failure*, anche detto *disaster*): un'interruzione del sistema in cui il contenuto della memoria permanente viene perso o danneggiato. Quando accade, il Recovery Manager usa un backup per ripristinare il database.

### File di Log

Oltre ai backup, un'altra protezione dai guasti è rappresentata dai **file di log**: questi file contengono righe che registrano ogni operazione transazionale eseguita nel sistema, comprese le transazioni abortite. Per ogni transazione si registra: quando inizia, quando fa commit, quando abortisce, e quando modifica un record (specificando la pagina, il valore precedente *before image* e il valore successivo *after image*). Ogni record è identificato univocamente da un **LSN** (Log Sequence Number), assegnato in ordine strettamente crescente.

Un algoritmo di recovery richiede un **undo** se un aggiornamento di una transazione non committed è già memorizzato nel database. Un **redo** è necessario se una transazione è committed ma non tutti i suoi aggiornamenti sono stati scritti in modo permanente (dopo un guasto di sistema).

### Checkpoint

Il tempo di inattività del sistema è dato dal prodotto tra il tasso di guasto e il tasso di recovery. Il modo per ridurre l'inattività è ridurre il tempo di recovery: nella pratica questo si fa tramite i **checkpoint**. Ne esistono diversi tipi:

> [!definition] Commit-consistent checkpoint
>
> Quando il checkpoint inizia, l'attivazione di nuove transazioni viene sospesa, e il sistema aspetta la terminazione delle transazioni attive; tutte le pagine "dirty" nel buffer vengono scritte in memoria permanente; il checkpoint viene scritto nel log, e un puntatore al record corrispondente viene memorizzato in un file speciale chiamato *restart file*. Il sistema riprende poi la normale attività. Questa strategia è semplice ma inefficiente, poiché il sistema deve fermarsi regolarmente.

> [!definition] Buffer-consistent checkpoint (Versione 1)
>
> Simile al precedente, ma una volta che il checkpoint inizia, sospende anche l'esecuzione delle transazioni attive. Più efficiente del precedente, ma comunque lento a causa delle operazioni di flush del buffer.

> [!definition] No-stop checkpoint
>
> Una volta avviato, il checkpoint viene scritto nel log, insieme agli id delle transazioni attive; un nuovo thread viene avviato, che scansiona il buffer e fa flush delle pagine dirty in parallelo con le transazioni standard, garantendo che tutte le pagine dirty all'inizio del checkpoint vengano scaricate prima della sua fine.

---

## Recovery da Guasti di Sistema e di Supporto

Per ripristinare il database, viene invocato l'operatore di **restart**, che porta il database nel suo stato committed rispetto all'esecuzione fino al guasto di sistema, e riavvia le normali operazioni di sistema. L'algoritmo usato ha due fasi: **rollback** e **rollforward**.

Nella **fase di rollback** il log viene letto all'indietro, per annullare gli aggiornamenti delle transazioni che non erano terminate prima del crash, e per trovare gli identificatori delle transazioni terminate con successo. Si costruiscono due insiemi: la *redo-list* e la *undo-list*. Fino al primo record di checkpoint, si eseguono queste operazioni:

1. Se un record è `(commit, Ti)`, $T_i$ viene aggiunto alla redo-list;
2. Se un record è un aggiornamento di una transazione $T_i$ non nella redo-list, viene aggiunto alla undo-list;
3. Se un record è `(begin, Ti)`, $T_i$ viene rimosso dalla undo-list;
4. Se un record è un checkpoint `(CKP, L)`, per ogni $T_i \in L$ non nella redo-list viene aggiunto alla undo-list. Se la undo-list non è vuota, la fase di rollback continua finché non è completamente svuotata.

Nella **fase di rollforward**, il log viene letto in avanti dal primo record dopo il checkpoint, rifacendo tutte le operazioni delle transazioni nella redo-list.

> [!note] Varianti dell'algoritmo Undo-Redo
>
> - **NoUndo-Redo**: richiede che tutti gli aggiornamenti di una transazione siano nel database *dopo* il commit. Usa la **NoSteal (Pin) Policy**: tutti i buffer usati dalle transazioni sono bloccati (*pinned*) finché non viene eseguito il commit.
> - **Undo-NoRedo**: richiede che tutti gli aggiornamenti siano nel database *prima* del commit. Usa la **Force policy**: le pagine del buffer vengono forzatamente scritte su disco prima del commit. Costoso per l'alto numero di scritture.
> - **NoUndo-NoRedo**: usa entrambe le policy. Richiede un meccanismo di **shadow pages** per scrivere atomicamente molte pagine.

### Shadow Pages

Il meccanismo **NoUndo-NoRedo** richiede un modo per scrivere molte pagine atomicamente. Questo si fa usando le **shadow pages**: quando una transazione aggiorna una pagina per la prima volta, viene creata una nuova pagina nel database chiamata *current page*, con un certo indirizzo $p$. La vecchia pagina rimane invariata e diventa la shadow page. La *New Page Table* viene aggiornata con l'indirizzo fisico $p$ della current page. Una volta che la transazione raggiunge il punto di commit, le shadow pages vengono sostituite con un'operazione atomica.

> [!question] Possibili domande d'esame
>
> - Cosa significa ACID? Quale componente del DBMS garantisce quale proprietà?
> - Quali sono i tre tipi di guasto in un DBMS? Come si differenziano?
> - Cosa sono i checkpoint e perché sono importanti per la recovery?
> - Descrivere le varianti dell'algoritmo Undo-Redo e le policy di buffer associate.
> - Come funzionano le shadow pages?
