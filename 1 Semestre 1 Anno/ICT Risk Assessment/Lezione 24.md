## Sandbox ed Evasione

Una **Sandbox** è un ambiente di esecuzione protetto, disgiunto e confinato rispetto al sistema reale, spesso ospitato in cloud. La sua funzione principale è duplice: analisi e difesa. Permette di scaricare ed eseguire codice potenzialmente rischioso per osservarne il comportamento senza compromettere l'integrità del sistema ospitante. Essenzialmente, una sandbox è una macchina virtuale altamente robusta progettata per rilevare comportamenti anomali come la cifratura o la cancellazione di file. Tuttavia, questo meccanismo presenta rischi intrinseci: se un malware riesce a evadere dalla sandbox ("escaping"), non esiste ulteriore linea di difesa. Inoltre, i malware moderni sono spesso _sandbox-aware_, ovvero capaci di rilevare se stanno girando in un ambiente virtualizzato e, in tal caso, inibire le proprie funzionalità malevole per apparire innocui.

### Strategie di Implementazione delle Sandbox

Esistono diverse strategie per implementare questi ambienti isolati, ognuna con specifici trade-off tra performance e sicurezza.

La tecnica del **De-privileging** (o "trap-and-emulate") consiste nell'eseguire il sistema operativo ospite (Guest OS) direttamente, ma a un livello di privilegio non amministrativo. Tutte le istruzioni che tentano di leggere o scrivere stati privilegiati vengono intercettate ("trapped") ed eseguite in modo controllato.

La **Para-Virtualization** richiede la modifica del Guest OS affinché comunichi direttamente con il Virtual Machine Monitor (VMM), fornendo informazioni di alto livello che facilitano la gestione.

L'**Interpretive Execution** aggiunge una modalità di esecuzione hardware dedicata per il Guest OS, riducendo il numero di trap necessarie e migliorando l'efficienza.

Infine, la **Binary Translation** sostituisce le istruzioni sensibili nel binario del guest direttamente a tempo di esecuzione ("on-the-fly") con codice di emulazione o _hypercall_. Questo approccio lavora sui binari senza richiedere il codice sorgente.

## Configurazione e Azioni in Snort

Snort, un popolare sistema IDS/IPS, definisce una serie di azioni che possono essere intraprese quando un pacchetto soddisfa una regola specifica.

L'azione **alert** genera un avviso e logga il pacchetto, mentre **log** si limita alla registrazione. **Pass** ordina di ignorare il pacchetto.

Esistono azioni più complesse come **activate**, che solleva un allarme e attiva una regola dinamica successiva, e **dynamic**, che rimane inattiva (idle) finché non viene innescata da una regola _activate_, dopodiché inizia a loggare.

In modalità IPS (prevenzione), si usano azioni di blocco: **drop** blocca e logga il pacchetto; **reject** blocca, logga e invia una risposta di errore al mittente (TCP Reset o ICMP Port Unreachable); **sdrop** blocca il pacchetto silenziosamente senza loggarlo.

### Opzioni delle Regole

Le regole di Snort possono essere raffinate utilizzando opzioni specifiche per ispezionare i campi dell'header IP e TCP.

Opzioni come **msg** e **logto** gestiscono l'output degli avvisi. Per il filtraggio tecnico, **ttl** e **tos** verificano i campi Time-To-Live e Type-Of-Service nell'header IP. È possibile controllare la frammentazione tramite **id** (fragment ID) e **fragbits**, o ispezionare le opzioni IP con **ipoption**.

A livello di payload e trasporto, **dsize** verifica la dimensione del payload, **flags** controlla i flag TCP (es. SYN, ACK) e **seq** verifica il numero di sequenza TCP.

### Ordine di Applicazione delle Regole

L'ordine con cui le regole vengono applicate determina la postura di sicurezza e le prestazioni del sistema.

L'ordine **Drop > Pass > Alert > Log** è considerato il più sicuro. Verificando prima cosa deve essere bloccato (_Drop_), si garantisce che il traffico malevolo venga fermato immediatamente. Tuttavia, poiché la maggior parte del traffico è legittimo, questo ordine può essere computazionalmente costoso, costringendo il sistema a scorrere le regole di blocco per ogni pacchetto valido.

Un ordine alternativo, **Pass > Drop > Alert > Log**, privilegia le prestazioni lasciando passare subito il traffico noto come sicuro, ma comporta rischi maggiori se le regole di _Pass_ sono configurate in modo impreciso.