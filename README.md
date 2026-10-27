# 3M-Consulting/.github

Il repository comune dell'organizzazione 3M Consulting: file di salute, moduli delle issue, preset di Renovate e CI riusabile. È pubblico perché GitHub legge i file di salute di riserva solo da un `.github` pubblico; per questo non contiene niente di interno — né clienti, né persone, né indirizzi dei sistemi.

## File di salute di riserva

GitHub mostra questi file in ogni repository dell'organizzazione che non ne ha uno proprio. Un file del repository **sostituisce per intero** quello di qui: i due non si fondono.

| File                                                   | Cosa contiene                                                        |
| ------------------------------------------------------ | -------------------------------------------------------------------- |
| [CONTRIBUTING.md](./CONTRIBUTING.md)                   | Lingue, issue, branch e pull request, commit, dipendenze             |
| [SECURITY.md](./SECURITY.md)                           | Come si segnala una vulnerabilità                                    |
| [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)             | Codice di comportamento (Contributor Covenant 2.1)                   |
| [profile/README.md](./profile/README.md)               | La pagina dell'organizzazione su GitHub                              |
| [.github/ISSUE_TEMPLATE/](./.github/ISSUE_TEMPLATE/)   | Un modulo per tipo di lavoro: difetto, funzionalità, task, sicurezza |
| [PULL_REQUEST_TEMPLATE.md](./PULL_REQUEST_TEMPLATE.md) | Il corpo di partenza di ogni pull request                            |

I moduli stanno per forza in `.github/ISSUE_TEMPLATE/`: GitHub non li cerca altrove, a differenza degli altri file. La sostituzione vale per cartella: un repository con una propria `.github/ISSUE_TEMPLATE/` — anche solo un `config.yml` — non mostra nessuno dei moduli di qui.

## Preset di Renovate

[default.json](./default.json) è il preset comune degli aggiornamenti delle dipendenze; ogni regola porta il proprio perché nel campo `description`, e la sezione Dipendenze di [CONTRIBUTING.md](./CONTRIBUTING.md) dice che cosa ne segue per chi lavora. Un repository lo adotta con un `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>3M-Consulting/.github"]
}
```

Le regole del repository si aggiungono sotto `extends` e vincono su quelle del preset. Perché il preset agisca servono due impostazioni fuori dai file:

- l'app Renovate dell'organizzazione deve includere il repository;
- per le PR sulle vulnerabilità segnalate da GitHub, nel repository vanno attivati _Dependency graph_ e _Dependabot alerts_ (Settings → Advanced Security), e l'app deve poter leggere gli avvisi. Le vulnerabilità del database OSV arrivano anche senza.

## CI riusabile

Due componenti, da referenziare sempre con lo **SHA completo** di un tag e la versione in commento — `@<sha> # vX.Y.Z`, mai `@main` né il solo tag, che si può spostare. Lo SHA di un tag:

```bash
gh api repos/3M-Consulting/.github/commits/vX.Y.Z --jq .sha
```

Renovate, col preset qui sopra, propone il tag nuovo al primo giro mensile, nella PR «non-major dependencies».

### Controlli di igiene

[.github/workflows/hygiene.yml](./.github/workflows/hygiene.yml) è un workflow da chiamare: un job, `checks`, che su GitHub compare come `hygiene / checks`.

| Evento                                  | gitleaks                  | actionlint | zizmor |
| --------------------------------------- | ------------------------- | ---------- | ------ |
| PR aperta, aggiornata o riaperta        | ✓ commit fra base e testa | ✓          | ✓      |
| PR modificata, **cambia la base**       | ✓ commit fra base e testa | ✓          | ✓      |
| PR modificata, cambiano titolo o corpo  | —                         | —          | —      |
| Push su `main` (solo questo repository) | ✓ commit del push         | ✓          | ✓      |

- **gitleaks** cerca segreti nei commit dell'intervallo; un intervallo vuoto è un errore, non un verde.
- **actionlint** controlla la sintassi dei workflow.
- **zizmor** ne controlla la sicurezza, con gli audit che interrogano GitHub; ogni `uses:` deve essere fissato allo SHA.

Quando cambia la base i controlli si ripetono sul nuovo intervallo. Quando cambiano soltanto titolo o corpo non parte niente, e l'esecuzione in corso non si annulla. Questi due comportamenti stanno nel workflow **chiamante**, che si copia così com'è in `.github/workflows/hygiene.yml`:

```yaml
name: hygiene

on:
  pull_request:
    types: [opened, synchronize, reopened, edited]

permissions: {}

# An edit of the PR title or body alone has a group of its own: it never cancels the run in progress.
concurrency:
  group: ${{ (github.event.action == 'edited' && !github.event.changes.base) && format('noop-{0}', github.run_id) || format('hygiene-{0}', github.ref) }}
  cancel-in-progress: true

jobs:
  hygiene:
    # Only the PR title or body changed: nothing to check, zero minutes.
    if: github.event.action != 'edited' || github.event.changes.base
    uses: 3M-Consulting/.github/.github/workflows/hygiene.yml@<sha> # vX.Y.Z
    permissions:
      contents: read # what the `checks` job of hygiene.yml asks for, nothing more
```

### Node e pnpm

[actions/setup-node-pnpm](./actions/setup-node-pnpm/action.yml) installa pnpm (versione dal campo `packageManager` di `package.json`) e Node (versione da `.node-version`), ripristina lo store di pnpm dalla cache e installa le dipendenze.

```yaml
- uses: actions/checkout@<sha> # vX.Y.Z
  with:
    persist-credentials: false
- uses: 3M-Consulting/.github/actions/setup-node-pnpm@<sha> # vX.Y.Z
  with:
    working-directory: frontend
```

| Input               | Predefinito     | A che serve                                                            |
| ------------------- | --------------- | ---------------------------------------------------------------------- |
| `working-directory` | `.`             | La cartella del progetto pnpm, relativa alla radice del repository     |
| `node-version-file` | `.node-version` | Il file della versione di Node, relativo a `working-directory`         |
| `cache-prefix`      | `pnpm-store-v1` | Prefisso della cache dello store: cambiarlo riparte da una cache vuota |
| `install`           | `true`          | `false` si ferma al setup, senza `pnpm install --frozen-lockfile`      |

## Licenza

Il contenuto del repository è sotto licenza MIT: vedi [LICENSE](./LICENSE). `CODE_OF_CONDUCT.md` è la traduzione italiana del Contributor Covenant 2.1, sotto CC BY 4.0.
