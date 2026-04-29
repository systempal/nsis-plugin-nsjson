# nsJSON NSIS plug-in

Autori: Stuart 'Afrow UK' Welch, Pieter Dewachter

Data: 10 Ottobre 2024

Versione: 1.1.1.1

Plugin NSIS per il parsing, la manipolazione e la generazione di JSON (JavaScript Object Notation).

Vedi Examples\nsJSON\*.

## Informazioni su JSON

Vedi http://www.json.org/ per informazioni su JSON, sintassi e sequenze di escape.

## Informazioni importanti

Tutte le funzioni del plug-in impostano il flag di errore NSIS in caso di errore. Per le funzioni che restituiscono un valore sullo stack, nessun elemento viene restituito se il flag di errore è stato impostato. Usare `IfErrors` o `${If} ${Errors}` per verificare che una chiamata sia riuscita prima di fare Pop di un valore.

Il plug-in supporta tutte le sequenze di escape nei valori stringa:
`\r`, `\n`, `\t`, `\b`, `\f`, `\"`, `\\` e `\uXXXX`

`[NodePath]`, usato in questo readme, è qualsiasi lista di chiavi e indici separati da spazi (gli indici sono preceduti da /index). Per esempio:

```
{
    "node1": {
        "node2": false,
        "node3": [ "Stuart", 1, { "node4": "Welch", "node5": "" }, 32.5, [ 0, "test", false ] ]
    }
}
```

Esempi di percorsi nodo in questo JSON:

"node1" "node2"
- Percorso per "node2" (valore: false)
- `nsJSON::Get "node1" "node2" /end`

"node1" "node3" /index 0
- Percorso per il primo elemento di "node3" (valore: "Stuart")
- `nsJSON::Get "node1" "node3" /index 0 /end`

Più alberi JSON possono essere manipolati contemporaneamente. Aggiungere `/tree NomeAlbero` prima degli altri argomenti per specificare quale albero JSON si sta manipolando.

## Lettura di file JSON

`nsJSON::Set [/tree Albero] [NodePath] /file [/unicode] "percorso\input.json"`

Carica il JSON dal file dato nel nodo specificato. Specificare `/unicode` se il file è Unicode.

---

`nsJSON::Set [/tree Albero] /file [/unicode] "percorso\input.json"`

Carica il JSON dal file dato in memoria, sovrascrivendo qualsiasi albero JSON esistente.

## JSON da richieste HTTP

`nsJSON::Set [/tree Albero] [NodePath] /http AlberoConfig`

Esegue una richiesta HTTP e carica la risposta JSON nel nodo dato. L'AlberoConfig specifica la configurazione della richiesta in formato JSON. I valori possibili sono:

**"Url"**: "http://..."
- L'URL della richiesta. Obbligatorio.

**"Verb"**: "GET"
- Il verbo della richiesta (GET, POST, ecc.). Default: GET.

**"Params"**: "..."
- Parametri opzionali da aggiungere all'URL (dopo ?).

**"Data"**: "..."
- Dati opzionali da inviare (tipicamente con POST).

**"Headers"**: "..."
- Header aggiuntivi da inviare.

**"Agent"**: "nsJSON NSIS plug-in/1.0.x.x"
- La stringa dell'agente della richiesta.

**"Decoding"**: false
- Abilita la decompressione GZIP automatica.

**"Async"**: true
- Esegue la richiesta in modo asincrono. Usare la funzione Wait per attendere il completamento.

## JSON dall'esecuzione di applicazioni console

`nsJSON::Set [/tree Albero] [NodePath] /exec AlberoConfig`

Esegue un'applicazione console e carica l'output nel nodo dato. I valori possibili della configurazione:

**"Path"**: "$INSTDIR\ConsoleApp.exe"
- Percorso all'eseguibile. Obbligatorio.

**"Arguments"**: ...
- Argomenti della riga di comando.

**"WorkingDir"**: "..."
- Directory di lavoro.

**"Input"**: ...
- Input da scrivere su STDIN.

**"Async"**: true
- Esegue in modo asincrono.

## Modifica del JSON

`nsJSON::Set [/tree Albero] [NodePath] /value "Valore"`

Imposta il valore del nodo dato. Il valore può essere un singolo valore o codice JSON. Il nodo viene creato se non esiste.

---

`nsJSON::Delete [/tree Albero] [NodePath] /end`

Elimina l'albero o il nodo dato. `/end` deve essere aggiunto alla fine della lista.

---

```
nsJSON::Quote [/unicode] [/always] Valore
Pop $Var
```

Racchiude il valore dato tra virgolette se necessario e fa l'escape dei caratteri che lo richiedono.

---

`nsJSON::Sort [/tree Albero] [NodePath] [/options Opzioni] /end`

Ordina il nodo dato. Sommare i seguenti valori per le opzioni di ordinamento:

1. Ordine decrescente.
2. Ordinamento numerico.
4. Ordinamento case-sensitive.
8. Ordina per chiavi anziché per valori.
16. Ordina ricorsivamente tutti i nodi nell'albero.

## Lettura di valori JSON

```
nsJSON::Get [/tree Albero] [/noexpand] [NodePath] /end
Pop $Var
```

Ottiene il valore del nodo dato. `/end` deve essere aggiunto alla fine della lista.

---

```
nsJSON::Get [/tree Albero] /type [NodePath] /end
Pop $Var
```

Ottiene il tipo di valore del nodo dato: "node", "array", "string", "value" o stringa vuota se il nodo non esiste.

---

```
nsJSON::Get [/tree Albero] /keys [NodePath] /end
Pop $VarKeyCount
Pop $VarKey1
Pop $VarKeyN
```

Ottiene le chiavi del nodo dato.

---

```
nsJSON::Get [/tree Albero] /count [NodePath] /end
Pop $Var
```

Ottiene il numero di nodi figli o elementi di un array.

## Generazione JSON

```
nsJSON::Serialize [/tree Albero] [/format]
Pop $Var
```

Serializza l'albero JSON corrente in `$Var`. Aggiungere `/format` per formattare l'output.

---

`nsJSON::Serialize [/tree Albero] [/format] /file [/unicode] "percorso\output.json"`

Serializza l'albero JSON corrente nel file dato.

## Attesa di task asincroni

```
nsJSON::Wait Albero [/timeout TimeoutInMillisecondi]
Pop $Var
```

Verifica se il task asincrono in esecuzione nell'albero JSON dato è terminato.

---

## ⚠️ Differenze nella versione personale

Questa versione è basata sul rilascio ufficiale v1.1.1.1 da [GitHub](https://github.com/Pieter-Dewachter/nsJSON/releases/tag/v1.1.1.1).

### Nuova architettura supportata: x64 (amd64-unicode)

L'originale v1.1.1.1 supporta solo:
- x86-ansi
- x86-unicode

La versione personale aggiunge il supporto per:
- **amd64-unicode** (x64)

### Progetto Visual Studio

L'originale includeva:
- `Contrib/nsJSON/nsJSON.sln` - Solution VS2012
- `Contrib/nsJSON/nsJSON.vcxproj` - Progetto VS2012 (solo x86)
- `Contrib/nsJSON/ConsoleApp/ConsoleApp.vcxproj` - App di test

La versione personale usa lo stesso progetto aggiornato per VS2022 con supporto x64.

### File aggiunti (infrastruttura di build)

- `build_plugin.cmd` - Script per compilare il plugin
- `build_plugin.py` - Script Python per compilare il plugin per tutte le architetture
- `BUILD_README.md` - Documentazione di compilazione
- `.gitignore` - File di esclusione Git

### File rimossi

I file DLL precompilati sono stati rimossi dalla distribuzione (vengono generati dalla compilazione):

- `Plugins/x86-ansi/nsJSON.dll`
- `Plugins/x86-unicode/nsJSON.dll`

### Modifiche funzionali

Nessuna modifica funzionale rispetto all'originale. Le modifiche riguardano solo l'infrastruttura di build e il supporto x64.

### Compilazione

```cmd
cd nsJson
python build_plugin.py
```

I DLL vengono copiati in `plugins/{platform}/nsJSON.dll`.

### Opzioni build

```powershell
python build_plugin.py --config x86-unicode      # Solo un'architettura (x86-ansi|x86-unicode|amd64-unicode|all)
python build_plugin.py --toolset 2026            # Toolset specifico (2022|2026|auto)
python build_plugin.py --jobs 4                  # Numero di job MSBuild paralleli (default: CPU count)
python build_plugin.py --clean                   # Pulizia dist/ prima della build
python build_plugin.py --install-dir "C:\NSIS\Plugins"  # Copia in directory NSIS aggiuntiva
python build_plugin.py --verbose                 # Output MSBuild esteso
python build_plugin.py --version                 # Stampa versione ed esce
```

---

*See [README.md](README.md) for the English version.*
