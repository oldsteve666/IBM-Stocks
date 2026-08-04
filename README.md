# IBM Stock Dashboard — GitHub Pages

Dashboard interattiva per le azioni IBM, deployata come pagina statica su GitHub Pages. Tutti i calcoli (regressione lineare, FFT, previsioni) sono eseguiti nel browser in JavaScript puro.

## Demo live

`https://<username>.github.io/<repo-name>/`

---

## Funzionalità

- **Storico prezzi IBM** con selettore 5Y / 1Y / 1M
- **Previsione naive 1 mese** con intervallo di confidenza 95%
- **Previsione decomposizione 5 anni** (trend lineare + ciclico armonico) con CI piatto ±1.96×RMSE
- **Periodogramma** con frequenze armoniche selezionate
- Tema dark, grafici Plotly interattivi, aggiornamento automatico ogni 60 minuti

---

## Setup

### 1. Ottieni una chiave Alpha Vantage gratuita

1. Vai su <https://www.alphavantage.co/support/#api-key>
2. Compila il form (nome, email, uso previsto)
3. Ricevi la chiave (25 chiamate/giorno — sufficiente per uso personale)

### 2. Crea `docs/config.js`

```js
// docs/config.js  ← NON committare questo file
const ALPHA_VANTAGE_KEY = "LA_TUA_CHIAVE_QUI";
```

> ⚠️ `docs/config.js` è in `.gitignore` — non verrà mai committato nel repository.

### 3. Primo push

```bash
git init
git remote add origin https://github.com/<username>/<repo-name>.git
git add -A
git commit -m "init: IBM Stock Dashboard"
git push -u origin main
```

### 4. Abilita GitHub Pages

1. Repository → **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` — Folder: `/docs`
4. Clicca **Save** → dopo ~1 minuto la pagina è live

---

## Struttura del repository

```
IBM Stocks/
├── docs/
│   ├── index.html        # pagina statica completa
│   └── config.js         # chiave API (NON committata — in .gitignore)
├── .gitignore
├── README.md
└── github-pages-plan.md
```

---

## Note di sicurezza

- La chiave Alpha Vantage è usata lato client (browser) — inevitabile per pagine statiche. È accettabile per uso personale/demo.
- Non usare questa configurazione per dati sensibili o in produzione.
- Il repository deve essere **pubblico** per GitHub Pages gratuito (piano Free).
