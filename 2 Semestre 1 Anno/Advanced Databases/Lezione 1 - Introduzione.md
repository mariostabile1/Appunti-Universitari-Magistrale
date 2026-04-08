---
tags:
  - università/advanced-databases
  - storage
  - memoria-permanente
data: 2024-01-01
lezione: "1 - Introduzione"
professore: ""
---

# Introduzione

L'uso più comune delle tecnologie informatiche è memorizzare e recuperare dati, che si tratti di testo, immagini, video o file audio. Con l'aumento della quantità di dati generati da diversi processi nel tempo, i sistemi di archiviazione devono evolversi per garantire un accesso affidabile ai dati, nonché un recupero rapido ed efficiente. I dati vengono tipicamente memorizzati in **database** (DB), conservati nella **memoria permanente**.

## La Tecnologia della Memoria Permanente

La tecnologia su cui si basa la memoria permanente utilizza i **dischi magnetici**, composti da un insieme di piatti (*platters*) che ruotano a velocità relativamente basse (rispetto alla velocità della CPU). L'interazione con i piatti avviene tramite testine (*heads*) fissate a bracci mobili. Ogni piatto presenta su entrambe le superfici un insieme di anelli concentrici chiamati **tracce** (*tracks*), utilizzati per memorizzare informazioni (eccetto le più interne e le più esterne). Ogni traccia è suddivisa in **settori** (*sectors*) della stessa dimensione, che corrispondono alla più piccola unità di trasferimento consentita dall'hardware. Le dimensioni tipiche dei settori sono 512 byte, 1 KB, 2 KB o 4 KB. Vi sono da 500 a 1000 settori per traccia e circa 100.000 tracce per superficie di un singolo piatto.

## Tempi di Accesso al Disco

Il tempo di accesso necessario per leggere una sezione del disco è composto da tre contributi:

- **Seek time** (tempo di posizionamento): tempo necessario a spostare la testina sulla traccia corretta;
- **Rotational delay** (ritardo rotazionale): tempo dato dalla rotazione del disco finché il settore corretto non si trova sotto la testina;
- **Transfer time** (tempo di trasferimento): tempo necessario per leggere o scrivere i dati.

Queste operazioni richiedono diversi millisecondi, il che le rende decisamente più lente di qualsiasi operazione sulla memoria principale (RAM), che richiede solo pochi nanosecondi.

> [!tip] Perché usiamo ancora i dischi?
>
> Nonostante la disparità di velocità, i dischi rimangono la tecnologia preferita per la memorizzazione dei dati. La memoria principale è **volatile**: una volta che la macchina si spegne, qualsiasi informazione nella RAM è persa per sempre. I dischi invece offrono un'archiviazione **affidabile**: le informazioni scritte possono essere recuperate anche dopo un riavvio.

## Memorie a Stato Solido

Una tecnologia più recente, chiamata **solid state storage** (SSD), e in particolare la **flash memory**, ha guadagnato popolarità negli ultimi anni. Offre l'affidabilità dei dischi magnetici ma con operazioni molto più rapide. Tuttavia, non è ancora diventato lo standard dominante a causa dei costi elevati.

> [!question] Possibili domande d'esame
>
> - Quali sono le componenti del tempo di accesso al disco?
> - Perché i dischi magnetici rimangono preferiti rispetto alla RAM per la memorizzazione persistente?
> - Quali sono i vantaggi e i limiti della flash memory rispetto ai dischi tradizionali?
