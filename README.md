# IBM Stock Dashboard — GitHub Pages

Dashboard interattiva per le azioni IBM, deployata come pagina statica su GitHub Pages. Tutti i calcoli (regressione lineare, FFT, previsioni) sono eseguiti nel browser in JavaScript puro. Nessuna chiave API richiesta.

## Demo live

`https://<username>.github.io/<repo-name>/`

---

## Funzionalità

- **Storico prezzi IBM** con selettore 5Y / 1Y / 1M
- **Previsione naive 1 mese** con intervallo di confidenza 95% (modello random-walk, CI = 1.96 × σ × √h)
- **Previsione decomposizione 5 anni** (trend lineare T + ciclo armonico C) con CI piatto ±1.96×RMSE
- **Periodogramma** con frequenze armoniche selezionate evidenziate
- Tema dark, grafici Plotly interattivi, aggiornamento automatico ogni 60 minuti

---

## Dati

I prezzi storici IBM provengono dall'endpoint pubblico di **Yahoo Finance** — nessuna registrazione, nessuna chiave API, nessun limite di chiamate.

---

## Setup e deploy

### 1. Crea il repository GitHub

1. Vai su <https://github.com/new>
2. Nome repository: `ibm-stocks` (o simile)
3. Visibilità: **Public** (necessario per GitHub Pages gratuito)
4. Non inizializzare con README — lo abbiamo già

### 2. Carica i file

Con **GitHub Desktop** (consigliato se non hai dimestichezza con git):

1. Scarica e installa [GitHub Desktop](https://desktop.github.com/)
2. Fai login con il tuo account GitHub
3. **File → Add Local Repository** → seleziona questa cartella
4. Se chiede di inizializzare git, clicca **Initialize Repository**
5. Clicca **Publish repository** → scegli il nome, spunta **Public** → Publish

Oppure con **upload manuale** su github.com:

1. Apri il repository appena creato
2. Clicca **"uploading an existing file"**
3. Trascina i file: `.gitignore`, `README.md`, `github-pages-plan.md`, `docs/index.html`
4. Clicca **Commit changes**

### 3. Abilita GitHub Pages

1. Repository → **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` — Folder: `/docs`
4. Clicca **Save**

Dopo ~1 minuto la pagina è live su `https://<username>.github.io/<repo-name>/`.

---

## Struttura del repository

```
IBM Stocks/
├── docs/
│   └── index.html        # pagina statica completa (JS + calcoli + Plotly)
├── .gitignore
├── README.md
└── github-pages-plan.md
```

---

## Tecnologie usate

| Componente | Tecnologia |
|---|---|
| Dati storici | Yahoo Finance (endpoint pubblico, nessuna chiave) |
| Grafici | [Plotly.js 2.35.2](https://cdn.plot.ly/plotly-2.35.2.min.js) |
| FFT (periodogramma) | Cooley-Tukey iterativo in JS vanilla (nessuna libreria esterna) |
| Trend + regressione | JavaScript vanilla (OLS con equazioni normali) |
| Deploy | GitHub Pages (`/docs` branch `main`) |
