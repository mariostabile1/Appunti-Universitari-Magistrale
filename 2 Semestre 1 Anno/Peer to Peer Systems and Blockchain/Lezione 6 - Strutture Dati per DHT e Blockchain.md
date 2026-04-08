# Strutture Dati per DHT e Blockchain

Questa lezione copre quattro strutture dati fondamentali per sistemi distribuiti e blockchain: **Hash Pointer**, **Filtri di Bloom**, **Merkle Tree** e **Merkle Patricia Trie**.

---

## Hash Pointer

> [!definition] Hash Pointer
>
> Un puntatore tradizionale indica *dove* si trova un dato. Un hash pointer indica *dove* si trova il dato **e** ne contiene l'hash crittografico, permettendo di verificare che non sia stato alterato. È un puntatore *tamper-evident*.

L'idea è costruire strutture dati concatenando componenti tramite hash pointer invece di puntatori normali. Per calcolare l'hash pointer a un blocco, si fa l'hash dell'intero blocco **incluso** il suo hash pointer al blocco precedente.

L'esempio più noto è la **blockchain**: ogni blocco contiene l'hash pointer al blocco precedente. Modificare il blocco $k$ invalida il suo hash, quindi non coincide con il puntatore nel blocco $k+1$. In una blockchain **Proof of Work**, ogni blocco contiene anche la prova che il PoW è stato eseguito con successo: modificare i dati richiede di rieseguire il PoW per tutti i blocchi successivi — computazionalmente impossibile.

Gli hash pointer funzionano su qualsiasi struttura senza cicli: liste, alberi, DAG. Applicazioni:
- **Bitcoin/Ethereum**: catena di blocchi con SHA-256 e RIPEMD-160 (doppio hash)
- **IPFS**: Merkle DAG
- **eMule** (*Advanced Intelligent Corruption Handling*): prima applicazione storica; verifica che i blocchi di un file scaricato dalla rete non siano stati manomessi, sfruttando i Merkle Tree

---

## Filtri di Bloom

### Il Problema

Dato un insieme $S = \{s_1, s_2, \ldots, s_n\}$ con $n$ molto grande, vogliamo rispondere alla domanda "l'elemento $k$ appartiene a $S$?" usando poca memoria. Memorizzare tutti gli elementi esplicitamente è proibitivo. La soluzione è un'approssimazione: si accetta un trade-off tra spazio occupato e probabilità di falsi positivi.

> [!definition] Filtro di Bloom
>
> Struttura dati probabilistica per membership query. Può rispondere:
> - "$k$ **non** appartiene a $S$" → **garantito** (nessun falso negativo)
> - "$k$ **forse** appartiene a $S$" → con una certa probabilità di falso positivo

### Costruzione

Un filtro di Bloom è un vettore $B[1 \ldots m]$ di $m$ bit (inizialmente tutti a 0) e $k$ funzioni di hash $h_1, \ldots, h_k$, indipendenti e uniformemente distribuite, ciascuna mappante in $[1, m]$. Non è necessario che siano crittografiche: bastano funzioni veloci. Esempio: $h_i(x) = MD5(x \| i)$.

**Inserimento** di un elemento $x \in S$: si calcolano $h_1(x), \ldots, h_k(x)$ e si impostano a 1 i bit corrispondenti. Un bit può essere target di più di un elemento.

**Lookup** di un elemento $y$: si calcolano $h_1(y), \ldots, h_k(y)$.
- Se **tutti** i bit corrispondenti sono 1 → $y$ è *probabilmente* in $S$
- Se **anche uno solo** è 0 → $y$ è *certamente* assente

I falsi positivi accadono quando tutti i bit di un elemento estraneo sono già stati impostati a 1 da altri elementi.

### Probabilità di Falso Positivo

L'analisi si può modellare col paradigma **balls and bins**: inserire $n$ elementi con $k$ funzioni equivale a lanciare $kn$ palline in $m$ secchi.

Con $n$ elementi inseriti, la probabilità che un bit specifico sia ancora a 0 è:

$$p' = \left(1 - \frac{1}{m}\right)^{kn} \approx e^{-kn/m}$$

La probabilità di un falso positivo (tutti i $k$ bit a 1 per un elemento assente) è:

$$P_{fp} = \left(1 - e^{-kn/m}\right)^k$$

Questa dipende da due parametri:
- **$m/n$** (bit per elemento): per $k$ fisso, al crescere di $m$ la $P_{fp}$ **decresce esponenzialmente**
- **$k$** (numero di funzioni di hash): fissato $m/n$, la $P_{fp}$ prima decresce poi **risale** all'aumentare di $k$. Con pochi bit per elemento (es. $m/n = 2$) troppe funzioni riempiono il filtro di 1 e peggiorano il risultato; con $m/n = 10$ aumentare $k$ diminuisce sempre la $P_{fp}$

> [!example] Regola pratica
>
> Il filtro diventa efficace quando $m = c \cdot n$ con $c$ costante. Con $m = 8n$ (8 bit per elemento) e $k \approx 5\text{–}6$ funzioni, la probabilità di falso positivo è circa $2\%$ — buon compromesso con un numero limitato di bit.

### Operazioni sugli Insiemi

| Operazione | Metodo | Note |
|---|---|---|
| Unione $S_1 \cup S_2$ | OR bit a bit di $B_1$ e $B_2$ | Esatto (stessi $m$ e $k$) |
| Intersezione $S_1 \cap S_2$ | AND bit a bit di $B_1$ e $B_2$ | Approssimato — un bit a 1 in entrambi può venire da elementi diversi non nell'intersezione |
| Cancellazione | Non supportata | Azzerare un bit rimuoverebbe anche altri elementi che lo condividono |

Per supportare la cancellazione si usano i **Counting Bloom Filter**: ogni entry è un contatore invece di un bit. L'inserimento incrementa, la cancellazione decrementa.

### Applicazioni Reali

| Sistema | Uso |
|---|---|
| **Bitcoin (SPV client)** | I client mobili costruiscono un filtro con gli indirizzi di interesse e lo inviano a un nodo completo (bandwidth saving + privacy). Il nodo filtra le transazioni rilevanti senza trasmettere l'intera blockchain |
| **Ethereum** | I *log bloom* negli header dei blocchi riassumono gli eventi degli smart contract. Esempio: trovare tutti i token venduti da un utente in un blocco di 500 transazioni — si interroga il log bloom per la presenza dell'utente e si analizza il blocco solo in caso di match, evitando il parsing sequenziale |
| **Google Chrome** | Filtro locale per verificare se un URL è in un database di siti malevoli, prima di fare una query remota al server |
| **Google BigTable** | Evita costosi disk lookup cercando prima nel filtro, aumentando le performance delle query al database |

---

## Merkle Tree

### Il Problema dell'Authenticated File Storage

Alice salva un file $F$ (contenuto $D$) su un server remoto e cancella la copia locale. Quando lo recupera, come verifica che il server non abbia restituito un file alterato $D' \neq D$?

- **Soluzione 1** (banale): non cancellare $D$ — inutile se manca la memoria
- **Soluzione 2** (hash singolo): Alice conserva $H(D)$; verifica confrontando $H(D') = H(D)$. Funziona per il file intero, ma se Alice vuole solo un frammento deve scaricare tutto e ricalcolare l'hash
- **Soluzione 3** (Merkle Tree): aggiungere struttura al commitment — non un singolo hash ma una gerarchia di hash

> [!definition] Merkle Tree
>
> Struttura dati introdotta da **Ralph Merkle nel 1979** per sintetizzare grandi quantità di dati con verifica efficiente. È un albero binario completo di hash costruito da un insieme iniziale $\{x_1, \ldots, x_n\}$ (con $n$ potenza di 2):
> - Le **foglie** contengono l'hash di ciascun dato: $y_i = H(x_i)$
> - I **nodi interni** contengono l'hash della concatenazione dei figli: $H(\text{figlio\_sx} \| \text{figlio\_dx})$
> - La **radice** (**Merkle Root Hash**) riassume crittograficamente l'intero dataset

### Costruzione

Con $n$ dati $x_1, x_2, \ldots, x_n$ e funzione hash $H$:
- Ogni nodo interno memorizza $H(x \| y) = H(\text{concatenazione dei figli})$
- Il commitment finale è il Merkle Root $y_{2n-1}$
- **Costo di costruzione**: $O(n)$ spazio e $O(n)$ hash
  
![[Pasted image 20260407112004.png]]
### Merkle Proof (Proof of Inclusion)

Protocollo file storage:
1. Alice invia $D$ al server; il server memorizza $(F, D)$
2. Alice calcola il **Merkle Tree Root** (MTR) da $D$, conserva MTR ($O(1)$ — 256 bit), cancella $D$
3. Più tardi: Alice chiede al server il chunk $x_i$
4. Il server (Prover) restituisce $x_i$ + **prova di inclusione** $p$ (gli hash dei fratelli lungo il percorso foglia→radice)
5. Alice (Verifier) verifica $p$ rispetto al MTR conservato

> [!example] Merkle Proof Concreta
>
> Per dimostrare che $D = x_4$ appartiene all'albero, si esibiscono i fratelli lungo il percorso: $y_3$, $y_9$, $y_{14}$.
>
> La verifica ricalcola:
> - $z_4 = H(D)$
> - $z_{10} = H(y_3 \| z_4)$
> - $z_{13} = H(y_9 \| z_{10})$
> - $z_{15} = H(z_{13} \| y_{14})$
>
> Si controlla che $z_{15} = y_{15}$ (la radice fidata). Se sì, il chunk è autentico.

| Proprietà | Valore |
|---|---|
| Costo costruzione | $O(n)$ spazio e hash |
| Spazio commitment | $O(1)$ — solo il Merkle Root (256 bit) |
| Dimensione prova | $O(\log n)$ hash |
| Costo di verifica | $O(\log n)$ operazioni |
| Falsi negativi | Impossibili — se $x_i \in \{x_1,\ldots,x_n\}$ la prova è sempre costruibile |
| Falsi positivi | Impossibili — una prova falsa richiederebbe trovare una collisione hash |

### Proof of Non-Membership

Per dimostrare che $\text{Data} \notin \{x_1, \ldots, x_n\}$: si ordinano le foglie e si trovano $x_i < \text{Data} < x_{i+1}$; si dimostrano le inclusioni di $x_i$ e $x_{i+1}$.

### Applicazioni

- **Bitcoin**: memorizza le transazioni in ogni blocco
- **Ethereum**: usa Merkle-Patricia Tries per stato e transazioni
- **IPFS**: Merkle DAG
- **Apache Cassandra**: verifica dell'integrità dei dati replicati

---

## Trie, Patricia Trie e Merkle Patricia Trie

### Trie (Prefix Tree / Radix Tree)

Il nome *trie* viene da "retrieval". È una struttura ad albero per stringhe in cui ogni arco è etichettato con un carattere (o lettera dell'alfabeto); un percorso dalla radice a un nodo *marcato* forma una stringa valida. Con l'alfabeto inglese ogni nodo può avere fino a 26 figli.

**Ricerca**: si scende dall'alto seguendo i caratteri della stringa; se si trova il percorso e il nodo finale è marcato → successo; se si è bloccati o il nodo finale è non marcato → fallimento.

**Problema**: la maggior parte dei nodi ha un solo figlio, creando lunghe catene inutili (es. "Ann", "Anna", "Annab", "Annabe", "Annabel" formano un percorso senza biforcazioni). Lo spazio richiesto è elevato.

### Patricia Trie

**PATRICIA** = *Practical Algorithm To Retrieve Information Coded In Alphanumeric* (1960). È una versione compressa del trie: le catene di nodi a figlio unico vengono **compresse in un singolo arco** con etichetta multipla (la concatenazione delle etichette dei nodi compressi). Il risultato è un albero in cui ogni nodo interno ha **almeno due figli**.

Il Patricia Trie funziona anche come **dizionario di coppie (chiave, valore)**: le chiavi sono le stringhe rappresentate nell'albero, i valori sono memorizzati nei nodi terminali. Usato in Ethereum per lo stato dei contratti.

### Nibble

In Ethereum le chiavi sono stringhe esadecimali suddivise in **nibble** (mezzo byte = 4 bit = 1 carattere esadecimale). Questo permette una condivisione più **granulare** dei prefissi: 16 possibili direzioni per ogni nodo interno (anziché 256 per i byte interi).

### Merkle Patricia Trie (Ethereum)

> [!definition] Merkle Patricia Trie
>
> Struttura ibrida introdotta nel **Yellow Paper di Ethereum** e ora usata nella maggior parte delle blockchain EVM-based. Combina:
> - La **compressione dei prefissi** del Patricia Trie → ricerca veloce e deterministica
> - La **garanzia crittografica** dei Merkle Tree → integrità e tamper-proof validation
>
> Usata da Ethereum per memorizzare lo stato degli account/contratti e le transazioni.

Le coppie (chiave, valore) in Ethereum possono essere:
- `(indirizzo account → saldo)`
- `(identificatore transazione → importo trasferito)`

I nodi del Merkle Patricia Trie sono di tre tipi:

| Tipo | Contenuto | Ruolo |
|---|---|---|
| **Leaf node** | Nibble finali della chiave + valore | Nodo terminale; memorizza il valore |
| **Extension node** (shared node) | Nibble condivisi (prefisso comune) + hash pointer al branch node | Compressione del prefisso comune |
| **Branch node** | Array di 16 elementi (uno per nibble 0–F) + eventuale valore | Punto di biforcazione tra prefissi |

> [!example] Costruzione MPT
>
> Inserzione di 'do' → creazione di un nodo foglia.
> Inserzione di 'puppy' → il prefisso comune (in esadecimale '646f') genera uno shared node + branch node; la nuova foglia viene collegata tramite un **hash pointer** (l'hash del nodo foglia stesso) → questa è la componente *Merkle* dell'albero. Il nibble '6' è condiviso tra tutti i prefissi → viene creato un nodo dedicato al livello di nibble.

Un singolo hash pointer alla radice del MPT garantisce l'integrità crittografica dell'intero stato di Ethereum: qualunque modifica cambia la radice.

---

## Riepilogo

| Struttura | Problema risolto | Complessità | Garanzie |
|---|---|---|---|
| **Hash Pointer** | Tamper-evidence su strutture concatenate | $O(1)$ per verifica | Qualunque manomissione è rilevabile |
| **Filtro di Bloom** | Membership query su insiemi enormi | $O(k)$ insert/lookup | No falsi negativi; falsi positivi controllabili con $m/n$ e $k$ |
| **Merkle Tree** | Autenticazione di frammenti di dati | $O(n)$ costruzione, $O(\log n)$ prova/verifica | Nessun falso positivo né negativo |
| **Merkle Patricia Trie** | Stato globale distribuito e verificabile | $O(\log n)$ lookup | Integrità crittografica + ricerca efficiente per prefisso |

> [!question] Possibili domande d'esame
>
> - Cos'è un **hash pointer**? In cosa differisce da un puntatore tradizionale e quali strutture dati può essere usato in?
> - Spiega la proprietà di **tamper-freeness** della blockchain basata su hash pointer. Cosa succede se si modifica un blocco a metà della catena?
> - Descrivi la struttura e il funzionamento di un **filtro di Bloom**: come si inserisce un elemento, come si esegue un lookup e perché sono possibili falsi positivi ma non falsi negativi?
> - Qual è la formula per la probabilità di falso positivo di un filtro di Bloom? Da quali parametri dipende e come si sceglie il valore ottimale di $k$?
> - Perché un filtro di Bloom non supporta la cancellazione? Qual è la struttura dati alternativa che risolve questo problema?
> - Descrivi come viene usato il filtro di Bloom negli **SPV client di Bitcoin**: qual è il vantaggio in termini di banda e privacy?
> - Costruisci concettualmente un **Merkle Tree** da un insieme di 8 elementi. Quali sono i costi di costruzione, la dimensione del commitment e il costo di una proof of inclusion?
> - Cos'è una **Merkle Proof** (proof of inclusion)? Descrivi il protocollo tra Prover e Verifier e spiega perché è impossibile ottenere falsi positivi.
> - Come si dimostra la **non-appartenenza** di un elemento a un Merkle Tree? Qual è il prerequisito?
> - Cos'è un **Merkle Patricia Trie**? Quali strutture combina e quali problemi risolve rispetto a un trie semplice? Quali sono i tre tipi di nodo e il loro ruolo?
