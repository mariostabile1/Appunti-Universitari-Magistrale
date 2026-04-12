# Unipi - Obsidian Vault

Appunti universitari della Magistrale all'Università di Pisa, organizzati in Obsidian e versionati con Git.

## Struttura del vault

```
Unipi - Obsidian/
├── 1 Semestre 1 Anno/
│   ├── ICT Risk Assessment/
│   └── Information Retrieval/
├── 2 Semestre 1 Anno/
│   ├── Advanced Databases/
│   ├── Datacenter Design and Operation/
│   ├── ICT Startup Building/
│   ├── Mobile and Cyber-Physical Systems/
│   └── Peer to Peer Systems and Blockchain/
└── General/
    ├── Images and Documents/
    └── SSH and CheatSheet/
```

All'interno di ogni corso gli appunti seguono la convenzione `Lezione N.md`. Alcuni corsi includono sottocartelle aggiuntive (es. `Appunti AI/`, `Esami/`, `Note sulle Slide/`).

## Pipeline di creazione degli appunti

```
Slide PDF/PPTX  →  Claude Code (skill appunti-universitari)  →  Revisione manuale  →  Commit
```

> [!info] Come funziona
> 1. Si passa a Claude Code il file della lezione (PDF o PPTX)
> 2. Claude genera gli appunti in Markdown usando la skill `appunti-universitari`
> 3. Si revisiona manualmente il file generato, lezione per lezione
> 4. Si committa nel repo Git

> [!tip] Skill usata
> La skill `appunti-universitari` è configurata per produrre appunti discorsivi, con callout Obsidian e tabelle, ottimizzati per la visualizzazione in Obsidian.

## Setup

### Prerequisiti

- [Obsidian](https://obsidian.md/) (qualsiasi versione recente)
- [Claude Code](https://claude.ai/code) CLI
- Git

### Aprire il vault

1. Clona il repository
2. In Obsidian: **Open folder as vault** → seleziona la cartella clonata
3. Abilita i plugin della community quando richiesto (vedi sezione Plugin)

### Plugin community utilizzati

| Plugin | Scopo |
|---|---|
| Obsidian Git | Sync automatico con Git |
| Admonition | Supporto callout estesi |
| Editing Toolbar | Barra strumenti per la formattazione |
| Better Export PDF | Esportazione PDF migliorata |
| Omnisearch | Ricerca avanzata nel vault |
| Pandoc | Conversione in altri formati |
| Style Settings | Personalizzazione del tema |

Il tema usato è **Minimal** con personalizzazioni via Style Settings.

### Convenzioni degli appunti

Le preferenze di formattazione per Claude Code sono in `CLAUDE.md`:

- Intestazioni mai numerate
- Callout Obsidian (`[!info]`, `[!warning]`, `[!tip]`, ecc.) per i concetti chiave
- Tabelle per confronti, glossari ed evoluzioni storiche
