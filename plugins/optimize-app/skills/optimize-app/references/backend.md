# Backend PHP, servizi Node, VPS — misure e colli di bottiglia ricorrenti

## Misurare

**Non misurare in produzione, e non fare test di carico contro un hosting condiviso.** Il confine del provider (WAF, cache, captcha, rate limit) non è replicabile, e un carico che sembra tuo può bloccarti l'IP. Se puoi, monta una replica locale dello stack — stessa catena di reverse proxy e stessa versione del runtime, dataset e date dei file veri — ed è lì che rompi e misuri. Il comando per farlo dipende dal tuo setup: se non ne hai uno, la baseline lo dichiara e le misure sotto carico restano `non misurato`.

| Cosa | Come |
|---|---|
| Latenza per endpoint | `curl -w '%{time_total} %{time_starttransfer}\n' -o /dev/null -s <url>`, ripetuto; mediana e coda, non la media |
| Profilo PHP | `microtime(true)` ai bordi delle funzioni sospette, oppure Xdebug/Blackfire sulla replica; mai in produzione |
| CPU e memoria di un processo Node | il tuo process manager (`pm2 describe <nome>`, `pm2 monit`) oppure `ps`/`top` sul pid; il picco RSS conta più della media |
| Salute della macchina | `htop`, `free -m`, `df -h` via ssh — su una VPS piccola il margine è stretto |
| Query SQL | `EXPLAIN QUERY PLAN` (SQLite) o `EXPLAIN ANALYZE` (Postgres/MySQL); un full scan dove ti aspettavi un lookup è il tuo indice mancante |
| Payload | dimensione della risposta in gzip, e quante richieste servono per una schermata |

Un ordine di grandezza utile da tenere a mente: un endpoint che risponde in pochi millisecondi a vuoto può andare a diversi secondi sotto un carico generato da un solo portatile. Se cerchi un tetto di capacità, parti da lì.

## Dove si nasconde il costo, in ordine di frequenza reale

1. **Query dentro un ciclo.** Il classico N+1: cento record, cento query. Si vede subito contando le query per richiesta, non leggendo il codice.
2. **File JSON grossi letti e riscritti interi** a ogni salvataggio. Leggere tutto il dataset per cambiare un campo costa, e in più è il punto dove un deploy sbagliato cancella dati veri.
3. **Immagini processate a ogni richiesta** invece che una volta e messe da parte.
4. **Assenza di cache HTTP** su risorse che non cambiano: ogni ricarica del pannello ripaga tutto.
5. **Middleware che gira su rotte a cui non serve** — parsing di body, sessioni, log verbosi su endpoint di sola lettura.
6. **Log che crescono senza limite.** Su un disco piccolo non è teoria.
7. **Dipendenze pesanti a runtime** su un servizio che fa tre cose: ogni pacchetto è memoria residente su una macchina condivisa.

## Vincoli da rispettare sempre

- **I database su file sono single-writer.** Query di lettura sempre ok, scritture mai da due sessioni in parallelo.
- **Il database non sta nella directory di deploy.** Se ci sta, spostarlo è un'ottimizzazione di sicurezza prima che di performance, e va concordata.
- **File sensibili mai toccati**: `.env`, file di password del server, credenziali, chiavi. Se un'ottimizzazione chiede di editarli, si ferma e si chiede.
- **Deploy manuale**: ottimizzato ≠ in produzione, e il deploy vuole un ok esplicito.
