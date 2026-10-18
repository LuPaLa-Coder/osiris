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

## Comandi

| Comando | Cosa fa |
|---|---|
| `osiris analyze` | Avvia, segue, annulla ed elenca gli audit |
| `osiris report` | Legge il report di un audit concluso (Markdown o JSON) |
| `osiris issues` | Elenca i problemi, con filtri e ordinamento per priorità |
| `osiris geo` | Punteggio di visibilità AI e file di raccomandazioni (`llms.txt` e simili) |
| `osiris gsc` | Google Search Console: query, pagine, ispezione URL (opzionale) |
| `osiris keywords` | Idee keyword e gap di contenuto rispetto ai concorrenti (opzionale) |
| `osiris config` | Legge e scrive la configurazione utente |
| `osiris context` | Mostra percorsi, audit corrente e integrazioni |
| `osiris mcp serve` | Avvia il server MCP su stdio |

`osiris <comando> --help` mostra le opzioni.

## Configurazione

```bash
osiris config list
osiris config set --key defaultMaxPages --value 100
```

Le integrazioni sono tutte opzionali. Senza credenziali crawl e audit funzionano comunque.

- **PageSpeed Insights** (Core Web Vitals): `osiris config set --key usePageSpeedInsights --value true`;
  una chiave propria alza la quota: `--key pageSpeedApiKey --value <chiave>`.
- **Google Search Console**: `gscServiceAccountPath` (service account JSON) oppure
  `gscOAuthClientSecret` (OAuth), poi `osiris gsc auth`.

I valori segreti non vengono mai stampati: `config list` mostra `***`.

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

L'agente può avviare un audit, seguirne l'avanzamento e leggere report e problemi con gli stessi
risultati della CLI.

## Sicurezza

- Gli indirizzi privati e locali sono bloccati per impostazione predefinita.
- Il crawler rispetta `robots.txt` e ha limiti di profondità, pagine e concorrenza.
- Tutto resta in locale sotto `~/.osiris`; nessun dato viene inviato a servizi che non hai configurato.
