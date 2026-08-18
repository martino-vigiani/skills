---
name: optimize-app
description: Passata di ottimizzazione performance completa su un'app o un sito — CPU, memoria, GPU, tempi di avvio, peso dei bundle, query, immagini. Misura prima di toccare, poi lavora a giri successivi finché i numeri smettono di scendere, e chiude con una tabella prima/dopo con le percentuali vere. Usa questa skill ogni volta che l'utente vuole ottimizzare, alleggerire, velocizzare o rendere più fluida un'app, un sito o un servizio — anche se dice solo "l'app è lenta", "consuma troppa RAM", "l'avvio ci mette una vita", "il pannello ci mette tre secondi a caricare", "ottimizza questo progetto", "fai una passata di performance", "riduci il consumo CPU", "il sito è pesante", "questa pagina scatta". Vale per iOS/SwiftUI, PWA e siti JS, backend PHP e servizi Node. Parte da sola: chiede solo quale progetto, e solo se non è deducibile dal contesto.
---

# optimize-app

Passata di performance su un intero progetto, condotta come la condurrebbe qualcuno che risponde della bolletta: si misura, si colpisce il collo di bottiglia vero, si rimisura, e si dice la verità sui numeri.

**Parti e lavora.** Non chiedere conferma per iniziare, non presentare un piano in attesa di approvazione, non fermarti a metà per chiedere se proseguire. L'unica domanda ammessa è **quale progetto**, e solo quando non è deducibile — vedi Step 0.

## Il principio che regge tutto

**Un numero che non hai misurato non esiste.** Questa skill produce alla fine una tabella di percentuali, ed è esattamente il punto in cui è più facile mentire: si stima un −30% perché "abbiamo tolto un decode inutile" e nessuno controlla. Se lo fai, la tabella diventa peggio che inutile — diventa una bugia che qualcuno userà per decidere.

Da qui discendono tre regole che non si negoziano:

1. **Baseline prima di toccare qualsiasi cosa.** Nessuna riga di codice cambia prima che esista una misura scritta su disco dello stato attuale.
2. **Ogni riga della tabella finale ha due numeri reali**, prima e dopo, ottenuti con lo stesso metodo e sulla stessa macchina. Se una metrica non è misurabile qui e ora — la GPU vuole un iPhone via cavo, il server di produzione è dietro un captcha — la riga dice `non misurato` e spiega cosa servirebbe. Mai una stima travestita da misura.
3. **Se una modifica non migliora un numero, si butta.** Non si tiene "perché comunque è più pulita": questo è un lavoro di performance, la pulizia è un altro lavoro.

Il motivo per cui queste regole sono scritte così secche è che i due modi di sbagliare sono entrambi frequenti. Un cold start lento si risolve in un giro solo se lo strumenti invece di indovinare: si scopre che quasi sei secondi se ne andavano in un decode di immagine e li si porta a zero. E all'opposto, un ambiente di misura sbagliato nasconde un difetto per settimane — un simulatore con le cache vuote perché i download fallivano, la suite verde e il bug lì. **Una misura sbagliata è peggio di nessuna misura.**

## Step 0 — bersaglio

Deduci il progetto dal contesto: cosa ha nominato l'utente, su che file si stava lavorando, qual è la directory corrente. Se il repo contiene un progetto solo, il bersaglio è quello. Se è un monorepo, guarda cosa c'è attorno alla cwd — i marker di progetto sono i file di manifest (`package.json`, `*.xcodeproj`, `Package.swift`, `composer.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`) e la loro directory è il bersaglio.

Chiedi **una volta sola**, con `AskUserQuestion`, solo se restano due o più candidati plausibili. Se il progetto è chiaro ma non lo è la risorsa (CPU? memoria? avvio?), **non chiedere**: misurale tutte nella baseline, sarà il numero peggiore a dirti su cosa lavorare. È il senso stesso della skill.

Se il repo è un repo git, lavora su un branch separato: `git worktree add ../perf-<slug> -b perf/<slug>` se preferisci tenere l'albero principale intatto, altrimenti un semplice branch. Da lì in poi **niente più git fino alla fine**: nessun commit, nessun checkout, nessuno stash. Il motivo è che una passata di performance tocca molti file in molti giri, e un commit a metà cristallizza uno stato che potresti dover buttare; il branch separato ti dà comunque la rete di sicurezza. Il commit lo fa l'utente, quando i numeri sono chiusi.

Se il progetto ha bisogno di file non tracciati per buildare (chiavi, `.env`, secret generati), procurateli o linkali prima di provare a compilare: altrimenti la baseline fallisce e sembrerà colpa del codice.

## Step 1 — baseline

Misura lo stato attuale e **scrivilo su disco**, in `.perf/<slug>/baseline.md` dentro la directory del progetto su cui lavori. Su disco e non in memoria, perché ti servirà fra molti giri e il tuo contesto nel frattempo verrà riassunto. Assicurati che `.perf/` non finisca nel repo: aggiungila a `.git/info/exclude` (vale solo per questo checkout e non sporca il `.gitignore` tracciato). In testa al file annota **commit hash, macchina e device usati**: due misure sono confrontabili solo se sai su che codice e su che ferro sono state prese.

Cosa misurare dipende dallo stack — i comandi concreti stanno nelle reference, leggi solo quella che ti serve:

| Stack | Reference |
|---|---|
| iOS / SwiftUI | [references/ios.md](references/ios.md) |
| PWA, siti, app JS | [references/web.md](references/web.md) |
| Backend PHP, servizi Node, VPS | [references/backend.md](references/backend.md) |

Regole di misura valide ovunque:

- **Ripeti ogni misura almeno tre volte** e annota mediana e spread. Una singola run non distingue un miglioramento da rumore, e su una macchina da sviluppo il rumore è facilmente ±15%.
- **Misura la cosa vera, non la sua controfigura.** Il simulatore iOS non ha la GPU del telefono, `localhost` non ha la latenza dell'hosting vero, una cache calda non è un cold start. Quando puoi misurare solo la controfigura, scrivilo accanto al numero.
- **Se un numero non si può prendere**, annota in baseline `non misurabile: <perché>`. Alla fine quella riga resta `non misurato`: è informazione, non un buco.

## Step 2 — inventario, senza saltare pezzi

Prima di cercare colli di bottiglia, mappa **tutta** la superficie del progetto. Il senso è non ottimizzare tre volte lo stesso file lasciando fuori metà dell'app: succede perché si parte dal file che si aveva già aperto.

Elenca, per il bersaglio scelto: schermate e componenti UI uno per uno; percorsi dati (fetch, decode, cache, persistenza); asset e immagini con il loro peso; dipendenze di terze parti col loro costo a runtime e nel bundle; build settings e configurazione del server; job in background, timer, observer, listener.

Salva l'inventario in `.perf/<slug>/inventario.md` e **segna cosa hai coperto e cosa no**. Un inventario che tace su ciò che non ha guardato è il modo in cui una passata "completa" lascia scoperta metà del lavoro.

Delega questa parte: è lavoro largo e meccanico. Più agenti di ricerca read-only in parallelo su aree disgiunte — l'agente `explore` del plugin `core-agents` va bene — e **dichiara il file scope di ciascuno nel prompt**, altrimenti si pestano i piedi.

## Step 3 — i giri

Ogni giro è: **audit → classifica per costo misurato → applica → rimisura**. Si ripete finché non converge.

### Audit — a carico di subagent, che non toccano il codice

Manda in parallelo un gruppo di auditor read-only, ognuno su un'area disgiunta dell'inventario. Sono **read-only per costruzione**: gli auditor guardano e riferiscono, le modifiche le fai tu nel main loop. Un agente che misura e insieme corregge non ha nessun modo di accorgersi di aver peggiorato le cose, perché sta valutando il proprio lavoro.

Routing: l'agente `explore` (plugin `core-agents`) per la ricerca larga, l'agente `verifier` dello stesso plugin quando un finding va rifiutato prima di agirci.

Per il codice UI serve in più un auditor specializzato — se ne hai uno tarato sul tuo framework usalo, altrimenti scrivi il prompt tu. Su SwiftUI deve cercare queste cose e riferire con `file:line`: osservazione troppo larga di un modello (`@Observable`/`ObservableObject` osservato intero dove la view usa un campo solo), lavoro dentro `body` (formatter, sort, filter, date), decode e I/O sul main actor, isolamento `@MainActor` messo su lavoro pesante senza motivo, `Task` senza propagazione di `Task.isCancelled`, identità instabile in `ForEach` che ricrea le righe a ogni update, cache che non hanno una politica di svuotamento. Su un framework web l'elenco cambia, il criterio no: **cosa viene ricalcolato e quante volte**.

Ogni auditor deve poter rispondere **"in quest'area non ho trovato niente"**: un auditor che sa di dover consegnare findings li inventa.

Pretendi da ognuno: sequenza di guasto concreta, `file:line`, e **quanto costa** — misurato o almeno stimabile con un ordine di grandezza dichiarato come tale. Un finding senza costo non è ordinabile, quindi non serve a niente.

### Ricerca esterna, quando il problema non è nel codice

Alcuni colli di bottiglia non si vedono leggendo il proprio repo: sono il default di una libreria, un flag di build, un comportamento noto di un framework, una configurazione del provider. Per quelli usa **Context7** (documentazione aggiornata delle librerie) e **WebSearch**. Sono utili soprattutto su versioni recenti, dove la tua memoria è più probabilmente vecchia del codice che stai leggendo.

Per le UI web, misura nel browser vero invece di dedurre dal sorgente: `npx lighthouse` per Web Vitals e punteggi, uno script Playwright lanciato via Bash (con tracing CDP) per long task, layout thrash e heap, e un MCP browser se ne hai uno collegato per ispezione interattiva e richieste di rete. È l'unico modo di vedere i costi di render veri.

### Applica, uno alla volta

Ordina i candidati per **costo misurato diviso rischio**, e parti dall'alto. Applica **una modifica per volta e rimisura**: se ne applichi cinque insieme e il numero scende del 20%, non sai quale delle cinque ha funzionato, e le altre quattro restano lì come debito di cui nessuno conosce il valore.

Preferisci **togliere** ad aggiungere. La soluzione migliore a un lavoro costoso è quasi sempre non farlo: un decode che non serve, un observer che rianima tutto, una libreria usata per una funzione, un'immagine servita a 4× la dimensione a cui viene mostrata. Aggiungere una cache è la seconda scelta, non la prima — la cache è un altro pezzo che può sbagliarsi.

Dopo ogni modifica lancia i test del progetto. Un'ottimizzazione che rompe il comportamento non è un'ottimizzazione, ed è un fallimento particolarmente insidioso perché i numeri migliorano davvero.

### Quando fermarsi

Non fermarti al primo fix, ma nemmeno all'infinito: i giri che spostano l'1% consumano budget vero. Fermati quando è vera una di queste:

- **Due giri consecutivi senza un miglioramento misurato ≥5%** su nessuna metrica tracciata.
- **I candidati rimasti costano tutti meno del rumore di misura**: sotto il ±15% di spread tipico non stai più misurando, stai leggendo il caso.
- **Il rimanente richiede una decisione che non è tua**: cambiare architettura, rompere una API, spendere soldi, toccare qualcosa che l'utente finale vede. Fermati e chiedi, con i numeri in mano.

Annota in `.perf/<slug>/giri.md` cosa hai provato in ogni giro, cosa ha funzionato e **cosa hai buttato perché non migliorava niente**. Quest'ultima colonna è la più preziosa: impedisce che al giro dopo qualcuno riprovi la stessa cosa.

## Step 4 — il benchmark finale

Chiudi con un report in `.perf/<slug>/REPORT.md`, e stampa la tabella nella risposta.

```markdown
# Ottimizzazione <progetto> — <data>

## Risultati

| Metrica | Prima | Dopo | Δ | Come è stata misurata |
|---|---|---|---|---|
| Cold start (cache calda) | 15,2 s | 0,9 s | −94% | media di 5 run, iPhone via cavo |
| Memoria da sospesa | 480 MB | 92 MB | −81% | Instruments, Allocations |
| Bundle JS iniziale | 412 KB | 198 KB | −52% | build di produzione, gzip |
| GPU sotto scroll | non misurato | non misurato | — | serve un device fisico via cavo |

## Chi consumava

| Componente | Costo prima | Causa | Fix |
|---|---|---|---|
| `HeroImageView` | 5.783 ms all'avvio | decode full-res sul main thread | decode fuori dal main, dimensione al bisogno |
| libreria di animazione | 84 KB, layout su ogni schermata | layout projection sempre attiva | disattivata dove non serve |

## Non fatto, e perché
- <cosa resta sul tavolo, con il numero che varrebbe e cosa lo blocca>
```

Regole del report, tutte per lo stesso motivo — che venga letto fra sei mesi da qualcuno che non c'era:

- **Nessuna riga senza due numeri veri**, o `non misurato` con la ragione.
- **La sezione "Non fatto" non si omette mai.** Una passata che dichiara tutto ottimizzato è quasi sempre una passata che ha smesso di guardare.
- Se il totale è modesto, **dillo**. «−8% sull'avvio, il resto era già a posto» è un esito legittimo e utile. Gonfiarlo con percentuali su metriche irrilevanti (−60% su una funzione che gira una volta all'avvio e costa 3 ms) è il modo classico di far sembrare grosso un lavoro piccolo.

## Alla fine

Il branch resta aperto, con le modifiche non committate: è la consegna. Dillo esplicitamente, con il path e il nome del branch, e ricorda che il commit e il merge restano all'utente.

Se il progetto ha un deploy separato (PWA, pannello server, servizi), **le ottimizzazioni non sono in produzione finché non si deploya**, e il deploy vuole il suo ok. Scrivilo nel report invece di lasciar credere che il lavoro sia in mano agli utenti.
