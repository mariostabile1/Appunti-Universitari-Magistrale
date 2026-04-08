## Segmentazione di Rete e VLAN

La **Network Segmentation** è la pratica di suddividere una rete in sottoreti distinte per ridurre la visibilità tra di esse. I segmenti sono connessi tramite un router che determina le regole di visibilità e accesso attraverso le reti. Esistono due modalità principali di segmentazione: logica e fisica. La segmentazione fisica prevede connessioni fisiche separate tra sottoreti distinte operando a livello 3, mentre la segmentazione logica crea reti virtuali (**VLAN**) dove i nodi, pur essendo fisicamente connessi alla stessa infrastruttura, sono logicamente isolati.

Le **VLAN (Virtual Local Area Networks)** operano al livello 2 (Data Link) dello stack TCP/IP. La separazione avviene basandosi sugli indirizzi MAC, facendo sì che ogni VLAN debba essere considerata come una LAN autonoma. Dato che i dati non possono essere scambiati direttamente tra VLAN diverse al livello Ethernet, ogni VLAN definisce il proprio dominio di broadcast isolato.

Per implementare questa separazione logica, i frame Ethernet vengono "taggati" (**VLAN Tagging**). Questo tag aggiunge quattro byte all'header Ethernet e gestisce informazioni cruciali: il **VLAN ID** (o tag), che definisce a quale rete logica appartiene il pacchetto, e la **Priority**, un valore tra 0 e 7 che permette di prioritizzare certi frame rispetto ad altri per la qualità del servizio (QoS).

## Egress Filtering e Igiene della Rete

Mentre i firewall sono tradizionalmente configurati per filtrare il traffico in ingresso (_Ingress Filtering_), l'**Egress Filtering** si occupa del monitoraggio e della restrizione del traffico in uscita dalla rete aziendale verso Internet. Questa pratica è fondamentale non solo per la sicurezza interna, ma per l'igiene complessiva dell'ecosistema informatico globale.

L'obiettivo primario è prevenire lo **IP Spoofing**. Un attaccante interno (o un malware) potrebbe inviare pacchetti con un indirizzo IP sorgente falsificato che non appartiene al range di indirizzi assegnati all'organizzazione. Bloccando in uscita qualsiasi pacchetto che non abbia un indirizzo sorgente legittimo della rete interna, si impedisce alla rete di diventare involontariamente una sorgente di attacchi DDoS basati su _reflection_ o _amplification_ e si contrasta lo spamming generato da botnet.

Inoltre, limitare le connessioni in uscita aiuta a bloccare il traffico di _Command & Control_ (C2) dei malware e l'esfiltrazione di dati sensibili derivante da configurazioni errate o attività malevole interne.

## Best Practices SANS per il Filtraggio

Il SANS Institute raccomanda di bloccare specifici protocolli e porte in uscita che non dovrebbero mai legittimamente attraversare il perimetro verso Internet. Questi filtri dovrebbero essere applicati il più vicino possibile al confine della rete, tipicamente sul _border router_ che collega all'ISP, fornendo un livello di protezione anche per il firewall stesso.

Le regole di blocco essenziali includono:

- Tutto il traffico diretto verso gli indirizzi IP gestiti internamente (per prevenire spoofing esterno).
    
- **MS RPC** (TCP/UDP 135), **NetBIOS** (TCP/UDP 137-139) e **SMB** (TCP 445): protocolli di condivisione file e stampanti Windows che espongono gravi vulnerabilità se accessibili pubblicamente.
    
- **TFTP** (UDP 69): protocollo di trasferimento file non autenticato.
    
- **Syslog** (UDP 514): per evitare la fuga di log di sistema.
    
- **SNMP** (UDP 161-162): protocollo di gestione rete che può rivelare dettagli infrastrutturali.
    
- **SMTP** (TCP 25): tutto il traffico email in uscita deve essere bloccato eccetto quello generato dal server di posta ufficiale dell'organizzazione, per impedire ai client infetti di inviare spam.
    
- **IRC** (TCP 6660-6669) e messaggi **ICMP** specifici (Echo/Reply, Host Unreachable) per ridurre la superficie di diagnosi disponibile agli attaccanti.

L'implementazione tecnica su router Cisco avviene tramite _Access Control Lists_ (ACL) estese applicate all'interfaccia esterna in direzione _outbound_, specificando esplicitamente il diniego (`deny`) per questi servizi critici.