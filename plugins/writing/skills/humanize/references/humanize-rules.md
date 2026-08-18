# Humanize — regole di riscrittura (dizionario completo)

Questo file è il riferimento operativo passato al subagent che riscrive. Obiettivo: prendere un testo che "sa di AI" — mail, articolo, post, documentazione, copy — e farlo suonare come scritto da una persona reale, **senza cambiare significato, fatti, lingua, registro o struttura**. Non è un cambio di contenuto: è un cambio di voce.

## Principio guida

L'AI scrive con bassa *perplexity* (sceglie sempre la continuazione più probabile → testo levigato, prevedibile) e bassa *burstiness* (frasi tutte della stessa lunghezza e forma). Gli umani no: alternano frasi corte e secche a frasi lunghe con incisi, sbagliano un po' il ritmo, vanno dritti al punto, usano la lingua viva. Il lavoro è ripristinare quel ritmo irregolare e quella voce diretta, non aggiungere fronzoli.

**Regola d'oro: meno levigato, più diretto.** Nel dubbio, taglia. Una frase più corta e un po' imperfetta batte una frase perfetta e anonima.

## I tell dell'AI — da rimuovere

### 1. Trattino lungo (—) — il tell numero uno
Gli umani non usano quasi mai il trattino lungo nella scrittura corrente (mail, post, prosa). L'AI lo infila ovunque, specie nella struttura "frase — coda esplicativa". **Eliminali TUTTI, zero eccezioni** — è il singolo segnale più riconoscibile (eccezione: documentazione tecnica il cui house style li usa già di proposito, lì si lasciano). Sostituisci ognuno con:
- una virgola (caso più comune)
- una parentesi, se è un vero inciso
- un punto, spezzando in due frasi
- "e" / "ma" / "così" / due punti, secondo il senso

Esempio — AI: `L'app è pronta — mancano solo le foto.` → umano: `L'app è pronta, mancano solo le foto.`

Anche il trattino medio (–) e il punto e virgola (;) sono spie quando abbondano: gli umani usano punti e virgole semplici. Diradali.

### 2. Vocabolario gonfio / da chatbot
Parole che segnalano AviaA. Sostituisci con sinonimi semplici o riscrivi la frase.
- **EN**: delve, tapestry, pivotal, leverage, utilize, robust, plethora, ecosystem, seamless, elevate, foster, underscore, navigate, realm, testament, intricate, paramount, vibrant, bustling, boasts, game-changer, unlock, empower, supercharge, streamline, holistic, "in today's fast-paced world", "in today's landscape".
- **IT**: approfondire ("delve"), fondamentale/cruciale/imprescindibile (a raffica), robusto, ecosistema, "senza soluzione di continuità", agevolare, valorizzare, "nel panorama attuale", "in un mondo sempre più...", "soluzione" usato come riempitivo, "ottimizzare/efficientare" a vuoto.

### 3. Transizioni robotiche a inizio frase
L'umano collega le idee con congiunzioni semplici (e, ma, però, quindi, allora / and, but, so), non con avverbioni.
- **EN da evitare**: Furthermore, Moreover, Additionally, In conclusion, Ultimately, Subsequently, "That being said", "With that in mind".
- **IT da evitare**: Inoltre, Pertanto, Difatti, Altresì, "In conclusione", "Ciò detto", "Detto ciò", "Va da sé".

### 4. Frasi-cuscinetto e hedging
Da tagliare quasi sempre, non aggiungono nulla:
- "È importante notare che…", "Vale la pena sottolineare che…", "Come saprà…" → "It's worth noting", "It's important to note".
- Aperture da modello: "Spero che questa mail la trovi bene" / "I hope this email finds you well".
- Chiusure da customer-service: "Resto a disposizione per qualsiasi chiarimento", "Non esiti a contattarmi", "Let me know if you have any questions", "Feel free to reach out", "I'd be happy to…". Una chiusura breve e reale va bene ("Resto a disposizione", "A presto"), la formula gonfia no.

### 5. L'antitesi "non solo X, ma Y" / "It's not just X, it's Y"
Costruzione amatissima dall'AI. Smontala in una frase diretta.
AI: `Non è solo un'app, è un'esperienza.` → umano: di solito si taglia del tutto, o si dice la cosa concreta.

### 6. La regola del tre / parallelismi perfetti
"Veloce, affidabile e scalabile." "Chiaro, conciso e completo." Gli umani non bilanciano tutto in triadi. Rompi il ritmo: due elementi, o tre ma di lunghezza diversa, o riscrivi in prosa.

### 7. Sovra-struttura
L'AI mette elenchi puntati, grassetti e titoletti ovunque. In una mail o un post veri, la maggior parte va in prosa. Tieni un elenco SOLO se sono davvero punti distinti che il lettore deve spuntare (es. istruzioni passo-passo, opzioni) — la documentazione tecnica fa eccezione, lì l'elenco è spesso lo strumento giusto.

### 8. Entusiasmo e cortesia eccessivi
"Ottima domanda!", "Certamente!", "Sarò felice di…", emoji a pioggia, punti esclamativi multipli. Togli. Tono calmo e diretto.

### 9. Uniformità (bassa burstiness)
Se tutte le frasi hanno 15-20 parole e la stessa struttura soggetto-verbo-oggetto, è AI. **Varia di proposito**: qualche frase di 4-6 parole, qualcuna lunga con un paio di incisi uniti da virgole. Il ritmo deve sembrare un po' irregolare.

## I pattern umani — da aggiungere

- **Burstiness**: alterna corto e lungo. Una frase secca dopo una lunga è molto umana.
- **Congiunzioni a inizio frase**: "E poi…", "Ma…", "Così…" / "And…", "But…", "So…". Usate con misura, suonano vere.
- **Contrazioni (solo EN)**: I'll, don't, we're, it's, can't. In italiano NON esistono: non forzarle. In italiano l'equivalente è la lingua viva — elisioni naturali (dell', all', un'), connettivi semplici, frasi che iniziano con "e"/"ma".
- **Concretezza**: nomi, numeri, dettagli specifici battono il vago. "la build di domani" > "la prossima iterazione".
- **Imperfezione lieve**: un inciso fra parentesi, una piccola digressione, una frase che non è perfettamente bilanciata. Va bene.
- **Non iniziare due paragrafi di fila con la stessa parola.**
- **Una colloquialità leggera** ogni tanto (non in ogni riga), se il registro lo consente.

## Vincoli invalicabili (non infrangere mai)

1. **Significato e fatti identici.** Non aggiungere, togliere o alterare informazioni, numeri, date, nomi, promesse, condizioni.
2. **Stessa lingua** dell'originale (IT resta IT, EN resta EN). Non tradurre.
3. **Stesso registro e relazione.** Se l'originale dà del "Lei" (es. cliente formale), resta "Lei": umano ≠ informale. Un testo formale può essere umano e formale insieme. Se dà del "tu", resta "tu".
4. **Stessa struttura logica e firma.** Mantieni oggetto, saluti, firma. Non inventare un tono che non è del mittente.
5. **Non "detection-evasion" fine a sé**: l'obiettivo è leggibilità naturale e voce reale, non ingannare un rilevatore. (In pratica le due cose coincidono, ma la bussola è il lettore umano, non il detector.)

## Esempi prima/dopo

**EN — robotico → umano**
> Prima: `I wanted to reach out to provide you with an update on the project. We have successfully implemented the new feature — a robust solution that will seamlessly elevate the user experience. Furthermore, it's worth noting that testing has been completed. Please don't hesitate to let me know if you have any questions.`
> Dopo: `Quick update on the project. The new feature is in and tested. It makes the whole flow smoother to use. Shout if anything's unclear.`

**IT — formale robotico → formale umano** (registro "Lei" mantenuto)
> Prima: `Le scrivo per fornirle un aggiornamento. Abbiamo implementato con successo la nuova funzionalità — una soluzione robusta che migliorerà in modo significativo l'esperienza utente. Inoltre, è importante sottolineare che i test sono stati completati. Resto a disposizione per qualsiasi chiarimento.`
> Dopo: `Le mando un aggiornamento. La nuova funzione è pronta e testata, e rende l'app più scorrevole da usare. Se qualcosa non torna me lo dica pure.`

Nota: in entrambi i casi i fatti sono identici, è sparito il trattino lungo, sono spariti "robust/seamlessly/elevate/Furthermore/it's worth noting/Inoltre/è importante sottolineare", il ritmo è più vario e corto, la chiusura è breve e reale. Il "Lei" è rimasto.

## Come consegnare

Restituisci **solo il testo riscritto**, pronto da usare (oggetto/titolo se presente, corpo, firma se c'è). Niente preamboli, niente "ecco la versione umanizzata", niente spiegazioni dopo. Se è stato chiesto esplicitamente un riepilogo delle modifiche, mettilo PRIMA del testo, mai dopo.
