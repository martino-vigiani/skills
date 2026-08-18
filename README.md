# skills

Skill e subagent per Claude Code, estratti da un repo di lavoro vero e ripuliti per essere usati altrove.

Sono quattro plugin separati. Si installano uno alla volta: nessuno ti obbliga a prendere il resto.

```
/plugin marketplace add martino-vigiani/skills
/plugin install optimize-app@martino-vigiani
```

Per vedere cosa c'è dentro prima di installare, `/plugin marketplace` dopo l'`add`.

## I quattro plugin

### `core-agents`

Quattro subagent generici: `fixer` e `fixer-quick` per gli edit, `explore` per il lookup read-only, `verifier` per la verifica adversariale.

Il pezzo che vale è come `fixer` tratta la verifica. Ha una scala esplicita: in cima la misura che può prendersi da solo, poi quella che richiede un ambiente ma non una persona, e per ultima la prova che richiede le tue mani — che deve dire perché le prime due non bastavano e chiedere un gesto preciso, non «prova e dimmi». Più due regole che sembrano ovvie e non lo sono: **la prova deve poter fallire** (rompi apposta il pezzo che il fix protegge e guarda se qualcosa protesta), e **il report va consegnato sempre**, anche quando è un fallimento.

`verifier` esiste per una ragione simmetrica: un agente mandato a «controllare» conferma quasi sempre. Questo è mandato a refutare, e un verdetto vale solo se poteva essere negativo.

### `optimize-app`

Una passata di performance condotta come la condurrebbe qualcuno che risponde della bolletta.

La regola che regge tutto è che **un numero che non hai misurato non esiste**. Quindi: baseline scritta su disco prima di toccare qualsiasi cosa; ogni riga della tabella finale con due numeri veri presi con lo stesso metodo sulla stessa macchina, oppure `non misurato` con la ragione; una modifica che non migliora un numero si butta, anche se è più pulita.

È scritta per il momento in cui è più facile mentire — la tabella finale — e per il difetto più comune, che è ottimizzare tre volte il file che si aveva già aperto lasciando fuori metà dell'app.

Copre iOS/SwiftUI, PWA e siti, backend e servizi.

### `swift-quality`

`swift-test-expert` e `e2e-tester` per i test, più una reference sulle API nuove di SwiftUI in iOS 27.

`e2e-tester` decide da sé quali percorsi utente meritano un test end-to-end, e poi verifica che il test **fallisca davvero** quando la cosa che protegge si rompe — rompendola. Non aggiusta i test per farli passare.

`swift-test-expert` distingue il bug del codice dal bug del test, e lo dice invece di far passare il test.

Entrambi conoscono la trappola che costa più tempo di tutte: uno scheme senza test action fa uscire `xcodebuild test` con codice 0 **senza eseguire niente**, e sembra verde. Un run con zero test eseguiti non è una prova.

### `writing`

`humanize` riscrive un testo perché suoni di una persona vera invece che di un modello. `persuade-mail` scrive una mail che muove chi la legge, a freddo o verso qualcuno che già ti conosce.

## Cosa non troverai qui

Sono in italiano. Non è una svista: sono scritte come le userei io, e tradurle le farebbe suonare come tutto il resto.

Alcune skill che uso ogni giorno non sono qui perché **non sono mie**. Se cerchi quelle:

- [`impeccable`](https://github.com/paulbakaus) — Paul Bakaus
- [`app-store-review`](https://github.com/JustinPerea/app-store-review-skill) — Justin Perea
- [`make-interfaces-feel-better`](https://github.com/jakubkrehel/make-interfaces-feel-better) — jakub krehel
- [`taste-skill`](https://github.com/leonxlnx/taste-skill) — leonxlnx

Altre non ci sono perché fuori dal mio setup non hanno niente da eseguire: il triage della posta, la chiusura di sessione, il preflight di deploy. Presuppongono un database, degli script e un tracker che sul tuo Mac non esistono, e pubblicarle sarebbe darti un guscio.

## Licenza

MIT. Se una di queste ti è utile e la migliori, mi fa piacere saperlo: martino@subralabs.com
