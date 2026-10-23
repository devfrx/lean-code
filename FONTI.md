# Fonti, scelte e limiti

Documento per chi mantiene la skill, non da caricare durante il lavoro. Tutto il contenuto è riscritto in italiano: da ponytail si riprendono idee e regole, non testo.

## Provenienza (fonti primarie lette il 20/10/2026)

| Fonte | Repo @ commit | Licenza | Cosa è stato preso |
|---|---|---|---|
| ponytail (Dietrich Gebert), `skills/ponytail/SKILL.md` v5.3.2 | DietrichGebert/ponytail @ 09c434a (18/10/2026) | MIT | La scala (serve → c'è già → libreria standard o piattaforma → dipendenza installata → una riga → codice minimo); la regola operativa sui chiamanti nei bug fix; il pavimento; "pigro sulla soluzione, mai sulla modifica", diventato il perimetro; il codice spostato che conserva la validazione; i commenti solo per il perché; l'idea di annotare una scorciatoia con il suo limite e il momento di rivederla; le righe finali su cosa è stato saltato e sui rischi |
| Versioni precedenti di `SKILL.md` | stessa storia git (v1–v4.13) | MIT | "Il diff più corto nel posto sbagliato è un secondo bug" e "la scala è un riflesso, non una ricerca" (v4) |
| Benchmark di ponytail, `benchmarks/results/*.md` | stesso commit | MIT | Le evidenze delle debolezze (tabella sotto) e la lezione di metodo: solo le regole operative spostano il comportamento. Sul bug con causa in una funzione condivisa, "traccia il flusso" scritto come prosa ha fatto 0/3, "grep di ogni chiamante e correggi una volta nel punto condiviso" 6/6 (Opus 4.8, 2026-06-22) |
| Issue di ponytail | github.com/DietrichGebert/ponytail/issues, lette il 20/10/2026 | — | #120, #245, #595, #602, #633, #640, #664, #682, #683, #685, #823, #866; di #597 e #679 solo il titolo e le citazioni in altre issue |
| Benchmark indipendente di KuldeepB19 | kuldeepb19.github.io/ponytail-benchmark (24/06/2026, ponytail v4) | — | Gestione dell'input sbagliato tagliata in 5 task su 24 (sintesi nel README di ponytail); robustezza e production-readiness in calo, soprattutto su dati e CLI |

Altre verifiche (20/10/2026, Python 3.13, Node 22, PHP 8): `email.utils.parseaddr("@missing-local.com")` restituisce l'indirizzo intatto; con `re.match` un pattern che termina in `$` accetta `"ok@ok.com\n"`, `re.fullmatch` no; `intval("12abc")` e `parseInt("12abc")` valgono 12. Dimensione del testo che ponytail v5 inietta a ogni sessione: circa 2.900 caratteri di regole più la mappa del codebase, fino a circa 2.000 (misurati da `SKILL.md` e dal budget in `hooks/ponytail-map.js`).

## Cosa è stato corretto, e su quale evidenza

| Debolezza di ponytail | Evidenza | Correzione in lean-code |
|---|---|---|
| La correttezza sui casi limite è solo uno spareggio "a parità di dimensione"; gli input sporchi ma legittimi vengono tagliati | `SKILL.md` v5 ("between options of equal size…"); KuldeepB19 | Regola 2, completo prima di corto; inventario degli input reali (fase 1) |
| Lo strumento della libreria standard scelto per il nome | Robustness audit 2026-06-16: `parseaddr` usato come validatore. Lo scivolone compare sui modelli OpenAI, non su Claude, e otto modifiche al testo della skill non l'hanno corretto | Verifica dello strumento sui casi dell'inventario (fase 2), con esempi controllati |
| Esposizione di dati non intercettata | Task `scope-recipients` (un'email a tutti gli iscritti senza mostrare gli indirizzi): 2/5 in tutti e tre i bracci, ponytail 5 incluso (benchmark 2026-10-07). La v5 lo chiede solo in prosa ("data it would … expose") | Controllo operativo "chi vede cosa" (fase 1); privacy nel pavimento |
| Test più frequenti ma più deboli | Bug iniettati catturati dove esiste un test: 69% con ponytail 5, 77% senza skill (2026-10-07) | Prova di mutazione a mente; test per ogni rottura plausibile (fase 4) |
| Un solo test anche sul codice di sicurezza | #823; la v5 chiede ancora "one small test" | Un test per confine e per caso di abuso |
| Il minimalismo taglia i test quando i test sono il deliverable | #602: 4 test dove l'utente se ne aspettava più di 50 | La copertura richiesta è il perimetro; il minimo vale per ciascun test |
| Nessun limite alle modifiche non richieste; "deletion beats addition" spinge a cancellare codice morto altrui | #640 | Ogni riga cambiata si ricollega alla richiesta; si pulisce solo ciò che la modifica ha reso orfano |
| Le cancellazioni lasciano riferimenti nel codice che resta | #866 | Una cancellazione è completa quando niente punta al vuoto |
| Codice iper-difensivo e `try/except` generici | #682 (chiesta come passata di review e audit) | Portata in scrittura: gestire non vuol dire avvolgere; niente guardie per casi impossibili |
| Test che ricalcano l'implementazione, asserzioni solo su mock o snapshot | #683 (chiesta come passata di audit) | Portata in scrittura, nei test come deliverable |
| Persona "sviluppatore pigro" fraintendibile | #633: un modello l'ha letta come "risparmia fatica umana" ed è uscito dall'harness | Un obiettivo esplicito al posto della persona |
| Marker con il nome del prodotto, messo anche su codice banale | #120; ristretto in #577, rinominato in `shortcut:` nella PR #1068 | Convenzione del debito già in uso nel repo, solo per tagli deliberati, mai il nome di una skill |
| Il modello rigurgita il banner delle istruzioni | #595 | Le fasi non si raccontano nella risposta |
| Sempre attivo, anche su test, task non di codice e sotto-agenti di sola lettura | #685: sulla v4 circa 2,8k token riletti a ogni chiamata, +6,9% di costo rispetto a una sola frase di istruzioni (misura di terzi, Sonnet 5); nella v5 il testo è sceso, ma resta un costo fisso a ogni chiamata. #597, #602 | Skill caricata su descrizione; ai sotto-agenti il criterio passa solo se scrivono codice |
| Livelli lite/full/ultra quasi identici | #664 (v4.8.4: 96 righe su 99 uguali); nella v5, oltre al nome del livello nell'intestazione, cambia una sola riga di regole (`hooks/ponytail-instructions.js`) | Niente livelli: la richiesta sovradimensionata si discute prima (`dev-communication`), lo spike si dichiara |

## Divergenze con le skill dell'utente e come sono state risolte

| Tema | Posizioni | Scelta |
|---|---|---|
| Test sulle modifiche banali | ponytail: nessuno. `dev-discipline`: aggiungi o aggiorna i test per il codice che tocchi | Nessun test nuovo oltre a quelli che la convenzione del repo richiede; gli esistenti si aggiornano |
| Dove si dichiara una scorciatoia | ponytail: commento `shortcut:` nel codice. `dev-discipline`: dichiarala. `decision-principles`: dove chi legge la troverà | Nel codice accanto al limite, con la convenzione del repo, e nella risposta |
| Livelli di intensità | ponytail: lite e ultra. `dev-communication`: se la richiesta va contro i principi, dillo prima | Vale `dev-communication`; niente livelli |
| Codice morto | ponytail: "deletion beats addition". `dev-discipline`: le incoerenze si segnalano, non si correggono di straforo | Si cancella solo ciò che la modifica ha reso orfano; il resto si segnala |
| Commenti | ponytail: solo il perché, in una riga | Solo il perché, salvo docstring richieste dal repo |

## Escluso, e perché

- **Hook e iniezione sempre attiva** (SessionStart, SubagentStart, UserPromptSubmit): costo fisso a ogni chiamata (#685) e regole applicate fuori contesto (#602). Una skill si carica quando serve; il prezzo è dipendere da una descrizione che la faccia attivare.
- **Livelli, statusline, band e board**: i livelli non cambiano il comportamento (#664); il resto è interfaccia di Claude Code.
- **ponytail-review, ponytail-audit, ponytail-debt**: fuori perimetro per scelta dell'utente, che ha chiesto solo la scrittura. Le loro debolezze note (#682, #683, #866 e #679, che chiede di registrare il commit di partenza) restano qui per un eventuale secondo giro; la revisione resta a `engineering:code-review`.
- **ponytail-gain e ponytail-help**: vetrina dei numeri del benchmark e guida ai comandi, nessun comportamento.
- **Mappa del codebase iniettata all'avvio**: richiede un hook; cercare prima di scrivere è già in `dev-discipline`.
- **Adattatori per altri agenti** (Cursor, Codex, Copilot e gli altri): non servono a una skill di Claude.

## Non verificato (limiti di questa sintesi)

- **L'effetto delle regole nuove non è misurato.** Per scelta dell'utente non è stato eseguito alcun benchmark: ogni correzione è un'ipotesi fondata sulle issue e sui benchmark di ponytail, da validare sul lavoro reale. Le regole nuove sono scritte come azioni o controlli, perché è il formato che in ponytail ha spostato il comportamento, ma nessuna è stata provata.
- **Rischio atteso**: rispetto a ponytail, più codice dove l'inventario trova casi reali e più test sul codice di sicurezza. È voluto, ma la misura che lo bilancia (quanto codice in più, quanta robustezza in più) non esiste ancora.
- **L'attivazione della descrizione non è testata**: il ciclo di `skill-creator` richiede sessioni dedicate.
- **Le issue sono state lette tramite pagine web riassunte**, non dal testo grezzo né dall'API: dettagli e commenti possono mancare, e alcuni elenchi apparivano non aggiornati rispetto ai commit.
- **Benchmark di KuldeepB19**: lette solo la pagina di sintesi e la riga nel README di ponytail, non le pagine per task; riguarda la v4.
- **La misura di #685** è di terzi, sulla v4, non riprodotta.
