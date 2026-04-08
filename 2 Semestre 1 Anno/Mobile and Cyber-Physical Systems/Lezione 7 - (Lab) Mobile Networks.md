# Reti Mobili: Architettura Cellulare 4G LTE e 5G

Questa sezione introduce i fondamenti e l'evoluzione delle reti cellulari, esplorando l'architettura dettagliata dei sistemi 4G LTE e l'innovativo paradigma del 5G, fino ad arrivare ai meccanismi fondamentali di gestione della mobilità e della sicurezza. Il materiale è basato sulle lezioni del corso "Mobile and cyber-physical systems" tenuto dalla professoressa Federica Paganelli.

> [!note] Nota Bibliografica
>
> I concetti illustrati si basano ampiamente su testi accademici di riferimento, in particolare "Computer Networking: A Top-Down Approach" (8a edizione, 2020) di James F. Kurose e Keith W. Ross, di cui le slide offrono un libero adattamento. Un'ulteriore fonte fondamentale per l'architettura moderna è "5G Mobile Networks: A Systems Approach" di Larry Peterson e Oguz Sunay.

---

## L'Evoluzione delle Reti Cellulari: Dal 1G al Futuro 6G

Le reti cellulari hanno subìto un'evoluzione radicale nel corso dei decenni, passando da semplici sistemi per chiamate vocali a infrastrutture globali per l'Internet delle Cose e applicazioni ad altissime prestazioni. Secondo una recente pubblicazione (Zakria Qadir et al., Elsevier, Giugno 2023), questa traiettoria ci sta guidando verso il futuro delle reti 6G.

| Generazione | Anni | Standard | Velocità | Accesso | Caratteristiche principali |
|---|---|---|---|---|---|
| **1G** | Anni '80 | AMPS, TACS | 2.4 kbps | FDMA | Servizio vocale analogico; commutazione a circuito per voce e dati; copertura e sicurezza limitate |
| **2G** | Anni '90 | GSM, CDMA | 64 kbps | TDM+FDM | Voce digitale e SMS; commutazione a circuito per voce e dati |
| **2.5G (GPRS)** | Fine anni '90 | GPRS | — | — | Passo intermedio fondamentale; prima introduzione dei dati a commutazione di pacchetto |
| **3G** | Anni 2000 | UMTS, CDMA2000 | fino a 2 Mbps | CDMA | Accesso Web in mobilità; core vocale a circuito, rete dati separata a pacchetto |
| **4G** | Anni 2010 | LTE Advanced | 100–1000 Mbps | OFDMA | Mobile Internet of Applications; All-IP core; tutto il traffico tunnelizzato come pacchetti IP |
| **5G** | Anni 2020 | 5G NR | fino a ~20 Gbps | OFDMA + mmWave | IIOT; separazione piano controllo/dati; Service Based Architecture; Network slicing; Sub-6 GHz e mmWave |
| **6G** | Anni 2030 (prev.) | — | ~1 Tbps | — | Ecosistema "completamente connesso"; potenziato da AI; architetture Cell-less |

---

## L'infrastruttura delle Reti Cellulari 4G/5G

Le reti 4G e 5G rappresentano oggi la soluzione standard per l'Internet mobile su vasta scala. La loro diffusione è impressionante: basti pensare che già nel 2019 c'erano più dispositivi connessi tramite banda larga mobile che tramite banda larga fissa (con un rapporto di 5 a 1). A livello di affidabilità, la disponibilità del 4G ha raggiunto il 97% del tempo in Corea e il 90% negli Stati Uniti, garantendo velocità di trasmissione di 20 Mbps e oltre. Dal punto di vista tecnico, gli standard sono definiti dal consorzio **3GPP** (3rd Generation Partnership Project), in particolare lo standard **LTE** (Long-Term Evolution) per il 4G.

L'architettura di una rete cellulare 4G/5G si divide in tre blocchi principali:

1. **Radio Access Network (RAN):** È la rete di accesso radio, composta da una collezione distribuita di **Base Stations** (stazioni base) che gestiscono lo spettro radio per connettere i dispositivi mobili (UEs).

2. **Backhaul Network:** È l'infrastruttura che interconnette la RAN con il Mobile Core. Può basarsi su reti cablate (come la fibra ottica) o su soluzioni wireless emergenti come l'Integrated Access Backhaul (IAB).

3. **Mobile Core Network:** Fornisce connettività Internet (IP) sia per i servizi dati che per la voce. Assicura i requisiti di Quality of Service (QoS), traccia la mobilità dell'utente per garantire un servizio ininterrotto, e gestisce i processi di fatturazione (billing and charging). Nel 4G questo blocco è chiamato **Evolved Packet Core (EPC)**, mentre nel 5G prende il nome di **NG-Core**.

![[Screenshot 2026-03-04 alle 10.23.14.png|480]]

---

## Gli Elementi dell'Architettura 4G LTE

Analizziamo nel dettaglio i componenti fondamentali che costituiscono l'architettura 4G LTE, comunemente definita all-IP Enhanced Packet Core (EPC):

- **Mobile device (UE - User Equipment):** È il terminale dell'utente (smartphone, tablet, dispositivi IoT) dotato di una radio 4G LTE. Implementa l'intero stack dei protocolli Internet a 5 livelli. La sua identità globale è definita dall'**IMSI** (International Mobile Subscriber Identity) a 64-bit, salvata sulla scheda **SIM**. L'IMSI identifica univocamente l'abbonato nel sistema globale delle reti cellulari, includendo la nazione e la rete del carrier "home" dell'utente.

> [!definition] UE (User Equipment)
>
> Con **UE** si intende qualsiasi dispositivo terminale dell'utente dotato di una radio 4G LTE — smartphone, tablet o dispositivo IoT. La sua identità globale è l'**IMSI** (International Mobile Subscriber Identity), un codice a 64-bit salvato sulla SIM che identifica univocamente l'abbonato nel sistema globale, includendo nazione e carrier "home".

- **Base station (eNode-B):** Situata al "margine" (edge) della rete del carrier, gestisce le risorse radio wireless e i dispositivi mobili all'interno della sua area di copertura, chiamata "cella". Coordina l'autenticazione del dispositivo interfacciandosi con altri elementi di rete. Rispetto a un Access Point WiFi, l'eNode-B ha un ruolo molto più attivo nella mobilità dell'utente: coordina gli _handover_ del dispositivo tra le celle, collabora con le stazioni base vicine per minimizzare le interferenze radio, e crea tunnel IP specifici per ogni dispositivo verso i gateway.

- **Mobility Management Entity (MME):** È il "cervello" per la gestione della mobilità. Coordina l'autenticazione bidirezionale (dispositivo-rete e rete-dispositivo) interfacciandosi con l'HSS della rete "home" dell'utente. Gestisce i dispositivi mobili occupandosi degli handover tra le celle e del tracciamento/localizzazione (tracking/paging) del dispositivo. Inoltre, si occupa di stabilire i percorsi (tramite tunneling) dal dispositivo mobile verso i gateway S-GW e P-GW.

- **Home Subscriber Service (HSS):** È il database centrale che contiene tutte le informazioni relative agli abbonati. Memorizza i dati dei dispositivi per i quali la rete in cui si trova l'HSS è considerata la "home network" (rete di casa) e lavora a stretto contatto con l'MME durante le procedure di autenticazione.

- **Serving Gateway (S-GW):** Si trova sul percorso dei dati da e verso il dispositivo mobile e Internet. Inoltra i pacchetti IP verso la RAN e, viceversa, si occupa dell'instradamento (routing) e dell'inoltro dei dati dell'utente. Funge da **Local Mobility Anchor**: quando un utente si sposta da un eNodeB a un altro all'interno della stessa rete, l'S-GW agisce come ancora locale per la sessione. È quindi direttamente coinvolto negli handover tra stazioni base.

- **PDN Gateway (P-GW):** Il Packet Data Network Gateway connette la rete core 4G a reti dati esterne, come Internet o reti aziendali private. In pratica, è l'ultimo elemento LTE che un datagramma incontra prima di immettersi in Internet. Si comporta come un normale router gateway, fornendo servizi di **NAT** (Network Address Translation) e funzioni aggiuntive come l'applicazione di policy, il traffic shaping e la gestione degli addebiti (charging). _Spesso S-GW e P-GW possono essere co-localizzati nello stesso nodo fisico_.

![[Screenshot 2026-03-04 alle 10.32.29.png]]

I componenti del Mobile Core possono essere implementati flessibilmente. Ad esempio, una singola coppia MME/P-GW potrebbe servire un'intera area metropolitana, mentre gli S-GW potrebbero essere distribuiti in vari siti "edge" nella città, ognuno a servizio di circa 100 stazioni base. Le specifiche 3GPP consentono molte configurazioni alternative.

---

## Piano Dati e Piano di Controllo in LTE: Il Meccanismo del Tunneling

Una caratteristica distintiva dell'architettura LTE è l'uso estensivo di nuovi protocolli, sia a livello di collegamento (link) e fisico, sia per la mobilità, la sicurezza e l'autenticazione. Questo si traduce in una netta separazione tra il **Piano di Controllo** (Control Plane) e il **Piano Dati** (User/Data Plane).

La Base Station inoltra sia i pacchetti di controllo che quelli utente tra il Mobile Core e l'UE.

- Nel **Piano di Controllo**, i pacchetti sono incapsulati su **SCTP/IP** (Stream Control Transmission Protocol, RFC 4960). L'SCTP è un'alternativa affidabile al TCP, specificamente progettata per trasportare informazioni di segnalazione telefonica e gestire registrazione, tracciamento e autenticazione.

- Nel **Piano Dati**, i pacchetti utente viaggiano all'interno di tunnel creati tramite il protocollo **GTP-U** (General Packet Radio Service Tunneling Protocol), progettato dal 3GPP per operare sopra al protocollo UDP.

Il datagramma mobile originario viene quindi incapsulato usando GTP e inviato all'interno di un datagramma UDP verso l'S-GW, che a sua volta lo re-incapsula per inviarlo al P-GW. **Questo uso del tunneling è vitale per supportare la mobilità:** quando un utente si sposta, gli indirizzi IP originali rimangono invariati; a cambiare sono semplicemente gli endpoint del tunnel GTP. Le sessioni vengono identificate tramite un **TEID** (Tunnel Endpoint Identifier).

> [!tip] Perché il tunneling GTP abilita la mobilità
>
> Il tunneling GTP risolve elegantemente il problema della mobilità IP: incapsulando il datagramma dell'utente dentro un nuovo pacchetto UDP/GTP, gli indirizzi IP visibili ai router di Internet rimangono fissi (quelli dell'S-GW e del P-GW). Quando l'utente cambia cella o si sposta in una nuova area, basta aggiornare gli endpoint del tunnel — il dispositivo mantiene il proprio indirizzo IP e le sessioni TCP attive non vengono interrotte.

![[Screenshot 2026-03-04 alle 10.47.58.png|529]]

A livello radio, lo stack del piano dati (tra dispositivo e stazione base) comprende:

1. **Packet Data Convergence Protocol (PDCP):** Si occupa della compressione dell'header e della crittografia.

2. **Radio Link Control (RLC):** Gestisce la frammentazione e il riassemblaggio dei pacchetti IP, garantendo un trasferimento affidabile a livello di collegamento tramite meccanismi di ACK/NACK.

3. **Medium Access Control (MAC):** Richiede e utilizza gli slot di trasmissione radio, gestendo inoltre la rilevazione e la correzione degli errori.

4. **Livello Fisico (Physical):** Utilizza tecniche come FDM e TDM, e specificamente l'**OFDM** (Orthogonal Frequency Division Multiplexing) in downlink per minimizzare l'interferenza tra i canali. Ogni dispositivo attivo riceve due o più time slot da 0.5 ms distribuiti su 12 frequenze, secondo un algoritmo di scheduling non standardizzato e a discrezione dell'operatore, garantendo velocità potenziali di centinaia di Mbps per dispositivo.

### Modalità "Sleep" e Associazione

Per preservare la batteria, i dispositivi LTE dispongono di due modalità di risparmio energetico simili a quelle del WiFi: _Light sleep_ (attivata dopo centinaia di millisecondi di inattività, con risvegli periodici per controllare trasmissioni in arrivo) e _Deep sleep_ (attivata dopo 5-10 secondi). Poiché un dispositivo in deep sleep potrebbe cambiare cella, deve ristabilire l'associazione per ricevere i messaggi di "paging" broadcastati dall'MME quando la rete cerca di localizzarlo per una chiamata in entrata.

Per associarsi a una rete, il dispositivo rileva un "primary synch signal" (inviato dalla BS ogni 5 ms), individua un secondo segnale di sincronizzazione, e legge i dati trasmessi dalla Base Station (larghezza di banda, configurazioni, informazioni del carrier) prima di decidere a quale rete associarsi, avviando infine l'autenticazione.

---

## La Rivoluzione del 5G: Prestazioni, Architettura e Servizi

Mentre il 4G mirava a circa 10 Mbps e 10 ms di latenza, gli obiettivi progettuali del 5G sono drasticamente più ambiziosi (basati su proiezioni del GAO e dati ITU):

- Un aumento di 10 volte nel bitrate di picco (100 Mbps di "User experienced data rate", con picchi potenziali $>10$ Gbps).

- Una riduzione di 10 volte della latenza (fino a $1\text{ ms}$).

- Un aumento di 100 volte della capacità di traffico e una Connection Density in grado di supportare 1 milione di dispositivi per chilometro quadrato.


La nuova tecnologia radio, **5G NR (New Radio)**, non è retrocompatibile con il 4G. Opera su due bande principali: **FR1** (450 MHz-6 GHz) e **FR2** (24 GHz-52 GHz) che include le onde millimetriche. Le frequenze millimetriche permettono velocità immense, ma su distanze brevi, richiedendo antenne direzionali (MIMO) e un'installazione massiccia di nuove e densissime stazioni base chiamate "pico-cells" (dal raggio di 10-100 metri).

Il 5G abilita tre macro-scenari d'uso (Use Cases):

1. **eMBB (Enhanced Mobile Broadband):** Larghezza di banda incrementata per applicazioni multimediali pesanti come realtà aumentata, realtà virtuale e streaming video in 4K/360°.

2. **URLLC (Ultra Reliable Low-Latency Communications):** Applicazioni critiche (Mission-Critical Control) che richiedono affidabilità estrema (99.999% o "five nines"), latenze di 1ms e supporto per mobilità ad alte velocità (fino a 100 km/h), come l'automazione di fabbrica o la guida autonoma.

3. **mMTC (Massive Machine Type Communications):** Accesso narrowband progettato per pervasività sensoriale. Richiede dispositivi a bassissimo costo, batterie che durano oltre 10 anni e altissima densità.

### La 5G Core e la Service Based Architecture (SBA)

A livello architetturale, la rete 5G Core è profondamente riprogettata per integrarsi con Internet e i servizi cloud (approccio "cloud-native"), pur mantenendo alcuni concetti ereditati dal 4G, come l'incapsulamento dei pacchetti tramite GTP-U. Adotta una **Service Based Architecture (SBA)** in cui la rete è organizzata come un set di funzioni di rete (Network Functions, NF) indipendenti e riutilizzabili, implementabili come macchine virtuali su hardware generico e attivabili su richiesta (Network Function Virtualization). Ogni funzione espone i propri servizi tramite una **Service-Based Interface (SBI)** che utilizza un'interfaccia REST ben definita su HTTP/2.

C'è una netta separazione tra piano di controllo e piano utente, con server distribuiti fino all'edge della rete per ridurre la latenza. Ecco come le funzioni del 4G si mappano nella nuova architettura 5G Core:

|**Componente 4G**|**Corrispettivo 5G**|**Funzione Principale**|
|---|---|---|
|**eNB**|**gNB**|Modulo radio 5G flessibile e "cloud-native", diviso in Distributed Unit e Central Unit.|
|**S-GW, P-GW**|**UPF (User Plane Function)**|Inoltra il traffico tra RAN e Internet. Gestisce ispezione dei pacchetti, QoS e report del traffico. Può essere distribuita (Multi-Access Edge Computing, MEC).|
|**MME**|**AMF + SMF**|L'**AMF** (Access and Mobility Management Function) si occupa unicamente di autenticazione, gestione delle connessioni, mobilità e reachability. L'**SMF** (Session Management Function) gestisce le sessioni dati, inclusa l'allocazione degli indirizzi IP e il controllo della QoS.|
|**HSS**|**AUSF + UDM**|L'**AUSF** (Authentication Server Function) gestisce le procedure di autenticazione. L'**UDM** (Unified Data Management) archivia i dati degli abbonati, le identità degli utenti e le credenziali, operando come un moderno HSS.|
|**PCRF**|**PCF**|Policy Control Function: gestisce le regole di policy che le altre funzioni di controllo applicano.|

Il 5G introduce anche funzioni di supporto (Helper) completamente nuove, rendendo i servizi "stateless" e facilmente scalabili:

- **SDSF e UDSF:** Funzioni di archiviazione per dati strutturati (Structured Data Storage) e non strutturati.

- **NEF (Network Exposure Function):** Espone in modo sicuro le capacità della rete a servizi di terze parti.

- **NRF (NF Repository Function):** Una sorta di registro per scoprire i servizi disponibili.

- **NSSF (Network Slicing Selector Function):** Seleziona le _Network Slices_ per gli utenti, permettendo di partizionare le risorse di rete per differenziare i servizi.

### Opzioni di Implementazione (Deployment)

Il passaggio dal 4G al 5G prevede due principali opzioni di _deployment_:

1. **Non-Standalone (NSA):** È un approccio ibrido (4G + 5G RAN) operante sopra all'Evolved Packet Core (EPC) del 4G. Le stazioni base 5G sono installate a fianco di quelle 4G per dare un _boost_ di velocità e capacità. In questo modello, il piano di controllo passa attraverso le stazioni 4G verso il Core 4G, mentre le stazioni 5G veicolano esclusivamente il traffico dati degli utenti.

2. **Standalone (SA):** È l'infrastruttura 5G pura, che poggia sulla nuova 5G Mobile Core (NG-Core) e gestisce autonomamente sia il controllo che i dati. Al momento, l'adozione del 5G SA varia enormemente nel mondo: si attesta all'80% in Cina, 52% in India, 24% negli Stati Uniti, ma è fermo a un mero 2% in Europa (Dati Ookla 2025).

---

## Gestione Globale, Sicurezza e Protocollo di Autenticazione (AKA)

Le reti cellulari odierne formano una complessa "rete di reti IP" globale interconnessa tramite punti di interscambio (IPX). Sebbene condividano tecnologie tipiche di Internet (HTTP, DNS, TCP, SDN), mantengono caratteristiche uniche come la mobilità intesa come servizio nativo di primaria importanza, l'uso di link radio, un modello di business su abbonamento e una forte identità dell'utente legata alla SIM card. Diventa quindi vitale gestire il **Roaming**, regolato dal rapporto di fiducia commerciale tra la rete "home" dell'utente (che possiede i dati HSS) e la rete "visitata".

La sicurezza e l'autenticazione in 4G avvengono attraverso un protocollo di _challenge-and-response_ stabilito dal 3GPP chiamato **AKA (Authentication and Key Agreement)**. Esso assicura autenticazione delle entità, integrità e confidenzialità dei messaggi. Ecco i passaggi chiave di come un dispositivo mobile si autentica su una rete visitata (4G LTE):

1. **Richiesta di Attach:** Il dispositivo mobile invia un messaggio di _attach_ alla Base Station contenente il proprio IMSI, che viene inoltrato all'MME della rete visitata.

2. **Auth_Req verso HSS:** L'MME invia l'IMSI e le informazioni della rete visitata all'HSS nella rete "home" del dispositivo.

3. **Elaborazione HSS (Challenge):** L'HSS, utilizzando una chiave segreta pre-condivisa (chiamiamola $K_{HSS-M}$), calcola un token di autenticazione (`auth_token`) e la risposta attesa (`xres_{HSS}`). L'HSS invia entrambi all'MME.

4. **Autenticazione della Rete (da parte del mobile):** L'MME inoltra l'`auth_token` al mobile. Poiché il token contiene informazioni criptate dall'HSS con la chiave segreta, il dispositivo può verificare matematicamente che la rete possieda effettivamente la chiave condivisa. A questo punto, _il mobile ha autenticato la rete_.

5. **Risposta del Mobile (Response):** Il dispositivo mobile esegue lo stesso calcolo crittografico utilizzando la sua chiave segreta per generare la risposta $res_{M}$ e la invia all'MME.

6. **Validazione Finale MME:** L'MME confronta la risposta calcolata dal mobile ($res_{M}$) con quella attesa fornita in precedenza dall'HSS ($xres_{HSS}$). Se corrispondono, l'utente è autenticato! Infine, l'MME genera e invia le chiavi di sessione (es. basate su AES, come $K_{BS-M}$) alla Base Station per crittografare il traffico radio sul canale wireless 4G.

### Differenze Cruciali nella Sicurezza 5G

Nel passaggio al 5G, il modello di sicurezza è stato ulteriormente irrobustito:

- Nel 4G la decisione finale di autenticazione spettava all'MME nella rete visitata; **nel 5G questa decisione viene presa direttamente dalla rete Home**, mentre l'AMF visitato agisce solo da "intermediario" (pur potendo ancora rifiutare la connessione).

- Nel 4G, il delicato codice IMSI veniva trasmesso alla Base Station in chiaro ("cleartext"); **nel 5G, l'identificativo viene protetto sfruttando crittografia a chiave pubblica** (Public Key Crypto), offrendo un livello di privacy superiore.

---

> [!abstract] Sintesi
>
> L'evoluzione dalle reti cellulari analogiche al 5G rappresenta una trasformazione architetturale profonda, non solo un incremento di velocità. Il cuore di questa trasformazione è il passaggio a un **All-IP core** (introdotto nel 4G con l'EPC) e poi a una **Service Based Architecture cloud-native** (nel 5G con il NG-Core), che separa nettamente il piano di controllo dal piano dati e organizza le funzioni di rete come microservizi indipendenti. Il meccanismo del **tunneling GTP** è il pilastro che abilita la mobilità trasparente all'utente: mantenendo fissi gli indirizzi IP mentre aggiorna dinamicamente gli endpoint dei tunnel, la rete garantisce continuità di sessione durante gli spostamenti. Infine, il protocollo **AKA** assicura la mutua autenticazione tra dispositivo, rete visitata e rete home, con il 5G che rafforza ulteriormente la privacy nascondendo l'IMSI tramite crittografia a chiave pubblica e centralizzando la decisione di autenticazione nella rete home.
