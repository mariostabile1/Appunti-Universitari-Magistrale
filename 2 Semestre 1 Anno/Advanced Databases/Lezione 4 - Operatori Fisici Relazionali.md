---
tags:
  - università/advanced-databases
  - operatori-fisici
  - join
  - query-optimizer
data: 2024-01-01
lezione: "4 - Operatori Fisici Relazionali"
professore: ""
---
# Operatori Fisici Relazionali

Una delle componenti più importanti di un DBMS è il **Query Manager**, responsabile di pianificare le query e dirigerle alle tabelle corrette. Una parte del Query Manager è il **Query Optimizer**, che ha il compito di determinare come eseguire una query nel modo più efficiente possibile, considerando i parametri fisici, l'organizzazione dei dati e la presenza o assenza di indici.

---

## Fattori di Selettività

Il **fattore di selettività** di una condizione è una stima della percentuale di record in una relazione che soddisfano quella condizione. Il modo più semplice per stimarla è assumere che i dati siano distribuiti uniformemente.

| Condizione | $sf$ calcolato | $sf$ approssimato |
|-----------|---------------|------------------|
| $A = v$ | $\frac{1}{N_{key}}$ | $\frac{1}{10}$ |
| $A > v$ | $\frac{max(A) - v}{max(A) - min(A)}$ | $\frac{1}{3}$ |
| $A < v$ | $\frac{v - min(A)}{max(A) - min(A)}$ | $\frac{1}{3}$ |
| $v_1 < A < v_2$ | $\frac{v_2 - v_1}{max(A) - min(A)}$ | $\frac{1}{4}$ |
| $A_1 = A_2$ | $\frac{1}{\max(N_{key}(A), N_{key}(B))}$ | $\frac{1}{10}$ |
| $\sigma_1 \wedge \sigma_2$ | $sf(\sigma_1) \cdot sf(\sigma_2)$ | — |
| $\sigma_1 \vee \sigma_2$ | $sf(\sigma_1) + sf(\sigma_2) - sf(\sigma_1) \cdot sf(\sigma_2)$ | — |

In molti casi, però, i valori degli attributi seguono distribuzioni non uniformi. La soluzione preferita dai DBMS è usare un **istogramma** con intervalli di valori per approssimare la distribuzione reale. Esistono due tipi:

- **Equi-width**: ogni bin contiene lo stesso numero di elementi $n$. La somma dei conteggi per ogni bin viene memorizzata;
- **Equi-height**: i bin sono divisi in modo che la "altezza" (somma dei conteggi) sia uguale per tutti. È più accurato dell'equi-width.

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 4.1]**: Un istogramma equi-height — i bin hanno la stessa altezza ma larghezze diverse, mostrando la distribuzione dei dati.

---

## Operatori Fisici

### Operatori per la Relazione

**TableScan(R)** — Restituisce tutti i record di $R$ nell'ordine in cui sono memorizzati. Costo: $C = N_{pag}(R)$. Dimensione del risultato: $E_{rec} = N_{rec}(R)$.

**SortScan(R, \{Ai\})** — Restituisce tutti i record di $R$ ordinati in ordine crescente sull'attributo $A_i$ (usando merge-sort). Costo:
$$C = \begin{cases} N_{pag}(R) & \text{se } N_{pag}(R) < B \\ 3 \cdot N_{pag}(R) & \text{se } N_{pag}(R) \leq B(B-1) \\ N_{pag}(R) + 2 \cdot N_{pag}(R) \cdot \log_{B-1}(N_{pag}(R)/B) & \text{altrimenti} \end{cases}$$

**IndexScan(R, I)** — Restituisce i record di $R$ ordinati per l'attributo su cui è definito l'indice $I$. Costo:
$$C = \begin{cases} N_{leaf}(I) + N_{pag}(R) & \text{se } I \text{ è clustered} \\ N_{leaf}(I) + N_{rec}(R) & \text{se } I \text{ è su una chiave di } R \\ N_{leaf}(I) + N_{key}(I) \cdot \phi(N_{rec}(R)/N_{key}(I), N_{pag}(R)) & \text{altrimenti} \end{cases}$$

**IndexSequentialScan(R, I)** — Restituisce i record di $R$ memorizzati con l'organizzazione primaria $I$, ordinati per valori di chiave primaria. Costo: $C = N_{leaf}(I)$.

### Operatori per la Proiezione

**Project(O, \{Ai\})** — Proietta i record di $O$ sugli attributi $\{A_i\}$. Costo: $C = C(O)$.

**IndexOnlyScan(R, I, \{Ai\})** — Restituisce i record di $R$ proiettati sugli attributi $\{A_i\}$ dell'indice $I$, accedendo solo all'indice. Costo: $C = N_{leaf}(I)$.

### Operatori per l'Eliminazione dei Duplicati

**Distinct(O)** — Restituisce i record di $O$ eliminando i duplicati. Richiede che i record siano già raggruppati (quando una collezione è ordinata è anche raggruppata). Costo: $C = C(O)$.

Dimensione del risultato: se $O$ ha un solo attributo, $E_{rec} = N_{key}(A)$; con più attributi: $E_{rec} = \min(|O|/2, \prod_i N_{key}(A_i))$.

**HashDistinct(O)** — Elimina i duplicati usando una tecnica hash in due fasi:

1. **Fase di partizionamento**: per ogni record di $O$ si applica la funzione hash $h_1$, distribuendo i record uniformemente su $B$ pagine. Quando una pagina $i$ è piena, viene scritta nel file di partizione $T_i$;
2. **Fase di eliminazione duplicati**: per ogni file $T_i$, si leggono i record pagina per pagina, eliminando i duplicati usando la funzione hash $h_2$.

Costo: $C = C(O) + 2 \cdot N_{pag}(O)$.

### Operatori per l'Ordinamento

**Sort(O, \{Ai\})** — Restituisce i record di $O$ ordinati sugli attributi $\{A_i\}$ usando merge-sort:

$$C = \begin{cases} C(O) & \text{se } N_{pag}(R) < B \\ C(O) + 2 \cdot N_{pag}(O) & \text{se } N_{pag}(O) \leq B(B-1) \\ C(O) + 2 \cdot N_{pag}(O) \cdot \log_{B-1}(N_{pag}(O)/B) & \text{altrimenti} \end{cases}$$

### Operatori per la Selezione

**Filter(O, σ)** — Restituisce i record di $O$ che soddisfano la condizione $\sigma$. Costo: $C = C(O)$. Dimensione: $E_{rec} = sf(\sigma) \cdot N_{rec}(O)$.

**IndexFilter(R, I, σ)** — Restituisce i record di $R$ che soddisfano la condizione $\sigma$ usando l'indice $I$. La condizione deve coinvolgere solo gli attributi nel prefisso della chiave dell'indice. Appare sempre come nodo foglia in un piano fisico.

Costo: $C = C_I + C_D$, dove:
- Se **clustered**: $C_I = sf(\sigma) \cdot N_{leaf}(I)$, $C_D = sf(\sigma) \cdot N_{pag}(R)$
- Se **unclustered**: $C_I = sf(\sigma) \cdot N_{leaf}(I)$, $C_D = sf(\sigma) \cdot N_{key}(I) \cdot \phi(N_{rec}(R)/N_{key}(I), N_{pag}(R))$
- Se definito su una **chiave di R**: $C_D = sf(\sigma) \cdot N_{rec}(R)$

**IndexSequentialFilter(R, I, σ)** — Restituisce i record di $R$ con organizzazione primaria $I$ che soddisfano $\sigma$. Costo: $C = sf(\sigma) \cdot N_{leaf}(I)$.

**IndexOnlyFilter(R, I, \{Ai\}, σ)** — Restituisce i record della proiezione su $R$ che soddisfano $\sigma$, usando solo l'indice. Costo: $C = sf(\sigma) \cdot N_{leaf}(I)$.

### Operatori per il Raggruppamento

**GroupBy(O, \{Ai\}, \{fi\})** — Restituisce i record di $O$ ordinati su $\{A_i\}$, applicando le funzioni di aggregazione $\{f_i\}$. I record devono essere già ordinati. Costo: $C = C(O)$.

**HashGroupBy(O, \{Ai\}, \{fi\})** — Raggruppa i record di $O$ per $\{A_i\}$ usando due fasi simili a HashDistinct:
1. **Partizionamento**: usando $h_1$;
2. **Raggruppamento**: per ogni partizione, si raggruppano i record usando $h_2$ sugli attributi di raggruppamento.

Costo: $C = C(O) + 2 \cdot N_{pag}(O)$.

---

### Operatori per il Join

#### NestedLoop(OE, OI, CJ)

Algoritmo:
```
for r ∈ OE do
    for s ∈ OI do
        if CJ: aggiungere ⟨r, s⟩ al risultato
    end for
end for
```

Costo: $C = C(O_E) + E_{rec}(O_E) \cdot C(O_I)$

Dimensione: $E_{rec} = sf(C_J) \cdot E_{rec}(O_E) \cdot E_{rec}(O_I)$

#### PageNestedLoop(OE, OI, CJ)

Scansiona $O_I$ una volta per ogni **pagina** di $O_E$ (invece che per ogni record). Algoritmo:
```
for pr di OE do
    for ps di OI do
        for r ∈ pr do
            for s ∈ ps do
                if CJ: aggiungere ⟨r, s⟩ al risultato
```

Costo: $C = C(O_E) + N_{pag}(O_E) \cdot C(O_I)$

Il costo è inferiore quando l'operando esterno ha meno pagine.

#### BlockNestedLoop(OE, OI, CJ)

Estende il PageNestedLoop usando più pagine del buffer ($B$ pagine per l'operando esterno, 1 pagina per l'input di $S$, 1 pagina come output buffer).

Costo: $C = N_{pag}(R) + \lceil N_{pag}(R)/B \rceil \cdot N_{pag}(S)$

Se $B$ pagine contengono una delle due relazioni: $C = N_{pag}(R) + N_{pag}(S)$.

> [!warning] Attenzione
>
> BlockNestedLoop non è conveniente quando gli operatori richiedono troppe pagine (cioè $N_{pag}(R) \geq B^2$).

#### IndexNestedLoop(OE, OI, CJ)

Richiede un indice sulla colonna di join dell'operando interno. Algoritmo:
```
for r ∈ OE do
    for s ∈ IndexFilter(OI, I, OE.e1 = OI.i1) do
        aggiungere ⟨r, s⟩ al risultato
```

Costo: $C = C(O_E) + E_{rec}(O_E) \cdot (C_I + C_D)$

dove $C_I$ e $C_D$ sono i costi per recuperare i record dell'indice e i dati dal disco.

#### MergeJoin(OE, OI, CJ)

Richiede che $O_E$ e $O_I$ siano ordinati sugli stessi attributi di join, e che nell'attributo di join, $O_E.A_i$ sia una chiave di $O_E$. Poiché l'attributo di join ha valori distinti in $O_E$, l'algoritmo legge i record di $O_E$ uno per uno, e legge tutti i record di $O_I$ con gli stessi valori.

Costo: $C = C(O_E) + C(O_I)$

#### HashJoin(OE, OI, CJ)

Due fasi:
1. **Partizionamento**: i record di entrambi gli operandi vengono partizionati usando $h_1$;
2. **Probing (matching)**: per ogni partizione $B_i$, i record di $O_E$ vengono inseriti in una hash table in buffer usando $h_2$. I record di $O_I$ vengono letti una pagina alla volta, $h_2$ viene applicato, e se c'è una corrispondenza il join viene aggiunto al risultato.

Costo (assumendo $N_{pag}(O_E)/B < B$ e distribuzione uniforme):
$$C = C(O_E) + C(O_I) + 2 \cdot (N_{pag}(O_E) + N_{pag}(O_I))$$

Caso generale:
$$C = (\log_B(N_{pag}(O_E)) \cdot 2 - 2) \cdot (N_{pag}(O_E) + N_{pag}(O_I))$$

Se $N_{pag}(O_E) < B$, il costo è 0 (tutto in memoria).

> [!abstract] Sintesi degli Operatori di Join
>
> | Operatore | Requisiti | Costo tipico | Nota |
> |-----------|-----------|--------------|------|
> | NestedLoop | Nessuno | Alto | Più semplice |
> | PageNestedLoop | Nessuno | Medio | Meglio se $O_E$ ha poche pagine |
> | BlockNestedLoop | $B+2$ pagine | Medio-basso | Ottimale se $O_E$ entra in buffer |
> | IndexNestedLoop | Indice su $O_I$ | Basso | Efficiente con indice clustered |
> | MergeJoin | Ordinamento su attributo join | Basso | Efficiente se già ordinati |
> | HashJoin | Distribuzione uniforme | Basso | Ottimale per equi-join |

> [!question] Possibili domande d'esame
>
> - Cos'è il fattore di selettività? Come si calcola per diverse condizioni?
> - Differenza tra istogrammi equi-width e equi-height.
> - Descrivi il funzionamento e il costo di TableScan, SortScan, IndexScan.
> - Come funziona HashDistinct? Qual è il suo costo?
> - Descrivi i sei operatori di join e il loro costo. Quando conviene usarne uno rispetto a un altro?
> - Cos'è il BlockNestedLoop e quando non è conveniente?
> - Descrivi l'algoritmo HashJoin con le sue due fasi.
