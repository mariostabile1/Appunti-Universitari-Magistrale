# Advanced Bitcoin Scripts e SPV Clients

Tre costrutti avanzati di Bitcoin — **multisignature**, **hash lock** e **time lock** — sono alla base della Lightning Network, la principale soluzione di scalabilità per Bitcoin.

---

## Multisignature (Multisig)

In un protocollo multi-firma, un gruppo di firmatari autorizza collettivamente una transazione; la verifica avviene tramite le chiavi pubbliche di tutti i partecipanti. L'approccio ingenuo concatena le firme individuali, ma la dimensione cresce linearmente con il numero di firmatari. L'ideale sarebbe una dimensione fissa indipendente dal numero di partecipanti.
![[Pasted image 20260407113044.png]]
Bitcoin ha adottato inizialmente la soluzione più semplice con **ECDSA**: firme multiple separate, non aggregate. Le **firme Schnorr**, che permettono l'aggregazione, sono state introdotte solo in seguito con il protocollo **Taproot**.

Un indirizzo multisig accoppia un indirizzo Bitcoin a un locking script che richiede **M** firme valide su **N** chiavi pubbliche associate.

### Script Multisig

**Locking script** M-of-N:
```
M <PubKey1> … <PubKeyN> N OP_CHECKMULTISIG
```

**Unlocking script** (qualsiasi combinazione valida di M firme):
```
<Signature 1> … <Signature M>
```

Esempio 2-of-3 con Alice, Bob e Judy:
- Locking: `2 <PkA> <PkB> <PkC> 3 OP_CHECKMULTISIG`
- Unlocking valido: `<Sig A> <Sig C>` — qualsiasi due delle tre

### Casi d'uso

| Schema | Caso d'uso |
|---|---|
| **1-of-2** | Conto corrente coniugale per piccole spese — basta la firma di uno |
| **2-of-2** | Conto risparmio — entrambi devono approvare |
| **2-of-2** | Wallet con autenticazione a due fattori (laptop + smartphone) — un trojan sul telefono non basta per rubare i fondi |
| **2-of-2** | Blocco fondante della Lightning Network |
| **2-of-3** | Conto risparmio genitore-figlio — il figlio non può prelevare senza il consenso di un genitore |
| **2-of-3** | Escrow trustless tra compratore e venditore con arbitro |

---

## Transazioni Escrow

Alice vuole acquistare un libro raro da Bob, ma vivono in città diverse e non si fidano l'uno dell'altro: Alice non vuole pagare prima di ricevere il libro, Bob non vuole spedire prima di essere pagato.

La soluzione è una **transazione escrow 2-of-3** con Judy come arbitro neutrale. Alice crea la transazione multisig con una chiave pubblica ciascuno per Alice, Bob e Judy, e la pubblica sulla blockchain. I fondi entrano in una sorta di "limbo": nessuno può muoverli da solo, servono sempre due firme. L'escrow fallisce solo se Judy collude esplicitamente con una delle parti.

> [!example] I tre scenari possibili
>
> **2a — Tutto ok**: Alice riceve il libro. Alice e Bob firmano insieme per rilasciare i fondi a Bob. Judy non viene coinvolta. Servono due transazioni: una per depositare nell'escrow, una per pagare Bob.
>
> **2b — Alice riceve ma rifiuta di pagare**: Bob fornisce a Judy la prova di spedizione. Bob e Judy firmano insieme per inviare i fondi a Bob.
>
> **2c — Bob non spedisce**: Alice dimostra a Judy di non aver ricevuto nulla. Alice e Judy firmano insieme per restituire i fondi ad Alice.

---

## Pay-To-Script-Hash (P2SH)

Gli script multisig sono scomodi in pratica. Se un cliente deve pagare un'azienda con un multisig 2-of-5, l'azienda deve trasmettere l'intero script al cliente, che ha bisogno di un wallet speciale per costruirlo. La transazione risultante è cinque volte più grande del normale: fee più alte (a carico del mittente), script troppo lungo per un QR code, e l'intero script resta in RAM nel set UTXO di ogni full node finché non viene speso.

![[Pasted image 20260407113120.png]]

**P2SH** (BIP-16, gennaio 2012) risolve il problema: il destinatario del pagamento è identificato dall'**hash dello script**, non dallo script stesso.

| | Locking script | Unlocking script |
|---|---|---|
| **Multisig classico** | `OP_1 <PK1> <PK2> OP_2 OP_CHECKMULTISIG` | `OP_0 <Sig1>` |
| **P2SH** | `OP_HASH160 <RedeemScriptHash> OP_EQUAL` | `OP_0 <Sig1> <Sig2> \| <OP_2 <PK1> <PK2> <PK3> OP_3 OP_CHECKMULTISIG>` |

Per riscattare un P2SH l'utente presenta: la firma richiesta + il *Redeem Script* originale in chiaro, che hashato deve coincidere con l'hash nel locking script.

> [!tip] Il vantaggio chiave del P2SH
>
> Il P2SH sposta tutti gli oneri **dal mittente al destinatario**:
> - La complessità di costruire lo script passa al destinatario
> - Le fee aggiuntive per lo script lungo le paga il destinatario (al momento della spesa), non il mittente
> - Lo script lungo non occupa RAM nell'UTXO set ora, ma viene registrato sulla blockchain solo quando viene speso (nell'input)
> - Gli script vengono codificati come normali indirizzi: qualsiasi wallet semplice può pagare

---

## Hash-Time Locked Contracts (HTLC)

Un HTLC combina due meccanismi:

> [!definition] HTLC
>
> - **Hash Lock**: l'hash di un segreto è pubblicato nello script. I fondi si sbloccano solo se il destinatario rivela pubblicamente il segreto originale.
> - **Time Lock**: condizione di fallback — se entro un timeout prestabilito il segreto non viene rivelato, i fondi tornano al mittente.

Gli HTLC sono usati nei **payment channel** (Lightning Network) e negli **Atomic Swap**.

### Atomic Swap

Un Atomic Swap permette di scambiare criptovalute su blockchain diverse in modo *trustless*, senza exchange centralizzati. Il problema classico: chi invia i fondi per primo rischia che la controparte non adempia. L'HTLC risolve sincronizzando i due lati dello scambio.

**Esempio**: Alice ha BTC e vuole ZEN di Bob.

1. Alice genera un segreto `s`, ne calcola `H(s)` e crea un HTLC sulla blockchain Bitcoin bloccando 1 BTC: Bob può riscattarli rivelando `s`, oppure dopo 24 ore i fondi tornano ad Alice. Alice invia `H(s)` a Bob.
2. Bob crea un HTLC identico sulla blockchain ZEN bloccando 200 ZEN con lo stesso `H(s)` e un timelock di 24 ore.
3. Alice usa `s` per sbloccare l'HTLC di Bob sulla rete ZEN e incassare i 200 ZEN — operazione pubblica e registrata sulla blockchain.
4. Bob legge `s` dalla blockchain ZEN e lo usa per sbloccare l'HTLC di Alice su Bitcoin, incassando 1 BTC.

L'hashlock sincronizza lo scambio; il timelock garantisce che nessuno perda i fondi se la controparte sparisce.

---

## Data Registering e Proof of Burn

La blockchain può fungere da **registro notarile**: si calcola l'hash di un documento e lo si registra on-chain per provare l'esistenza di quel file in una data specifica.

Il metodo originale era simulare un pagamento verso un indirizzo falso (nessuno ha la chiave privata corrispondente) usando 20 byte liberi come campo dati. Il problema: quell'UTXO non può mai essere rimosso dalla RAM dei full node — **data pollution**.

**OP_RETURN** (introdotto dopo il 2013 come compromesso) standardizza la registrazione:

```
OP_RETURN <Data>
```

L'output è esplicitamente non spendibile: i coin associati vengono distrutti, ma l'entry non entra nell'UTXO set e non inquina la RAM. Si usa in due modi:
- Solo per registrare dati (senza bruciare coin significativi)
- **Proof of Burn**: distruggere deliberatamente coin in modo verificabile

> [!note] Usi della Proof of Burn
>
> - **Bootstrap di nuove criptovalute**: gli utenti bruciano BTC per ottenere token della nuova chain, distribuendo la supply in modo decentralizzato
> - **Consenso alternativo**: si vince la "lotteria dei blocchi" bruciando coin invece di consumare energia (come nella PoW). Il bilanciamento matematico tra coin bruciati e ricompense è però difficile da implementare correttamente.

---

## Simplified Payment Verification (SPV)

Scaricare l'intera blockchain (>649 GB ad aprile 2025) non è pratico su smartphone. I **client SPV** (o *lightweight client*) scaricano solo gli **header dei blocchi** — circa 80 byte ciascuno, mille volte più leggeri del blocco completo — e sono interessati solo alle transazioni che riguardano gli indirizzi nel proprio wallet.

La sicurezza è mantenuta su due livelli:
- L'header contiene il **nonce**: si può verificare che la Proof of Work sia stata completata
- La validità di una transazione si verifica tramite **Merkle proof**: il full node invia il ramo del Merkle Tree che collega la transazione alla Merkle Root nell'header. L'SPV ricalcola ricorsivamente gli hash dal basso verso l'alto e confronta il risultato con la radice — se coincide, la transazione è autentica.

### Bloom Filter e Privacy

Richiedere transazioni specifiche per indirizzo rivelerebbe al full node quali indirizzi appartengono all'utente. Per proteggere la privacy, l'SPV invia al full node un **Bloom filter** costruito sugli indirizzi del wallet (OR bit a bit degli hash di ogni indirizzo).

Il full node testa ogni output di ogni transazione contro il filtro:
- Se tutti gli hash restituiscono un bit a 1 → la transazione è *probabilmente* rilevante → viene inviata all'SPV
- Se anche un solo hash restituisce uno 0 → la transazione è *certamente* irrilevante → viene ignorata

I falsi positivi sono accettati deliberatamente: nascondono quali indirizzi interessano davvero all'SPV (privacy) e rimangono comunque pochi (efficienza di banda).

---

## Bitcoin Protocol Stack e Rete P2P

| Livello | Descrizione |
|---|---|
| **Application layer** | Applicazioni user-facing che usano la blockchain |
| **Transaction layer** | Script e logica di validità delle transazioni |
| **Consensus layer** | Algoritmi per l'accordo sull'incorporazione delle transazioni (es. PoW) |
| **Network (P2P) layer** | Broadcasting dei dati tra i nodi |

La rete P2P è **non strutturata**: chiunque può connettersi. Di default ogni nodo mantiene 117 connessioni TCP in uscita e accetta fino a 8 in entrata sulla porta 8333, senza autenticazione né cifratura.
![[Pasted image 20260407113308.png]]

### Bootstrap e Peer Discovery

Un nodo nuovo deve prima trovare qualcuno con cui parlare. I metodi in ordine di preferenza:
1. **Seed address hard-coded** nel client — nodi stabili con IP statici
2. **DNS bootstrap** — server DNS dedicati che restituiscono liste di IP
3. **Forum e chat** — fallback manuale se tutto il resto fallisce

Una volta connesso, il nodo ricorda gli indirizzi dei peer con cui ha comunicato con successo: al riavvio può riconnettersi rapidamente senza ripartire da zero.

Per scoprire ulteriori peer, il nodo invia messaggi `GETADDR` ai vicini, che rispondono con messaggi `ADDR` contenenti liste di IP. Il nuovo nodo annuncia anche se stesso inviando un `ADDR` con il proprio IP, che i vicini propagano ai loro vicini.

### Handshake

All'apertura di una connessione, i nodi si scambiano un messaggio `VERSION` che contiene tra l'altro il campo **bestHeight** — l'altezza corrente della blockchain del nodo. Se un nodo ha una catena più corta di quella del vicino, richiede i blocchi mancanti.
![[Pasted image 20260407113353.png]]
### Gossip Protocol e Propagazione

La propagazione di transazioni e blocchi avviene tramite **gossip** (*any-to-all*): ogni nodo propaga ai propri vicini ciò che riceve.

Il flusso standard per una transazione o un blocco:

1. **`INV`** — messaggio di annuncio: il nodo invia ai vicini l'hash della transazione/blocco (non il contenuto). È una notifica, non un invio.
2. **`GETDATA`** — i vicini che non hanno già quel dato richiedono il contenuto completo.
3. **`BLOCK` / `TRANSACTION`** — il nodo invia il dato effettivo.

> [!tip] Minimizzare il consumo di banda
>
> Se lo stesso hash arriva da più peer contemporaneamente, il nodo invia `GETDATA` a **uno solo** di essi. Questo evita di scaricare lo stesso dato più volte.

**Unsolicited Block Push**: quando un miner trova un blocco, sa con certezza di essere l'unico ad averlo. Salta il passaggio `INV` e invia direttamente il blocco ai vicini — ogni secondo conta per non perdere il vantaggio competitivo.

**GETBLOCK**: un nodo che si è disconnesso o si avvia per la prima volta deve sincronizzarsi. Chiede al vicino la sua visione locale della blockchain; il vicino risponde con gli hash dei blocchi a varie altezze. Il nodo trova il primo hash in comune con la propria catena e richiede i blocchi successivi tramite `GETDATA`. Il processo è iterativo: dopo aver scaricato un batch, invia un nuovo `GETBLOCK` fino a essere aggiornato.

### Protezione contro il DoS

Nodi malevoli potrebbero inondare la rete con oggetti invalidi, saturando la banda. La protezione è integrata nel protocollo per design:

- Un nodo invia un messaggio `INV` ai vicini **solo dopo aver validato** il blocco o la transazione (firma valida, UTXO valido)
- Ogni nodo mantiene uno **score di reputazione** per ciascun peer
- Se un peer si comporta male (es. invia transazioni con firme invalide), il suo score viene degradato
- Sotto una certa soglia, il peer viene disconnesso

> [!question] Possibili domande d'esame
>
> - Cos'è un indirizzo **multisig M-of-N**? Scrivi il locking script e l'unlocking script per un caso 2-of-3. Porta due esempi d'uso concreti.
> - Descrivi il protocollo di **escrow trustless** 2-of-3: quali sono i tre scenari possibili e come vengono risolti?
> - Cos'è il **Pay-to-Script-Hash (P2SH)**? Quali problemi del multisig classico risolve e su chi sposta gli oneri di fee e complessità?
> - Spiega il funzionamento di un **HTLC** (Hash-Time Locked Contract): quali sono i due meccanismi che combina e come garantisce l'atomicità?
> - Descrivi il protocollo di un **Atomic Swap** tra due blockchain diverse. Quale ruolo svolge l'HTLC nel sincronizzare i due lati dello scambio?
> - Cos'è **OP_RETURN** e perché è stato introdotto? Qual è la differenza tra il suo uso e quello dei pagamenti verso indirizzi falsi per registrare dati?
> - Cos'è la **Proof of Burn** e in quali contesti viene usata? Quali problemi introduce come meccanismo di consenso?
> - Spiega come funziona un **client SPV** (Simplified Payment Verification): quali dati scarica e come verifica l'autenticità di una transazione tramite Merkle proof?
> - Come vengono usati i **Bloom filter** negli SPV client di Bitcoin per proteggere la privacy? Perché i falsi positivi sono accettati deliberatamente?
> - Descrivi il protocollo di propagazione dei blocchi nella rete P2P di Bitcoin (messaggi `INV`, `GETDATA`, `BLOCK`). Cos'è l'**Unsolicited Block Push** e perché viene usato dai miner?
