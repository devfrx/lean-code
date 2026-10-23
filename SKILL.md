---
name: lean-code
description: Dimensionare il codice al minimo che risolve il problema per intero — la scala che decide cosa scrivere davvero (serve? c'è già? lo fa la libreria standard, la piattaforma o una dipendenza installata?), il perimetro della modifica, gli input reali da gestire, test proporzionati al rischio, modifiche chirurgiche e cancellazioni complete. Usa questa skill OGNI VOLTA che stai per scrivere o modificare codice — feature, bugfix, refactoring, script, componenti, endpoint, piccole app da zero — o scegli tra codice nuovo, riuso e una dipendenza, anche se l'utente non parla mai di minimalismo; e quando chiede "la soluzione più semplice", "senza over-engineering", "yagni" o si lamenta di codice gonfio. Si attiva insieme a dev-discipline. NON usarla per spiegare codice (code-walkthrough), per revisionare un diff o un repo (engineering:code-review), né per la documentazione (lean-docs, design-docs).
---

# Lean Code

Questa skill governa quanto codice si scrive e come si dimostra che basta.

Il principio di fondo: **ogni riga scritta va letta, testata e mantenuta da qualcuno, e ogni caso reale non gestito è un bug che qualcuno troverà dopo.** Si pagano entrambi, quindi la misura è la brevità *tra le soluzioni complete*: quelle che raggiungono ogni punto toccato dalla modifica, reggono gli input che arriveranno davvero e portano la prova che funzionano. Più corta e incompleta non è più snella. Deriva da ponytail; provenienza, scelte e limiti sono in `FONTI.md`.

**Prerequisiti.** Questa skill si appoggia a skill che governano aspetti che qui non vengono ripetuti:

- `dev-discipline` — esplorare prima di scrivere, riuso, YAGNI, convenzioni, cultura dei test. Qui diventano la scala e la prova; dove il repo ha una convenzione, vince la convenzione.
- `decision-principles` — la proporzione, di cui questa skill è la versione operativa per il codice, e la distinzione tra verificato, dedotto e assunto.
- `dev-communication` — quando fermarsi: dipendenze nuove, cambi di pattern, richieste ambigue o sovradimensionate.

Leggile se non sono già in contesto.

## Le tre regole che governano tutto

In caso di tensione vincono queste, in quest'ordine:

1. **Il pavimento non si taglia**: validazione ai confini di fiducia, gestione degli errori che evita perdita o corruzione di dati, sicurezza, privacy, accessibilità, la calibrazione che l'hardware reale richiede, tutto ciò che l'utente ha chiesto.
2. **Completo prima di corto**: si accorcia solo tra soluzioni che raggiungono il perimetro, gestiscono gli input reali e portano la prova.
3. **Tra le complete, la più piccola da leggere**: si misura in ciò che un revisore deve capire, non in caratteri. Una riga da decifrare non è corta, il noioso batte il furbo, e la struttura del progetto — layer, interfacce, convenzioni — non è codice da risparmiare.

Le fasi che seguono sono un riflesso, non una cerimonia: per una modifica di una riga il perimetro è un grep e l'inventario un'occhiata; diventano lavoro vero quando la modifica tocca dati, soldi, permessi o più moduli. Non si raccontano nella risposta: si vedono nel risultato.

## Fase 1 — Il perimetro, prima della soluzione

Il diff più corto nel posto sbagliato è un secondo bug.

- **Cosa deve cambiare**, in una frase sugli effetti osservabili. Se non riesci a scriverla, o la richiesta ammette due letture con costi diversi, fai la domanda minima (`dev-communication`).
- **Dove deve arrivare**: traccia il flusso dall'ingresso all'effetto ed elenca chiamanti, test, fixture, configurazione, export, migrazioni e documentazione di quel comportamento. I chiamanti si cercano con grep su tutto l'albero, compresi i riferimenti dinamici: nomi in stringhe, configurazione, template, reflection.
- **Nei bug, la causa**: prima di modificare, grep di ogni chiamante della funzione che stai per toccare, poi correggi una volta sola nel punto condiviso. Una guardia lì è un diff più piccolo di una per chiamante, e correggere solo il percorso della segnalazione lascia rotti gli altri.
- **L'inventario degli input reali**: parti da dove arrivano i dati — utenti, file esportati, API esterne, altri moduli — ed elenca ciò che arriverà davvero: vuoto, zero e negativi, ultimo elemento, duplicati, campi mancanti o malformati, encoding e separatori, fusi orari e arrotondamenti, volumi reali, accessi concorrenti. I casi plausibili sono requisiti, non extra. Quelli impossibili per costruzione — esclusi da un tipo, da uno schema, da una validazione a monte che hai letto — non si gestiscono: una guardia per un caso che non può accadere è rumore.
- **Chi vede cosa**: se la modifica porta dati verso persone o sistemi — email, notifiche, log, risposte API, export, messaggi d'errore — chiediti chi vede adesso qualcosa che prima non vedeva: gli indirizzi degli altri destinatari, un token, dati personali o di un altro utente. Un'esposizione è un difetto anche quando la funzione fa esattamente ciò che è stato chiesto.

## Fase 2 — La scala

Fermati al primo gradino che copre l'intero perimetro:

1. **Serve?** Opzioni, configurabilità e casi futuri che nessuno ha chiesto non si costruiscono: si nominano in una riga a fine lavoro. A una richiesta vaga ("fammi un X") risponde la versione più piccola che fa il lavoro centrale, dicendo cosa resta fuori.
2. **C'è già nel progetto** un helper, un componente, un servizio, un pattern? Usalo come lo usa il codice intorno.
3. **Lo fa la libreria standard o la piattaforma?** `<input type="date">` invece di un calendario scritto a mano, un vincolo del database invece di un controllo applicativo, CSS invece di JavaScript, `functools.lru_cache` invece di una classe di cache — a meno che il progetto non abbia già il suo modo.
4. **Lo fa una dipendenza già installata?** Usala. Una nuova per poche righe non entra; una nuova in generale è una decisione strutturale.
5. **Sta in una riga che si capisce al primo colpo?** Una riga.
6. **Solo allora**, il codice minimo che copre il perimetro.

Prima di fermarti, **verifica lo strumento sui casi dell'inventario, non sul nome**. Un parser non è un validatore: `email.utils.parseaddr("@dominio.it")` restituisce l'indirizzo intatto. In una regex Python `$` accetta anche un a capo finale, quindi per validare una stringa intera serve `re.fullmatch`; `intval("12abc")` in PHP e `parseInt("12abc")` in JavaScript valgono 12. Controlla la documentazione della versione in uso, o prova la funzione.

## Fase 3 — Scrivere la modifica

- **Gestire non vuol dire avvolgere.** Ogni caso dell'inventario ha un comportamento deciso — errore chiaro, elemento saltato e segnalato, default — come il progetto fa già per casi simili; senza precedenti, il meno sorprendente, dichiarato nella consegna. Un `try/except` generico che inghiotte l'errore non lo gestisce, lo nasconde: un solo gestore, al confine reale.
- **Ogni riga cambiata si ricollega alla richiesta.** Niente riformattazioni, commenti riscritti o "miglioramenti" al codice vicino: costringono chi rivede a capire perché sono lì.
- **Si pulisce solo ciò che hai reso orfano tu**: import, variabili, funzioni, flag. Il codice morto preesistente si segnala in una riga e resta dov'è: non è nel perimetro, e può avere chiamanti che il grep non vede.
- **Il codice spostato o unito** si porta dietro la sua gestione degli errori e la sua validazione.
- **Commenti**: solo il perché che il codice non dice, salvo docstring richieste dal repo.
- **Una cancellazione è completa quando niente di ciò che resta punta al vuoto**: chiamanti, anche dinamici; commenti che rimandano a un ragionamento nel codice tolto, il cui perché va spostato dove serve ancora; test che coprivano solo il ramo rimosso, da riorientare su ciò che resta se il comportamento sopravvive e da eliminare solo se sparisce; marker e TODO legati a ciò che hai tolto.

## Fase 4 — La prova, proporzionata al rischio

Un test vale se fallisce quando la logica si rompe. Prova di mutazione a mente: se invertissi un confronto, togliessi una guardia, sbagliassi di uno un indice, il test se ne accorgerebbe? Se no, non prova niente ed è solo codice in più da leggere.

- **Banale** — una riga, un rename, una costante, collegamenti senza logica: nessun test nuovo oltre a quelli che la convenzione del repo richiede; gli esistenti si aggiornano e devono passare.
- **Logica** — un ramo, un ciclo, un parser, soldi, scritture di dati: i test che bastano a far fallire ogni rottura plausibile, di solito uno per ramo che conta e uno per caso dell'inventario che hai gestito. Per un bug, il test che falliva prima della correzione.
- **Sicurezza** — autenticazione, autorizzazione, input non fidati, segreti, isolamento tra utenti o tenant: un test per ogni confine e per ogni caso di abuso pertinente (input malevolo, credenziale manomessa o malformata, accesso ai dati di un altro), non uno solo. Qui risparmiare sui test è il risparmio più caro.
- **Test come deliverable**: la copertura dei comportamenti richiesti è il perimetro. Il minimo vale per ciascun test — niente duplicati, niente test che ricalcano l'implementazione metodo per metodo, asserzioni su ciò che si osserva da fuori — non per il loro numero.

Framework e stile sono quelli del repo; se non c'è un framework, un file di test minimo o un self-check con `assert`, senza introdurne uno. Esegui i test che hai scritto o toccato; se non puoi, dillo.

## Fase 5 — La consegna

- **Le scorciatoie si dichiarano accanto al loro limite.** Un taglio deliberato con un tetto noto — lock globale, scansione quadratica, euristica ingenua, nessuna paginazione — si annota con la convenzione del debito già in uso nel repo (TODO, FIXME, un tag) o, se manca, con un commento che nomina il limite e quando rivederlo. Il codice normale non si annota, e niente porta il nome di una skill.
- **La risposta**, dopo il lavoro e in poche righe: cosa hai lasciato fuori di proposito e il segnale che lo renderebbe necessario ("niente cache: aggiungila se il profiler la indica"); cosa hai eseguito e cosa no; i rischi che chi rivede deve conoscere. Per una modifica piccola, se la spiegazione supera il codice, accorciala; una spiegazione richiesta si dà per intero.

## Casi edge

- **La richiesta chiede più del bisogno** — una cache con scadenza dove basta `lru_cache`, una libreria per dieci righe: dillo prima di costruire, con l'alternativa in una riga. Se l'utente conferma, costruisci ciò che ha chiesto, per intero e senza rilitigare (`dev-communication`).
- **Prototipo o spike dichiarato**: inventario e test si riducono al caso dimostrativo, ogni scorciatoia si dichiara, il pavimento resta.
- **Progetto già sovra-ingegnerizzato**: la modifica segue i layer esistenti. Semplificarli è un altro task, da proporre in una riga.
- **Deleghi codice a un sotto-agente**: il criterio non passa da solo. Nel prompt vanno la scala, l'inventario già fatto e il pavimento; a un sotto-agente che legge soltanto non servono.

## Checklist prima di consegnare

1. Il perimetro è raggiunto tutto — chiamanti anche dinamici, test, fixture, configurazione, documentazione?
2. Ogni input plausibile ha un comportamento deciso, senza guardie per casi impossibili, e chi riceve i dati vede solo ciò che deve?
3. Mi sono fermato al primo gradino che regge, con uno strumento verificato sui casi dell'inventario?
4. Ogni riga cambiata si ricollega alla richiesta, e nessuna cancellazione lascia riferimenti nel vuoto?
5. I test falliscono se la logica si rompe, sono proporzionati al rischio, e li ho eseguiti — o ho detto che non ho potuto?
6. Le scorciatoie sono dichiarate accanto al loro limite, e solo loro?
7. La risposta dice cosa è rimasto fuori e con quale segnale, cosa ho eseguito e quali rischi restano?

Se una risposta è "no", sistemala prima di consegnare — o dichiarala esplicitamente.
