# PWA, siti, app JS — misure e colli di bottiglia ricorrenti

## Misurare

Misura nel browser vero, non deducendo dal sorgente: layout thrash, long task e costi di render non si leggono nel codice.

| Cosa | Come |
|---|---|
| Trace runtime, long task, render | script Playwright via Bash con tracing CDP (`page.context().newCDPSession` / `page.tracing`), poi leggi il trace JSON |
| Rete, peso e cache degli asset | le richieste raccolte dallo script Playwright, o un MCP browser se ne hai uno collegato |
| Core Web Vitals, punteggi | `npx lighthouse <url> --output json` — utile come indice, non come diagnosi |
| Peso del bundle | l'output del build di produzione (molti bundler stampano il peso per route) o `du -h` sulla directory di output; confronta **gzip**, non il raw |
| Memoria | heap snapshot via CDP dallo script Playwright, e la stessa pagina lasciata aperta qualche minuto per vedere se cresce |
| Regressioni | se il progetto ha già test di performance, guardali prima di scriverne di nuovi |

Se il progetto ha già un report di performance in repo, leggilo prima di rifare il lavoro da capo.

**Attenzione al branch.** Prima di misurare controlla `git branch -a`: se esiste un branch con lavoro perf non ancora mergiato, misurare il branch principale e attribuirsi quei guadagni è il modo più facile di produrre una tabella falsa. Mergialo prima, o dichiara nel report che quei numeri appartengono a un altro branch.

## Dove si nasconde il costo, in ordine di frequenza reale

1. **Immagini.** Servite più grandi di come vengono mostrate, in formato pesante, senza dimensioni dichiarate (e quindi con layout shift). Il passaggio a un formato moderno (AVIF, WebP) vale tipicamente un quarto del peso a parità di qualità: misuralo sul tuo set, non fidarti del numero generico.
2. **JavaScript che arriva su pagine che non lo usano.** Una libreria di validazione o una libreria di mappe importata in un modulo condiviso finisce nel bundle di tutte le route. Controlla cosa entra per route.
3. **Librerie di animazione.** La layout projection e i motori di animazione basati su JS costano su ogni schermata, anche dove nessuno anima nulla.
4. **Mappe e canvas.** Vanno montati al primo frame libero, non dentro una transizione di navigazione.
5. **Service worker con strategia sbagliata.** Una risposta servita da cache stantia sembra un bug di dati, non di performance.
6. **Animazioni fuori da `transform`/`opacity`.** Tutto il resto passa da layout e paint, e su mobile si vede.
7. **Font e CSS bloccanti** in testa al documento.

## Deploy

Se il sito ha un deploy separato e manuale, codice ottimizzato nel branch principale **non è in produzione**: scrivilo nel report, e il deploy chiedilo.
