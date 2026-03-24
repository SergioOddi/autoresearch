L'architettura di autoresearch
Il concetto centrale è elegante: un agente AI itera autonomamente su un singolo file, misura un risultato, e mantiene o scarta le modifiche — come un loop evolutivo senza supervisione umana.

I tre elementi chiave sono:

Elemento	Ruolo	Chi lo modifica
train.py	Il "soggetto" degli esperimenti	L'agente AI
program.md	Le istruzioni/obiettivi	Tu (l'umano)
La metrica (val_bpb)	Il criterio di successo	Fisso
Come puoi applicarla ai tuoi progetti creativi
1. Generazione di domande per Moodle (GIFT)
Adatta perfettamente il pattern:

File soggetto: una banca domande GIFT
program.md: istruzioni su topic, difficoltà target, distribuzione RS/RM/CO
Metrica: percentuale di domande valide, varietà lessicale, copertura del topic
L'agente itera, genera varianti, scarta quelle che non rispettano il formato
2. Ottimizzazione di prompt per NotebookLM
File soggetto: un file di prompt/configurazione del notebook
Metrica: qualità delle risposte generate (puoi fare il judge con un secondo LLM)
L'agente sperimenta varianti di prompt overnight
3. Creazione di materiale didattico
File soggetto: uno script/slide in formato testo
program.md: "mantieni la chiarezza, aumenta gli esempi pratici, riduci il gergo"
Metrica: valutazione automatica della leggibilità (es. Flesch-Kincaid)
Il pattern da replicare
La cosa più potente non è il codice specifico — è il loop:


leggi program.md → modifica file → misura metrica → confronta → tieni/scarta → ripeti
Puoi replicarlo con Claude Code su qualsiasi dominio creativo purché tu:

Definisca un solo file da modificare
Scriva un program.md chiaro con obiettivi
Abbia una metrica misurabile (anche valutazione da LLM)
Vuoi che ti aiuti a costruire un program.md per uno dei tuoi casi d'uso specifici?

