## VPN e Modalità IPsec

Il protocollo IPsec opera secondo due modalità fondamentali che determinano come i pacchetti vengono incapsulati e protetti: **Transport Mode** e **Tunnel Mode**.

Nel **Transport Mode**, viene cifrato e/o autenticato solamente il payload del pacchetto IP originale, mentre l'header IP originale rimane in chiaro. Questa modalità è tipicamente utilizzata per sessioni end-to-end tra due host specifici. Poiché l'header IP non è protetto, gli indirizzi sorgente e destinazione rimangono visibili, il che non nasconde le statistiche di comunicazione e lascia il traffico esposto ad analisi dei metadati e del traffico stesso .

La struttura del pacchetto in Transport Mode varia a seconda del protocollo di sicurezza utilizzato. Con l'**Authentication Header (AH)**, l'header AH viene inserito tra l'header IP originale e il payload (livello 4), autenticando l'intero pacchetto ma lasciando i dati in chiaro. Con l'**Encapsulating Security Payload (ESP)**, l'header ESP viene inserito dopo l'header IP; il payload e il trailer ESP vengono cifrati, mentre un campo di autenticazione opzionale chiude il pacchetto. È possibile combinare AH ed ESP, applicando prima l'incapsulamento ESP e successivamente il calcolo AH .

Nel **Tunnel Mode**, utilizzato prevalentemente per le VPN Site-to-Site, l'intero pacchetto IP originale viene incapsulato all'interno di un nuovo pacchetto IPsec. Questo significa che il pacchetto originale diventa il payload del nuovo pacchetto, che avrà un nuovo header IP esterno. Questa configurazione nasconde i pattern di traffico e gli indirizzi IP interni (privati), proteggendo la topologia della rete interna. L'indirizzo esterno appartiene solitamente al gateway di sicurezza .

## Algoritmi di Crittografia in IPsec

La robustezza di IPsec dipende dalla scelta degli algoritmi di crittografia. L'**AES (Advanced Encryption Standard)** è l'algoritmo simmetrico più diffuso, noto per efficienza e sicurezza, disponibile con chiavi a 128, 192 e 256 bit. Il **3DES (Triple Data Encryption Standard)** applica l'algoritmo DES tre volte; sebbene ancora in uso, è in fase di dismissione in favore di AES. Algoritmi come **Blowfish** e il suo successore **Twofish** offrono flessibilità con lunghezza delle chiavi variabile. Infine, **ChaCha20-Poly1305** combina un cifrario a flusso con un codice di autenticazione (MAC), risultando ideale per ambienti con risorse hardware limitate grazie alla sua velocità .

La selezione dell'algoritmo deve bilanciare tre fattori: i requisiti di sicurezza (dati sensibili richiedono AES-256), le considerazioni sulle prestazioni (la crittografia consuma risorse) e le capacità hardware dei dispositivi (dispositivi legacy potrebbero non supportare algoritmi intensivi) .

## Sicurezza Hardware e Trusted Execution Environments

Per garantire la sicurezza in ambienti IoT e ICS, dove il sistema è spesso esposto fisicamente, si ricorre a estensioni hardware che creano ambienti di esecuzione fidati.

### ARM TrustZone

L'architettura **ARM TrustZone** partiziona tutte le risorse hardware e software in due mondi distinti: il **Secure World**, destinato al sottosistema di sicurezza, e il **Normal World** per tutto il resto. Questa separazione è garantita a livello hardware, impedendo ai componenti del mondo normale di accedere alle risorse del mondo sicuro .

La separazione è implementata tramite il **NS bit (Non-Secure bit)** sul bus di sistema. Tutti i master nel mondo non sicuro hanno questo bit impostato a livello hardware, rendendo impossibile indirizzare periferiche o memoria mappata nel mondo sicuro .

Il processore stesso è virtualizzato: ogni core fisico fornisce due core virtuali (uno sicuro e uno non sicuro) che operano in _time-slice_. Il passaggio tra i due mondi è gestito da una modalità speciale chiamata **Monitor Mode**, attivata da istruzioni dedicate come `SMC` (Secure Monitor Call) o da interruzioni hardware . Il monitor agisce come un gatekeeper robusto, salvando e ripristinando lo stato dei mondi durante il context switch .

Anche la memoria è partizionata. La cache L1 e L2 include bit aggiuntivi per marcare lo stato di sicurezza delle linee, permettendo la coesistenza di dati sicuri e non sicuri senza necessità di svuotare la cache ad ogni switch, migliorando le prestazioni . Esiste inoltre la **World-shared memory**, una porzione di memoria non sicura mappata anche nel mondo sicuro, che permette lo scambio efficiente di dati tra applicazioni normali e trusted (es. per decifrare contenuti multimediali) .

### Intel SGX (Software Guard Extensions)

Intel SGX introduce il concetto di **Enclave**, una regione di memoria isolata per codice e dati, protetta anche da processi privilegiati come il sistema operativo o l'hypervisor. Questo modello assume che il proprietario della piattaforma possa essere malevolo, fidandosi esclusivamente della CPU e dell'enclave stessa .

La memoria dell'enclave risiede nell'**Enclave Page Cache (EPC)**, una porzione di memoria riservata (PRM) cifrata hardware. L'accesso a questa memoria è mediato dal **Memory Encryption Engine (MEE)**, che cifra e decifra i dati on-the-fly solo all'interno del core della CPU, garantendo che sulla RAM fisica i dati siano sempre cifrati e protetti da integrità . Il controllo degli accessi è gestito tramite la **Enclave Page Cache Map (EPCM)**, che memorizza permessi e configurazioni per ogni pagina, impedendo accessi non autorizzati anche se mappati nelle tabelle delle pagine del sistema operativo .

SGX supporta l'**Attestazione**, un processo per verificare che il codice in esecuzione sia autentico e non manomesso. La **Local Attestation** permette a due enclavi sulla stessa CPU di autenticarsi a vicenda. La **Remote Attestation** permette a un'enclave di provare la propria identità a una terza parte remota. Il processo coinvolge una _Quoting Enclave_ (QE) che firma il report dell'enclave con una chiave di attestazione fornita da Intel, permettendo la verifica esterna .

## Confidential Computing

Il **Confidential Computing** è una tecnologia che mira a isolare i dati sensibili in un'enclave protetta della CPU durante l'elaborazione. Mentre la crittografia tradizionale protegge i dati _at rest_ (archiviazione) e _in transit_ (rete), il confidential computing elimina la vulnerabilità rimanente proteggendo i dati _in use_ (durante il runtime). Questo è cruciale per il cloud computing, permettendo di elaborare dati sensibili o proprietà intellettuale su infrastrutture pubbliche senza che il cloud provider possa accedervi .

## SGX-LKL: Library OS per Enclavi

Per facilitare l'esecuzione di applicazioni esistenti all'interno di enclavi SGX (che normalmente non supportano chiamate di sistema standard), è stato sviluppato **SGX-LKL**. Questo sistema porta l'architettura **Linux Kernel Library (LKL)** all'interno dell'enclave, permettendo di eseguire binari Linux non modificati. SGX-LKL fornisce un supporto completo al filesystem e allo stack di rete, gestendo internamente le chiamate di sistema e l'allocazione della memoria, interfacciandosi con l'host solo per operazioni di I/O strettamente necessarie .