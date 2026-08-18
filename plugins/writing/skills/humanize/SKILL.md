---
name: humanize
description: Rewrite a piece of writing — email, article, social post, documentation, README, changelog, site copy, any message — so it reads like a real person wrote it, not an AI. Strips the AI tells — em dashes, words like "delve/leverage/robust/seamless", robotic transitions ("Furthermore", "Inoltre", "In conclusione"), the "not just X but Y" construction, rule-of-three parallelism, customer-service closers, and uniform robotic rhythm — while keeping the meaning, facts, language, register, and structure identical. Detects the text type and language on its own, no need to ask. Delegates the rewrite to a Sonnet subagent. Use this whenever the user wants to humanize, de-AI-ify, or naturalize a draft, says a text "sounds like AI / sounds robotic", asks to "make this sound human / rendi umana questa mail/questo testo / fai si che sia umana / suona da IA", or right after drafting a client email, an article, a post, or docs and before publishing/sending. Trigger even if they don't say the word "humanize" — any request to make written text feel less machine-made and more natural belongs here, regardless of what kind of text it is.
---

# Humanize

Prende un testo che suona da AI e lo riscrive in una voce umana vera. Stesso significato, stessi fatti, stessa lingua, stesso registro, stessa struttura — non è un cambio di contenuto, è un cambio di voce. La riscrittura vera gira su un subagent **Sonnet**, perché è un lavoro di scrittura single-shot e Sonnet lo fa bene senza bisogno di giudizio; tu orchestri.

## Why this exists

La prosa da AI è statisticamente levigata e ritmicamente uniforme, e si appoggia a un piccolo set di tell — il trattino lungo su tutti, più un vocabolario e delle transizioni standard. Un lettore le riconosce subito, che sia una mail a un cliente, un articolo o un post. Il fix non è aggiungere fronzoli: è ripristinare un ritmo irregolare e diretto e tagliare il vocabolario da macchina. Il dizionario completo dei tell e dei pattern umani sta in `references/humanize-rules.md` — quel file è la sostanza, questo è solo il flusso.

## Workflow

### 1. Trova il testo
Identifica cosa riscrivere, in quest'ordine:
- testo esplicito o un path nel messaggio dell'utente,
- un file a cui punta,
- altrimenti il testo/messaggio più recente scritto prima nella conversazione (caso comune: hai appena scritto una mail o un paragrafo e dicono "rendilo umano").

Se è genuinamente ambiguo quale testo intendano, chiedi una volta sola. Altrimenti procedi, non fermarti su una domanda a cui puoi rispondere dal contesto.

### 2. Rileva il tipo e la lingua

**Il tipo si deduce dal testo, non si chiede.** Ogni tipo ha una forma che lo tradisce e un motivo per cui va trattato diverso:
- **Mail**: ha un destinatario, spesso un saluto e una firma. Va mantenuta nel suo registro — formale ("Lei") col cliente, diretto con un pari — perché umano non vuol dire informale.
- **Articolo / blog post**: ha un titolo, sezioni, una voce d'autore che regge tutto il pezzo. Va mantenuta la sua voce, non appiattita su un tono neutro.
- **Post social**: corto, ha un ritmo suo (spesso frasi molto brevi, a volte niente punteggiatura formale). Non allungarlo né infilargli struttura che non aveva.
- **Documentazione tecnica / README / changelog**: non va addolcita né resa "calda" — va solo de-robotizzata. Il lettore cerca precisione, non compagnia; tieni i tell fuori (vocabolario gonfio, transizioni robotiche) ma non aggiungere colloquialità che lì suonerebbe fuori posto.
- **Copy di sito / messaggio generico**: se non rientra chiaramente in nessuno dei casi sopra, trattalo come prosa generica — de-tellizzalo mantenendo tono e struttura originali — e dillo nell'output ("trattato come testo generico, tipo non chiaramente identificabile").

**La lingua si rileva dal testo e non si cambia mai.** Un testo in tedesco esce in tedesco, uno in inglese resta inglese. Vale anche per i diacritici: non sostituirli mai con equivalenti ASCII (mai "e" per "è", "fur" per "für").

### 3. Nota i vincoli da preservare
Prima di delegare, fissa (dal testo stesso + conversazione): **lingua**, **tipo di testo**, **registro e relazione** (Lei vs tu; cliente vs pari — ad esempio una mail a un cliente formale resta "Lei" per tutta la riscrittura), **struttura** (titolo/sezioni per un articolo, saluto/firma per una mail, niente struttura aggiunta per un post), e fatti/numeri/promesse che devono sopravvivere intatti. Passa tutto questo esplicitamente al subagent perché non deriva.

### 4. Delega la riscrittura a Sonnet
Spawna un subagent con l'Agent tool, `subagent_type: general-purpose`, **`model: sonnet`**. Un prompt sulla falsariga di questo (riempi i valori reali):

> Rewrite the text below so it reads like a real person wrote it, not an AI. First read the rules at `<ABSOLUTE_PATH_TO>/references/humanize-rules.md` and apply them. This is a `<TYPE: email/article/social post/documentation/generic>`, written in `<LANGUAGE>`, register `<formal "Lei" / informal "tu" / N/A>`. Hard constraints: keep the meaning, facts, numbers, dates, names and promises exactly, keep the structure (title/sections for an article, greeting/signature for an email, no structure invented for a short post), and ADD NOTHING — not one sentence, clause, or promise that isn't already in the draft. Remove EVERY em dash (—), no exceptions, replacing each with a comma, a period, or parentheses — unless the text is documentation/technical and em dashes there are a deliberate house-style choice, in which case leave them. Keep the language exactly as given, do not translate, do not swap accented characters for ASCII equivalents. Keep the register if applicable. Return ONLY the rewritten text, ready to use, with no preamble, no "here is the humanized version", and no notes after it.
>
> TEXT:
> `<the draft>`

Risolvi `<ABSOLUTE_PATH_TO>` sulla `references/humanize-rules.md` di questa skill, così il subagent legge il dizionario completo invece che tu lo reincolli.

### 5. Presenta
Output del testo riscritto come **ultima cosa del turno**, pronto da copiare. Niente commento meta dopo. Se l'utente ha chiesto esplicitamente cosa è cambiato, metti una nota di una o due righe *prima* del testo, mai dopo.

## Guardrail

- Questa è una riscrittura di **stile**, mai di contenuto. Se il subagent torna avendo perso un fatto, cambiato un numero, ammorbidito un impegno, alterato la struttura (titolo/sezioni di un articolo, saluto/firma di una mail) o cambiato "Lei"→"tu", scartalo e rilancia segnalando il problema. La fedeltà batte la naturalezza.
- Non tradurre mai. L'italiano resta italiano, il tedesco resta tedesco, coi diacritici intatti.
- Non è documentazione da riscaldare in prosa amichevole: se il tipo rilevato è tecnico, il target resta preciso e diretto, non caldo.
- L'obiettivo è una lettura genuinamente naturale per chi legge, non ingannare un rilevatore AI. In pratica coincidono, ma la bussola è il lettore.
- Un controllo veloce prima di presentare: cerca "—" nell'output e i peggiori vocaboli (leverage/robust/seamless/Furthermore/Inoltre/"è importante sottolineare"). Se qualcosa è passato, rimanda per un altro giro.
