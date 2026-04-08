---
tags:
  - università/advanced-databases
  - query-optimization
  - dipendenze-funzionali
  - piano-fisico
data: 2024-01-01
lezione: "5 - Ottimizzazione delle Query"
professore: ""
---
# Ottimizzazione delle Query

L'**ottimizzatore** è una componente chiave del Query Manager. Il suo ruolo è selezionare il piano fisico ottimale per eseguire le query usando gli operatori e le strutture dati forniti dallo Storage Engine. Le **dipendenze funzionali** vengono usate non solo per la progettazione dello schema relazionale, ma anche per l'ottimizzazione.

---

## Elaborazione ed Esecuzione delle Query

In generale, esistono molte strategie per eseguire una query, specialmente quando è complessa. Il problema dell'ottimizzazione è influenzato dal fatto che una query può essere scritta in modi equivalenti diversi, e gli operatori dell'algebra relazionale possono essere implementati usando diversi operatori fisici.

L'elaborazione delle query è divisa in quattro fasi:

1. **Analisi della query** (*Query analysis*): si verifica la correttezza della query SQL e si traduce in forma interna, tipicamente basata sull'algebra relazionale;
2. **Trasformazione della query** (*Query transformation*): il piano logico viene trasformato in uno equivalente con migliori prestazioni;
3. **Generazione del piano fisico** (*Physical plan generation*): vengono generati piani fisici alternativi e valutati; viene scelto quello con il costo minore;
4. **Valutazione della query** (*Query evaluation*): il piano fisico scelto viene eseguito.

La trasformazione della query e la generazione del piano fisico sono spesso considerate parte della stessa fase, chiamata **Query Optimizer**. Le trasformazioni più interessanti riguardano l'eliminazione di DISTINCT, GroupBy, sottoqueries nella WHERE e di viste.

---

## Dipendenze Funzionali

> [!definition] Dipendenza Funzionale
>
> Dato uno schema di relazione $R$ e $X, Y$ sottoinsiemi di attributi di $R$, una **dipendenza funzionale** $X \to Y$ (letta "X determina Y") è un vincolo tale che, per ogni istanza possibile $r$ di $R$ e per qualsiasi coppia di tuple $t_1, t_2 \in r$:
> $$t_1[X] = t_2[X] \Rightarrow t_1[Y] = t_2[Y]$$

Una dipendenza funzionale è **triviale** se il conseguente contiene attributi che appaiono anche nell'antecedente:
$$XY \to X$$

È **atomica** se il conseguente è composto da un solo attributo: $X \to A$.

È **canonica** se $X \to A$ vale, ma $X' \to A$ con $X' \subset X$ non vale. Ogni dipendenza non triviale contiene una o più dipendenze canoniche.

Una **chiave** è un insieme di attributi $K$ tale che $K \to T$ è canonica.

La **regola dell'unione** afferma che:
$$X \to A_1, \ldots, A_n \Leftrightarrow X \to A_1, \ldots, X \to A_n$$

> [!definition] Implicazione Logica
>
> Dato un insieme di dipendenze funzionali $F$ su uno schema di relazione $R$, una dipendenza $X \to Y$ è **derivata** (implicata) da $F$ se ogni istanza di $R$ che soddisfa $F$ soddisfa anche $X \to Y$. Questo vale se $X \to Y$ può essere derivata da $F$ usando gli **assiomi di Armstrong**:
> - $Y \subseteq X \Rightarrow X \to Y$ (Riflessività)
> - $X \to Y, Z \to T \Rightarrow XZ \to YZ$ (Aumentazione)
> - $X \to Y, Y \to Z \Rightarrow X \to Z$ (Transitività)

> [!definition] Chiusura di un Insieme di Attributi
>
> Dato uno schema $R\langle T, F \rangle$ e $X \subseteq T$, la **chiusura** di $X$ è:
> $$X^+ = \{A_i \in T \mid F \models X \to A_i\}$$

> [!theorem] Teorema
>
> $$F \models X \to Y \Leftrightarrow Y \subseteq X^+$$

Consideriamo una query su un insieme di tabelle $R_1(T_1), \ldots, R_n(T_n)$, con la condizione WHERE $C$ in CNF. Dopo il join e una selezione, queste dipendenze valgono sul risultato finale:
- $K_{ij} \to T_i$ per ogni chiave $K_{ij}$ di $T_i$;
- $\emptyset \to A$ per ogni $A = c$ in $C$;
- $A_i \to A_j$ e $A_j \to A_i$ per ogni $A_i = A_j$ in $C$.

---

## Eliminazioni

### Distinct Elimination

DISTINCT normalmente richiede un piano fisico con eliminazione dei duplicati, con dati raggruppati via ordinamento (operazione costosa). Per decidere se l'eliminazione dei duplicati è inutile, si usa un algoritmo basato sulla teoria delle dipendenze funzionali:

> [!theorem] Teorema per Distinct Elimination
>
> Sia $A$ l'insieme degli attributi del risultato e $K$ l'unione degli attributi della chiave di ogni tabella usata nella query. Se $A \to K$, allora l'eliminazione dei duplicati è inutile.

Per scoprire se $A \to K$, si calcola la chiusura di $A$. Se la query contiene una clausola GroupBy, il teorema vale per $G$ (attributi di raggruppamento) invece di $K$.

### GroupBy Elimination

Un GroupBy può essere eliminato se: ogni gruppo ha solo un record (equivalente al caso Distinct), oppure c'è solo un gruppo. Il secondo caso si verifica controllando che il valore degli attributi di raggruppamento sia lo stesso per ogni tupla: si verifica che la chiusura dell'insieme vuoto $(\{\})^+$ contenga tutti gli attributi di raggruppamento.

Se si usano funzioni di aggregazione: `COUNT` viene sostituito con 1, e `MIN(A)`, `MAX(A)`, `SUM(A)`, `AVG(A)` vengono sostituiti con $A$.

### Where-subquery Elimination

Questa è una delle trasformazioni più comuni e importanti. Per eseguire queste query, l'ottimizzatore genera un piano fisico per la subquery, che viene eseguito per ogni record processato dalla query esterna. Assumendo che tutte le subquery siano state convertite in forma equivalente con EXISTS, e che non ci sia GroupBy nella subquery, una query del tipo:

```sql
SELECT ... FROM R1 WHERE EXISTS (SELECT ... FROM R2 WHERE ...)
```

è equivalente a un join tra $R_1$ e $R_2$. Il DISTINCT nel join è necessario quando esiste una relazione 1:N tra $R_1$ e $R_2$.

Se la subquery ha una funzione di aggregazione, la query unnested richiede una clausola GroupBy sugli attributi di proiezione. Quando la funzione è `COUNT`, il join deve essere sostituito con un **outer join** (NATURAL LEFT/RIGHT/FULL JOIN), per risolvere il problema detto **count bug**.

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 5.1]**: Tipi di join — mostra INNER JOIN, LEFT JOIN, RIGHT JOIN e FULL JOIN con i rispettivi insiemi di record restituiti.

### View Merging

Query complesse sono più semplici con le viste. La clausola `WITH` definisce viste temporanee disponibili solo alla query in cui occorre. Quando l'ottimizzatore genera un piano per una vista, ottimizza la vista separatamente dalla query principale.

Nel piano logico, la trasformazione sostituisce il riferimento al nome di una vista con il corrispondente piano logico. Il nuovo piano logico viene poi riscritto usando le regole di equivalenza dell'algebra relazionale per ottenere la forma canonica:

$$\sigma_b(\pi(\bowtie(R_i)))$$

La trasformazione di una query con GroupBy richiede di spostare il GroupBy sopra un join, usando la regola di equivalenza algebrica:

$$\gamma_{X_R, F}(R) \bowtie_{fk=pk} S \equiv \gamma_{X_R, A(S), F}(R \bowtie_{fk=pk} S)$$

---

## Generazione del Piano Fisico

Con le eliminazioni si genera un piano fisico "casuale" per la query, che poi viene ottimizzato. La generazione trova direttamente il miglior piano fisico possibile in due passi: generazione di piani fisici alternativi e scelta del piano con il costo stimato minore.

### Query su Relazione Singola

Per queste query, l'unica domanda è se usare gli indici invece di una semplice scansione. Di solito, usare un indice conviene se il fattore di selettività è abbastanza restrittivo. Un caso speciale: se gli attributi nella SELECT sono inclusi nel prefisso della chiave di un indice, la query può essere valutata leggendo solo l'indice.

### Query su Relazioni Multiple

La questione più importante è l'**ordine in cui le relazioni vengono joinato**. Ogni permutazione di $n$ relazioni dà $n!$ permutazioni diverse, ognuna con molti candidati. La **ricerca completa** (full search) nello spazio dei candidati:

1. Si trovano i piani più economici per accedere a ogni singola relazione;
2. Si sceglie il piano più economico tra quelli non selezionati e quelli generati da un join con la relazione precedentemente scelta;
3. Si continua fino a coprire l'intera query.

Ad ogni passo vengono valutati solo i piani dietro una "frontiera", cioè tutti i piani che sono figli diretti della radice o di un piano scelto come costo minimo.

```
Algorithm Full Search:
1. Inizializza Plans con i migliori piani per accedere a ogni relazione
2. loop:
3.     Estrai il piano più veloce P da Plans
4.     if P è completo: return P
5.     for R non in P:
6.         Inserisci in Plans il migliore tra P ⊳⊲ R e R ⊳⊲ P
7.     Rimuovi P
```

**Euristiche** per velocizzare la ricerca:

- **Limitazione dei successori** (*Left-deep trees*): le operazioni di join vengono associate solo verso sinistra, creando **alberi left-deep** in cui ogni nodo ha un join come figlio sinistro e una relazione come figlio destro. Un albero left-deep permette l'uso dell'operatore IndexNestedLoop.

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 5.2]**: Albero left-deep a sinistra, albero bushy a destra.

- **Greedy search**: una volta espanso il nodo logico di costo minimo, gli altri non vengono più considerati, evitando backtracking. La soluzione trovata è tipicamente sub-ottimale, ma trovata in meno tempo.

- **Iterative full search**: approccio misto — la ricerca completa viene fatta fino a un numero predeterminato di livelli, poi si sceglie il nodo migliore e si continua come nella greedy search.

- **Interesting orders**: quando l'operatore di MergeJoin è disponibile e il risultato della query deve essere ordinato (per OrderBy o GroupBy), è utile organizzare la ricerca preservando anche i piani con un "interesting order" (ordine potenzialmente utile per il piano finale).

### Altre Query

**Query con GroupBy** — L'ottimizzatore produce un piano fisico per la SELECT, ordina i dati sugli attributi di raggruppamento, ed estende il piano con GroupBy e Project.

**Scambio con Join** — Può essere conveniente fare il GroupBy prima del Join (con il vincolo che il GroupBy sia fatto sugli stessi attributi usati nel Join). La condizione fondamentale perché questa equivalenza valga è che il join sia **unario** rispetto agli attributi $X$, producendo esattamente un record per ogni valore di quell'attributo.

**Scambio con Filter** — Si verifica se vale:
$$\sigma_{cond}(\gamma_{X,F}(E)) \equiv \gamma_{X,F}(\sigma_{cond}(E))$$

Due casi possibili:
- La selezione è sulle dimensioni di una tabella: può essere spostata sotto il GroupBy;
- La selezione è sui risultati di funzioni aggregate: vale solo per MAX e MIN:
$$\sigma_{M_b \geq v}(\gamma_{X, MAX(b) \text{ AS } M_b}(E)) \equiv \gamma_{X, MAX(b) \text{ AS } M_b}(\sigma_{b \geq v}(E))$$

> [!question] Possibili domande d'esame
>
> - Quali sono le quattro fasi dell'elaborazione di una query?
> - Cos'è una dipendenza funzionale? Elenca gli assiomi di Armstrong.
> - Cos'è la chiusura di un insieme di attributi? Come si usa per l'eliminazione del DISTINCT?
> - Quando può essere eliminato un GroupBy?
> - Cos'è la Where-subquery elimination? Cos'è il count bug?
> - Come funziona la ricerca completa per i piani fisici? Quali euristiche esistono?
> - Cos'è un albero left-deep e qual è il suo vantaggio?
> - Quando conviene scambiare GroupBy con un Join?
