# Introduzione al Corso — P2P Systems and Blockchain

## Struttura del Corso

Il corso (9 CFU) è tenuto da Laura Ricci e Damiano Di Francesco Maesa. Si divide in lezioni teoriche (~6 CFU) e laboratorio (~3 CFU), dove si sviluppano smart contract con **Solidity** e **Hardhat**.

L'esame consiste in un progetto su blockchain seguito da un orale che discute il progetto e gli argomenti teorici non coperti da esso. Esempi di progetti passati: MasterMind o BattleShip sulla blockchain, aste smart, lotterie smart, commercio di contenuti con ricompensa, Splitwise sulla blockchain.

Il programma copre: sistemi P2P (overlay non strutturati, DHT), Bitcoin ed Ethereum, smart contract, soluzioni di scalabilità Layer-2, e applicazioni avanzate come DeFi, Supply Chain e Self Sovereign Identity (SSI). Gli strumenti fondamentali utilizzati durante il corso sono: algoritmi distribuiti, metodi crittografici e strutture dati probabilistiche.

---

## Il Paradigma Peer-to-Peer

Nel modello **Client-Server** classico i server sono macchine dedicate con IP fisso che erogano servizi; i client li consumano senza comunicare tra loro. Il server è un potenziale collo di bottiglia.

> [!definition] Rete Peer-to-Peer (P2P)
>
> Insieme di entità autonome (**peer**) che si auto-organizzano e condividono risorse distribuite (calcolo, memoria, banda). Il sistema è in grado di adattarsi a un continuo **churn** dei nodi mantenendo connettività e prestazioni ragionevoli senza un'entità centrale. Ogni nodo è contemporaneamente fornitore e consumatore di servizi (funzionalità simmetrica: **servent**).

I server possono esistere per il bootstrap iniziale, ma non sono necessari per lo scambio effettivo delle risorse. Una sfida caratteristica delle reti P2P è il **churn**: i nodi entrano ed escono continuamente, ottenendo spesso un nuovo indirizzo IP ad ogni connessione. Questo rende inutilizzabile l'indirizzamento tramite IP statici e richiede meccanismi applicativi — non a livello IP — per localizzare le risorse.
![[Pasted image 20260407110113.png]]
Ogni peer che partecipa a una rete P2P deve affrontare quattro problemi fondamentali: come **unirsi** alla rete (*join*), come **scoprire altri peer** (*peer discovery*), come comportarsi sia da fornitore che da consumatore di servizi, e come **prevenire il free riding** — ovvero impedire che alcuni nodi consumino risorse senza contribuire — incentivando la partecipazione e la reciprocità.

### Condivisione di Risorse

Le risorse condivise in una rete P2P si trovano "ai bordi" di Internet: sono direttamente messe a disposizione dai peer, senza nodi speciali per la loro gestione. Possono essere: **ledger** distribuiti, spazio di archiviazione in lettura/scrittura, potenza di calcolo, banda.

La partecipazione può avvenire per motivi molto diversi. Un peer può offrire risorse gratuitamente per contribuire a un progetto collettivo (es. ricerca di vita extraterrestre con SETI, ricerca su terapie contro il cancro). Oppure può essere **ricompensato** per il contributo alla gestione della rete, come avviene per i **Bitcoin miners**. In ogni caso, la proprietà più interessante è quella di **auto-scalabilità**: la partecipazione di un numero crescente di utenti aumenta naturalmente le risorse del sistema e la sua capacità di servire più richieste.

### L'Evoluzione del File Sharing

Il file sharing è stata la prima *killer application* del P2P. Il funzionamento tipico è il seguente: un utente U ha un client P2P sul proprio computer; ad ogni connessione ottiene un nuovo indirizzo IP. U memorizza i file condivisi in una directory, associando a ciascuno dei metadati identificativi (titolo, autore, data di pubblicazione). Quando U vuole trovare un file, invia una query al sistema, riceve la lista dei peer che lo posseggono, sceglie il peer P secondo certi criteri, e avvia il trasferimento diretto. Nel frattempo, altri utenti possono già scaricare da U le parti del file che U ha già ottenuto.

**Napster** (prima generazione) usava un indice centralizzato dei file su server dedicati, mentre il trasferimento avveniva direttamente tra peer. La centralizzazione dell'indice era però un punto di vulnerabilità — tecnica e legale — e portò alla chiusura del servizio per violazione del copyright: attraverso l'analisi dell'indice centralizzato era possibile risalire ai contenuti scambiati tra gli utenti. La seconda generazione (Gnutella, Kazaa, BitTorrent) ha eliminato ogni punto centrale, distribuendo sia la ricerca che il trasferimento.

---

## Blockchain: Cos'è e Quando Usarla

> [!definition] Blockchain
>
> Database condiviso, replicato e consistente, mantenuto senza un'autorità centrale. È **append-only** (solo aggiunta), immutabile e resistente alle manomissioni. In termini di teoria dei giochi: una macchina a stati decentralizzata mantenuta da attori non fidati, incentivati economicamente a comportarsi correttamente. Non può essere spenta o censurata, e i dati non possono essere eliminati.

Le tecnologie fondamentali sono:
- **Firme digitali** — autenticazione
- **Hash crittografici** — immutabilità e integrità
- **Replicazione** — disponibilità tramite copie distribuite
- **Consenso distribuito** — coordinamento tra repliche mutuamente diffidenti

> [!tip] Quando usare la blockchain
>
> Ha senso considerare la blockchain quando: è necessario memorizzare uno stato condiviso in modalità append-only, ci sono più scrittori con diversi gradi di fiducia reciproca, l'applicazione deve girare in modo distribuito, il processo di settlement è complesso e richiede una terza parte fidata, servono integrità/autenticazione/non-ripudio, le regole sono precise e semplici da codificare, e la trasparenza è preferibile alla privacy.

> [!warning] La blockchain non è sempre la soluzione giusta
>
> Ha senso usarla solo se i partecipanti non sono noti o fidati. Se tutte le parti sono fidate, un database tradizionale è preferibile. Se serve solo immutabilità con parti fidate, bastano database con checksum crittografici (es. AWS QLDB, Kafka). Molti usi proposti della blockchain in ambito aziendale rientrano in questa categoria e non ne hanno effettivamente bisogno.

---

## Bitcoin ed Ethereum

**Bitcoin** nasce nel 2008 dal paper di Satoshi Nakamoto come sistema di cassa elettronica P2P: pagamenti online diretti senza intermediari, con il problema del **double-spending** risolto tramite una catena di blocchi con timestamping basato su hash. Il *Genesis Block* conteneva il messaggio *"Chancellor on brink of second bailout for banks"* — un riferimento esplicito alla crisi finanziaria e all'obiettivo di sottrarre il controllo del denaro alle banche centrali. La *cypherpunk vision* alla base di Bitcoin è che si possa rivoluzionare il mondo costruendo protocolli sicuri.

**Ethereum** espande il concetto: non è solo valuta, ma una piattaforma programmabile. Introduce gli **smart contract**, programmi eseguiti dalla blockchain stessa tramite linguaggi Turing-completi come Solidity. L'intera rete si comporta come un singolo computer globale replicato e consistente (**EVM**, Ethereum Virtual Machine). A differenza di Bitcoin, in cui gli script hanno potere computazionale limitato, Ethereum può risolvere qualunque problema computazionale — con il meccanismo del **gas** per prevenire attacchi denial-of-service.

> [!example] Smart contract assicurativo
>
> Un contratto connesso a un database di voli rimborsa automaticamente il passeggero se il ritardo supera una soglia prestabilita — senza pratiche burocratiche, senza intermediari. Il rimborso in criptovaluta viene trasferito automaticamente al wallet di Bob non appena il ritardo viene verificato.

---

## Le Sfide: Trilemma della Blockchain

> [!warning] Trilemma della Blockchain
>
> È difficile ottenere contemporaneamente **sicurezza**, **decentralizzazione** e **scalabilità**. Migliorare una delle tre proprietà tende a penalizzare le altre. È una delle grandi sfide scientifiche aperte del settore.

### Privacy

Le transazioni su ledger pubblici sono visibili a tutti. Le identità degli utenti possono a volte essere inferite, con rischio di esposizione di dati sensibili. Bilanciare privacy e auditabilità è particolarmente delicato nei protocolli **DeFi**: da un lato richiedono confidenzialità (transazioni, depositi, prestiti senza rivelare importi o indirizzi), dall'altro richiedono auditabilità (verifica del double-spending, conferma della collateralizzazione). Le soluzioni principali sono:

- **Zero Knowledge Proofs (ZKP)**: un *prover* dimostra la validità di un'affermazione a un *verifier* senza rivelare i dati sottostanti. Applicazioni: nascondere dati sensibili mantenendo la correttezza, esecuzione privata di smart contract, verifica d'identità privata on-chain, DeFi privacy-preserving.
- **Fully Homomorphic Encryption (FHE)**: eseguire calcoli su dati cifrati senza decifrarli — il risultato, una volta decifrato, coincide con quello ottenuto dal testo in chiaro. L'idea chiave è: *"posso calcolare sui tuoi dati segreti senza mai vederli"*. Esempio: un protocollo di prestito calcola l'idoneità al credito o il tasso di interesse su saldi cifrati — i saldi restano privati ma il sistema produce risultati corretti.
- **Multiparty Computation (MPC)**: calcolo distribuito tra più parti che collaborano senza rivelare i propri input privati alle altre.

### Scalabilità

Le blockchain tradizionali hanno throughput limitato e costi energetici elevati. Le soluzioni principali spostano l'esecuzione **off-chain**, usando la catena principale solo come ancora di fiducia (*trust anchor*) per la validazione finale:

- **Layer-2** (Optimistic Rollups, ZK-rollups): raggruppano molte transazioni off-chain e ne pubblicano solo la prova sulla catena principale.
- **Payment Channel** (Lightning Network): canali di pagamento bidirezionali che consentono scambi diretti tra peer senza toccare la blockchain per ogni transazione.

---

## Applicazioni

| Applicazione | Descrizione |
|---|---|
| **Criptovalute e Token** | Alternativa alle valute fiat: la blockchain non richiede che un governo emetta moneta né che le banche validino le transazioni. L'offerta è legata a un bene virtuale limitato crittograficamente. La blockchain risolve il double spending e supporta sia token **fungibili** (interscambiabili) che **non fungibili** |
| **NFT** | Prova di proprietà di asset digitali unici (arte, diritti d'autore). Un'opera in .jpeg è facilmente copiabile, ma l'NFT certifica chi è il proprietario originale |
| **DeFi** | Piattaforme come **Uniswap** per il trading diretto tra pari (DEX) con pool di liquidità e market maker automatizzati (AMM). In Uniswap V3, le posizioni di liquidità sono rappresentate come **LP NFTs**: ogni posizione ha parametri distinti e personalizzabili che ne determinano valore e rendimento |
| **Self Sovereign Identity** | L'utente controlla i propri dati e rivela solo le informazioni minime necessarie (minimalismo, portabilità, consenso) |
| **Supply Chain** | Tracciamento provenienza e qualità: es. Walmart-IBM (Hyperledger) per sicurezza alimentare, con sensori IoT che registrano temperatura e posizione. Un altro esempio: ristoranti possono verificare la catena di custodia del pesce, con sensori attaccati al prodotto che registrano posizione, temperatura e umidità lungo tutta la filiera |
| **Intellectual Property** | Il proprietario di un contenuto digitale fa l'hash del contenuto insieme alla propria identità e lo registra sulla blockchain. Se nessun altro può dimostrare di averlo pubblicato prima di quel commit, questo costituisce prova di proprietà — più comodo di un ufficio brevetti e senza dover divulgare i dettagli del contenuto |

---

## Conclusioni e Prospettive

I sistemi P2P offrono vantaggi su più fronti. Per gli utenti: sfruttamento delle risorse in eccesso (cicli CPU inutilizzati, storage libero, banda disponibile) in cambio di risorse, servizi o partecipazione a reti sociali. Per la comunità: la **proprietà di auto-scalabilità** — la partecipazione di un numero maggiore di utenti aumenta naturalmente le risorse del sistema. Per chi sviluppa applicazioni: riduzione dei costi rispetto al modello client-server (server farm ad alta connettività, replicazione per fault tolerance, disponibilità 24×7 sono tutte responsabilità distribuite tra i peer).

Il successo di un'applicazione P2P dipende in larga misura dalla formazione di una **massa critica** di utenti: una soglia di partecipazione che permette all'applicazione di autosostenersi. Nei primi sistemi P2P questa soglia è stata raggiunta grazie alla novità dell'applicazione (contenuti gratuiti nel file sharing, criptovalute come asset redditizio, token come mezzo semplice per scambiare asset). Per le nuove applicazioni, il successo dipenderà oltre che dalla qualità tecnica anche dall'**appeal dell'applicazione** e dalla definizione di nuovi **modelli di business**.

> [!note] Sfide scientifiche aperte
>
> Lo sviluppo di applicazioni P2P su larga scala richiede strumenti nuovi. Le metodologie classiche per i sistemi distribuiti non scalano: un sistema P2P opera su milioni di nodi (non centinaia), e il fallimento o la disconnessione di un nodo è un evento normale, non un'eccezione. Servono: **teoria dei giochi** (cooperazione tra peer, equilibrio di Nash), **nuove tecniche crittografiche**, **nuovi algoritmi di consenso**, **strumenti di analisi di sistemi complessi**.

> [!question] Possibili domande d'esame
>
> - Quali sono le differenze fondamentali tra il modello client-server e il modello peer-to-peer? Quali vantaggi e svantaggi presenta ciascuno?
> - Cos'è il **churn** in una rete P2P e perché rende inutile l'indirizzamento tramite IP statici?
> - Quali sono i quattro problemi fondamentali che un peer deve affrontare partecipando a una rete P2P?
> - Descrivi il meccanismo di file sharing di Napster: perché la sua architettura lo rese vulnerabile sia tecnicamente che legalmente?
> - Cos'è una blockchain? Elenca le tecnologie fondamentali su cui si basa e spiega il ruolo di ciascuna.
> - In quali scenari ha senso usare una blockchain e in quali no? Quali condizioni devono essere soddisfatte perché il suo utilizzo sia giustificato?
> - Cos'è il trilemma della blockchain? Descrivi il trade-off tra sicurezza, decentralizzazione e scalabilità con esempi concreti.
> - Quali sono le principali differenze tra Bitcoin ed Ethereum in termini di scopo e capacità computazionale?
> - Cos'è la proprietà di **auto-scalabilità** nelle reti P2P? In che modo si differenzia dal comportamento dei sistemi client-server?
> - Cos'è il **free riding** e perché rappresenta un problema per le reti P2P? Quali meccanismi possono mitigarlo?
