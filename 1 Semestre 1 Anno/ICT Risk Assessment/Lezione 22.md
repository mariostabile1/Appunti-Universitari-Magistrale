## Evasione e Obfuscation

Gli attaccanti utilizzano tecniche di evasione per aggirare i sistemi di rilevamento e le analisi di _reverse engineering_.

### Polymorphic Malware

Il malware polimorfico cifra il proprio corpo principale e allega un modulo di decifratura (_decryptor_). Ogni infezione genera una nuova chiave e un nuovo corpo cifrato, cambiando la firma del file. Tuttavia, il codice del decryptor rimane presente e può essere rilevato, a meno che non venga utilizzata la forza bruta per la decifratura (come nel virus Crypto) .

### Tecniche di Obfuscation

L'obfuscation trasforma il codice per renderlo inintelligibile mantenendone la semantica. Le trasformazioni principali sono:

- **Lexical**: Modifica dei nomi delle variabili (es. nomi casuali).
    
- **Control**: Alterazione del flusso di controllo (es. riordinamento dei loop, inserimento di codice morto).
    
- **Data**: Modifica delle strutture dati (es. codifica delle stringhe, split degli array) .
    
- **Anti-disassembly/Anti-debugging**: Tecniche per impedire l'analisi dinamica o statica .

### Casi di Studio e Tecniche Avanzate

- **Emotet**: Utilizza catene di esecuzione complesse (_Execution Chains_) che coinvolgono documenti Office con macro, script PowerShell offuscati e file HTA eseguiti tramite `mshta.exe`. Le varianti mostrano un'alta diversità nelle catene di invocazione per evadere le firme statiche .
    
- **LockBit 3.0**: Introduce la protezione tramite password per l'esecuzione. Senza la chiave crittografica corretta (la password), l'eseguibile non può essere decifrato né analizzato dalle sandbox automatiche .
    
- **Zmist**: Un virus metamorfico avanzato ("ZOmbie's Mistfall") che integra i propri frammenti di codice all'interno dell'applicazione ospite ("islands"), collegandoli tramite salti casuali e inserendo istruzioni spazzatura (_Executable Trash Generator_). Questo rende estremamente difficile individuare il punto di ingresso o la firma del virus .

Anche applicazioni legittime come **Skype** utilizzano tecniche di obfuscation e anti-dumping per proteggere la proprietà intellettuale, rendendo difficile il reverse engineering del protocollo.