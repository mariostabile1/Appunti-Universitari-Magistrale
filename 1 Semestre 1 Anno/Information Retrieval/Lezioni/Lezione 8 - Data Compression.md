## <font color="#244061">Lezione 8: Compressione dei Dati</font>

### <font color="#366092">Architettura: Sharding e distribuzione</font>

I moderni sistemi di Information Retrieval devono gestire volumi di dati che superano la capacità di memoria e calcolo di una singola macchina. Per questo motivo, l'indice inverso viene partizionato e distribuito su più nodi fisici, una tecnica nota come **Sharding**. Esistono due approcci principali per il partizionamento. Il partizionamento per termini (global index organization) distribuisce il dizionario tra i nodi: ogni nodo è responsabile di un sottoinsieme di termini e delle relative posting list complete. Sebbene semplifichi la gestione delle statistiche globali, questo approccio soffre di problemi di concorrenza, poiché una query con più termini coinvolge quasi tutti i nodi.

L'approccio predominante è il partizionamento per documenti (local index organization). In questo schema, ogni nodo è responsabile di un sottoinsieme di documenti della collezione e costruisce un indice locale completo per quei documenti. Quando arriva una query, viene inviata a tutti i nodi in parallelo (scatter), e i risultati parziali (top-k) vengono poi uniti dal nodo broker (gather). Questo metodo scala meglio e bilancia il carico in modo più uniforme.
### <font color="#366092">Proprietà dei codici e Integer Encoders</font>

La compressione nell'indice inverso riguarda principalmente la riduzione dello spazio occupato dai _docID_ e dalle frequenze. I codici utilizzati devono possedere la proprietà di **decodificabilità univoca**, ovvero ogni sequenza di bit codificata deve corrispondere a un solo possibile messaggio originale. Inoltre, per garantire l'efficienza, si preferiscono codici **prefix-free** (o istantanei), dove nessuna parola di codice è prefisso di un'altra. Questo permette al decoder di identificare la fine di un codice e l'inizio del successivo senza dover leggere bit futuri.
#### <font color="#366092">Codice Unario e Codici Elias</font>

Il **Codice Unario** rappresenta un intero $x$ come $x-1$ bit a '1' seguiti da un terminatore '0' (es. $3 \rightarrow 110$). È efficiente solo per numeri molto piccoli. Per numeri più grandi si utilizzano i codici di Elias, che separano il numero in una componente di lunghezza e una di valore.

L'Elias Gamma ($\gamma$) codifica un intero $x$ rappresentando la sua lunghezza in unario seguita dal valore binario di $x$ troncato del bit più significativo. La lunghezza totale è $2\lfloor \log_2 x \rfloor + 1$.

L'Elias Delta ($\delta$) migliora la compressione per numeri grandi codificando la componente lunghezza stessa con Elias Gamma invece che in unario.
#### <font color="#366092">Variable-Byte (VByte)</font>

Mentre i codici Elias lavorano a livello di bit, il **Variable-Byte** lavora allineato al byte, rendendo la decodifica molto più veloce sulle CPU moderne. Ogni byte utilizza 7 bit per il payload (i dati) e 1 bit di controllo (flag). Il bit di controllo è settato a 1 se il byte successivo fa parte dello stesso numero, e a 0 se è l'ultimo byte del numero. Sebbene sprechi leggermente più spazio rispetto a Elias $\delta$, la velocità di decompressione lo rende molto diffuso.
### <font color="#366092">List Encoders: d-gaps e Combinatorial Lower Bound</font>

Nelle posting list, i _docID_ sono ordinati in modo crescente ($d_1 < d_2 < \dots < d_n$). Memorizzare i valori grezzi richiederebbe sempre più bit man mano che i docID crescono. Per comprimere efficacemente, si memorizzano i **d-gaps** (differenze): invece di salvare $d_i$, si salva $d_i - d_{i-1}$. Poiché le differenze sono generalmente numeri piccoli, possono essere codificate efficientemente con i metodi visti sopra (es. VByte o Elias).

L'efficienza di compressione di una lista è spesso confrontata con il **Combinatorial Lower Bound**. Se dobbiamo scegliere $n$ elementi da un universo di dimensione $U$, il numero minimo di bit necessari per rappresentare tale insieme, indipendentemente dall'ordine, è $\lceil \log_2 \binom{U}{n} \rceil$. Nessun metodo di compressione lossless può scendere sotto questa soglia teorica.
### <font color="#366092">Algoritmi per liste: Packing e Interpolazione</font>

Per migliorare ulteriormente le prestazioni rispetto alla lettura byte per byte, si utilizzano algoritmi che sfruttano la word del processore (32 o 64 bit) per "impacchettare" più interi.

**Simple9** e **Simple16** cercano di inserire il maggior numero possibile di d-gaps in una word di 32 bit. Utilizzano alcuni bit (selector) per indicare lo schema di partizionamento (es. "i restanti 28 bit contengono 28 numeri da 1 bit" oppure "14 numeri da 2 bit"). Questo riduce drasticamente le istruzioni di branching della CPU.

Il **PFor (Patched Frame of Reference)** è un algoritmo robusto agli outlier. Codifica la maggior parte dei numeri (es. il 90%) utilizzando un numero fisso di bit $b$ (frame of reference). I numeri che eccedono $b$ bit sono considerati eccezioni e vengono memorizzati a parte ("patched"). Questo garantisce una decompressione rapidissima per la maggior parte dei dati.

Il **Binary Interpolative Coding** è ideale per liste dense (es. liste di stopword o termini comuni). Dato un intervallo noto $[L, H]$ e sapendo che ci sono $n$ elementi in esso, l'algoritmo codifica l'elemento mediano $m$. Poiché $m$ deve trovarsi tra $L+n/2$ e $H-n/2$, l'intervallo di valori possibili è ridotto, permettendo di usare pochissimi bit. Il processo viene ripetuto ricorsivamente per le due metà sinistra e destra.
### <font color="#366092">Rappresentazioni Succinte: Elias-Fano</font>

La codifica **Elias-Fano** permette di avvicinarsi al Combinatorial Lower Bound supportando però l'accesso casuale (random access) agli elementi, cosa non possibile con i d-gaps standard. Elias-Fano divide ogni numero ordinato $x$ in due parti: i bit bassi (low bits) e i bit alti (high bits).

I Low Bits vengono memorizzati esplicitamente in un array $L$. Gli High Bits vengono rappresentati come un array di bit $H$ costruito concatenando rappresentazioni unarie delle quantità di numeri che condividono lo stesso prefisso (bucket).

Per migliorare le prestazioni su distribuzioni locali irregolari, si utilizza il Partitioned Elias-Fano, che divide la lista in blocchi e applica Elias-Fano su ciascun blocco indipendentemente, ottimizzando il numero di bit usati per i low bits in base alle caratteristiche locali dei dati.
### <font color="#366092">Query Rank e Select</font>

Per navigare efficientemente strutture succinte come i vettori di bit generati da Elias-Fano (in particolare il vettore $H$), sono necessarie due operazioni primitive che devono essere eseguite in tempo costante $O(1)$:

1. **Rank(B, i)**: Restituisce il numero di bit impostati a '1' nel vettore binario $B$ fino alla posizione $i$ inclusa.
    
2. **Select(B, i)**: Restituisce la posizione nel vettore $B$ dell'$i$-esimo bit impostato a '1'.

Queste operazioni sono implementate utilizzando strutture ausiliarie che memorizzano conteggi pre-calcolati per blocchi e super-blocchi del vettore di bit, permettendo di saltare il conteggio lineare e accedere direttamente alla porzione di interesse.