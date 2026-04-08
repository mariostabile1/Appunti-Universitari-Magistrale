# 16 Intrusion Analysis

L'analisi delle intrusioni è il processo metodico di esame degli eventi di sicurezza per identificare tentativi di accesso non autorizzato o compromissione dei sistemi. Questo campo si avvale di modelli matematici e strutture logiche per distinguere tra attività legittime (rumore di fondo) e attacchi reali, cercando di ridurre l'incertezza intrinseca nel monitoraggio di reti complesse.
## 16.1 Bayes theorem

L'applicazione del teorema di Bayes nell'Intrusion Detection è fondamentale per comprendere il problema del **Base Rate Fallacy**. Questo paradosso statistico dimostra che, in un contesto dove le intrusioni reali sono eventi rari (basso "base rate") rispetto al traffico legittimo, anche un sistema di rilevamento con un'accuratezza molto elevata produrrà un numero preponderante di falsi positivi.

La probabilità che un allarme corrisponda effettivamente a un'intrusione è data dalla formula:
$$P(Intrusion|Alarm) = \frac{P(Alarm|Intrusion) \times P(Intrusion)}{P(Alarm)}$$
Dove $P(Alarm|Intrusion)$ è la sensibilità del sistema (True Positive Rate), $P(Intrusion)$ è la probabilità a priori che avvenga un attacco, e $P(Alarm)$ è la probabilità totale che suoni l'allarme. Se $P(Intrusion)$ è molto piccolo, il valore finale sarà basso, rendendo gli allarmi poco affidabili agli occhi dell'analista, che tenderà a ignorarli (fenomeno della "alarm fatigue").
## 16.2 Measuring #intrusions

Per quantificare e analizzare sistematicamente le intrusioni, non è sufficiente contare gli eventi isolati. È necessario modellare le relazioni tra le vulnerabilità e i permessi per comprendere come un attaccante possa muoversi all'interno del sistema. Esistono due modelli formali principali per rappresentare questi scenari: i grafi e gli alberi.
### 16.2.1 Graph

Un **Attack Graph** è una rappresentazione che modella lo stato del sistema e le transizioni possibili tra stati. I nodi del grafo rappresentano lo stato di sicurezza di una macchina o i privilegi di un utente, mentre gli archi (edges) rappresentano le azioni o gli exploit che permettono di passare da uno stato all'altro (ad esempio, sfruttare una vulnerabilità SSH per ottenere una shell root).

Il vantaggio dell'Attack Graph è che fornisce una visione completa di _tutti_ i percorsi possibili che un attaccante può intraprendere partendo da un punto di ingresso per raggiungere un obiettivo critico. Questo permette di identificare i colli di bottiglia o i nodi critici attraverso cui passano la maggior parte dei percorsi di attacco.
### 16.2.2 Tree

Un **Attack Tree** offre una rappresentazione gerarchica e orientata all'obiettivo. La radice dell'albero (Root Node) rappresenta l'obiettivo finale dell'attaccante (es. "Rubare dati sensibili"). I nodi sottostanti rappresentano i sotto-obiettivi o le azioni elementari necessarie per raggiungere il genitore.

La struttura utilizza logica booleana:

- **AND nodes**: Tutte le azioni figlie devono essere completate per soddisfare il nodo genitore.
    
- **OR nodes**: È sufficiente completare una qualsiasi delle azioni figlie.

Gli Attack Trees sono particolarmente utili per il calcolo del rischio quantitativo. Assegnando valori ai nodi foglia (costo dell'attacco, probabilità di successo, impatto), è possibile propagare questi valori verso l'alto per calcolare il costo o la probabilità dell'attacco complessivo alla radice.
### 16.2.3 Building and stopping intrusions

La costruzione di questi modelli richiede la scansione delle vulnerabilità e la mappatura della topologia di rete. Una volta costruito il modello, l'obiettivo difensivo è identificare il **Minimal Cut Set**. Questo è l'insieme minimo di archi o nodi che, se rimossi o patchati, disconnettono completamente l'attaccante dall'obiettivo finale, rendendo l'attacco impossibile. L'analisi si concentra sul trovare il cut set con il "costo" minore per il difensore (patching più economico o rapido) o il costo maggiore per l'attaccante.
## 16.3 Automating intrusion

L'automazione delle intrusioni sposta l'approccio dalla teoria alla simulazione pratica tramite **Adversary Emulation**. A differenza del semplice vulnerability scanning, l'emulazione cerca di replicare il comportamento, le tattiche e le procedure (TTPs) di un avversario umano specifico o generico, eseguendo catene di attacco complete in modo controllato.
### 16.3.1 Possible attacker’s actions

Un sistema automatizzato deve replicare il ciclo decisionale dell'attaccante. Le azioni tipiche includono la **Reconnaissance** (raccolta informazioni), l'**Initial Access** (ottenere un punto d'appoggio), la **Privilege Escalation** (ottenere diritti di amministratore), il **Lateral Movement** (spostarsi su altri host nella rete) e la **Collection/Exfiltration** (furto dati). L'automazione deve gestire non solo l'esecuzione dei comandi, ma anche la logica condizionale: decidere l'azione successiva in base all'output dell'azione precedente (ad esempio, scegliere un exploit diverso se il sistema operativo rilevato non è quello previsto).
### 16.3.2 Mitre att&ck matrix

Il **MITRE ATT&CK** (Adversarial Tactics, Techniques, and Common Knowledge) è la base di conoscenza standard de facto per descrivere le azioni degli attaccanti.

È strutturato come una matrice in cui le colonne rappresentano le **Tactics** (l'obiettivo tattico dell'avversario, il "perché", es. Persistence) e le celle rappresentano le **Techniques** (il metodo specifico usato, il "come", es. Registry Run Keys).

Questa matrice fornisce un linguaggio comune per mappare le capacità offensive dei tool di emulazione e le capacità difensive dei sistemi di rilevamento.
## 16.4 Caldera

**Caldera** è un framework di cybersicurezza sviluppato da MITRE per automatizzare le operazioni di adversary emulation. Funziona secondo un'architettura client-server.

Il server centrale gestisce le operazioni e la pianificazione, mentre gli **Agents** distribuiti sugli endpoint eseguono le azioni. Caldera utilizza il concetto di **Abilities**, che sono implementazioni specifiche di tecniche ATT&CK (spesso script o comandi shell). Le abilità possono essere concatenate in **Adversaries** (profili di attaccanti) che vengono eseguiti durante un'**Operation**.

L'obiettivo di Caldera è permettere al Blue Team (difensori) di testare la propria visibilità e le proprie regole di detection contro attacchi reali e riproducibili, identificando lacune nella copertura difensiva.
## 16.5 Cascade

Mentre Caldera si concentra sulla generazione dell'attacco (Red Teaming automatizzato), **Cascade** è un progetto di ricerca MITRE focalizzato sull'automazione dell'analisi difensiva (Blue Teaming).

Cascade affronta il problema del sovraccarico di dati analizzando i log di sistema e di rete per cercare relazioni causali tra eventi apparentemente disgiunti. L'obiettivo è identificare automaticamente la sequenza di eventi che costituiscono un'intrusione, riducendo il tempo che gli analisti umani devono dedicare alla correlazione manuale degli allarmi. Utilizza una logica di query investigativa per "unire i puntini" tra un evento sospetto (es. un processo anomalo) e la sua causa radice (es. una connessione di rete malevola precedente).