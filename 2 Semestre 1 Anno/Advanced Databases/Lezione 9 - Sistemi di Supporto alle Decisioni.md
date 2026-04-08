---
tags:
  - università/advanced-databases
  - data-warehouse
  - OLAP
  - DSS
data: 2024-01-01
lezione: "9 - Sistemi di Supporto alle Decisioni"
professore: ""
---

# Sistemi di Supporto alle Decisioni

I **sistemi di supporto alle decisioni** (*Decision Support Systems*, DSS) sono un tipo di sistema informativo. Un **sistema informativo** è una raccolta organizzata di risorse, persone e procedure finalizzate a raccogliere, memorizzare, elaborare e comunicare informazioni necessarie a supportare le attività in corso. Le informazioni possono essere rappresentate come immagini, testo, audio o qualsiasi tipo di dato.

Le applicazioni tradizionali che usano database sono dette **OLTP** (On Line Transactional Processing), mentre le applicazioni di supporto alle decisioni sono dette **OLAP** (On Line Analytical Processing). La differenza chiave è che i sistemi OLTP sono usati per operazioni quotidiane e transazionali, mentre i sistemi OLAP sono usati per analizzare i dati ed estrarre conoscenza nascosta senza aggiornare i dati.

| Funzione | OLTP | OLAP |
|----------|------|------|
| Tipo | Operazioni | Decisioni |
| Utenti | Operativi (impiegati, IT) | Knowledge workers (manager, analisti) |
| Dettaglio | Dettagliato | Può essere aggregato |
| Utilizzo | Ripetitivo | Ad hoc |
| Dati | Correnti, interni | Storici, interni ed esterni |
| Operazioni per transazione | Pochi (decine) | Molti (milioni) |
| Aggiornamenti | Frequenti, piccoli | Rari, massivi |
| Memorizzato in | Database | Data Warehouse/DSS |

Mentre nei sistemi tradizionali esiste solitamente un unico database, nei DSS ci sono più **data warehouse**, uno per ogni analisi.

---

## Data Warehouse

> [!definition] Data Warehouse
>
> Un **data warehouse** è una raccolta di dati orientata ai soggetti, integrata, non volatile e variabile nel tempo a supporto delle decisioni del management.
>
> - **Orientata ai soggetti**: i dati sono memorizzati per soggetto, non per applicazione;
> - **Integrata**: i dati sono raccolti da fonti diverse e unificati;
> - **Non volatile**: i dati non vengono modificati interattivamente, ma possono essere aggiornati periodicamente;
> - **Variabile nel tempo**: i dati sono assunti come storici, raccolti su un lungo periodo.

### Architettura

Il termine "data warehousing" si usa per il processo di organizzazione dei dati in un data warehouse. L'architettura più comune è a **tre livelli**: sorgenti dati, data staging e data warehouse.

Il data warehouse viene caricato con dati estratti tramite applicazioni **ETL** (Extract, Transform, and Load) dalle diverse sorgenti dati, portandoli in una forma consistente e effettuando le necessarie operazioni di pulizia, infine memorizzandoli nel data staging.

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 9.1]**: Schema dell'architettura di un data warehouse — mostra i tre livelli: data sources → ETL → data staging → data warehouse.

### Modellazione

La creazione di un data warehouse avviene gradualmente, a diversi livelli di astrazione: modello concettuale, modello logico e infine modello a cubo multidimensionale.

#### Modello Concettuale

Il **Dimensional Fact Model** (DFM) è un modello concettuale grafico per i data warehouse che permette di rappresentare:

- **Fatti** (*Facts*): la più importante astrazione, sono le osservazioni delle prestazioni di un processo di business (es. vendite, click, reclami, visite). Possono essere periodici (un fatto per un gruppo di transazioni in un periodo) o accumulanti (un fatto per l'intera vita dell'evento). Sono modellati con **tabelle dei fatti** (rettangoli divisi in due parti: nome del fatto e misure);

- **Misure** (*Measures*): informazioni quantitative la cui aggregazione è di interesse. La funzione di aggregazione è spesso la somma, ma può essere anche count, media, max, min, ecc. Sono specificate all'interno delle tabelle dei fatti;

- **Dimensioni** (*Dimensions*): danno contesto ai fatti (rispondono a "chi, cosa, perché, quando, dove" accade un fatto). Ogni dimensione è descritta da un insieme di attributi usati per qualificare, categorizzare o riassumere i fatti. Sono rappresentate da un cerchio collegato alla tabella dei fatti, con ulteriori cerchi per ogni attributo.

In presenza di dimensioni, si può modellare una **relazione gerarchica** tra di esse. Ad esempio, per una dimensione temporale (Anno, Trimestre, Mese, Settimana, Giorno), si può costruire una gerarchia $Giorno \to Settimana \to Mese \to Trimestre \to Anno$, dove ogni elemento è più generale dei precedenti. Ogni arco della gerarchia modella una dipendenza funzionale tra i due attributi.

> [!note] Dove inserire l'immagine
>
> **[IMMAGINE - Fig. 9.2]**: Schema concettuale senza e con gerarchie — mostra come le dimensioni possono essere organizzate in strutture ad albero.

#### Modello Relazionale

Un modello concettuale può essere trasformato in uno schema relazionale applicando regole di mapping. Gli schemi più comuni sono:

- **Star schema**: la tabella dei fatti è al centro e per ogni dimensione viene aggiunta una chiave esterna. Le dimensioni sono trasformate in tabelle (con tutti gli attributi, senza considerare le gerarchie), collegate alla tabella dei fatti con relazioni uno-a-molti;

- **Snowflake schema**: simile allo star schema, ma alcune tabelle di dimensione sono normalizzate, suddividendo i dati in ulteriori tabelle;

- **Constellation schema**: ha più tabelle dei fatti che condividono tabelle di dimensione.

> [!note] Dove inserire le immagini
>
> **[IMMAGINE - Fig. 9.3]**: Star schema — tabella dei fatti al centro, dimensioni come raggi.
> **[IMMAGINE - Fig. 9.4]**: Snowflake schema — dimensioni normalizzate in ulteriori tabelle.
> **[IMMAGINE - Fig. 9.5]**: Constellation schema — più tabelle dei fatti con dimensioni condivise.

#### Modello a Cubo Multidimensionale

Un **modello di dati multidimensionale** (data cube) rappresenta i fatti con $n$ dimensioni come punti in uno spazio $n$-dimensionale. Un punto (fatto) è identificato dai valori delle sue dimensioni e ha un insieme associato di misure. È un modo intuitivo per pensare alle query OLAP.

Le operazioni di base su un data cube sono:

- **Slice**: "taglio" del cubo selezionando un valore specifico per una dimensione;
- **Dice**: simile allo slice, ma si selezionano due o più dimensioni;
- **Pivot**: rotazione degli assi dei dati, fornendo una presentazione alternativa;
- **Roll-up**: compressione del cubo lungo una dimensione, riassumendo l'informazione. Simile a un GROUP BY su tutte le altre dimensioni;
- **Drill-down**: inverso del roll-up; produce dati più dettagliati lungo una dimensione.

Assumendo che ogni dimensione di un data cube sia estesa con un valore aggiuntivo $*$ (che rappresenta la sommarizzazione lungo quella dimensione), il cubo può essere esteso con bordi contenenti i valori della somma lungo ogni possibile combinazione di dimensioni. Quando si seleziona un subcubo specificando una dimensione invece del suo valore (es. $(StoreId, *)$), si ottiene un **cuboid**, cioè un roll-up dei dati originali lungo tutte le altre dimensioni.

> [!note] Dove inserire le immagini
>
> **[IMMAGINE - Fig. 9.6]**: Tabella dei fatti con StoreId, ProductId, Qty.
> **[IMMAGINE - Fig. 9.7]**: Esempio di data cube — visualizzazione 2D o 3D dei dati.
> **[IMMAGINE - Fig. 9.8]**: Lattice del data warehouse — mostra le relazioni di ordine tra cuboid.

Tra i cuboid è definita una relazione d'ordine: $C_1 \leq C_2$ se $C_1$ può essere calcolato da $C_2$. Questo ordine è visualizzato usando un **data warehouse lattice**. Come si sceglie quali cuboid memorizzare? I dati sorgente permettono di calcolare tutti gli altri cuboid; se le query più comuni considerano solo un sottoinsieme delle dimensioni, si può scegliere di memorizzare quelli corrispondenti.

> [!question] Possibili domande d'esame
>
> - Qual è la differenza tra OLTP e OLAP?
> - Cosa si intende per data warehouse? Quali sono le sue quattro proprietà?
> - Descrivi l'architettura a tre livelli di un data warehouse e il ruolo dell'ETL.
> - Cos'è il Dimensional Fact Model? Quali elementi rappresenta?
> - Differenza tra star schema, snowflake schema e constellation schema.
> - Cos'è un data cube e quali sono le operazioni di base (slice, dice, roll-up, drill-down)?
> - Cos'è un cuboid e come si relazionano tra loro nel data warehouse lattice?
