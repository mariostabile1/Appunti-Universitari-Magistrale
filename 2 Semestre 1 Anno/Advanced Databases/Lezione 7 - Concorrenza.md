---
tags:
  - università/advanced-databases
  - concorrenza
  - locking
  - serializzabilità
data: 2024-01-01
lezione: "7 - Concorrenza"
professore: ""
---
# Concorrenza

Quando le transazioni vengono eseguite concorrentemente, le loro operazioni sono spesso **interleaved** (interlacciate), cioè l'esecuzione delle operazioni di una transazione si alterna con quella di un'altra. Questo può causare interferenze che lasciano il sistema in uno stato inconsistente. È responsabilità del **Concurrency Manager** prevenire questo. Si assume che tutte le transazioni siano consistenti: se una transazione venisse eseguita in isolamento non violerebbe alcun vincolo. Esistono tre tipi di conflitti possibili durante l'esecuzione concorrente:

- **Dirty Read** (conflitto Write-Read): una transazione non committed scrive su un dato $x$, e l'altra legge quello stesso $x$ dopo la scrittura. Se la prima transazione abortisce, le modifiche non avrebbero dovuto influenzare la seconda;
- **Unrepeatable Read** (conflitto Read-Write): una transazione legge lo stesso dato $x$ due volte, ma tra le due letture un'altra transazione esegue una scrittura su $x$. La prima transazione leggerà due valori diversi, sebbene si aspetti che siano uguali;
- **Lost Update** (conflitto Write-Write): due transazioni leggono il valore dello stesso dato $x$ (senza conflitti), ma poi entrambe aggiornano $x$ con un nuovo valore. L'ultima a scrivere è quella il cui cambiamento avrà effetto sul database, indipendentemente da quando è iniziata.

> [!definition] Esecuzione Seriale
>
> Un'esecuzione di un insieme di transazioni $T = \{T_1, \ldots, T_n\}$ è **seriale** se, per ogni coppia di transazioni $T_i$ e $T_j$, tutte le operazioni di $T_i$ vengono eseguite prima di quelle di $T_j$, o viceversa.

L'esecuzione seriale è corretta ma impraticabile, poiché una transazione alla volta occupa l'accesso ai dati per potenzialmente lungo tempo. È sufficiente garantire che l'esecuzione di transazioni interlacciate sia **serializzabile**.

> [!definition] Esecuzione Serializzabile
>
> Un'esecuzione di un insieme di transazioni $T$ è **serializzabile** se ha lo stesso effetto sul database di una qualche esecuzione seriale dell'insieme delle transazioni committed $T' \subseteq T$.

Le transazioni abortite sono ignorate perché non devono cambiare lo stato del database.

---

## Storie (Histories)

Una transazione è una sequenza di operazioni di lettura/scrittura. Una **storia** (*history*) rappresenta l'esecuzione interlacciata di più transazioni.

> [!definition] Storia
>
> Dato $T = \{T_1, \ldots, T_n\}$ un insieme di transazioni, una storia $H$ su $T$ è un insieme ordinato di operazioni tale che:
> - Le operazioni di $H$ sono quelle di $T_1, \ldots, T_n$;
> - $H$ preserva l'ordinamento tra le operazioni della stessa transazione.

Ad esempio, date le transazioni:
$$T_1 = r_1[x], r_1[y], w_1[x], c_1$$
$$T_2 = r_2[y], r_2[z], c_2$$
$$T_3 = w_3[x], w_3[y], c_3$$

una storia possibile è:
$$H = r_1[x], r_2[y], r_1[y], w_3[x], w_3[y], w_1[x], c_1, r_2[z], c_3, c_2$$

In una storia, due operazioni di transazioni diverse possono essere **in conflitto** se riguardano lo stesso dato e almeno una delle due è una scrittura. La c-equivalenza (conflict-equivalence) considera solo l'ordine delle operazioni in conflitto delle transazioni committed.

> [!definition] c-Serializzabilità
>
> Una storia di transazioni $T$ è **c-serializzabile** se è c-equivalente a una storia seriale sulle stesse transazioni di $T$.
>
> La c-serializzabilità implica sempre la serializzabilità, ma non viceversa.

Per verificare se una storia è c-serializzabile si usa il **grafo di serializzazione** (*serialization graph*):

> [!definition] Grafo di Serializzazione
>
> Sia $H$ una storia di transazioni committed $T = \{T_1, \ldots, T_n\}$. Il grafo di serializzazione $SG(H)$ è un grafo diretto tale che:
> - C'è un nodo per ogni transazione committed in $H$;
> - C'è un arco diretto $T_i \to T_j$ ($i \neq j$) se e solo se qualche operazione $p_i$ in $T_i$ precede e confligge con qualche operazione $p_j$ in $T_j$.

> [!theorem] Teorema di c-Serializzabilità
>
> Una storia $H$ è c-serializzabile se e solo se il suo grafo di serializzazione è **aciclico**.

Se il grafo è aciclico, un ordinamento seriale può essere ottenuto con un ordinamento topologico.

---

## Serializzabilità con Locking

### Strict Two-Phase Locking (2PL)

Lo **Strict Two-Phase Locking** (2PL Stretto) è il protocollo di scheduling più usato nei sistemi commerciali. Secondo questo protocollo, ogni elemento di dato usato da una transazione ha associato un lock: un **read lock** (condiviso) o un **write lock** (esclusivo). Si seguono due regole:

1. Se una transazione vuole leggere (scrivere) un dato, richiede prima un lock condiviso (esclusivo) su di esso. Prima che una transazione possa accedere a un dato, lo scheduler esamina il lock associato: se nessun'altra transazione ha il lock, viene concesso; altrimenti la transazione deve aspettare.
2. Tutti i lock di una transazione vengono rilasciati insieme nel momento in cui essa fa commit o abort.

Le politiche di concessione dei lock sono descritte dalla **matrice di compatibilità**:

|  | S | X |
|--|---|---|
| **S** | sì | no |
| **X** | no | no |

In breve: se un dato ha un read lock, può ricevere altri read lock ma non write lock; se ha un write lock, non può ricevere altri lock di alcun tipo.

> [!theorem] Teorema
>
> Un protocollo strict 2PL garantisce la c-serializzabilità.

### Deadlock

Il gestore ha bisogno di una strategia per rilevare i **deadlock**, situazioni in cui una transazione $T_i$ ha bloccato un elemento $A$ e richiede un lock su $B$, mentre contemporaneamente $T_j$ ha bloccato $B$ e richiede un lock su $A$: nessuna delle due può procedere.

**Deadlock Detection** — Strategia semplice: usare un **wait-for graph** in cui ogni nodo è una transazione, e un arco diretto $T_i \to T_j$ significa che $T_i$ sta aspettando che $T_j$ rilasci un lock. Se nel grafo compare un ciclo, si è verificato un deadlock, e una delle transazioni coinvolte (di solito la più giovane) deve essere abortita.

In applicazioni reali, gestire questo grafo può essere costoso. Si usano quindi strategie alternative: verifica dei cicli a intervalli predefiniti, oppure **timeout strategy** (se una transazione aspetta più di `timeout`, il sistema assume che ci sia un deadlock e abortisce la transazione, rischiando però il *thrashing*). PostgreSQL usa un timeout breve: allo scadere, costruisce un grafo wait-for locale per quella transazione.

**Deadlock Prevention** — Un'altra strategia che previene i deadlock: ogni transazione $T_i$ riceve un timestamp $ts(T_i)$ all'avvio. Quando $T_i$ richiede un lock già detenuto da $T_j$, due algoritmi sono possibili:

- **Wait-Die**: se $T_i$ è più vecchia di $T_j$ aspetta, altrimenti abortisce;
- **Wound-Wait**: se $T_i$ è più vecchia di $T_j$ la abortisce (*wound*), altrimenti aspetta.

In entrambi i casi, quando la transazione uccisa viene riavviata mantiene la sua priorità originale. Entrambi garantiscono assenza di starvation.

---

## Serializzabilità senza Locking

I metodi precedenti sono detti **pessimistici**, perché assumono che le operazioni di diverse transazioni siano molto probabilmente in conflitto. I metodi senza lock sono detti **ottimistici**: le transazioni eseguono liberamente le proprie operazioni, e il sistema controlla solo che non siano occorsi errori.

Un tale metodo, usato da Oracle, è la **snapshot isolation**. In questa soluzione, tutte le letture e scritture vengono eseguite senza lock. Ogni transazione $T_i$ legge i dati da una versione del database chiamata **snapshot**, contenente lo stato di tutti i dati come erano stati modificati da tutte le transazioni committed prima di essa. Le scritture della transazione sono visibili solo a $T_i$ stessa. Se una transazione esegue una scrittura, può fare commit solo se il suo write set non si interseca con quello di un'altra transazione committed (regola **"First-Committer-Wins"**). In caso contrario, la transazione viene abortita. Questa soluzione permette esecuzioni non serializzabili.

---

## Locking a Granularità Multipla

Le tecniche di locking viste finora assumono lock a livello di singoli record. Nelle applicazioni reali, le transazioni operano spesso su insiemi di record, quindi sono necessari lock con **granularità diversa** (database, file, pagina, record, campo), con una relazione di inclusione tra i livelli.

I tipi di lock S e X non sono sufficienti, quindi si introducono i **lock di intenzione**:

- **IS** (Intentional Shared): permette il locking esplicito dei discendenti in modalità S o IS;
- **IX** (Intentional Exclusive): permette il locking esplicito dei discendenti in modalità S, IS, X, IX o SIX;
- **SIX** (Shared Intentional Exclusive): blocca implicitamente tutti i discendenti in modalità S, e permette il locking esplicito dei discendenti in modalità X, SIX o IX.

La **matrice di compatibilità** per questi lock è:

|  | S | X | IS | IX | SIX |
|--|---|---|----|----|-----|
| **S** | sì | no | sì | no | no |
| **X** | no | no | no | no | no |
| **IS** | sì | no | sì | sì | sì |
| **IX** | no | no | sì | sì | no |
| **SIX** | no | no | sì | no | no |

### Il Problema del Phantom Locking

Un problema con il locking a granularità multipla è il **phantom locking**. Esempio:

- $T_1$: `SELECT * FROM Students`
- $T_2$: `INSERT INTO Students VALUES (100, 'Rossi')`; `commit`
- $T_1$: `SELECT * FROM Students`

$T_1$ blocca i record della tabella uno per uno. Anche con lo strict 2PL, questa storia è possibile: $T_1$ non può bloccare un record che ancora non esiste quando $T_2$ lo inserisce.

Per risolvere questo problema, si introduce il **Multi-granularity Strict 2PL**, che aggiunge queste due regole:

1. Un nodo (non radice) può essere bloccato da $T_i$ in modalità S o IS solo se il padre è bloccato da $T_i$ in modalità IS o IX;
2. Un nodo (non radice) può essere bloccato da $T_i$ in modalità X, IX o SIX solo se il padre è bloccato da $T_i$ in modalità SIX o IX.

> [!question] Possibili domande d'esame
>
> - Quali sono i tre tipi di conflitto nella concorrenza? Descrivili.
> - Cos'è la c-serializzabilità e come si verifica tramite il grafo di serializzazione?
> - Descrivi il protocollo Strict 2PL e perché garantisce la c-serializzabilità.
> - Differenza tra Deadlock Detection e Deadlock Prevention (Wait-Die vs Wound-Wait).
> - Cos'è la snapshot isolation e quali problemi può causare?
> - Cosa sono i lock di intenzione (IS, IX, SIX) e perché sono necessari?
> - Cos'è il phantom locking e come viene risolto?
