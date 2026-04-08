## Vulnerabilità Critiche e Trend Recenti

L'analisi delle vulnerabilità maggiormente sfruttate nel 2023 evidenzia la predominanza di difetti critici in software infrastrutturali e di gestione. Tra le tipologie più ricorrenti figurano la **Remote Code Execution (RCE)**, che permette l'esecuzione di codice arbitrario da remoto (es. CVE-2021-44228 in Log4j2), e la **Code Injection** o **SQL Injection** (es. CVE-2023-34362 in MOVEit Transfer). Altre categorie rilevanti includono i **Buffer Overflow** (sia heap-based che standard) e le **Privilege Escalation**, che consentono agli attaccanti di elevare i propri diritti amministrativi su dispositivi di rete come gateway e router.

## Case Study: Stuxnet e la Guerra Cyber-Fisica

Stuxnet rappresenta un punto di svolta nella storia della cybersecurity, essendo il primo malware noto progettato specificamente come arma cyber-fisica per sabotare sistemi industriali (**SCADA**). Il suo obiettivo primario erano i controllori logici programmabili (**PLC**) Siemens, in particolare il sistema di controllo di processo SIMATIC PCS 7, programmato tramite software WinCC e STEP 7. La specificità dell'attacco risiedeva nella capacità di colpire configurazioni PLC uniche, richiedendo una conoscenza profonda dei processi industriali target.

### Metodi di Propagazione e Air Gap

Una delle caratteristiche distintive di Stuxnet è la sua capacità di superare l'**Air Gap**, ovvero l'isolamento fisico tra la rete industriale critica e Internet. L'infezione iniziale avveniva tramite drive USB infetti, agendo come un ponte fisico trasportato inconsapevolmente dal personale.

Una volta all'interno della rete, Stuxnet utilizzava molteplici vettori per propagarsi lateralmente:

- **Zero-day Exploits:** Sfruttava vulnerabilità non note all'epoca, come la MS10-046 (vulnerabilità dei collegamenti LNK) e la MS10-061 (Print Spooler).
    
- **Exploit RPC:** Utilizzava vulnerabilità nel protocollo Remote Procedure Call, simili a quelle sfruttate dal worm Conficker.
    
- **Credenziali di Default:** Tentava l'accesso ai server database WinCC utilizzando password predefinite di Siemens codificate nel software.
    
- **Network Shares:** Si copiava attraverso le condivisioni di rete e sfruttava meccanismi di aggiornamento Peer-to-Peer.

### Vettore di Infezione USB: La Vulnerabilità LNK

Il meccanismo di infezione tramite USB sfruttava una vulnerabilità critica nella gestione delle icone di collegamento (**LNK**) da parte di Windows (CVE-2010-2568). Quando il sistema operativo tentava di visualizzare l'icona di un file di collegamento malevolo presente sul drive USB, caricava ed eseguiva automaticamente una **DLL** malevola (spesso mascherata all'interno di un file del pannello di controllo, .CPL), senza richiedere alcun click esplicito o esecuzione da parte dell'utente. Parallelamente, veniva utilizzato anche il classico metodo `AutoRun.inf` per eseguire il payload all'inserimento del dispositivo.