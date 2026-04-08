# 14 Polymorphic malwares and Sandboxes

La continua evoluzione delle minacce informatiche ha portato a una corsa agli armamenti tra attaccanti e difensori. Mentre i sistemi di difesa tradizionali si basavano sul riconoscimento di firme statiche, il malware moderno ha sviluppato meccanismi per mutare il proprio aspetto mantenendo inalterata la funzionalità malevola.
## 14.1 Polymorphic malwares and viruses

Un malware **polimorfico** è un codice malevolo progettato per cambiare la propria firma digitale (hash o sequenza di byte) ogni volta che si replica o infetta un nuovo sistema. L'obiettivo è eludere i sistemi di rilevamento basati su firme (Signature-based Detection), che cercano pattern fissi all'interno dei file. Sebbene il "contenitore" cambi aspetto, il "payload" (la parte funzionale che esegue l'azione malevola) rimane semanticamente identico.
### 14.1.1 Encryption

La tecnica più comune per ottenere il polimorfismo è l'utilizzo della cifratura accoppiata a un motore di mutazione. Il corpo principale del virus viene cifrato con una chiave variabile, rendendolo illeggibile agli scanner antivirus. Al codice cifrato viene aggiunto un piccolo segmento di codice in chiaro, chiamato **Decryption Loop** o **Stub**.

Quando il malware viene eseguito, lo stub decifra il corpo del virus in memoria e trasferisce il controllo al codice malevolo. Per evitare che lo stub stesso diventi una firma riconoscibile, il motore polimorfico genera una routine di decifratura diversa per ogni infezione. Questo avviene utilizzando istruzioni assembly equivalenti (es. `ADD EAX, 1` al posto di `INC EAX`), inserendo istruzioni "spazzatura" (junk code) che non alterano il flusso logico (NOPs), o permutando l'ordine delle istruzioni indipendenti.
### 14.1.2 Emotet example

**Emotet** rappresenta uno degli esempi più sofisticati di malware polimorfico moderno. Nato come Banking Trojan nel 2014, si è evoluto in una piattaforma modulare di distribuzione malware (Dropper/Loader). La sua capacità di persistenza e diffusione è dovuta a un motore polimorfico avanzato che ricompila costantemente il proprio codice e modifica le routine di offuscamento. Emotet utilizza macro VBA malevole all'interno di documenti Office, le quali, una volta abilitate, scaricano payload che variano per ogni vittima, rendendo inefficaci le firme statiche generate poche ore prima.
### 14.1.3 Zmist example

Il virus **Zmist** (Zombie Mist) è considerato uno dei primi esempi di malware "metamorfico" con capacità di integrazione del codice. A differenza del polimorfismo che cifra il payload, il **Metamorfismo** riscrive interamente il codice del virus ad ogni iterazione, cambiandone la struttura interna, la logica e il flusso di controllo, pur mantenendo lo stesso comportamento. Zmist utilizza una tecnica chiamata "Code Integration": disassembla il file ospite, inserisce le proprie istruzioni tra quelle del programma legittimo, rigenera il codice e ricostruisce i riferimenti. Questo rende il virus parte integrante dell'eseguibile infetto, estremamente difficile da isolare o rimuovere senza danneggiare il file originale.
## 14.2 Sandboxes

Una **Sandbox** è un ambiente di esecuzione isolato e controllato, utilizzato per analizzare il comportamento di software sospetto in sicurezza. In ambito di analisi malware, la sandbox (spesso basata su macchine virtuali) permette di eseguire il campione (detonation) e monitorarne le interazioni con il file system, il registro di sistema e la rete. I risultati dell'esecuzione vengono confrontati con policy di sicurezza o euristiche per determinare se il file è malevolo.
### 14.2.1 Detecting Sandboxes

Per sopravvivere all'analisi, il malware moderno implementa tecniche di **Sandbox Evasion** o **VM Awareness**. Prima di eseguire il payload malevolo, il codice effettua dei controlli ambientali per determinare se sta girando su una macchina fisica reale o in un ambiente virtualizzato.

Le tecniche di rilevamento includono:

- Controllo dei driver di periferica o delle chiavi di registro tipiche di VMware, VirtualBox o QEMU.
    
- Verifica della risoluzione dello schermo (spesso le sandbox non usano risoluzioni standard user-friendly).
    
- Controllo dell'attività dell'utente, come il movimento del mouse o i click (in una sandbox automatizzata non c'è interazione umana reale).
    
- Analisi delle istruzioni CPU privilegiate (es. "Red Pill" attack) che restituiscono valori diversi su hardware virtualizzato.
### 14.2.2 More-specific detection

Tecniche più avanzate sfruttano le discrepanze temporali o hardware.

- **Timing Attacks**: Il malware utilizza l'istruzione `RDTSC` (Read Time-Stamp Counter) per misurare i cicli di clock necessari a eseguire un blocco di codice. Le operazioni in una VM sono generalmente più lente o hanno latenze inconsistenti rispetto all'esecuzione su bare metal.
    
- **Sleep Calls**: Il malware esegue lunghe chiamate di sospensione (es. `Sleep(1000000)`). Poiché le sandbox hanno un tempo limitato per analizzare ogni file (timeout di pochi minuti), il malware spera che l'analisi termini prima che il periodo di sonno finisca, venendo classificato come benigno. Alcune sandbox contrastano questa tecnica accelerando artificialmente il tempo di sistema (sleep skipping), ma il malware può rilevare anche questa manipolazione confrontando il tempo di sistema con una fonte esterna (NTP).
### 14.2.3 WannaCry example

Il ransomware **WannaCry** (2017) includeva un meccanismo che, involontariamente o meno, agiva come rilevamento di sandbox. Il malware tentava di connettersi a un dominio web specifico, lungo e senza senso. Se la connessione aveva successo, il malware terminava l'esecuzione (kill switch).

Molte sandbox di analisi sono configurate per simulare una connessione Internet fittizia ("FakeNet") che risponde positivamente a qualsiasi richiesta HTTP per osservare il comportamento di rete del malware. In questo scenario, la sandbox rispondeva "sì" alla connessione del kill switch, inducendo WannaCry a credere di essere in un ambiente di analisi (o sotto controllo del creatore) e a disattivarsi.
# 15 Detection Tools

Gli strumenti di rilevamento sono essenziali per identificare minacce note e comportamenti sospetti all'interno dell'infrastruttura.
## 15.1 Rule based Signature Detection

La **Rule Based Signature Detection** è il metodo tradizionale di identificazione delle minacce. Si basa sul confronto del contenuto di un file o di un pacchetto di rete con un database di firme note. Una firma è una sequenza univoca di byte, una stringa o un pattern esadecimale che identifica in modo univoco una specifica minaccia o famiglia di malware. Sebbene efficace e veloce per le minacce note, questo approccio è cieco di fronte a malware polimorfico, cifrato o zero-day.
## 15.2 Yara

**Yara** è uno strumento open-source progettato per aiutare i ricercatori di malware a identificare e classificare i campioni. È spesso descritto come "il coltellino svizzero del pattern matching". Yara permette di creare descrizioni di famiglie di malware basate su pattern testuali o binari.

Una regola Yara è composta da tre sezioni principali:

1. **Meta**: Contiene metadati descrittivi (autore, data, descrizione) che non influenzano il funzionamento della regola.
    
2. **Strings**: Definisce le variabili contenenti i pattern da cercare (stringhe di testo, sequenze esadecimali o espressioni regolari).
    
3. **Condition**: Espressione booleana che determina la logica di attivazione della regola (es. "la stringa A deve essere presente E la dimensione del file deve essere inferiore a 2MB").
## 15.3 Snort

**Snort** è un **Network Intrusion Detection System** (NIDS) open-source. A differenza di Yara che analizza file statici, Snort analizza il traffico di rete in tempo reale. Agisce come uno sniffer di pacchetti che applica un motore di regole al traffico passante per rilevare tentativi di intrusione, scansioni di porte e attacchi exploit.

L'architettura di Snort comprende:

- **Packet Decoder**: Prepara i pacchetti per l'elaborazione.
    
- **Preprocessors**: Normalizzano il traffico (es. riassemblano flussi TCP frammentati) per evitare tecniche di evasione.
    
- **Detection Engine**: Confronta i pacchetti normalizzati con le regole caricate.
    
- **Logging/Alerting System**: Genera output in base all'esito del confronto.
### 15.3.1 Rules

Le regole di Snort sono strutturate in due parti: l'**Header** e le **Options**.

L'Header definisce l'azione (es. `alert`, `log`, `drop`), il protocollo (TCP, UDP, ICMP), gli indirizzi IP sorgente/destinazione e le porte.

Le Options, racchiuse tra parentesi, definiscono i criteri specifici di payload detection. Esempio: cercare una specifica sequenza di byte (content) all'interno del payload del pacchetto, o verificare lo stato dei flag TCP.

La sintassi è posizionale e permette un'analisi molto granulare del traffico di rete.
## 15.4 Merging Signatures and Anomalies

I sistemi di sicurezza moderni non si affidano più esclusivamente alle firme o alle anomalie, ma fondono entrambi gli approcci per ridurre i falsi positivi e aumentare la capacità di rilevamento (Hybrid Detection).
### 15.4.1 Endpoint Detection and Response

L'**Endpoint Detection and Response** (EDR) rappresenta l'evoluzione dell'antivirus tradizionale. Mentre l'antivirus si concentra sul blocco preventivo tramite firme, l'EDR si concentra sul monitoraggio continuo e sulla registrazione degli eventi che avvengono sull'endpoint (creazione processi, modifica file, connessioni rete, modifica registro).

L'EDR raccoglie questa telemetria e utilizza l'analisi comportamentale (Behavioral Analysis) e l'apprendimento automatico per identificare pattern di attacco complessi che potrebbero sfuggire a una scansione statica. Inoltre, fornisce strumenti per la risposta agli incidenti, come l'isolamento dell'host dalla rete o l'uccisione di processi remoti.
### 15.4.2 Introspection

La **Virtual Machine Introspection** (VMI) è una tecnica di monitoraggio avanzata utilizzata in ambienti virtualizzati. Invece di installare un agente di sicurezza all'interno della macchina virtuale (che potrebbe essere disabilitato o ingannato dal malware se quest'ultimo ottiene privilegi di kernel - rootkit), la VMI osserva lo stato della macchina virtuale dall'esterno, ovvero dal livello dell'Hypervisor.

Dall'Hypervisor è possibile ispezionare la memoria, i registri della CPU e lo stato del disco della VM ospite. Poiché il sistema di monitoraggio è isolato in un dominio di sicurezza superiore, è estremamente difficile per il malware rilevare di essere osservato o manipolare i risultati dell'analisi.