## Introduzione al Learned Sparse Retrieval (LSR)

L'avvento dei modelli linguistici pre-trained ha rivoluzionato l'Information Retrieval portando alla nascita del Neural Information Retrieval (NIR). In questo ambito, il Learned Sparse Retrieval (LSR) emerge come un paradigma che trasforma il testo in vettori sparsi all'interno di uno spazio vettoriale dove ogni dimensione è mappata direttamente a un termine del vocabolario del modello. A differenza dei modelli densi, le rappresentazioni sparse sono interpretabili per design: un valore diverso da zero in una coordinata indica che il termine corrispondente è semanticamente rilevante per l'input. La similarità tra questi vettori viene calcolata tramite il prodotto interno, trasformando il recupero dei documenti in un problema di Maximum Inner Product Search (MIPS).

Nonostante la loro natura sparsa sembri ideale per l'uso con gli indici invertiti classici, i vettori LSR presentano sfide strutturali. I modelli tradizionali come BM25 si basano su distribuzioni di frequenza dei termini di tipo Zipfiano e query brevi, assunzioni che spesso non valgono per le rappresentazioni sparse apprese, dove le query sono più lunghe e i pesi seguono distribuzioni statistiche differenti. Questa discrepanza porta a latenze elevate quando si utilizzano algoritmi di potatura dinamica standard come WAND o MaxScore.

## Il Fenomeno della Concentration of Importance

Un'osservazione empirica fondamentale per l'efficienza dei sistemi LSR è la cosiddetta concentration of importance. Questo fenomeno indica che le tecniche di rappresentazione sparsa, come SPLADE, tendono a concentrare una quantità sproporzionata della massa totale $L_{1}$ del vettore su un sottoinsieme molto ridotto di coordinate. In termini pratici, l'analisi mostra che i primi 50 ingressi di un documento (circa il 30% dei suoi valori non nulli) sono sufficienti per preservare il 75% della sua massa $L_{1}$.

Questa proprietà ha implicazioni dirette sul calcolo della rilevanza: il prodotto interno tra query e documenti può essere approssimato con estrema precisione utilizzando solo una frazione delle coordinate più significative. Ad esempio, mantenendo solo il top 10% delle coordinate più importanti, si preserva mediamente l'85% del prodotto interno totale. Questa intuizione permette di progettare algoritmi di Approximate Nearest Neighbor (ANN) che sacrificano una precisione trascurabile per ottenere enormi guadagni in termini di throughput e latenza.

## Architettura e Algoritmo SEISMIC

L'algoritmo SEISMIC (Spilled Clustering of Inverted Lists with Summaries for Maximum Inner Product Search) introduce una nuova organizzazione dell'indice invertito che combina potatura statica e dinamica. La struttura si basa sull'integrazione di due componenti principali: un indice invertito modificato, che identifica i candidati, e un forward index, che memorizza i vettori esatti per il calcolo finale dei punteggi.

### Organizzazione dell'Indice e Blocking

L'innovazione principale risiede nella suddivisione delle liste invertite in blocchi geometricamente coesi. Invece di una scansione lineare, le liste vengono partizionate tramite un algoritmo di clustering (una variante shallow di K-Means) che raggruppa i documenti con rappresentazioni locali simili. Ogni blocco $B$ è corredato da un summary vector $\phi(B)$, un "vettore di sintesi" le cui coordinate rappresentano il valore massimo di quella dimensione tra tutti i documenti nel blocco. Matematicamente, questo vettore funge da limite superiore conservativo per il prodotto interno: $\langle q, \phi(B) \rangle \ge \langle q, x \rangle$ per ogni documento $x$ nel blocco.

### Processo di Query e Potatura Dinamica

Durante l'elaborazione di una query, SEISMIC utilizza la concentration of importance per selezionare solo le coordinate più rilevanti della query stessa. L'algoritmo attraversa l'indice una coordinata alla volta e valuta i blocchi tramite i loro summary vector. Se il prodotto interno tra la query e il riassunto del blocco non supera una determinata soglia (legata al punteggio minimo presente nella heap dei risultati top-k), l'intero blocco viene saltato, risparmiando una quantità significativa di calcoli. Se invece il blocco è promettente, SEISMIC accede al forward index per recuperare i vettori completi e calcolare i prodotti interni esatti.

### Ottimizzazione e Compressione

Per gestire lo spazio occupato dai summary vector, che tenderebbero a diventare densi all'aumentare della dimensione del blocco, SEISMIC applica tecniche di compressione come la potatura dei sotto-vettori di massa $\alpha$ e la quantizzazione scalare a 8-bit. La quantizzazione riduce l'impronta di memoria dei riassunti di un fattore 4 senza compromettere l'efficacia del sistema. Inoltre, l'implementazione sfrutta istruzioni di prefetching per mitigare i problemi di latenza dovuti ai cache miss durante l'accesso al forward index, dato che i documenti dello stesso blocco potrebbero non essere contigui in memoria.

---

**Sintesi dell'Architettura SEISMIC**

| **Componente**       | **Funzione Principale**       | **Tecnica Utilizzata**                       |
| -------------------- | ----------------------------- | -------------------------------------------- |
| **Indice Invertito** | Selezione dei candidati       | Partizionamento in blocchi geometrici        |
| **Summary Vectors**  | Potatura dinamica dei blocchi | Vettore dei massimi con quantizzazione 8-bit |
| **Forward Index**    | Calcolo esatto dei punteggi   | Lookup table dei vettori originali           |
| **Logica di Query**  | Efficienza temporale          | Coordinate-at-a-time con early skipping      |
