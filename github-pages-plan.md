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

Tutto questo deve essere riscritto in JavaScript client-side, i dati devono provenire da **Alpha Vantage** (API REST con CORS abilitato), e la chiave API deve essere salvata localmente nel file `docs/config.js` (escluso da git).

### Approccio scelto
- **Dati:** Alpha Vantage API (TIME_SERIES_DAILY_ADJUSTED, simbolo IBM, outputsize=full)
- **Calcoli:** JavaScript puro nel browser — nessuna libreria statistica esterna; solo Plotly per i grafici
- **Deploy:** GitHub Pages dalla cartella `/docs` del branch `main`
- **CI/CD:** nessun build step — `docs/index.html` è il file finale, deployato direttamente con `git push`
- **Chiave API:** salvata in `docs/config.js` localmente (NON committata — in `.gitignore`); l'utente la copia a mano dopo il clone
- **Repository GitHub:** da creare da zero (nuovo repo pubblico)
- **Test:** visivo — aprire `docs/index.html` nel browser dopo aver compilato `config.js`

### Struttura finale del repository
```
IBM Stocks/
├── app.py                   # mantenuto per sviluppo locale (Flask)
├── requirements.txt         # mantenuto per sviluppo locale
├── templates/
│   └── index.html           # mantenuto per sviluppo locale
├── docs/                    # ← nuova cartella, servita da GitHub Pages
│   ├── index.html           # pagina statica completa (JS + calcoli + Plotly)
│   └── config.js            # chiave Alpha Vantage (NON committata — in .gitignore)
├── .gitignore               # aggiornato per escludere docs/config.js
└── github-pages-plan.md
```

GitHub Pages viene configurato per servire dalla cartella `/docs` del branch `main`.

---

## Sub-Tasks

---

### Sub-Task 1 — Creazione repository GitHub, struttura cartelle e configurazione

**Intent**
Creare il repository GitHub da zero, inizializzare git localmente, creare la cartella `docs/`, configurare `.gitignore`, e abilitare GitHub Pages. Questo sub-task copre anche le istruzioni passo-passo per ottenere la chiave Alpha Vantage gratuita.

**Expected Outcomes**
- Repository GitHub pubblico creato e collegato al progetto locale
- Cartella `docs/` con `index.html` placeholder e `config.js` template
- `.gitignore` che esclude `docs/config.js`
- `README.md` con istruzioni complete: Alpha Vantage key, git setup, GitHub Pages
- GitHub Pages abilitato e visibile su `https://<username>.github.io/<repo-name>/`

**Todo List**
1. **Ottenere chiave Alpha Vantage** (manuale, da fare prima di tutto):
   - Andare su https://www.alphavantage.co/support/#api-key
   - Compilare il form (nome, email, uso previsto)
   - Ricevere la chiave via email o mostrata a schermo
   - Chiave gratuita: 25 chiamate/giorno, sufficiente per uso personale
2. **Creare il repository GitHub** (manuale):
   - Andare su https://github.com/new
   - Nome repository: `ibm-stocks` (o simile)
   - Visibilità: **Public** (necessario per GitHub Pages gratuito)
   - NON inizializzare con README (lo creiamo noi)
   - Cliccare "Create repository"
3. Inizializzare git nella cartella del progetto: `git init`, `git remote add origin <url>`
4. Creare `docs/config.js` con `const ALPHA_VANTAGE_KEY = "YOUR_KEY_HERE";` — da compilare a mano con la chiave reale, NON committare
5. Creare `.gitignore` con le regole: `docs/config.js`, `__pycache__/`, `*.pyc`, `.env`
6. Creare `docs/index.html` come shell HTML5 minimale (placeholder per Sub-Task 5)
7. Creare `README.md` con istruzioni: come ottenere la chiave, come compilare `config.js`, come fare il primo push, come abilitare Pages
8. Primo commit e push: `git add -A`, `git commit -m "init"`, `git push -u origin main`
9. **Abilitare GitHub Pages** (manuale):
   - Repo → Settings → Pages
   - Source: Deploy from a branch
   - Branch: `main`, Folder: `/docs`
   - Cliccare Save → dopo ~1 minuto la pagina è live

**Relevant Context**
- Sicurezza: `docs/config.js` NON deve mai essere in git — verificare `.gitignore` prima di ogni `git add`
- La chiave Alpha Vantage è in chiaro nel browser (inevitabile per pagine client-side) — accettabile per uso personale/demo; non usare per dati sensibili
- GitHub Pages gratuito richiede repo pubblico; con piano GitHub Pro/Team è possibile anche privato
- URL finale: `https://<username>.github.io/<repo-name>/`

**Status** — `[ ] pending`

---

### Sub-Task 2 — Fetch dati Alpha Vantage in JavaScript

**Intent**
Implementare in `docs/index.html` la funzione JS che chiama Alpha Vantage, scarica i dati IBM storici (10 anni), e li trasforma nella stessa struttura del payload Python (`hist_5y`, `hist_1y`, `hist_1m`).

**Expected Outcomes**
- Funzione `fetchIBMData()` che chiama `https://www.alphavantage.co/query?function=TIME_SERIES_DAILY_ADJUSTED&symbol=IBM&outputsize=full&apikey=${ALPHA_VANTAGE_KEY}`
- I dati vengono parsati e trasformati in array `{dates, closes}` ordinati cronologicamente
- Le tre finestre storiche (`hist_5y`, `hist_1y`, `hist_1m`) vengono calcolate filtrando per data
- Gestione errori: messaggio visibile all'utente se la chiave è assente o la chiamata fallisce
- Loading spinner mostrato durante il fetch

**Todo List**
1. Implementare `fetchIBMData()` con `fetch()` + `await`
2. Parsare il JSON di risposta Alpha Vantage: chiave `"Time Series (Daily)"`, sotto-chiave `"5. adjusted close"`
3. Ordinare le date cronologicamente (Alpha Vantage restituisce ordine decrescente)
4. Implementare `sliceWindow(allData, years, months)` equivalente a Python `slice_window()`
5. Costruire l'oggetto `DATA` con le tre finestre
6. Mostrare un `<div id="loading">` durante il fetch, nasconderlo al completamento
7. Mostrare un `<div id="error">` se la chiave è placeholder o la API restituisce errore

**Relevant Context**
- Alpha Vantage endpoint: `https://www.alphavantage.co/query?function=TIME_SERIES_DAILY_ADJUSTED&symbol=IBM&outputsize=full&apikey=KEY`
- La risposta ha la forma: `{"Time Series (Daily)": {"2025-01-10": {"5. adjusted close": "215.3", ...}, ...}}`
- `outputsize=full` restituisce 20+ anni di dati — filtrare a 10 anni dopo il parse
- La chiave API viene letta da `ALPHA_VANTAGE_KEY` definita in `config.js`

**Status** — `[ ] pending`

---

### Sub-Task 3 — Naive forecast in JavaScript

**Intent**
Portare la funzione Python `naive_forecast()` in JavaScript. La logica è semplice (std di diff, bdate_range, CI = 1.96 × σ × √h) e si traduce direttamente.

**Expected Outcomes**
- Funzione JS `naiveForecast(closes, dates)` che restituisce un oggetto con la stessa struttura del payload Python: `{last_date, last_close, dates, forecast, lower, upper}`
- I giorni di business vengono generati saltando sabato (6) e domenica (0) tramite `Date.getDay()`
- Il risultato è identico numericamente alla versione Python

**Todo List**
1. Implementare `stdDiff(arr)` — std delle prime differenze degli ultimi 30 valori
2. Implementare `nextBusinessDays(lastDate, count)` — genera `count` giorni lavorativi successivi
3. Implementare `naiveForecast(closes, dates)` con CI = `1.96 × sigma × Math.sqrt(h)`
4. Verificare che l'output corrisponda alla struttura attesa dal codice Plotly esistente

**Relevant Context**
- Python `pd.bdate_range` = giorni Mon–Fri, esclude solo weekend (non festivi)
- La funzione JS deve restituire le date come stringhe ISO `"YYYY-MM-DD"` (stessa convenzione del payload Python)

**Status** — `[ ] pending`

---

### Sub-Task 4 — Decomposition forecast in JavaScript (T + C + CI)

**Intent**
Portare in JavaScript le funzioni Python `decomposition_forecast()`: regressione OLS lineare (trend T), FFT periodogramma, selezione armonici, regressione armonica (ciclo C), forecast 5 anni, CI piatto ±1.96×RMSE.

Questa è la sub-task più complessa: richiede implementare OLS e FFT in JS puro senza librerie esterne.

**Expected Outcomes**
- Funzione JS `decompositionForecast(closes, dates)` che restituisce lo stesso oggetto del payload Python
- OLS implementato con le equazioni normali: `coeffs = (X'X)⁻¹ X'y` — per 2 parametri (intercetta + slope) è una formula diretta, non serve inversione matriciale
- FFT implementata con `computePeriodogram(r)` usando l'algoritmo Cooley-Tukey (o una versione iterativa semplice per array di lunghezza arbitraria)
- Selezione armonici: power > mean + 2×std, top 10
- Regressione armonica: OLS su matrice sin/cos, coefficienti calcolati con equazioni normali
- Forecast 5 anni su business days
- CI piatto: `± 1.96 × RMSE`
- Output identico strutturalmente al payload Python

**Todo List**
1. Implementare `olsLinear(t, y)` — regressione lineare semplice con formula diretta (intercetta + slope)
2. Implementare `fftReal(r)` — FFT real-input; usare la libreria **fft.js** (CDN: `https://cdn.jsdelivr.net/npm/fft.js@4.0.4/lib/fft.js`) per evitare di reimplementare Cooley-Tukey da zero
3. Implementare `computePeriodogram(r)` — chiama `fftReal`, calcola potenza one-sided, restituisce `{freqs, power}`
4. Implementare `selectHarmonics(freqs, power, maxN=10)` — soglia mean + 2×std
5. Implementare `olsHarmonic(t, r, selectedFreqs)` — OLS su design matrix sin/cos (equazioni normali con `(X'X)⁻¹ X'y`, usando eliminazione gaussiana per la matrice 2K×2K)
6. Implementare `decompositionForecast(closes, dates)` che assembla tutti i passi
7. Verificare che le date forecast siano business days (stesso algoritmo Sub-Task 3)
8. Verificare struttura output compatibile con il codice Plotly in `docs/index.html`

**Relevant Context**
- `fft.js` è una libreria leggera (< 5KB) per FFT real-input in JS: `https://cdn.jsdelivr.net/npm/fft.js@4.0.4/lib/fft.js`
- OLS lineare semplice: `b = (Σxy - n·x̄·ȳ) / (Σx² - n·x̄²)`, `a = ȳ - b·x̄` — nessuna libreria necessaria
- OLS armonico: per K armonici la matrice è 2K×2K — implementare eliminazione gaussiana con pivoting parziale
- I periodi nel periodogramma sono in trading days (1/freq), identici alla versione Python
- La funzione `harmonic_matrix` Python non ha intercetta — stessa scelta in JS

**Status** — `[ ] pending`

---

### Sub-Task 5 — Assemblaggio `docs/index.html` finale, test e push

**Intent**
Assemblare la pagina finale `docs/index.html` integrando fetch dati, calcoli, e rendering Plotly. Il CSS e il codice Plotly sono già scritti in `templates/index.html` — vengono copiati e adattati (rimozione Jinja2, aggiunta logica async).

**Expected Outcomes**
- `docs/index.html` è un file HTML autosufficiente che funziona aprendo direttamente nel browser (file://) o da GitHub Pages
- Il flusso di avvio è: mostra loading → fetch Alpha Vantage → calcola naive + decomp → renderizza 4 grafici Plotly
- Il selettore 5Y/1Y/1M funziona identicamente alla versione Flask
- Hover sui CI mostra Upper/Lower come nella versione Flask
- Il file non contiene la chiave API (letta da `config.js` separato)
- Testato localmente aprendo `docs/index.html` direttamente nel browser con `config.js` compilato

**Todo List**
1. Copiare CSS dark-theme e struttura HTML da `templates/index.html` in `docs/index.html`
2. Rimuovere tutti i tag Jinja2 (`{{ data | safe }}`)
3. Aggiungere `<script src="config.js">` come primo script (definisce `ALPHA_VANTAGE_KEY`)
4. Aggiungere `<script src="https://cdn.jsdelivr.net/npm/fft.js@4.0.4/lib/fft.js">`
5. Integrare le funzioni delle Sub-Task 2, 3, 4 nello script della pagina
6. Implementare la funzione `main()` async: fetch → calcola → renderizza
7. Sostituire `const DATA = {{ data | safe }};` con la chiamata a `main()`
8. Aggiungere stati loading/error visibili all'utente
9. **Test locale**: compilare `docs/config.js` con la chiave reale → aprire `docs/index.html` nel browser → verificare visivamente i 4 grafici
10. **Deploy**: `git add docs/index.html`, `git commit -m "static page"`, `git push` → verificare su `https://<username>.github.io/<repo-name>/`

**Relevant Context**
- `templates/index.html` contiene tutto il codice Plotly riusabile (LAYOUT_BASE, CONFIG, ciRibbon, renderHist, ecc.)
- Il cambio principale è: da dati iniettati server-side a dati calcolati client-side in funzione async
- `config.js` NON viene mai committato — solo `docs/index.html` va in git
- Alpha Vantage free tier: **25 chiamate/giorno** — la pagina fa 1 chiamata al caricamento; il limite è sufficiente per uso personale
- Il repo non ha step di build: ogni `git push` aggiorna direttamente la pagina live

**Status** — `[ ] pending`

---

## Dipendenze aggiuntive (solo per docs/)

| Risorsa | URL CDN | Motivo |
|---|---|---|
| Plotly 2.35.2 | `https://cdn.plot.ly/plotly-2.35.2.min.js` | Grafici (già in uso) |
| fft.js 4.0.4 | `https://cdn.jsdelivr.net/npm/fft.js@4.0.4/lib/fft.js` | FFT per periodogramma in JS |

Nessuna altra libreria JS è necessaria — OLS, slice finestre, CI sono tutti implementabili in JS vanilla.

---

## Note di sicurezza

- `docs/config.js` contiene la chiave Alpha Vantage → **mai committare**, sempre in `.gitignore`
- La chiave API gratuita Alpha Vantage è pubblica per natura (usata nel browser) — accettabile per uso personale/demo
- Il repository GitHub deve essere **pubblico** per GitHub Pages gratuito (piano Free), oppure privato con piano Team/Enterprise

---

## Cosa NON cambia

- `app.py` e `templates/index.html` rimangono intatti per lo sviluppo locale con Flask
- `requirements.txt` rimane invariato
- La logica statistica è identica — solo la lingua cambia (Python → JavaScript)
