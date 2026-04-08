# Introduzione alla Blockchain

## Il Ledger Distribuito

> [!definition] Ledger
>
> Un registro cronologico di eventi, **append-only**: si possono solo aggiungere voci, mai modificare o cancellare quelle passate. Funziona come un notaio digitale: tutti i partecipanti devono concordare sul suo contenuto.

Un ledger non serve solo per le transazioni finanziarie: qualunque applicazione che necessita di un log di eventi può beneficiarne. Un ledger distribuito viene replicato tra i nodi di una rete P2P: ogni nodo possiede la stessa copia immutabile.

> [!example] Alice come intermediaria
>
> Alice gestisce un'azienda che fa da intermediaria tra dettaglianti e grossisti. Ha difficoltà a mantenere un ledger consistente perché serve centinaia di clienti. Decide di condividere il ledger con tutti: ciascuno mantiene una copia e la aggiorna. Emerge immediatamente il problema della **consistenza**: chi decide cosa è scritto nel ledger?

Quando il ledger è organizzato come una sequenza di blocchi concatenati prende il nome di **blockchain**. Esistono architetture alternative — **IOTA**, ad esempio, usa un DAG invece di una catena lineare.

Ogni blocco contiene tre elementi:
- **Dati** — le transazioni (es. "Alice invia X a Bob")
- **Hash del blocco** — impronta crittografica del contenuto
- **Hash del blocco precedente** — il collegamento che forma la catena (hash pointer)

La catena parte dal **Genesis Block**. Modificare un blocco cambia il suo hash, invalidando tutti i successivi. In una blockchain **Proof of Work**, ripristinarli non richiede solo ricalcolare gli hash: bisogna trovare un valore che, combinato con il nuovo hash, risolva il PoW per ogni blocco successivo. Questo è computazionalmente impossibile. Altre blockchain possono usare meccanismi diversi (es. PoS).

---

## Il Problema del Consenso

In un sistema distribuito senza autorità centrale, chi decide quale operazione aggiungere al registro?

> [!definition] Consenso
>
> Accordo tra i nodi di una rete distribuita sull'**ordine e la validità** delle informazioni memorizzate su un ledger condiviso e replicato. Il consenso è il meccanismo che definisce: chi decide quale operazione viene aggiunta alla blockchain, e quale tra le operazioni da confermare viene scelta.

Il consenso è essenziale per prevenire il **double spending**: le informazioni digitali si copiano facilmente — i bit sono più facili da copiare della carta — quindi senza un accordo collettivo nulla impedisce di spendere la stessa moneta due volte. Se la maggioranza è onesta, i nodi dovrebbero votare contro le transazioni di double spending.

Un approccio classico è il **consenso basato sul voto**: ogni operazione viene trasmessa in broadcast sulla rete e si raccolgono i voti; la maggioranza determina la correttezza delle transazioni e l'ordine dei blocchi.

In un mondo ideale funzionerebbe, ma in reti P2P aperte ci sono due ostacoli reali:
- Ritardi e partizioni di rete (*jitter*, nodi che non si vedono temporaneamente)
- Nodi malintenzionati (**byzantine parties**) che possono comportarsi in modo arbitrario

---

## L'Attacco Sybil

Il punto debole del consenso a voto in reti aperte: cosa significa "maggioranza" se chiunque può creare identità false?

> [!definition] Attacco Sybil
>
> Un attaccante inietta nella rete un numero arbitrario di identità fittizie per ottenere la maggioranza dei voti e manipolare il consenso. Il nome viene dalle Sibille dell'antica Grecia — profetesse attraverso cui parlava una divinità, metafora di più identità per la stessa persona; rappresentate nel pavimento del **Duomo di Siena**. In informatica il libro di riferimento è *Sybil* (Flora Rheta Schreiber, 1973), su una donna con 16 personalità distinte.

Iniettare identità false è banale: basta registrarsi molte volte con la stessa DHT. Il singolo nodo impersona così molteplici identità logiche.

**Obiettivi dell'attacco Sybil:**
- Routing attacks (controllare i percorsi di instradamento)
- Controllare la replica dei dati
- Disturbare la connettività della rete
- Essere la maggioranza nel voto → consenso nelle criptovalute

**Difese:**
- **Proof of Work** (Bitcoin, Ethereum): richiede potere computazionale
- **Proof of Stake**: richiede stake economico
- **Certified Node-IDs**: richiede un'autorità centrale — non è una soluzione P2P

### Double Spending con Sybil

Alice esegue un attacco Sybil assumendo più del 50% delle identità nella rete. Poi tenta di spendere lo stesso bitcoin sia verso Bob che verso Charlie. Usando le sue identità multiple (>50%), approva con il consenso entrambe le transazioni di double spending → l'attacco ha successo.

---

## La Proof of Work

**La soluzione di Bitcoin**: invece di contare le identità, si conta la **potenza di calcolo**.

> [!definition] Proof of Work
>
> Per partecipare al consenso, un nodo deve risolvere un problema computazionalmente difficile ma facile da verificare. Creare identità multiple è inutile se si ha un solo computer: per manipolare la rete bisogna controllare la maggioranza della potenza di calcolo globale. Gli attacchi Sybil diventano costosi e inutili.

La PoW funziona come una **lotteria** in cui i biglietti sono molto costosi (risolvere il problema). Il vincitore della lotteria decide **unilateralmente** quale sarà il prossimo blocco e riceve una ricompensa economica per comportarsi onestamente — incentivo a seguire le regole.

---

## Proof of Ownership e UTXO

Oltre al consenso, serve dimostrare la **proprietà** degli asset.

> [!example] ICO di Alice
>
> Alice vuole aprire un ristorante ma l'affitto è alto e i venture capitalist sono avidi. Usa una **ICO** (*Initial Coin Offering*): propone un progetto su blockchain, raccoglie finanziamenti, crea **token** come compensazione per i finanziatori — nel caso concreto, *cryptocoupon* per pasti scontati all'apertura del ristorante.
>
> Alice usa un ledger per registrare i trasferimenti di token. Il problema: come può un finanziatore dimostrare di possedere un coupon? Come può Alice dimostrare di essere autorizzata a emetterlo? Senza una CA centralizzata.

La soluzione è puramente crittografica: Alice genera una coppia `(chiave pubblica, chiave privata)`. Chiunque conosca la chiave privata corrispondente alla chiave pubblica di Alice possiede i suoi cryptocoupon — Alice stessa, chiunque lei voglia (a cui cede la chiave privata), o chiunque la rubi.

- **Chiave pubblica** → identifica il proprietario del token
- **Chiave privata** → conferisce il possesso effettivo e autorizza i trasferimenti tramite firma digitale

### Esempio di Spesa

Alice (con chiave privata `af876f536...`) vuole trasferire il 50% di un coupon a ciascuno di due finanziatori:
1. Identifica le chiavi pubbliche dei destinatari (es. `1FE1W2EEJE...`, `A5d65ab38...`)
2. Firma il trasferimento con la propria chiave privata → dimostra di essere autorizzata
3. Registra la transazione firmata sul ledger
4. I due destinatari usano le proprie chiavi private per riscuotere il mezzo coupon

> [!warning] La chiave privata è tutto
>
> Chiunque ottenga la chiave privata di un utente ha il pieno controllo dei suoi asset — non esiste recupero, non esiste banca che possa aiutare.

### UTXO

Bitcoin non ha il concetto di "conto" con saldo. Esistono solo trasferimenti. Per sapere quanto possiede, un utente scorre l'intero registro cercando gli **UTXO** (*Unspent Transaction Output*): le transazioni ricevute e non ancora spese. Spendere un UTXO lo distrugge e ne crea di nuovi per il destinatario (e per il resto).

---

## Permissionless vs Permissioned

| | Permissionless | Permissioned |
|---|---|---|
| **Accesso** | Aperto a chiunque | Limitato a nodi autorizzati |
| **Identità** | Anonima/pseudonima | Certificata (umani con password/chiavi, sensori con chiavi) |
| **Consenso** | PoW, PoS (costoso ma senza fiducia) | PBFT e simili (efficiente, ambiente controllato) |
| **Esempi** | Bitcoin, Ethereum, Steemit, Algorand | Hyperledger, Ethereum Quorum, Corda |
| **Rischi** | Fork, attacchi (es. DAO hack da $54M) | Richiede fiducia negli operatori |

### Permissionless

Chiunque può partecipare e minare. Il sistema funziona senza fidarsi di nessuno, grazie agli incentivi economici. Non richiede autorità centrale.

### Permissioned

Accesso e consenso limitati a partecipanti con identità verificata. Le differenze chiave rispetto alle permissionless sono:
- Le parti hanno **identità**: umani con password/chiavi, sensori con chiavi proprie — tutti **autenticati**
- Si usano meccanismi di consenso diversi, basati sul voto tra nodi noti
- **Accountability**: se un nodo viene scoperto a barare, è identificabile — diverso approccio agli attacchi Sybil

> [!example] Supply chain del frozen yogurt
>
> Alice vende il ristorante e apre un'attività di frozen yogurt. Le spedizioni arrivano sciolte: di chi è la colpa — Bob (il trasportatore) o Carol (la fabbrica)? Con una blockchain permissioned accessibile solo a Bob, Carol e sensori di temperatura certificati, ogni evento è tracciato e firmato digitalmente. La responsabilità è attribuibile con certezza.

Il **PBFT** (*Practical Byzantine Fault Tolerance*, Castro e Liskov, 1999) è il protocollo di consenso tipico delle blockchain permissioned: tolera comportamenti arbitrari di nodi malevoli in reti asincrone, con prestazioni nettamente superiori alla PoW in ambienti con partecipanti noti e controllati.

> [!question] Possibili domande d'esame
>
> - Cos'è un **ledger** e quali sono le sue proprietà fondamentali? Perché emerge il problema della consistenza in un ledger distribuito?
> - Descrivi la struttura di un blocco in una blockchain: quali sono i tre elementi fondamentali e qual è il ruolo dell'hash del blocco precedente?
> - Cos'è il **consenso** in una blockchain? Perché è essenziale per prevenire il double spending?
> - Cos'è l'**attacco Sybil**? Spiega con un esempio come un attaccante può manipolare il consenso a voto sfruttando identità false.
> - Quali sono le difese contro gli attacchi Sybil nelle blockchain? Perché la Proof of Work è efficace contro di essi?
> - Spiega il modello **UTXO**: cosa sono gli output non spesi, come si calcola il saldo di un utente e cosa succede quando un UTXO viene speso?
> - Qual è la differenza tra blockchain **permissionless** e **permissioned**? Confrontale su accesso, identità, consenso ed esempi reali.
> - Cos'è la **Proof of Work** come meccanismo di consenso? Perché sostituisce il conteggio delle identità con la potenza di calcolo?
> - Come funziona la **Proof of Ownership** in Bitcoin? Qual è il ruolo della chiave pubblica e della chiave privata?
> - Descrivi uno scenario d'uso concreto per una blockchain permissioned (es. supply chain): perché una blockchain tradizionale non sarebbe adeguata?
