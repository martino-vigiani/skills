# iOS / SwiftUI — misure e colli di bottiglia ricorrenti

## Misurare

**Il simulatore non è il telefono.** Non ha la GPU del device, la memoria è quella del Mac, e la rete è quella del Mac: un difetto di memoria che dipende da cache piene può restare invisibile per mesi se in simulatore quelle cache non si riempiono mai. Le misure che contano si prendono su un iPhone via cavo. Se non ce l'hai, la baseline lo dichiara e le righe GPU/memoria restano `non misurato`.

| Cosa | Come |
|---|---|
| Tempi di fase (avvio, decode, fetch) | `OSSignposter` / `os_signpost` attorno alle fasi sospette, poi Instruments → Points of Interest. In alternativa, per una misura grezza ma immediata, un `print` con il delta in ms e un grep sul log |
| CPU, memoria, hang | Instruments: Time Profiler, Allocations, Hangs |
| Aggiornamenti di vista superflui | `Self._printChanges()` in cima al `body`, oppure Instruments → SwiftUI → View Body |
| Build e binario | `xcodebuild -showBuildTimingSummary`, e la dimensione del `.app` a parità di configurazione |
| Test | `xcodebuild test` (o l'equivalente del tuo MCP Xcode, se ne hai uno collegato) |

Prendi ogni misura almeno tre volte: su device i tempi ballano.

## Dove si nasconde il costo, in ordine di frequenza reale

1. **Decode immagini sul main thread.** È il collo di bottiglia numero uno delle app con contenuti fotografici: quasi sei secondi all'avvio in un caso misurato, poi zero. Decodifica fuori dal main e alla dimensione a cui l'immagine viene mostrata, non a quella nativa.
2. **Cache che non si svuotano.** Mezzo giga di foto in RAM da app sospesa fa uccidere l'app da iOS: l'utente la riapre e riparte da zero. Svuota su `scenePhase == .background` e su memory warning.
3. **Avvio che aspetta la rete.** Se i dati sono già sul telefono, mostrali e aggiorna in sottofondo. Aspettare la rete quando hai la cache è tempo regalato.
4. **Osservazione troppo larga.** Una view che osserva un modello grosso si ridisegna per ogni campo che cambia, anche quelli che non usa. Restringi l'osservazione a ciò che la view legge davvero.
5. **Lavoro nel `body`.** Il `body` viene rivalutato spesso: formatter, sort, filter e calcoli di date lì dentro si pagano a ogni valutazione.
6. **Framework di sistema pesanti (MapKit, PhotoKit).** Spesso il tempo se ne va dentro il framework e non nel codice tuo: prima di ottimizzare la propria annotazione, misura quanto resta davvero da guadagnare.
7. **Isolamento `@MainActor`.** Lavoro pesante marcato main actor senza motivo blocca l'interfaccia; `Task.isCancelled` non propagato tiene vivo lavoro che nessuno guarda più.
8. **Identità instabile nelle liste.** Un `id` che cambia a ogni update fa ricreare le righe invece di aggiornarle, e lo scroll perde i frame.

## Prima di tagliare, chiedi se è un margine

Non ogni attesa è uno spreco. Una splash screen tenuta lunga apposta, un'animazione con una durata voluta, un ritardo che serve a far leggere un messaggio: sono decisioni di prodotto travestite da lentezza, e a ogni passata di performance qualcuno prova a tagliarle. Se una lentezza sembra ovvia da eliminare, cerca prima se qualcuno l'ha messa lì di proposito — spesso c'è un commento nel codice — e in mancanza di risposta chiedi invece di decidere.

Vale anche per il perimetro dei dispositivi supportati: se un layout è stato tolto per una scelta esplicita, non si "ottimizza" reintroducendolo.
