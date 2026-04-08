## <font color="#3f3151">Lezione 3: Proprietà del Linguaggio Naturale e Text Processing</font>

### <font color="#5f497a">Caratteristiche del linguaggio</font>

Il linguaggio naturale è il mezzo primario di comunicazione umana e si distingue nettamente dai linguaggi formali (come quelli di programmazione) per la sua natura intrinsecamente disordinata e complessa. La struttura del linguaggio è governata dalla **sintassi**, che definisce le regole per combinare le parole in frasi grammaticalmente corrette. Tuttavia, la correttezza sintattica non garantisce il senso: entra qui in gioco la **semantica composizionale**, il principio secondo cui il significato di una frase complessa è derivato dalla combinazione dei significati delle sue parti costituenti.

L'ostacolo principale nell'elaborazione automatica è l'**ambiguità**, che permea ogni livello linguistico. L'ambiguità può essere _lessicale_ (polisemia, dove una parola ha molteplici significati come "pesca"), _sintattica_ (strutture interpretabili in modi diversi) o _semantica_. Inoltre, il linguaggio è caratterizzato da una forte **dinamicità**: è un sistema in continua evoluzione dove nuovi termini vengono coniati (neologismi), le regole grammaticali si adattano all'uso (slang) e i significati mutano nel tempo, rendendo difficile mantenere modelli statici sempre validi.
### <font color="#5f497a">Natural Language Understanding (NLU)</font>

Il Natural Language Understanding (NLU) è la branca dell'intelligenza artificiale che si occupa della comprensione profonda del significato del testo, andando oltre la semplice manipolazione di simboli. I problemi di NLU sono spesso classificati come **AI-complete** (o AI-hard), poiché risolverli pienamente equivarrebbe a risolvere il problema centrale dell'Intelligenza Artificiale Generale: richiedono non solo capacità linguistiche, ma anche una vasta conoscenza del mondo e la capacità di ragionamento contestuale.

Nonostante la difficoltà teorica, l'NLU ha abilitato numerose applicazioni pratiche fondamentali per l'Information Retrieval moderno, tra cui la classificazione automatica dei testi, l'estrazione di informazioni strutturate da fonti non strutturate, la _Question Answering_ (rispondere a domande formulate in linguaggio naturale), la _Text Summarization_ (riassunto automatico) e la _Machine Translation_.
### <font color="#5f497a">Pipeline di elaborazione del testo</font>

Per trasformare un flusso di caratteri grezzi in unità informative gestibili da un algoritmo di IR, si applica una pipeline di elaborazione sequenziale.

La **Tokenizzazione** è il primo passo critico e consiste nella segmentazione del flusso di caratteri in unità minime chiamate _token_ (generalmente parole o numeri). Sebbene possa apparire banale, presenta complessità notevoli legate alla gestione della punteggiatura, degli acronimi, dei caratteri speciali e delle lingue che non usano spazi separatori (come il cinese). Strettamente correlato è il **Sentence Splitting**, che individua i confini delle frasi, distinguendo ad esempio un punto fermo di fine frase da un punto utilizzato in un'abbreviazione (es. "ecc.").

Per ridurre la variabilità del vocabolario e mappare varianti morfologiche allo stesso concetto, si utilizzano due tecniche principali:

1. Lo **Stemming** è un processo euristico e grezzo che rimuove i suffissi delle parole per ottenerne la radice (stem). Algoritmi come il _Porter Stemmer_ operano attraverso regole di troncamento; questo approccio è computazionalmente leggero ma può generare errori di _over-stemming_ (unificare parole con significati diversi) o _under-stemming_.
    
2. La **Lemmatizzazione** è un processo più raffinato che richiede un'analisi morfologica completa e spesso l'uso di dizionari. L'obiettivo è ricondurre ogni parola alla sua forma base o di dizionario, chiamata _lemma_ (es. da "sono" a "essere"). A differenza dello stemming, la lemmatizzazione tiene conto del contesto e della parte del discorso (Part-of-Speech), garantendo una maggiore precisione a fronte di un costo computazionale più elevato.
### <font color="#5f497a">Strumenti Python per il Text Processing</font>

Nell'ecosistema Python, l'elaborazione del testo è supportata da librerie standardizzate. **NLTK (Natural Language Toolkit)** è la suite di riferimento per la didattica e la prototipazione in NLP, offrendo moduli pre-costruiti per tokenizzazione, stemming, lemmatizzazione e accesso a corpora linguistici. Per la fase di acquisizione dati dal web, **urllib** è il modulo standard per gestire richieste HTTP e scaricare risorse, mentre **BeautifulSoup** è la libreria essenziale per il parsing di documenti HTML e XML. BeautifulSoup permette di navigare l'albero del DOM, pulire il codice sorgente delle pagine web ("tag soup") ed estrarre il contenuto testuale rilevante, scartando i tag di formattazione non necessari all'indicizzazione.