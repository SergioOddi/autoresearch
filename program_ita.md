
# autoresearch

Questo è un esperimento per far sì che l'LLM conduca la propria ricerca in autonomia.

### Configurazione

Per impostare un nuovo esperimento, collabora con l'utente per:

1. **Concordare un tag di esecuzione**: proponi un tag basato sulla data odierna (es. `mar5`). Il branch `autoresearch/<tag>` non deve esistere già — si tratta di una nuova esecuzione.
2. **Creare il branch**: esegui `git checkout -b autoresearch/<tag>` dal master attuale.
3. **Leggere i file pertinenti**: La repository è piccola, leggi questi file per avere il contesto completo (`README.md` per il contesto della repository; `prepare.py` per costanti fisse, preparazione dei dati, tokenizer, dataloader e valutazione che non devi modificare; `train.py` come unico file da modificare per l'architettura del modello, l'ottimizzatore e il ciclo di addestramento).
4. **Verificare l'esistenza dei dati**: Controlla che `~/.cache/autoresearch/` contenga i frammenti di dati (data shards) e un tokenizer, altrimenti chiedi all'utente umano di eseguire `uv run prepare.py`.
5. **Inizializzare results.tsv**: Crea `results.tsv` contenente solo la riga di intestazione, poiché la baseline verrà registrata dopo la prima esecuzione.
6. **Confermare e procedere**: Conferma che la configurazione sia corretta.

Una volta ottenuta la conferma, dai il via alla sperimentazione.

### Sperimentazione

Ogni esperimento viene eseguito su una singola GPU. Lo script di addestramento viene eseguito con un **budget di tempo fisso di 5 minuti** (tempo effettivo di addestramento, escludendo l'avvio e la compilazione). Puoi avviarlo semplicemente con: `uv run train.py`.

**Cosa PUOI fare:**
- Modificare `train.py` — questo è l'unico file che editi e tutto è concesso (architettura del modello, ottimizzatore, iperparametri, ciclo di addestramento, dimensione del batch, dimensione del modello, ecc.).

**Cosa NON PUOI fare:**
- Modificare `prepare.py`, poiché è in sola lettura e contiene la valutazione fissa, il caricamento dei dati, il tokenizer e le costanti di addestramento.
- Installare nuovi pacchetti o aggiungere dipendenze, in quanto puoi usare solo ciò che è già presente in `pyproject.toml`.
- Modificare il sistema di valutazione, perché la funzione `evaluate_bpb` in `prepare.py` rappresenta la metrica di riferimento (ground truth).

**L'obiettivo è semplice: ottenere il val_bpb più basso possibile.** Poiché il budget di tempo è fisso, non devi preoccuparti del tempo di addestramento — è sempre di 5 minuti. Tutto è ammesso: cambia l'architettura, l'ottimizzatore, gli iperparametri, la dimensione del batch o del modello. L'unico vincolo è che il codice venga eseguito senza andare in crash e termini entro il tempo previsto.

La **VRAM** è un vincolo flessibile. Un leggero aumento è accettabile a fronte di miglioramenti significativi del val_bpb, ma non dovrebbe crescere a dismisura.

**Criterio di semplicità**: A parità di condizioni, più semplice è, meglio è. Un piccolo miglioramento che aggiunge una complessità disordinata non ne vale la pena. Al contrario, rimuovere qualcosa e ottenere risultati uguali o migliori è un ottimo risultato — è una vittoria in termini di semplificazione. Nel valutare se mantenere una modifica, soppesa il costo della complessità rispetto all'entità del miglioramento. Un miglioramento di 0.001 nel val_bpb che aggiunge 20 righe di codice abborracciato? Probabilmente non ne vale la pena. Un miglioramento di 0.001 derivato dall'eliminazione di codice? Assolutamente da mantenere. Un miglioramento quasi nullo ma con un codice molto più semplice? Da mantenere.

**La prima esecuzione**: La tua primissima esecuzione dovrebbe sempre servire a stabilire la baseline, quindi eseguirai lo script di addestramento così com'è.

### Formato di output

Al termine dello script, verrà stampato un riepilogo simile a questo:

```
---
val_bpb:          0.997900
training_seconds: 300.1
total_seconds:    325.9
peak_vram_mb:     45060.2
mfu_percent:      39.80
total_tokens_M:   499.6
num_steps:        953
num_params_M:     50.3
depth:            8
```

Nota che lo script è configurato per interrompersi sempre dopo 5 minuti, quindi a seconda della piattaforma di calcolo del computer in uso i numeri potrebbero variare. Puoi estrarre la metrica chiave dal file di log:

```
grep "^val_bpb:" run.log
```

### Registrazione dei risultati

Quando un esperimento è concluso, registralo in `results.tsv` (separato da tabulazioni, NON da virgole — le virgole causano problemi nelle descrizioni).

Il file TSV ha una riga di intestazione e 5 colonne:

```
commit	val_bpb	memory_gb	status	description
```

1. hash del commit git (breve, 7 caratteri).
2. val_bpb ottenuto (es. 1.234567) — usa 0.000000 in caso di crash.
3. memoria di picco in GB, arrotondata a un decimale (es. 12.3 — dividi peak_vram_mb per 1024) — usa 0.0 in caso di crash.
4. stato: `keep`, `discard` o `crash`.
5. breve descrizione testuale di ciò che questo esperimento ha tentato di fare.

Esempio:

```
commit	val_bpb	memory_gb	status	description
a1b2c3d	0.997900	44.0	keep	baseline
b2c3d4e	0.993200	44.2	keep	increase LR to 0.04
c3d4e5f	1.005000	44.0	discard	switch to GeLU activation
d4e5f6g	0.000000	0.0	crash	double model width (OOM)
```

### Il ciclo degli esperimenti

L'esperimento viene eseguito su un branch dedicato (es. `autoresearch/mar5` o `autoresearch/mar5-gpu0`).

CICLO INFINITO:

1. Controlla lo stato di git: il branch/commit attuale su cui ci troviamo.
2. Modifica `train.py` applicando un'idea sperimentale mettendo direttamente mano al codice.
3. Esegui un git commit.
4. Esegui l'esperimento: `uv run train.py > run.log 2>&1` (reindirizza tutto — NON usare tee e non lasciare che l'output intasi il tuo contesto).
5. Leggi i risultati: `grep "^val_bpb:\|^peak_vram_mb:" run.log`.
6. Se l'output del grep è vuoto, l'esecuzione è andata in crash; esegui `tail -n 50 run.log` per leggere lo stack trace di Python e tentare una correzione (se non riesci a risolvere dopo alcuni tentativi, lascia perdere).
7. Registra i risultati nel tsv (NOTA: non committare il file results.tsv, lascialo non tracciato da git).
8. Se il val_bpb è migliorato (più basso), "avanza" sul branch mantenendo il commit di git.
9. Se il val_bpb è uguale o peggiore, esegui un git reset tornando al punto di partenza.

L'idea è che tu sia un ricercatore completamente autonomo che fa dei tentativi: se funzionano mantieni, se non funzionano scarta. Facendo avanzare il branch hai la possibilità di iterare. Se ti senti bloccato in qualche modo, puoi tornare indietro, ma è una mossa che dovresti fare con estrema parsimonia (se non mai).

**Timeout**: Ogni esperimento dovrebbe richiedere ~5 minuti in totale (+ qualche secondo per il caricamento iniziale e l'overhead di valutazione). Se un'esecuzione supera i 10 minuti, interrompila e trattala come un fallimento (scarta e ripristina).

**Crash**: Se un'esecuzione va in crash (errore di memoria OOM, un bug, ecc.), usa il tuo giudizio: se è un errore banale e facile da sistemare (es. un refuso, un'importazione mancante), correggilo e ripeti l'esecuzione. Se l'idea in sé è fondamentalmente sbagliata, saltala, registra "crash" come stato nel tsv e vai avanti.

**NON FERMARTI MAI**: Una volta iniziato il ciclo di esperimenti (dopo la configurazione iniziale), NON metterti in pausa per chiedere all'utente se devi continuare. NON chiedere "devo andare avanti?" o "è un buon punto per fermarsi?". L'umano potrebbe dormire o essersi allontanato dal computer e si aspetta che tu continui a lavorare *indefinitamente* finché non vieni fermato manualmente. Sei autonomo. Se sei a corto di idee, pensa più a fondo: leggi gli articoli scientifici citati nel codice, rileggi i file di riferimento per trovare nuove prospettive, prova a combinare tentativi precedenti quasi riusciti o sperimenta modifiche architettoniche più radicali. Il ciclo continua fino a quando l'umano non ti interrompe, punto e basta.

Come esempio di caso d'uso, un utente potrebbe lasciarti in esecuzione mentre dorme. Se ogni esperimento ti richiede ~5 minuti, puoi eseguirne circa 12 all'ora, per un totale di un centinaio durante una normale dormita umana. Al suo risveglio l'utente troverà i risultati sperimentali, tutti completati da te durante la notte!

***
