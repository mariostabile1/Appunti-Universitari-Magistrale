## Prospettiva Adversary-Focused e Punti di Strozzatura

L'approccio moderno alla difesa dei sistemi ICT/OT si definisce **Adversary Focused**. Invece di limitarsi a considerare le vulnerabilità isolate, questo metodo analizza le azioni e le strategie degli attaccanti, valutando la probabilità che un'intrusione venga tentata e che abbia successo. L'obiettivo è dispiegare contromisure _cost-effective_, ottimizzando il rapporto tra investimento e riduzione del rischio.

In questo contesto, è fondamentale il concetto di **Choke Point** (punto di strozzatura) nei percorsi di attacco (_Attack Paths_). Un'intrusione è una sequenza di attacchi e azioni; spesso, più percorsi di intrusione distinti convergono su un singolo nodo o sfruttano la medesima vulnerabilità critica. Identificando e rimuovendo la vulnerabilità in questo nodo cruciale, è possibile interrompere simultaneamente molteplici catene di attacco. Questo dimostra che non è necessario correggere tutte le vulnerabilità del sistema (approccio spesso impraticabile ed economicamente inefficiente), ma è sufficiente concentrarsi su quelle che abilitano le azioni essenziali per l'avversario.

## Sicurezza Incondizionata e Condizionata

Esistono due approcci filosofici e operativi distinti alla sicurezza:

1. **Unconditional Security**: Assume che qualsiasi vulnerabilità presente nel sistema verrà inevitabilmente sfruttata dagli attaccanti, indipendentemente dal costo o dalla complessità dell'attacco. Questo approccio mira a difendere il sistema contro _qualsiasi_ agente di minaccia, risultando spesso in una spesa di risorse insostenibile.
    
2. **Conditional Security (Risk Management)**: Si basa sulla _Threat Intelligence_ per scoprire chi è interessato ad attaccare il sistema e quali vulnerabilità specifiche sono funzionali ai loro obiettivi. Si assume che alcune vulnerabilità non verranno sfruttate perché abilitano attacchi inutili, troppo costosi, complessi o rumorosi (facilmente rilevabili). La domanda guida non è "come difendere tutto", ma "cosa proteggere e da chi".

## Risk Assessment e Management

L'approccio moderno al rischio cyber segue un processo strutturato che inizia con l'analisi degli asset e del loro valore (impatto medio di un'intrusione). Successivamente, si procede con l'analisi dei **Threat Agents** (tramite Threat Intelligence e Threat Modelling) e delle vulnerabilità (Inventory e Vulnerability database). Una componente emergente è l'_Adversary Emulation_, spesso supportata dall'AI, per simulare il comportamento degli attaccanti.

Il rischio viene calcolato combinando l'impatto di un'intrusione riuscita con la probabilità che questa avvenga. Essendo il rischio un limite superiore al costo delle contromisure, la gestione del rischio prevede tre opzioni principali:

- **Accettare il rischio**: Se il rischio calcolato è molto inferiore al costo delle contromisure necessarie per mitigarlo, l'organizzazione può decidere di non modificare il sistema.
    
- **Ridurre il rischio**: Se il rischio è troppo alto, si adottano contromisure per eliminare vulnerabilità (riducendo la probabilità di successo dell'intrusione) o per limitare i danni (ad esempio tramite la cifratura dei database).
    
- **Trasferire il rischio residuo**: La parte di rischio che rimane dopo le mitigazioni (_Residual Risk_) può essere trasferita a terzi, tipicamente attraverso assicurazioni, a un costo fisso.

## La Matrice di Rischio (Risk Matrix)

Uno strumento comune per visualizzare e prioritizzare i rischi è la **Risk Matrix**. Questa matrice pone in relazione due dimensioni:

1. **Likelihood (Probabilità)**: La frequenza con cui si prevede che l'evento avverso accada (es. da "Raro" a "Quasi Certo").
    
2. **Impact (Impatto/Conseguenza)**: La severità del risultato se il rischio si concretizza (es. da "Insignificante" a "Severo").

In base alla combinazione di queste due variabili, la matrice assegna un livello di rischio (Basso, Medio, Alto, Estremo) che suggerisce l'azione manageriale da intraprendere. Ad esempio, un rischio "Estremo" (alta probabilità e alto impatto) richiede l'interruzione immediata dell'attività e azioni correttive urgenti, mentre un rischio "Basso" potrebbe richiedere solo un monitoraggio periodico.

### Critiche alle Matrici di Rischio

Nonostante la loro diffusione, le matrici di rischio presentano limitazioni significative che possono portare a decisioni subottimali:

- **Compressione del Range**: Le matrici possono confrontare in modo non ambiguo solo una piccola frazione delle coppie di rischi, assegnando spesso valutazioni identiche a rischi quantitativamente molto diversi (es. un evento raro ma catastrofico potrebbe finire nella stessa cella "Alta" di un evento frequente ma moderato).
    
- **Errori di Classificazione**: Possono assegnare rating qualitativi più alti a rischi quantitativamente minori, specialmente quando si confrontano eventi con frequenze e severità correlate negativamente.
    
- **Allocazione Subottimale delle Risorse**: Le categorie ampie e soggettive non permettono una allocazione precisa del budget di sicurezza.
    
- **Ambiguità**: Gli input (frequenza e severità) richiedono interpretazioni soggettive che possono variare drasticamente tra diversi valutatori.
    
- **Inconsistenza Isorisk**: Le linee di "uguale rischio" nella matrice non sempre corrispondono alla realtà matematica ($Probabilità \times Impatto$), portando a incongruenze dove rischi maggiori vengono classificati come minori rispetto al punto medio, e viceversa. Inoltre, le matrici tradizionali tendono a trascurare le dipendenze tra i rischi, come nel caso dei _choke points_ delle intrusioni.