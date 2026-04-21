# Crittografia per P2P e Blockchain

Due strumenti crittografici sono alla base di blockchain e DHT: le **funzioni di hash** (collegano i blocchi rendendoli tamper-proof) e le **firme digitali** (impediscono agli utenti di ripudiare le proprie azioni). Strumenti più avanzati — zero knowledge, strutture dati autenticate, accumulatori crittografici — verranno trattati più avanti nel corso.

---

## Funzioni di Hash Crittografiche

Una funzione di hash converte una stringa binaria di lunghezza arbitraria (anche 0) in una stringa di lunghezza fissa. L'output — detto *digest*, *fingerprint* o informalmente *checksum* — ha sempre la stessa dimensione indipendentemente dall'input (video, audio, testo, eseguibili).

Per le strutture dati ordinarie, una funzione hash deve essere: deterministica, efficiente da calcolare (es. `y = x mod table_dim`), pseudo-casuale per distribuire uniformemente gli elementi, minimizzare le collisioni. Le funzioni hash **crittografiche** aggiungono ulteriori proprietà di sicurezza.

### Proprietà Fondamentali

**Determinismo** — La stessa funzione hash applicata più volte allo stesso documento produce sempre lo stesso risultato.

**Fast Computation** — L'hash deve essere computazionalmente efficiente da calcolare.

**One-Way (Pre-image Resistance)** — Dato un hash $y$, deve essere computazionalmente impossibile trovare un $x$ tale che $H(x) = y$. Con SHA-1 (160 bit) un attacco brute-force richiederebbe $2^{71}$ anni.

**Avalanche Effect** — Un singolo bit di differenza nell'input produce un hash completamente diverso; cambiare anche solo 1 bit modifica tutto l'output.

**Collision Resistance** — Le collisioni esistono matematicamente (per il *Pigeonhole Principle*: con più input che output, almeno un output deve corrispondere a più di un input), ma devono essere computazionalmente impossibili da costruire intenzionalmente.

> [!example] Pigeonhole Principle
>
> Se si scelgono 51 numeri interi tra 1 e 100, almeno due devono essere consecutivi. Dimostrazione: le "buche" sono le coppie {1,2}, {3,4}, ..., {99,100} — 50 coppie per 51 numeri. Per il principio del piccione, almeno una buca contiene due piccioni.

**Weak Collision Resistance (Second Pre-image Resistance)** — Dato un input $x$ noto, deve essere difficile trovare un $x' \neq x$ tale che $H(x') = H(x)$. Questo previene, ad esempio, la distribuzione di software corrotto spacciato per autentico: un attaccante non può generare un file malevolo con lo stesso hash del software legittimo.

### Il Paradosso del Compleanno e la Sicurezza Effettiva

Per trovare una collisione in modo garantito bastano $2^n + 1$ input distinti (per Pigeonhole). Tuttavia il **Birthday Paradox** abbassa drasticamente il limite pratico:

In una stanza di $n$ persone scelte a caso, $n = 23$ è sufficiente perché la probabilità che due condividano il compleanno superi il 50%. Questo perché la seconda persona deve evitare 1 compleanno su 365, la terza 2 su 365, ecc.; il prodotto delle probabilità di "nessuna collisione" scende rapidamente. Formalmente, con $\sqrt{2^n} = 2^{n/2}$ input casuali si ha già alta probabilità di collisione.

Per una funzione hash con $n$-bit di output, bastano $\approx 2^{n/2}$ input casuali per trovare una collisione con probabilità 0,5. La sicurezza effettiva è quindi **metà dei bit di output**.

> [!warning] Quantificazione brute force
>
> Se un computer calcola 10.000 hash/sec, calcolare $2^{128}$ hash richiederebbe $10^{27}$ anni. Anche se tutti i computer mai costruiti dall'umanità avessero calcolato sin dall'inizio dell'universo, la probabilità di aver trovato una collisione sarebbe infinitesimamente piccola.

| Algoritmo | Bit output | Sicurezza effettiva | Stato |
|---|---|---|---|
| MD2, MD4 | 128 bit | 64 bit | Ritirati — vulnerabili |
| MD5 | 128 bit | 64 bit | Vulnerabile (ok per app non di sicurezza) |
| SHA-1 | 160 bit | 80 bit | Ritirato — debole |
| SHA-256 | 256 bit | 128 bit | Usato da Bitcoin — sicuro |
| SHA-512 | 512 bit | 256 bit | Corrente — massima sicurezza |

Almeno **80 bit** di sicurezza sono necessari. Le funzioni hash hanno storicamente una vita utile di circa **10 anni**. Lo standard attuale è **SHA-3** (Keccak), adottato anche da Ethereum.

> [!warning] Hash non crittografici
>
> La **parità a blocco a 8 bit**: è banale trovare una collisione invertendo un numero pari di bit nella stessa colonna. Il **CRC** (Cyclic Redundancy Check) è il resto di una divisione polinomiale lunga — ottimo per rilevare burst error nelle comunicazioni, ma facile da collidere intenzionalmente. CRC è stato usato erroneamente nel protocollo **WEP** (Wired Equivalent Privacy) dove si richiedeva integrità crittografica.

### Cryptanalysis

Oltre al brute force, la **crittoanalisi** cerca debolezze logiche nell'algoritmo: scorciatoie, buchi nella funzione. Una funzione si dice *broken* quando è possibile trovare collisioni significativamente più velocemente del brute force.

### Proprietà Avanzate

**Hiding** — Dato $H(R \| x)$, deve essere impossibile ricavare informazioni su $x$. Formalmente: $H$ è *hiding* se, scelto $R$ da una distribuzione con alta min-entropy (nessun valore più probabile degli altri), dato $H(R \| x)$ è computazionalmente impossibile trovare $x$.

Le funzioni base non garantiscono hiding se lo spazio di input è piccolo e prevedibile (es. password), rendendole vulnerabili ai **Rainbow Table Attack**: si pre-calcolano le hash di tutte le password possibili in una tabella; al login si cerca il hash nel database. La soluzione è concatenare un valore casuale $R$ a 256 bit ad alta min-entropy: $H(R \| x)$, rendendo lo spazio di ricerca enorme.

**Puzzle-Friendliness** — Per qualsiasi output target $y$ e valore casuale $k$ ad alta min-entropy, trovare $x$ tale che $H(k \| x) = y$ richiede un attacco esaustivo in tempo $\approx 2^n$, senza scorciatoie algoritmiche. Questa proprietà implica che nessuna strategia di risoluzione è significativamente migliore della ricerca esaustiva.

---

## Applicazioni delle Funzioni di Hash

### Applicazioni non-blockchain

**Gestione password** — I sistemi memorizzano $H(\text{password})$ invece del testo in chiaro. Anche in caso di compromissione del database, le password originali non sono recuperabili.

**Integrity checks (anti-tampering)** — Si calcolano checksum per i file da scaricare o trasmettere. Prima di aprire o eseguire un file, si calcola il suo hash e lo si confronta con il checksum atteso: se coincidono, il file non è stato alterato o corrotto.

**Data deduplication** — Se due file hanno lo stesso hash, sono identici — senza dover confrontare i file interi. Usato ad esempio da **eMule** con MD5 per verificare che due file siano identici anche se descritti da keyword diverse.

**DHT** — Le chiavi vengono hashate per localizzare il nodo responsabile in modo efficiente e deterministico.

### Applicazioni blockchain

**Hash Pointers** — Un hash pointer è sia un riferimento a una posizione (dove si trova il dato) sia un hash crittografico del dato in quella posizione. Permette di verificare che i dati non siano stati alterati. Bitcoin usa una **hash chain** (blockchain) per memorizzare il ledger delle transazioni: ogni blocco contiene l'hash del blocco precedente, garantendo la *tamper-freeness* — modificare un blocco invalida tutti i successivi.

![[Pasted image 20260407111722.png]]

**Commitment Scheme** — Permette di "chiudere in una busta" una decisione senza terze parti fidate.

**Hash Puzzles (Proof of Work)** — Trovare $x$ tale che $H(r \| x) \in S$.

---

## Commitment Scheme

### Motivazione: Sasso-Carta-Forbici online

Alice e Bob giocano a sasso-carta-forbici via Internet senza terze parti fidate. Il problema: chi va per primo perde, perché l'avversario può adattarsi. La soluzione è che chi va per primo *si impegna* a una scelta senza rivelarla.

Il flusso con commitment crittografico ($R_A$ = valore casuale scelto da Alice):

$$A \to B: h_A = H(R_A \| \text{paper})$$
$$B \to A: \text{scissors}$$
$$A \to B: R_A, \text{paper}$$

Bob verifica che $h_A = H(R_A \| \text{paper})$. Se sì, sa che Alice non ha barato.

- Bob non riesce a determinare la scelta di Alice perché non conosce $R_A$ (*hiding* + *pre-image resistance*)
- Alice non può cambiare idea dopo aver ricevuto "scissors": dovrebbe trovare $R'_A$ tale che $H(R_A \| \text{paper}) = H(R'_A \| \text{stone})$ → violazione della **second-preimage resistance**

La proprietà per cui il mittente non può cambiare il valore impegnato si chiama **binding**.

### API e Implementazione

```
com   ← commit(value, nonce)     // sigilla il valore
                                 // pubblica com
match ← verify(com, nonce, value) // apre la busta
```

Implementazione con funzione hash:
- `commit(msg, nonce)` = $H(\text{msg} \| \text{nonce})$
- `verify(com, nonce, msg)` = $(H(\text{msg} \| \text{nonce}) == \text{com})$

L'uso del `nonce` (number used once) garantisce la proprietà hiding anche quando lo spazio dei messaggi è piccolo.

---

## Search Puzzle (Proof of Work)

Un search puzzle consiste in: una funzione hash crittografica $H$, un valore casuale $r$, un insieme target $S$. La soluzione è un valore $x$ tale che:

$$H(r \| x) \in S$$

Si tratta di un **partial pre-image attack**: si deve trovare parte dell'input affinché l'output appartenga a un insieme (non a un singolo valore come nella pre-image resistance). La difficoltà si modula definendo la dimensione di $S$: un $S$ più grande rende il puzzle più facile. In **Bitcoin**, $S$ è definito dal numero di zeri iniziali richiesti nell'hash SHA-256 del blocco.

La puzzle-friendliness garantisce che non esistano scorciatoie: l'unico metodo è il tentativo esaustivo.

---

## Crittografia Asimmetrica e Firme Digitali

La crittografia asimmetrica usa una coppia di chiavi: una **chiave privata** ($K^-$), nota solo al proprietario, e una **chiave pubblica** ($K^+$), derivata matematicamente da essa ma non invertibile. La proprietà fondamentale è la **reciprocità**: ciò che una chiave cifra, l'altra decifra, e viceversa.

**Cifratura (riservatezza)** — Alice cifra un messaggio con la chiave *pubblica* di Bob; solo Bob, con la sua chiave *privata*, può decifrarlo.

**Vantaggi rispetto alla crittografia simmetrica**: nessun bisogno di concordare preventivamente una chiave condivisa. Chi vuole ricevere messaggi cifrati deve solo rendere pubblica la propria chiave pubblica. Finché la chiave privata è tenuta segreta, nessun altro può decifrare.

### Firme Digitali

> [!definition] Firma Digitale
>
> Meccanismo equivalente a una firma autografa ma molto più sicuro. Fornisce tre garanzie: **autenticazione** (il messaggio è stato creato dal mittente riconosciuto), **non ripudio** (il mittente non può negare di aver firmato — la firma può essere portata in tribunale come prova), **integrità** (il messaggio non è stato alterato durante la trasmissione).

Il flusso è l'**inverso della cifratura**: il mittente firma con la propria chiave *privata*, chiunque può verificare con la chiave *pubblica*.

**Firma naive**: Bob firma $m$ cifrando con la sua chiave privata $K^-_B$, creando $K^-_B(m)$. Invia ad Alice la coppia $(m, K^-_B(m))$. Alice verifica applicando la chiave pubblica: $K^+_B(K^-_B(m)) = m$. Se coincide, sa che solo Bob — possessore di $K^-_B$ — ha potuto firmare.

Poiché firmare asimmetricamente un messaggio lungo è computazionalmente costoso, in pratica si firma solo il **digest**:

$$\text{firma} = K^-_B(H(m))$$

Il verificatore calcola $H(m)$ e lo confronta con $K^+_B(\text{firma})$: se coincidono, l'autenticità è garantita.

Per avere simultaneamente **riservatezza + integrità**: il mittente firma con la propria chiave privata e cifra il pacchetto completo con la chiave pubblica del destinatario. Il destinatario decifra con la propria chiave privata e verifica con la chiave pubblica del mittente.

### API Standard

```
(sk, pk) := generateKeys(keysize)    // sk = signing key, pk = public key
sig      := sign(sk, message)        // cifra con sk → firma
isValid  := verify(pk, message, sig) // decifra sig con pk, confronta con message
```

Proprietà richiesta: `verify(pk, message, sign(sk, message)) == true`.

**Sfida principale**: cosa impedisce a un avversario di imparare a firmare messaggi analizzando la chiave pubblica? Le costruzioni basate su problemi hard (fattorizzazione, logaritmo discreto) rendono questo computazionalmente impossibile.

| Algoritmo | Base matematica | Note |
|---|---|---|
| RSA | Fattorizzazione di numeri primi (one-way trapdoor function) | Standard storico |
| DSA | Logaritmo discreto | Standard NIST |
| ECDSA | Logaritmo discreto su curve ellittiche | Usato da Bitcoin |

**Bitcoin e ECDSA**: ogni transazione Bitcoin contiene in input una firma e una chiave pubblica; in output il codice (script) per la procedura di verifica.

### Certification Authorities

La debolezza degli schemi asimmetrici è la **weak authentication**: verificare la firma garantisce solo che chi ha firmato possiede la chiave privata corrispondente — non che sia davvero chi afferma di essere.

> [!example] Pizza Prank
>
> Alice crea un ordine di pizze a nome di Bob, lo firma con la *propria* chiave privata, e invia alla pizzeria la propria chiave pubblica spacciandola per quella di Bob. La pizzeria verifica la firma (correttamente), consegna le pizze a Bob. La weak authentication non basta.

Le **Certification Authority (CA)** risolvono il problema: emettono certificati digitali che legano crittograficamente un'identità alla sua chiave pubblica, firmando essi stessi il certificato con la propria chiave privata (di cui tutti si fidano).

---

## Hash vs Cifratura

> [!note] Differenza concettuale
>
> La **cifratura** è bidirezionale: con la chiave giusta si cifra e si decifra. L'**hashing** è irreversibile per definizione: non esiste un'operazione di "de-hashing". Sono strumenti complementari, non intercambiabili.

---

## RIPEMD-160 e Bitcoin

Bitcoin usa **due** funzioni hash: SHA-256 per il Proof of Work e **RIPEMD-160** (160 bit, output) per la derivazione degli indirizzi wallet. Il doppio hash `RIPEMD160(SHA256(pubkey))` produce l'indirizzo Bitcoin a partire dalla chiave pubblica.

> [!question] Possibili domande d'esame
>
> - Elenca e descrivi le proprietà fondamentali di una funzione hash crittografica: determinismo, one-way, avalanche effect, collision resistance, second pre-image resistance.
> - Cos'è il **Paradosso del Compleanno** e come abbassa la sicurezza effettiva delle funzioni hash? Qual è la sicurezza effettiva in bit di SHA-256?
> - Perché le funzioni hash non crittografiche (es. CRC, parità) non sono adatte a contesti di sicurezza? Porta un esempio di sistema reale in cui è stato usato erroneamente CRC.
> - Cos'è la proprietà **hiding** di una funzione hash? Perché è necessario concatenare un nonce ad alta min-entropy? Quale attacco diventa possibile senza questa precauzione?
> - Cos'è la proprietà **puzzle-friendliness**? Come viene sfruttata nella Proof of Work di Bitcoin?
> - Descrivi il protocollo del **commitment scheme** applicato al gioco sasso-carta-forbici. Quali proprietà della funzione hash garantiscono rispettivamente hiding e binding?
> - Spiega il meccanismo delle **firme digitali** in Bitcoin: come si genera una firma, come si verifica, e perché in pratica si firma il digest invece del messaggio originale?
> - Qual è la differenza concettuale tra **cifratura** e **hashing**? Perché non sono intercambiabili?
> - Cos'è una **Certification Authority** e perché è necessaria nella crittografia asimmetrica? Qual è l'attacco che mitiga?
> - Come viene derivato un indirizzo Bitcoin a partire dalla chiave privata? Quali funzioni hash vengono usate e in quale ordine?
