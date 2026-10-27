# Come si contribuisce

Questo file vale per ogni repository di 3M Consulting che non ne ha uno proprio: GitHub lo mostra al suo posto. Un repository con un proprio `CONTRIBUTING.md` lo sostituisce per intero; per questo il file del repository rimanda qui in apertura e aggiunge soltanto ciò che vale per lui.

Le regole valgono allo stesso modo per chi lavora a mano e per gli agenti AI.

## Lingue

| In italiano                                         | In inglese                                                |
| --------------------------------------------------- | --------------------------------------------------------- |
| Testi per l'utente: interfaccia, messaggi di errore | Identificatori, nomi di file e di cartelle                |
| Titolo e corpo di issue e pull request              | Commenti, docstring, messaggi di log, nomi dei test       |
| Documenti e moduli di questo repository             | Messaggi di commit                                        |
|                                                     | `description` delle regole di Renovate, job e input di CI |

I termini di dominio italiani restano tali anche nel codice: `codice_fiscale`, `pratica`, `cedolino`.

## Issue

### Moduli ed etichette

Le issue si aprono dai moduli di [.github/ISSUE_TEMPLATE/](./.github/ISSUE_TEMPLATE/), uno per tipo di lavoro:

| Modulo                           | Quando                                                                     | Etichetta         |
| -------------------------------- | -------------------------------------------------------------------------- | ----------------- |
| Segnala un difetto               | Il sistema si comporta in modo diverso da quello atteso                    | `bug`             |
| Proponi una funzionalità         | Una capacità nuova, o la modifica di una esistente                         | `enhancement`     |
| Apri un task                     | Lavoro che non cambia il comportamento: manutenzione, strumenti, documenti | —                 |
| Registra un rilievo di sicurezza | Un gap di sicurezza o di conformità già noto                               | `security-review` |

Una issue creata da riga di comando (`gh issue create`) non passa dal modulo: riporta le stesse sezioni, con gli stessi titoli, e la stessa etichetta. Le etichette proprie di un repository, come la gravità, e le sue milestone le definisce il suo `CONTRIBUTING.md`.

Una vulnerabilità sfruttabile non diventa una issue: segue [SECURITY.md](./SECURITY.md).

### Titolo e criteri

- Il **titolo** descrive l'esito osservabile, non l'attività: «la ricevuta arriva anche quando l'indirizzo contiene lettere accentate», non «sistemare le email».
- I **criteri di accettazione** sono un elenco puntato, senza caselle, di criteri verificabili — un comando, un test, un esito osservabile. Una casella spuntata dice che qualcuno ha cliccato, non con quale prova: la prova la dà la pull request che chiude la issue, criterio per criterio, nella sezione «Come è stato verificato» del [modello della PR](./PULL_REQUEST_TEMPLATE.md). Per un criterio che non si applica si dà il motivo al posto della prova.

### Che cosa non entra in una issue

Dati personali di clienti o dipendenti, credenziali, testi di contratti. Al loro posto gli identificativi tecnici e il link al documento, che resta dove è archiviato.

## Branch e pull request

- **I rami si creano quando il lavoro comincia**, da `main` aggiornato. `gh issue develop <numero> --base main --checkout` crea un ramo collegato: la issue lo mostra nella propria pagina e la PR la ritrova da sola.
- **Ogni modifica entra in `main` da una pull request.** Nei repository privati del piano gratuito di GitHub la piattaforma non applica regole di branch: la regola regge sulla disciplina di chi lavora.
- **Il corpo della PR** segue il [modello](./PULL_REQUEST_TEMPLATE.md). Per chiudere una issue al merge: una riga `Closes #<numero>` per issue, con la parola chiave in inglese (`Closes`, `Fixes`, `Resolves`; «Chiude» non chiude niente) e fuori dai backtick. Una PR che non soddisfa tutti i criteri di una issue la nomina con `Refs #<numero>`, dice che cosa manca, e la issue resta aperta.
- **I controlli di igiene** (`hygiene / checks`: segreti nei commit, sintassi e sicurezza dei workflow) girano su ogni PR dei repository che li chiamano; sono descritti nel [README](./README.md).

## Commit e hook

- **Conventional Commits, in inglese**: `tipo(scope): descrizione`, che dice l'effetto — `fix(logging): a UUID stays readable in the logs`.
- **Un commit fa una cosa.** Un lavoro a più passi ha un commit per passo, ognuno verificato prima del successivo.
- **Mai `--no-verify`.** Se un hook blocca, il messaggio dice che cosa correggere; aggirarlo sposta soltanto la scoperta più avanti.

## Dipendenze

Le regole di questa sezione stanno in tre posti: il preset di Renovate [default.json](./default.json), la configurazione dei gestori di pacchetti di ogni progetto e la sua CI. Ogni regola dice dove.

### Lock file

- Ogni progetto tiene nel repository il proprio lock file (`pnpm-lock.yaml`, `uv.lock`; quello di uv porta anche gli hash). Lo scrivono il gestore di pacchetti e Renovate, mai una mano.
- La CI lo verifica: `pnpm install --frozen-lockfile`, `uv lock --check`. Le immagini Docker installano con `--frozen-lockfile` o `--locked`, che rifiutano un lock non allineato.

### Versioni dichiarate

- **Python: ogni dipendenza diretta con `==`.** Renovate (manager `pep621`) salta una dipendenza senza versione: la sposterebbe soltanto la manutenzione mensile del lock, che passa anche i major senza approvazione.
- **Nessun limite superiore scritto a mano** (`<3`, `~=2.1`): a fermare i major ci pensa l'approvazione nella Dependency Dashboard.
- **npm**: gli intervalli di `package.json` restano quelli del progetto; le `devDependencies` Renovate le propone fissate alla versione esatta (`:pinDevDependencies`, da `config:best-practices`).

### Cadenza

- **Un giro al mese**, il primo del mese prima delle 4 (ora di Roma). Ne escono una PR «non-major dependencies», con minor, patch, pin e SHA delle action insieme, e una PR di manutenzione del lock file.
- **Un major è una migrazione**, e lo è anche un minor di un pacchetto `0.x`, perché un intervallo `^0.8` non accetta la `0.9`. La PR si apre soltanto dopo l'approvazione nella Dependency Dashboard (la issue che Renovate tiene aperta in ogni repository), una per pacchetto. Prima di approvare si leggono le note di rilascio e si cercano gli usi nel codice.
- Renovate non fonde niente da sé (`automerge: false`).

### Periodo di attesa di 7 giorni

Una versione entra soltanto 7 giorni dopo la pubblicazione: un pacchetto compromesso viene di norma scoperto e ritirato entro pochi giorni. Due livelli, perché Renovate non vede le dipendenze indirette che il gestore di pacchetti risolve da sé:

- Renovate non propone la versione prima dei 7 giorni (`minimumReleaseAge` nel preset).
- Il gestore di pacchetti non la risolve, dipendenze indirette comprese: pnpm con `minimumReleaseAge: 10080` (minuti) in `pnpm-workspace.yaml`, uv con `exclude-newer = "7 days"` sotto `[tool.uv]` in `pyproject.toml`.

Le immagini su GHCR non pubblicano una data di rilascio: si aggiornano senza attesa.

### Correzioni di sicurezza

- Renovate apre una PR per ogni vulnerabilità nota di una dipendenza diretta (avvisi di GitHub e database OSV) fuori dal giro mensile e senza approvazione, appena la versione corretta ha 7 giorni: a quel punto anche pnpm e uv la accettano, e il lock si aggiorna da solo. Si guarda prima delle altre.
- Le vulnerabilità delle dipendenze indirette le mostra l'audit della CI (`pip-audit`, `pnpm audit`, dove il repository lo esegue) e le chiude la manutenzione mensile del lock.

**Canale d'emergenza, facoltativo.** Per avere una correzione prima dei 7 giorni si apre un'eccezione per quel solo pacchetto, e la si toglie trascorsi i 7 giorni:

- uv: sotto `[tool.uv]` in `pyproject.toml`, l'istante di oggi, con il riferimento dell'avviso; poi `uv lock --upgrade-package <pacchetto>` (per una dipendenza diretta, prima si alza il suo `==`):

  ```toml
  # GHSA-xxxx-xxxx-xxxx: remove after 2026-11-03
  exclude-newer-package = { urllib3 = "2026-10-27T12:00:00Z" }
  ```

  Sempre un istante, mai `false` né una durata: `false` spegnerebbe l'attesa per quel pacchetto per sempre, senza che nessuno se ne accorga. Con un istante, un'eccezione dimenticata ferma il pacchetto, e il suo aggiornamento successivo non si risolve finché non la si toglie. Renovate non scrive da sé questa eccezione ([renovatebot/renovate#41654](https://github.com/renovatebot/renovate/issues/41654)): quando lo farà, questo passo sparisce.

- pnpm: il nome del pacchetto in `minimumReleaseAgeExclude` in `pnpm-workspace.yaml`, poi `pnpm install`.

### Una dipendenza nuova

Entra soltanto se:

- è mantenuta: rilasci recenti, issue che ricevono risposta;
- ha una licenza permissiva (MIT, BSD, Apache-2.0, ISC); per ogni altra si chiede prima;
- non fa ciò che fa già una dipendenza presente (date, richieste HTTP, validazione…).

Si aggiunge con il comando del gestore — `pnpm add`, `uv add "<pacchetto>==<versione>"` — e la PR dice perché serve.

### Le PR di Renovate

- **«non-major dependencies» e manutenzione del lock**: CI verde, si fonde. Se dopo qualcosa si rompe, si annulla la PR con un revert e si apre una issue sul pacchetto colpevole.
- **Major e minor di un `0.x`**: dopo l'approvazione la PR si tratta come ogni migrazione, con i test e la verifica a mano dei percorsi toccati.
- **Vulnerabilità**: per prima, con la stessa regola della CI verde.

## Documentazione

- **Un solo proprietario per informazione.** Chi la possiede la scrive per esteso; gli altri documenti mettono una riga e un link, mai una copia.
- **I documenti descrivono il sistema com'è**, al presente: niente avanzamento, niente «per ora» o «prossimamente». La storia sta in git.

## Sicurezza e comportamento

Le vulnerabilità si segnalano come dice [SECURITY.md](./SECURITY.md). Chi partecipa segue il [codice di comportamento](./CODE_OF_CONDUCT.md).
