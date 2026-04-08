# 12 Trusted Zone & Intel SGX

La necessità di eseguire codice sensibile e proteggere dati critici (come chiavi crittografiche o dati biometrici) su dispositivi che eseguono sistemi operativi complessi e potenzialmente vulnerabili ha portato allo sviluppo dei Trusted Execution Environment (TEE). Questi ambienti isolano l'esecuzione sicura dal resto del sistema.
## 12.1 TrustZone - Overall architecture

**TrustZone** è un'estensione architetturale hardware per i processori ARM, progettata per fornire una base sicura per le applicazioni. Il concetto fondamentale è la divisione dell'hardware e del software in due mondi distinti: il **Secure World** e il **Normal World** (spesso chiamato anche Non-Secure World).

Il Normal World ospita il sistema operativo "ricco" (Rich OS, come Android o Linux) e le applicazioni standard. Il Secure World ospita un kernel sicuro e servizi fidati (Trusted Apps). L'isolamento hardware garantisce che nessun software in esecuzione nel Normal World possa accedere direttamente alle risorse del Secure World, nemmeno se quel software ha privilegi di root o kernel.
## 12.2 TrustZone - System architecture

L'architettura non si limita al processore, ma si estende all'intero sistema on-chip (SoC). Perché la sicurezza sia garantita, l'isolamento deve essere mantenuto attraverso il bus, la memoria, le periferiche e le interfacce di debug.
### 12.2.1 System Bus

Il bus di sistema (tipicamente protocollo AMBA AXI) è modificato per trasportare un segnale di controllo aggiuntivo per ogni transazione di lettura o scrittura. Questo segnale è noto come **NS bit** (Non-Secure bit).

Quando l'NS bit è impostato a 1, la transazione appartiene al Normal World; se è 0, appartiene al Secure World. I componenti hardware collegati al bus (controller di memoria, periferiche) controllano questo bit per decidere se accettare o rifiutare la richiesta, garantendo che le risorse sicure siano invisibili e inaccessibili alle transazioni non sicure.
### 12.2.2 Processor

Il processore è esteso per supportare due stati di sicurezza virtuali. Invece di avere core fisici dedicati alla sicurezza, un singolo core fisico esegue codice di entrambi i mondi in time-slicing. Il core mantiene una copia separata dei registri di stato critici per ciascun mondo. Per il software, sembra di avere a disposizione due processori virtuali: uno sicuro e uno non sicuro. Il passaggio tra i due mondi è strettamente controllato e non può avvenire arbitrariamente.
### 12.2.3 Monitor

Il **Monitor Mode** è la modalità del processore che funge da gatekeeper tra i due mondi. È il software di livello più basso e più privilegiato nel sistema. Il passaggio dal Normal World al Secure World avviene tramite un'eccezione o un'istruzione esplicita chiamata **SMC** (Secure Monitor Call). Quando viene invocata, il processore entra in Monitor Mode, salva lo stato del contesto non sicuro (registri, stack pointer), ripristina lo stato del contesto sicuro e passa il controllo al Secure OS. Questo context switch è trasparente al sistema operativo del Normal World.
### 12.2.4 Memory subsystem

La memoria fisica (RAM) deve essere partizionata. Questo compito è affidato a un componente hardware chiamato **TrustZone Address Space Controller** (TZASC). Il TZASC si interpone tra la CPU e la DRAM e permette di configurare regioni di memoria come "Secure only" o "Non-Secure". Se un accesso con NS bit=1 tenta di leggere o scrivere in una regione marcata come Secure, il TZASC blocca la transazione e può generare un'eccezione di sicurezza.
### 12.2.5 Interrupts

La gestione delle interruzioni è divisa per garantire che il Secure World possa sempre rispondere a eventi critici, anche se il Normal World è bloccato o compromesso. ARM utilizza due linee di interrupt: **IRQ** (Interrupt Request) e **FIQ** (Fast Interrupt Request).

Tipicamente, gli IRQ sono assegnati al Normal World (gestiti dal Rich OS), mentre i FIQ sono riservati al Secure World. Se arriva un FIQ mentre la CPU è nel Normal World, il processore entra immediatamente in Monitor Mode, permettendo al codice sicuro di gestire l'evento con priorità assoluta.
### 12.2.6 Debug

L'accesso di debug (JTAG) rappresenta un vettore di attacco fisico significativo. L'architettura TrustZone permette di configurare le porte di debug in modo granulare. È possibile abilitare il debug del Normal World mantenendo disabilitato il debug del Secure World. Per effettuare il debug del Secure World, è necessario un meccanismo di autenticazione crittografica (Secure Debug), impedendo l'analisi del codice sicuro o il dump della memoria sicura da parte di attori non autorizzati.
### 12.2.7 Secure OS

Nel Secure World gira un sistema operativo minimale e altamente ottimizzato, spesso conforme allo standard **GlobalPlatform TEE**. Questo Secure OS gestisce le risorse sicure, schedula le Trusted Applications (TA) e comunica con il Normal World tramite driver dedicati. Poiché la superficie di attacco del Secure OS è molto ridotta (pochissime righe di codice rispetto a Linux/Android), è più facile verificarne formalmente la correttezza e la robustezza.
## 12.3 Intel Software Guard Extensions SGX

A differenza di TrustZone che divide l'intero sistema in due mondi, **Intel SGX** (Software Guard Extensions) adotta un approccio basato sull'isolamento a livello di applicazione. L'obiettivo è proteggere porzioni specifiche di codice e dati anche se il sistema operativo o l'hypervisor sono compromessi.
### 12.3.1 Enclave

L'unità fondamentale di protezione in SGX è l'**Enclave**. Un Enclave è una regione di memoria privata e cifrata all'interno dello spazio di indirizzamento di un'applicazione.

Il contenuto dell'Enclave è protetto via hardware:

1. **Memory Encryption**: I dati sono decifrati dalla CPU solo quando si trovano all'interno del processore (cache/registri). Quando vengono scritti in RAM, sono automaticamente cifrati dalla Memory Encryption Engine (MEE).
    
2. **Access Control**: La CPU impedisce a qualsiasi software esterno all'Enclave, incluso il kernel del sistema operativo, il BIOS o l'hypervisor, di leggere o scrivere nella memoria dell'Enclave.

Questo modello inverte la gerarchia di fiducia tradizionale: l'applicazione non si fida più del sistema operativo, ma solo dell'hardware della CPU. L'Enclave contiene il codice e i dati sensibili, ed è possibile attestarne l'integrità tramite **Remote Attestation**, che prova a una terza parte remota che l'Enclave sta eseguendo un codice specifico su un processore Intel genuino.
# 13 Intrusion Detection

L'**Intrusion Detection System** (IDS) è un componente di sicurezza progettato per monitorare le reti o i sistemi alla ricerca di attività malevole o violazioni delle policy. A differenza dei firewall che agiscono preventivamente bloccando il traffico, gli IDS tradizionali analizzano ciò che accade (passive monitoring) e generano allarmi. Se il sistema è configurato per bloccare attivamente le minacce rilevate, si parla di **Intrusion Prevention System** (IPS).
## 13.1 Anomaly Based

I sistemi **Anomaly Based** (o comportamentali) basano il rilevamento sulla deviazione da un modello di "normalità".

Il funzionamento prevede due fasi:

1. **Training Phase**: Il sistema osserva il traffico o il comportamento del sistema per un periodo di tempo in condizioni "pulite", costruendo un profilo statistico di base (Baseline). Vengono appresi parametri come la larghezza di banda media, i protocolli usati, gli orari di connessione, ecc.
    
2. **Detection Phase**: Il sistema confronta l'attività corrente con la Baseline. Se l'attività devia oltre una certa soglia di tolleranza, viene generato un allarme.

Il vantaggio principale è la capacità di rilevare **Zero-Day Attacks** o minacce sconosciute, purché queste causino un comportamento anomalo. Lo svantaggio critico è l'alto tasso di **False Positives**: un comportamento legittimo ma inusuale (es. un picco di traffico per il rilascio di un nuovo prodotto) viene classificato come attacco. Inoltre, è suscettibile ad attacchi di _poisoning_ durante la fase di training, dove l'attaccante inietta traffico malevolo lentamente per farlo accettare come "normale".
### 13.1.1 Specification Based

Lo **Specification Based** IDS è una variante dell'approccio ad anomalie, ma invece di basarsi su profili statistici appresi, si basa su specifiche formali o standard (come le RFC dei protocolli). Il sistema definisce manualmente cosa costituisce un comportamento "corretto".

Ad esempio, in una comunicazione HTTP, lo standard definisce la struttura degli header. Se un pacchetto viola queste regole sintattiche o logiche, viene segnalato. Questo metodo riduce i falsi positivi rispetto all'approccio statistico puro, poiché le regole sono deterministiche, ma richiede un grande sforzo manuale per definire e mantenere le specifiche complete per ogni protocollo o applicazione monitorata.
## 13.2 Signature Based

I sistemi **Signature Based** (o Knowledge Based) operano in modo simile agli antivirus tradizionali. Utilizzano un database di firme (signatures) che descrivono pattern noti di attacchi.

Il motore di analisi ispeziona il contenuto dei pacchetti o dei log alla ricerca di corrispondenze esatte con le firme nel database (es. una specifica sequenza di byte in un exploit buffer overflow, o una stringa in una richiesta SQL Injection).

I vantaggi includono un tasso di **False Positives** molto basso (se la firma è precisa, l'attacco è reale) e un'alta velocità di elaborazione. Lo svantaggio fondamentale è l'incapacità di rilevare attacchi nuovi (**Zero-Day**) o varianti di attacchi noti che sono state modificate per non corrispondere alla firma (offuscamento o polimorfismo). Questo approccio richiede aggiornamenti costanti del database delle firme.