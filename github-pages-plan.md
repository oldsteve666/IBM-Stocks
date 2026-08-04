# IBM Stock Dashboard — Deploy su GitHub Pages

## Top-Level Overview

Trasformare l'app Flask (server Python + Jinja2) in una **pagina HTML statica 100% client-side** deployabile su GitHub Pages.

### Il problema
GitHub Pages serve solo file statici. L'attuale `app.py` fa in Python:
- Fetch dei dati via `yfinance`
- Calcolo della regressione lineare (trend T)
- Calcolo del periodogramma FFT e selezione degli armonici
- Regressione armonica (ciclo C)
- Calcolo dei CI naive e decomposizione
- Injection del JSON nel template Jinja2

Tutto questo deve essere riscritto in JavaScript client-side, i dati provengono da **Yahoo Finance** (API pubblica, nessuna chiave richiesta, CORS abilitato).

### Approccio scelto
- **Dati:** Yahoo Finance endpoint pubblico — `https://query1.finance.yahoo.com/v8/finance/chart/IBM?interval=1d&range=10y` — nessuna chiave API, nessun limite
- **Calcoli:** JavaScript puro nel browser — nessuna libreria statistica esterna; solo Plotly per i grafici e fft.js per la FFT
- **Deploy:** GitHub Pages dalla cartella `/docs` del branch `main`
- **CI/CD:** nessun build step — `docs/index.html` è il file finale, deployato direttamente con `git push`
- **Chiave API:** non necessaria
- **Repository GitHub:** da creare da zero (nuovo repo pubblico)
- **Test:** visivo — aprire `docs/index.html` nel browser

### Struttura finale del repository
```
IBM Stocks/
├── docs/                    # ← cartella servita da GitHub Pages
│   └── index.html           # pagina statica completa (JS + calcoli + Plotly)
├── .gitignore
├── README.md
└── github-pages-plan.md
```

GitHub Pages viene configurato per servire dalla cartella `/docs` del branch `main`.

---

## Sub-Tasks

---

### Sub-Task 1 — Creazione repository GitHub, struttura cartelle e configurazione

**Intent**
Creare il repository GitHub da zero, inizializzare git localmente, creare la cartella `docs/`, configurare `.gitignore`, e abilitare GitHub Pages.

**Expected Outcomes**
- Repository GitHub pubblico creato e collegato al progetto locale
- Cartella `docs/` con `index.html`
- `.gitignore` con regole standard Python
- `README.md` con istruzioni complete: git setup, GitHub Pages
- GitHub Pages abilitato e visibile su `https://<username>.github.io/<repo-name>/`

**Todo List**
1. **Creare il repository GitHub** (manuale):
   - Andare su https://github.com/new
   - Nome repository: `ibm-stocks` (o simile)
   - Visibilità: **Public** (necessario per GitHub Pages gratuito)
   - NON inizializzare con README (lo creiamo noi)
   - Cliccare "Create repository"
2. Creare `.gitignore` con le regole: `__pycache__/`, `*.pyc`, `.env`
3. Creare `docs/index.html` come shell HTML5 minimale (placeholder per Sub-Task 5)
4. Creare `README.md` con istruzioni: come fare il primo push, come abilitare Pages
5. Primo commit e push via GitHub Desktop o upload manuale su github.com
6. **Abilitare GitHub Pages** (manuale):
   - Repo → Settings → Pages
   - Source: Deploy from a branch
   - Branch: `main`, Folder: `/docs`
   - Cliccare Save → dopo ~1 minuto la pagina è live

**Relevant Context**
- GitHub Pages gratuito richiede repo pubblico; con piano GitHub Pro/Team è possibile anche privato
- URL finale: `https://<username>.github.io/<repo-name>/`

**Status** — `[x] done`

---

### Sub-Task 2 — Fetch dati Yahoo Finance in JavaScript

**Intent**
Implementare in `docs/index.html` la funzione JS che chiama Yahoo Finance, scarica i dati IBM storici (10 anni), e li trasforma nella struttura `{dates, closes}` con le tre finestre storiche (`hist_5y`, `hist_1y`, `hist_1m`).

**Expected Outcomes**
- Funzione `fetchIBMData()` che chiama `https://query1.finance.yahoo.com/v8/finance/chart/IBM?interval=1d&range=10y`
- I dati vengono parsati da timestamps UNIX e prezzi `quote[0].close`
- Le tre finestre storiche vengono calcolate con `sliceWindow()`
- Gestione errori: messaggio visibile all'utente se la chiamata fallisce
- Loading spinner mostrato durante il fetch

**Todo List**
1. Implementare `fetchIBMData()` con `fetch()` + `await`
2. Parsare il JSON di risposta Yahoo Finance: `chart.result[0].timestamp` e `indicators.quote[0].close`
3. Convertire timestamps UNIX in stringhe ISO `"YYYY-MM-DD"`, filtrare valori null
4. Implementare `sliceWindow(allDates, allCloses, {years, months})`
5. Costruire l'oggetto `DATA` con `all`, `hist_5y`, `hist_1y`, `hist_1m`
6. Mostrare `<div id="loading">` durante il fetch, nasconderlo al completamento
7. Mostrare `<div id="error-msg">` in caso di errore

**Relevant Context**
- Endpoint: `https://query1.finance.yahoo.com/v8/finance/chart/IBM?interval=1d&range=10y`
- Struttura risposta: `{chart: {result: [{timestamp: [...], indicators: {quote: [{close: [...]}]}}]}}`
- Nessuna chiave API richiesta
- CORS abilitato — funziona direttamente dal browser

**Note post-implementazione**
- Alpha Vantage è stato abbandonato: `TIME_SERIES_DAILY_ADJUSTED` è diventato premium, `outputsize=full` è diventato premium anche per `TIME_SERIES_DAILY`. Yahoo Finance è la fonte dati definitiva.

**Status** — `[x] done`

---

### Sub-Task 3 — Naive forecast in JavaScript

**Intent**
Portare la funzione Python `naive_forecast()` in JavaScript. La logica è semplice (std di diff, bdate_range, CI = 1.96 × σ × √h) e si traduce direttamente.

**Expected Outcomes**
- Funzione JS `naiveForecast(closes, dates)` che restituisce `{last_date, last_close, dates, forecast, lower, upper, sigma}`
- I giorni di business vengono generati saltando sabato (6) e domenica (0) tramite `Date.getDay()`
- Il risultato è identico numericamente alla versione Python

**Todo List**
1. Implementare `stdDiff(arr)` — std delle prime differenze degli ultimi 30 valori
2. Implementare `nextBusinessDays(lastDate, count)` — genera `count` giorni lavorativi successivi
3. Implementare `naiveForecast(closes, dates)` con CI = `1.96 × sigma × Math.sqrt(h)`

**Relevant Context**
- Python `pd.bdate_range` = giorni Mon–Fri, esclude solo weekend (non festivi)
- La funzione JS restituisce le date come stringhe ISO `"YYYY-MM-DD"`

**Status** — `[x] done`

---

### Sub-Task 4 — Decomposition forecast in JavaScript (T + C + CI)

**Intent**
Portare in JavaScript le funzioni Python `decomposition_forecast()`: regressione OLS lineare (trend T), FFT periodogramma, selezione armonici, regressione armonica (ciclo C), forecast 5 anni, CI piatto ±1.96×RMSE.

**Expected Outcomes**
- Funzione JS `decompositionForecast(closes, dates)` con lo stesso output strutturale della versione Python
- OLS lineare: formula diretta (intercetta + slope)
- FFT: via libreria fft.js (CDN), zero-padding a potenza di 2
- Selezione armonici: power > mean + 2×std, top 10
- Regressione armonica: OLS su design matrix sin/cos (nessuna intercetta), eliminazione gaussiana con pivoting parziale
- Forecast 5 anni su business days (~252×5 giorni)
- CI piatto: `± 1.96 × RMSE`, lower bound floored a $0.01

**Todo List**
1. Implementare `olsLinear(t, y)`
2. Implementare `computePeriodogram(r)` tramite fft.js, correzione one-sided
3. Implementare `selectHarmonics(freqs, power, maxN=10)` — soglia mean + 2×std
4. Implementare `olsHarmonic(t, r, selectedFreqs)` — eliminazione gaussiana
5. Implementare `decompositionForecast(closes, dates)` che assembla tutti i passi

**Relevant Context**
- `fft.js` 4.0.4: `https://cdn.jsdelivr.net/npm/fft.js@4.0.4/lib/fft.js`
- OLS lineare: `b = (Σxy - n·x̄·ȳ) / (Σx² - n·x̄²)`, `a = ȳ - b·x̄`
- Periodi nel periodogramma in trading days (1/freq)
- `t_future = arange(n, n + len(future_dates))` — continua l'indice intero

**Status** — `[x] done`

---

### Sub-Task 5 — Assemblaggio `docs/index.html` finale, test e push

**Intent**
Assemblare la pagina finale `docs/index.html` integrando fetch dati, calcoli, e rendering Plotly con dark theme e selettore 5Y/1Y/1M.

**Expected Outcomes**
- `docs/index.html` è un file HTML autosufficiente (~730 righe)
- Flusso: loading → fetch Yahoo Finance → calcola naive + decomp → renderizza 4 grafici Plotly
- Selettore 5Y/1Y/1M funzionante via `Plotly.react`
- Dark theme: bg `#0d1117`, surface `#161b22`, accent `#58a6ff`
- CI ribbon semi-trasparenti (`fill: 'tonexty'`)
- Auto-refresh ogni 60 minuti
- Nessuna chiave API nel file

**Todo List**
1. HTML + CSS dark theme con variabili CSS
2. `<script>` CDN: Plotly 2.35.2, fft.js 4.0.4
3. Funzioni JS: `fetchIBMData`, `naiveForecast`, `decompositionForecast`
4. Funzioni Plotly: `LAYOUT_BASE`, `ciRibbon`, `renderHist`, `renderNaive`, `renderDecomp`, `renderPeriodo`
5. `main()` async: fetch → calcola → renderizza
6. **Deploy**: push `docs/index.html` → verifica su GitHub Pages

**Status** — `[x] done`

---

## Dipendenze (solo per docs/)

| Risorsa | URL CDN | Motivo |
|---|---|---|
| Plotly 2.35.2 | `https://cdn.plot.ly/plotly-2.35.2.min.js` | Grafici |
| fft.js 4.0.4 | `https://cdn.jsdelivr.net/npm/fft.js@4.0.4/lib/fft.js` | FFT per periodogramma in JS |

Nessuna altra libreria JS necessaria — OLS, slice finestre, CI sono tutti in JS vanilla.

---

## Note di sicurezza

- Nessuna chiave API nel repository — Yahoo Finance è pubblico e non richiede autenticazione
- Il repository GitHub deve essere **pubblico** per GitHub Pages gratuito (piano Free)

---

## Note post-implementazione

### Abbandono di Alpha Vantage (risolto)

Alpha Vantage ha ristretto progressivamente il piano gratuito durante l'implementazione:
1. `TIME_SERIES_DAILY_ADJUSTED` → diventato **premium**
2. `outputsize=full` su `TIME_SERIES_DAILY` → diventato **premium**

**Risoluzione**: sostituito con Yahoo Finance endpoint pubblico (`/v8/finance/chart`), nessuna chiave, nessun limite, 10 anni di dati disponibili. `docs/config.js` è stato eliminato.

---

## Cosa NON cambia

- La logica statistica è identica alla versione Flask — solo la lingua cambia (Python → JavaScript)
