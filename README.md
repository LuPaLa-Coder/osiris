# Osiris

Audit SEO + GEO da terminale e da agenti AI. Un solo eseguibile: una CLI e un server MCP che
espone le stesse funzioni. Osiris scansiona un sito, esegue controlli tecnici, di contenuto e di
performance, salva l'audit in locale e produce un report con le priorità.

Sito: <https://osiris.lupala.com>

## Download

Ultima versione: [Releases](https://github.com/LuPaLa-Coder/osiris/releases/latest).
Eseguibili self-contained: non serve installare .NET.

| Sistema | File |
|---|---|
| macOS Apple Silicon | `osiris-osx-arm64.tar.gz` |
| macOS Intel | `osiris-osx-x64.tar.gz` |
| Linux x64 | `osiris-linux-x64.tar.gz` |
| Windows x64 | `osiris-win-x64.zip` |

Ogni release include `SHA256SUMS` per verificare i file scaricati.

## Installazione

**macOS / Linux**

```bash
tar -xzf osiris-osx-arm64.tar.gz          # usa il file del tuo sistema
sudo mv osiris /usr/local/bin/osiris
osiris --version
```

Su macOS, se il file scaricato da browser viene bloccato da Gatekeeper:

```bash
xattr -d com.apple.quarantine /usr/local/bin/osiris
```

**Windows**

Estrai `osiris-win-x64.zip`, sposta `osiris.exe` in una cartella inclusa nel `PATH`, poi:

```powershell
osiris --version
```

**Verifica checksum** (macOS / Linux): `shasum -a 256 -c SHA256SUMS --ignore-missing`

## Primi passi

```bash
# Audit di un sito e attesa del risultato
osiris analyze start https://example.com --depth 2 --max-pages 50 --wait

# Report in Markdown
osiris report get --format md

# Problemi, solo i quick win
osiris issues list --quick-wins

# Visibilità per motori AI (GEO)
osiris geo check --url https://example.com
```

Gli audit e la configurazione stanno in `~/.osiris`. Con `OSIRIS_HOME` si sposta l'intera cartella.
Ogni comando accetta `--json` per l'output strutturato.

## Riferimento comandi

Ogni comando accetta `--json` (stesso output dello strumento MCP corrispondente) e `-h` per l'help.
Quando `--audit-id` è omesso si usa l'audit corrente (vedi `osiris context set`).
Il comando `osiris --version` mostra la versione.

### `osiris analyze` — audit

| Comando | Cosa fa |
|---|---|
| `analyze start <url>` | Avvia un audit; di default attende la fine |
| `analyze status` | Avanzamento e contatori di un audit |
| `analyze cancel` | Richiede l'annullamento di un audit in corso |
| `analyze list` | Ultimi audit, dal più recente (`--limit`, default 20, max 200) |

Opzioni di `analyze start`:

| Opzione | Descrizione |
|---|---|
| `--depth <n>` | Profondità massima del crawl (0–10) |
| `--max-pages <n>` | Numero massimo di pagine |
| `--max-concurrency <n>` | Richieste parallele |
| `--include-subdomains` | Scansiona anche i sottodomini |
| `--respect-robots-txt` / `--no-respect-robots-txt` | Rispetta o ignora robots.txt |
| `--user-agent <ua>` | User-Agent del crawler |
| `--analyze-content` / `--no-analyze-content` | Analisi dei testi (contenuto sottile, leggibilità, heading, duplicati) |
| `--probe-resources` / `--no-probe-resources` | Misura CSS, JS e immagini (peso, richieste, compressione, cache) |
| `--max-resource-probes <n>` | Massimo di risorse misurate (0–200; 0 le disattiva) |
| `--min-words <n>` | Soglia di parole sotto cui la pagina è "sottile" (50–5000, default 300) |
| `--use-page-speed-insights` / `--no-use-page-speed-insights` | Core Web Vitals con PageSpeed Insights |
| `--use-gsc` / `--no-use-gsc` | Allega query e pagine di Search Console |
| `--use-keywords` / `--no-use-keywords` | Allega le opportunità di keyword |
| `--wait` / `--no-wait` | Attende il risultato (default) o ritorna subito |
| `--timeout <min>` | Attesa massima con `--wait` (default 40) |
| `--fail-on <livello>` | Esce con codice 3 se esiste un problema almeno a quella gravità (`none`, `low`, `medium`, `high`, `critical`) |

### `osiris report get` — report

| Opzione | Descrizione |
|---|---|
| `--audit-id <id>` | Audit da leggere |
| `--format <md\|json>` | Formato (default `md`) |
| `--output <percorso>` | Scrive il report su file e stampa solo il percorso |
| `--overwrite` | Sovrascrive il file di output se esiste |
| `--compact` | Riduce i campi verbosi |

### `osiris issues list` — problemi

| Opzione | Descrizione |
|---|---|
| `--audit-id <id>` | Audit da leggere |
| `--severity <livello>` | `info`, `low`, `medium`, `high`, `critical` |
| `--category <cat>` | `technical`, `onpage`, `content`, `performance`, `accessibility`, `security`, `geo`, `other` |
| `--url <testo>` | Solo problemi il cui URL contiene il testo |
| `--check <id>` | Solo un controllo preciso (es. `hreflang`, `page-size`) |
| `--min-priority <n>` | Punteggio di priorità minimo |
| `--band <P0\|P1\|P2\|P3>` | Fascia di priorità (P0 = subito) |
| `--quick-wins` | Solo alto impatto e poco sforzo |
| `--sort <criterio>` | `priority` (default), `severity`, `impact`, `quick-wins` |
| `--limit <n>` | Massimo di risultati (default 100, max 1000) |
| `--compact` | Riduce descrizioni ed evidenze |
| `--fail-on <livello>` | Esce con codice 3 se esiste un problema almeno a quella gravità |

### `osiris geo` — visibilità per i motori AI

| Comando | Cosa fa |
|---|---|
| `geo check` | Punteggio: crawler AI in robots.txt, llms.txt, citabilità, E-E-A-T, dati strutturati |
| `geo generate` | Scrive `llms.txt`, aggiunte a `robots.txt` e JSON-LD consigliati |

Opzioni comuni: `--audit-id <id>`, oppure `--url <url>` per analizzare un sito senza audit
(`--max-pages <n>`, 1–50, default 25). `geo check` accetta anche `--compact`.
`geo generate` richiede `--output <cartella>` e accetta `--policy <allow|block-training>` e `--overwrite`.

### `osiris gsc` — Google Search Console (opzionale)

| Comando | Cosa fa | Opzioni |
|---|---|---|
| `gsc auth` | Verifica la credenziale e le proprietà leggibili | `--interactive` (avvia il consenso OAuth nel browser) |
| `gsc queries` | Query principali della proprietà | `--site-url`, `--audit-id`, `--days` (1–480, default 28), `--limit` |
| `gsc pages` | Pagine di atterraggio principali | `--site-url`, `--audit-id`, `--days`, `--limit` |
| `gsc inspect` | Stato di indicizzazione di un URL | `--url` (obbligatorio), `--site-url`, `--audit-id` |

`--site-url` accetta una proprietà (`https://example.com/` o `sc-domain:example.com`).

### `osiris keywords` — keyword (opzionale)

| Comando | Cosa fa | Opzioni |
|---|---|---|
| `keywords ideas` | Espande i seed in idee di keyword | `--seeds`, `--audit-id`, `--limit`, `--location`, `--language` |
| `keywords gap` | Keyword dei concorrenti che il sito non copre (richiede DataForSEO) | `--competitor` (fino a 5), `--audit-id`, `--limit`, `--location`, `--language` |

Senza `--seeds` si usano i temi principali dell'audit.

### Contesto e configurazione

| Comando | Cosa fa |
|---|---|
| `context` | Percorsi, audit corrente, valori predefiniti, integrazioni |
| `context set --audit-id <id>` | Imposta l'audit corrente |
| `config list` | Tutte le chiavi con il valore effettivo |
| `config get --key <chiave>` | Legge una chiave |
| `config set --key <chiave> --value <valore>` | Scrive una chiave (valore vuoto = azzera) |
| `mcp serve` | Avvia il server MCP su stdio |

## Configurazione

Le integrazioni sono tutte opzionali: senza credenziali crawl e audit funzionano comunque.
I valori segreti non vengono mai stampati (`config list` mostra `***`).

| Chiave | Predefinito | Descrizione |
|---|---|---|
| `userAgent` | `OsirisBot/1.0` | User-Agent del crawler |
| `requestTimeoutSeconds` | 20 | Timeout per richiesta |
| `defaultMaxPages` | 50 | Pagine massime per audit |
| `defaultMaxDepth` | 2 | Profondità massima |
| `defaultMaxConcurrency` | 4 | Richieste parallele |
| `respectRobotsTxt` | true | Rispetta robots.txt |
| `maxRequestsPerSecond` | 10 | Limite di velocità |
| `maxRetries` | 2 | Tentativi per richiesta |
| `allowPrivateNetworks` | false | Consente indirizzi privati e locali (solo da CLI o file) |
| `analyzeContent` | true | Analisi dei contenuti |
| `probeResources` | true | Misura delle risorse |
| `maxResourceProbes` | 30 | Risorse misurate per audit |
| `minWordCount` | 300 | Soglia contenuto sottile |
| `usePageSpeedInsights` | false | Core Web Vitals con PageSpeed Insights |
| `pageSpeedApiKey` | – | Chiave PageSpeed (segreta; senza chiave vale la quota pubblica) |
| `useGoogleSearchConsole` | false | Allega Search Console agli audit |
| `gscSiteUrl` | – | Proprietà predefinita |
| `gscDateRangeDays` | 28 | Finestra di dati |
| `gscMaxRows` | 100 | Righe massime |
| `gscServiceAccountPath` | – | File JSON del service account |
| `gscOAuthClientSecret` | – | File dei client secret OAuth |
| `geoAiPolicy` | `allow` | Politica per i crawler AI: `allow` o `block-training` |
| `useKeywords` | false | Allega le opportunità di keyword |
| `keywordMaxIdeas` | 50 | Idee massime |
| `keywordCompetitors` | – | Concorrenti predefiniti |
| `keywordLocation` | `Italy` | Mercato dei dati |
| `keywordLanguage` | `it` | Lingua dei dati |
| `dataForSeoLogin` / `dataForSeoPassword` | – | Credenziali DataForSEO (segrete) |

Esempi:

```bash
osiris config set --key defaultMaxPages --value 100
osiris config set --key usePageSpeedInsights --value true
osiris config set --key gscServiceAccountPath --value ~/.osiris/gsc-service-account.json
osiris gsc auth
```

Alcune chiavi si impostano anche con variabili d'ambiente (es. `OSIRIS_PAGESPEED_API_KEY`,
`OSIRIS_GEO_AI_POLICY`).

## Uso con agenti AI (MCP)

Osiris espone le sue funzioni come strumenti MCP su stdio. Esempio per Claude Code:

```bash
claude mcp add osiris -- osiris mcp serve
```

Oppure nel file di configurazione di un client MCP:

```json
{
  "mcpServers": {
    "osiris": {
      "command": "osiris",
      "args": ["mcp", "serve"]
    }
  }
}
```

Gli strumenti hanno gli stessi nomi dei comandi, con underscore: `osiris_context`, `osiris_context_set`,
`osiris_config_get|list|set`, `osiris_analyze_start|status|cancel|list`, `osiris_report_get`,
`osiris_issues_list`, `osiris_geo_check|generate`, `osiris_gsc_auth|queries|pages|inspect`,
`osiris_keywords_ideas|gap`. Come prima chiamata di una sessione conviene `osiris_context`.

L'agente può avviare un audit, seguirne l'avanzamento e leggere report e problemi con gli stessi
risultati della CLI.

## Sicurezza

- Gli indirizzi privati e locali sono bloccati per impostazione predefinita.
- Il crawler rispetta `robots.txt` e ha limiti di profondità, pagine e concorrenza.
- Tutto resta in locale sotto `~/.osiris`; nessun dato viene inviato a servizi che non hai configurato.
