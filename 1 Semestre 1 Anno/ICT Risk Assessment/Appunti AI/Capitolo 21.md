# 21 Internet of Things

L'Internet of Things (IoT) rappresenta l'estensione della connettività di rete a oggetti fisici e dispositivi di uso quotidiano. Questi dispositivi, che spaziano dai termostati domestici ai sensori industriali, non sono più entità isolate, ma nodi attivi in grado di generare, scambiare e consumare dati con minima interazione umana. Questo paradigma fonde il mondo fisico con quello digitale, creando sistemi cyber-fisici complessi.
## 21.1 Inside IoT

Per comprendere le vulnerabilità dell'IoT, è necessario analizzarne l'anatomia interna, che differisce significativamente dai tradizionali computer general-purpose.
### 21.1.1 Hardware components

L'hardware IoT è tipicamente caratterizzato da vincoli stringenti in termini di risorse computazionali, memoria ed energia. Il cuore del dispositivo è solitamente un **Microcontroller Unit** (MCU) o un **System on Chip** (SoC) che integra processore, memoria e interfacce di I/O.

A differenza dei PC, questi dispositivi operano spesso con pochi kilobyte di RAM e storage flash limitato. La connettività è garantita da moduli radio che supportano protocolli a basso consumo come ZigBee, Bluetooth Low Energy (BLE), LoRaWAN o Wi-Fi. I sensori (per raccogliere dati dall'ambiente) e gli attuatori (per modificare l'ambiente) completano l'interfaccia fisica. Questa eterogeneità hardware e la necessità di ridurre i costi di produzione spesso portano alla rimozione di funzionalità di sicurezza hardware native, come le unità di gestione della memoria (MMU) o i coprocessori crittografici.
### 21.1.2 Firmware

Il software che governa questi dispositivi è definito **Firmware**. Nei dispositivi più semplici ("bare-metal"), il firmware è un unico blocco di codice che gira direttamente sull'hardware senza un sistema operativo intermedio. Nei dispositivi più complessi, si utilizzano sistemi operativi real-time (RTOS) o versioni ridotte di Linux (es. Embedded Linux).

Una criticità fondamentale del firmware IoT è la sua natura monolitica e statica: spesso include librerie di terze parti obsolete e credenziali hardcoded. L'aggiornamento del firmware è storicamente un processo complesso e rischioso, che molte volte non viene mai eseguito durante l'intero ciclo di vita del prodotto, lasciando vulnerabilità note aperte per anni.
## 21.2 IoT Attacks

L'ecosistema IoT ha ampliato drasticamente la superficie di attacco globale. La scarsa sicurezza intrinseca dei dispositivi, combinata con la loro connessione "always-on", li rende bersagli ideali per la compromissione. Gli attacchi non mirano solo al furto di dati, ma spesso al reclutamento del dispositivo in **Botnet** (come la tristemente nota Mirai). Una volta compromessi, migliaia di dispositivi IoT insicuri (telecamere, DVR, router) vengono aggregati per sferrare attacchi **DDoS** (Distributed Denial of Service) volumetrici devastanti contro terze parti, sfruttando la banda di upload collettiva.
### 21.2.1 Shodan

Mentre Google indicizza i contenuti del World Wide Web, **Shodan** è un motore di ricerca che indicizza i dispositivi connessi a Internet. Shodan scansiona costantemente l'intero spazio di indirizzamento IPv4, interrogando le porte di servizio e raccogliendo i "banner" di risposta.

Questi banner contengono metadati preziosi: tipo di server web, versione del firmware, configurazione predefinita e talvolta indizi su vulnerabilità specifiche. Per un attaccante, Shodan è lo strumento primario di ricognizione: permette di localizzare istantaneamente migliaia di dispositivi vulnerabili, come webcam senza password o sistemi di controllo industriale (SCADA) esposti pubblicamente, senza dover effettuare scansioni attive che potrebbero essere rilevate.
## 21.3 Device Classification

Non tutti i dispositivi IoT sono uguali; la IETF (RFC 7228) propone una classificazione basata sulle risorse disponibili, fondamentale per determinare quali meccanismi di sicurezza siano implementabili.

I dispositivi **Class 0** sono estremamente limitati (pochi byte di RAM), incapaci di comunicare direttamente su IP in modo sicuro; necessitano di gateway per connettersi. I dispositivi **Class 1** (es. 10KB RAM, 100KB Flash) possono gestire stack IP leggeri (come CoAP su UDP) ma non supportano protocolli crittografici pesanti come TLS completo. I dispositivi **Class 2** e superiori hanno risorse comparabili a smartphone o piccoli PC e possono supportare stack di sicurezza standard.
### 21.3.1 Architecture

L'architettura tipica di un sistema IoT si sviluppa su tre livelli. L'**Edge** è costituito dai dispositivi fisici (sensori/attuatori) che operano sul campo. Il **Gateway** (o Fog Layer) funge da intermediario, aggregando i dati dai dispositivi Edge, traducendo protocolli (es. da ZigBee a IP) e talvolta eseguendo una prima elaborazione locale. Il **Cloud** è il backend centralizzato dove i dati vengono archiviati, analizzati e dove risiede la logica di business di alto livello. La sicurezza deve essere garantita non solo sui singoli nodi, ma su tutti i canali di comunicazione che attraversano questi tre livelli.
## 21.4 Security in IoT

La sicurezza nell'IoT affronta sfide uniche rispetto all'IT tradizionale. La longevità dei dispositivi (che possono restare in campo per decenni) contrasta con il rapido ciclo di obsolescenza del software. L'assenza di interfacce utente (headless devices) rende difficile comunicare avvisi di sicurezza o richiedere input all'utente per l'autenticazione. Inoltre, l'eterogeneità dei protocolli rende impossibile applicare una soluzione di sicurezza unica. L'obiettivo è garantire confidenzialità e integrità dei dati sensibili raccolti, ma soprattutto la disponibilità e la resilienza del servizio, dato che molti dispositivi IoT svolgono funzioni critiche.
## 21.5 New perspective on Attacks

L'introduzione dell'IoT cambia la natura stessa del rischio: si passa da un impatto puramente digitale (perdita di dati) a un impatto **cinetico** (danno fisico).

Se un attaccante compromette un termostato intelligente, può causare danni all'impianto o un consumo energetico eccessivo. Se compromette un dispositivo medico impiantabile o il sistema di frenata di un'auto connessa, il rischio riguarda la sicurezza fisica delle persone (**Safety**). In questo contesto, la Security (protezione da attacchi intenzionali) diventa un prerequisito indispensabile per la Safety (protezione da danni accidentali o malfunzionamenti).
## 21.6 Privacy Concerns

La pervasività dei sensori IoT solleva gravi preoccupazioni per la privacy. I dispositivi raccolgono dati in modo continuo e granulare all'interno di spazi privati (case, auto, corpo umano). Anche dati apparentemente innocui, se aggregati nel tempo, permettono di inferire abitudini di vita, stato di salute, orari di presenza in casa e preferenze personali. Spesso l'utente non ha controllo su dove questi dati vengano inviati o come vengano utilizzati dal produttore (e dai suoi partner), e le policy di privacy sono spesso oscure o inesistenti.
## 21.7 IoT Best Practices

Per mitigare i rischi intrinseci dell'IoT, è necessario adottare un approccio "Security by Design" che copra l'intero ciclo di vita del dispositivo.
### 21.7.1 Physical Security

Poiché i dispositivi IoT sono spesso dislocati in ambienti non presidiati, sono esposti al rischio di manomissione fisica. La sicurezza fisica include l'uso di case (involucri) difficili da aprire, la rimozione di porte di debug esterne (come JTAG o UART) che darebbero accesso diretto al processore, e l'integrazione di meccanismi di **tamper detection** che cancellano i dati sensibili se rilevano un tentativo di intrusione fisica.
### 21.7.2 Secure Boot

Il **Secure Boot** è la fondazione della fiducia nel dispositivo. Assicura che, all'accensione, il dispositivo esegua solo firmware autentico e firmato digitalmente dal produttore.

Il processo si basa su una catena di fiducia (**Chain of Trust**): il primo stadio del bootloader (scritto in una memoria ROM immutabile) verifica la firma crittografica del secondo stadio, che a sua volta verifica il sistema operativo, e così via. Se una verifica fallisce (indicando malware o corruzione), il dispositivo si rifiuta di avviarsi.
### 21.7.3 Secure OS

L'uso di un sistema operativo sicuro, possibilmente basato su microkernel o che supporti l'isolamento dei processi (come ARM TrustZone), limita i danni in caso di compromissione di un'applicazione. Il Secure OS deve fornire servizi di crittografia affidabili e gestire l'accesso alle risorse hardware in modo rigoroso, impedendo a un processo vulnerabile di accedere a tutta la memoria del sistema.
### 21.7.4 Credential Management

Una delle peggiori pratiche nell'IoT è l'uso di credenziali predefinite uguali per tutti i dispositivi (es. admin/admin). Le best practices impongono che ogni dispositivo abbia una password unica generata in fabbrica o che l'utente sia forzato a cambiarla al primo avvio. Inoltre, le credenziali (come le chiavi Wi-Fi o i token API) non devono mai essere memorizzate in chiaro nel firmware, ma salvate in aree di memoria sicura (Secure Element).
### 21.7.5 Secure Software Update

La capacità di aggiornare il dispositivo da remoto (**OTA - Over The Air**) è critica per la sicurezza a lungo termine. Il processo di aggiornamento deve essere robusto: il firmware scaricato deve essere autenticato (tramite firma digitale) per evitare l'installazione di firmware malevolo, e cifrato per proteggere la proprietà intellettuale. Inoltre, il meccanismo deve supportare il rollback automatico in caso di fallimento dell'aggiornamento, per evitare di rendere il dispositivo inutilizzabile ("bricking").
### 21.7.6 Side Channel Attack

Data l'accessibilità fisica, i dispositivi IoT sono vulnerabili ai **Side Channel Attacks**. Questi attacchi non sfruttano debolezze nell'algoritmo crittografico, ma nell'implementazione fisica. Analizzando il consumo di potenza (**Power Analysis**) o le emissioni elettromagnetiche del processore durante le operazioni di cifratura, un attaccante può estrarre le chiavi segrete. Le contromisure includono schermature hardware e l'uso di algoritmi implementati in modo da avere un tempo di esecuzione e un consumo energetico costanti, indipendentemente dai dati elaborati.