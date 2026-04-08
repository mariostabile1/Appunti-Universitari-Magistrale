## Endpoint Detection and Response (EDR)

L'evoluzione delle strategie difensive ha portato allo sviluppo di soluzioni **Endpoint Detection and Response (EDR)**, talvolta definite anche _Endpoint Threat Detection and Response (ETDR)_. Questi sistemi rappresentano una convergenza tra il monitoraggio basato su firme e quello basato su anomalie, automatizzando la raccolta di informazioni in tempo reale e la risposta alle intrusioni rilevate.

Un sistema EDR si compone principalmente di **agenti** software installati sui nodi finali (endpoint) e di un **motore analitico** centralizzato. Gli agenti monitorano lo stato di sicurezza del nodo, analizzando la memoria, i file e i pacchetti di rete, e possono eseguire azioni attive come l'applicazione di patch o l'arresto di processi malevoli. Le informazioni raccolte vengono trasmesse al motore centrale, che correla i dati per scoprire intrusioni in corso e attivare contromisure dinamiche per minimizzare l'impatto .

Le funzionalità di un EDR moderno sono vaste e includono:

- **Asset Discovery e Inventory:** Mantenimento di un inventario aggiornato di hardware, software e patch.
    
- **Vulnerability Assessment:** Scansione continua per identificare vulnerabilità note (CVE) e configurazioni errate.
    
- **Intrusion Detection:** Rilevamento di malware, rootkit e comportamenti anomali tramite analisi dei log e monitoraggio dell'integrità dei file (FIM).
    
- **Incident Response:** Risposte automatiche basate su policy (es. isolamento del nodo) e supporto per l'analisi forense .

## Trusted Platform Module (TPM) e Root of Trust

La sicurezza software tradizionale soffre di un limite intrinseco: se il sistema operativo o il kernel sono compromessi, nessun software di sicurezza che gira su di essi può essere considerato affidabile. Per superare questo problema, è necessario ancorare la fiducia a un componente hardware immutabile, noto come **Hardware Root of Trust**.

Il **Trusted Platform Module (TPM)** è un microchip dedicato (criptoprocessore) progettato per fornire funzioni di sicurezza basate sull'hardware. Esso include un generatore di numeri casuali, un motore crittografico (RSA, SHA-1) e una memoria persistente sicura per la memorizzazione di chiavi. Il TPM funge da radice di fiducia per la misurazione dell'integrità del sistema e per l'archiviazione sicura di segreti.

### Componenti Chiave del TPM

Il TPM gestisce diverse chiavi crittografiche fondamentali:

- **Endorsement Key (EK):** Una coppia di chiavi RSA univoca (pubblica/privata) generata al momento della fabbricazione del chip. La parte privata non lascia mai il TPM e serve a identificare univocamente il dispositivo.
    
- **Storage Root Key (SRK):** La radice della gerarchia di archiviazione, utilizzata per proteggere le altre chiavi create dall'utente o dalle applicazioni.
    
- **Attestation Identity Keys (AIK):** Chiavi utilizzate per firmare i report sullo stato del sistema durante il processo di attestazione, proteggendo la privacy dell'utente (evitando l'uso diretto della EK).

Un elemento distintivo del TPM sono i **Platform Configuration Registers (PCR)**. Questi registri non memorizzano dati arbitrari, ma accumulano hash crittografici (tipicamente SHA-1) che rappresentano la misurazione dei componenti software caricati durante l'avvio (BIOS, bootloader, kernel). L'operazione di aggiornamento di un PCR è unidirezionale ed è definita come `Extend(PCR, measurement) = SHA1(PCR || measurement)`. Questo meccanismo crea una catena di fiducia (_Chain of Trust_) che registra in modo inalterabile la sequenza di avvio del sistema.

## Boot Sicuro e Remote Attestation

Il processo di avvio sicuro inizia con il **Core Root of Trust for Measurement (CRTM)**, tipicamente una porzione immutabile del BIOS. Il CRTM misura il prossimo componente da caricare (es. il resto del BIOS), estende il valore nel PCR e poi passa il controllo. Ogni componente successivo ripete il processo ("Measure then Execute"), garantendo che lo stato finale dei PCR rifletta l'esatta configurazione software del sistema .

La **Remote Attestation** è il protocollo che permette a un verificatore remoto di accertarsi dello stato di fiducia di un dispositivo. Il TPM firma digitalmente i valori dei PCR utilizzando una _Attestation Identity Key (AIK)_ e invia questa "quote" al verificatore. Quest'ultimo confronta i valori ricevuti con quelli attesi (noti come "golden values") per determinare se il sistema è integro o è stato compromesso (es. da un rootkit che ha alterato il bootloader).

## Strumenti EDR Open Source

Esistono soluzioni open source che implementano funzionalità EDR. **OSSEC** è un sistema di _Host-based Intrusion Detection_ (HIDS) che esegue analisi dei log, controllo dell'integrità dei file, monitoraggio del registro e risposta attiva. **TheHive** è una piattaforma di risposta agli incidenti scalabile che facilita la collaborazione tra analisti, permettendo l'analisi di osservabili (IP, hash, domini) e l'integrazione con servizi di threat intelligence.