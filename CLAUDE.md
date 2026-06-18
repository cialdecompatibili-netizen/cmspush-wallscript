# CLAUDE.md — Memoria progetto cmspush-wallscript

## Cos'è questo progetto

Landing page di vendita per **CmsPush** — CMS commerciale basato su Jekyll + GitHub Pages.
File unico: index.html (HTML + Tailwind + JS tracker).
Non è un sito Jekyll — è HTML puro, si pubblica direttamente su GitHub Pages.

---

## Percorsi importanti

| Cosa | Valore |
|------|--------|
| File locale | C:\Users\mirco\Desktop\onepage site\cmspush-wallscript\index.html |
| Repo GitHub | https://github.com/cialdecompatibili-netizen/cmspush-wallscript |
| Sito live | https://cialdecompatibili-netizen.github.io/cmspush-wallscript/ |
| Owner GitHub | cialdecompatibili-netizen |
| Repo name | cmspush-wallscript |

---

## Come ottenere il token GitHub

Il token è salvato nel remote URL della repo cmspush principale.
Estrarlo così con PowerShell:

`powershell
git -C "C:\Users\mirco\Desktop\cmspush" remote get-url origin
# Output: https://ghp_XXXX@github.com/cialdecompatibili-netizen/cmspush.git
# Il token è la parte ghp_XXXX
`

---

## Come caricare / aggiornare index.html su GitHub (via API)

Questo è il metodo usato da Claude — NON serve Git installato, funziona sempre.

`powershell
 = "ghp_XXXXXXXXXXXXXXXXXX"   # <-- prendi dal remote cmspush
 = "cialdecompatibili-netizen"
  = "cmspush-wallscript"
  = "index.html"
 = "C:\Users\mirco\Desktop\onepage site\cmspush-wallscript\index.html"

# Leggi SHA attuale (serve per aggiornare file esistente)
 = @{ Authorization = "token "; Accept = "application/vnd.github+json" }
try {
     = Invoke-RestMethod -Uri "https://api.github.com/repos///contents/" -Headers 
     = .sha
} catch {  =  }

# Leggi file e converti in base64
  = [System.IO.File]::ReadAllBytes()
 = [Convert]::ToBase64String()

# Prepara body
 = @{ message = "Update index.html"; content =  }
if () { .sha =  }   # obbligatorio se file esiste già
 =  | ConvertTo-Json

# Carica
 = Invoke-RestMethod -Uri "https://api.github.com/repos///contents/" 
    -Method PUT -Headers  -Body  -ContentType "application/json"
Write-Host "✅ Caricato! SHA: "
`

**REGOLA CRITICA:** se il file esiste già nella repo, devi passare il campo sha nel body,
altrimenti l'API risponde 422. Leggi sempre lo SHA prima di sovrascrivere.

---

## Come attivare GitHub Pages (solo prima volta)

`powershell
 = "ghp_XXXXXXXXXXXXXXXXXX"
 = "cialdecompatibili-netizen"
  = "cmspush-wallscript"

 = @{ source = @{ branch = "main"; path = "/" } } | ConvertTo-Json
 = @{ Authorization = "token "; Accept = "application/vnd.github+json" }

Invoke-RestMethod -Uri "https://api.github.com/repos///pages" 
    -Method POST -Headers  -Body  -ContentType "application/json"
Write-Host "✅ GitHub Pages attivato!"
`

Se già attivo risponde con errore — ignoralo, Pages è già on.

---

## Come usare Git da terminale (alternativa all'API)

Se Git è configurato con token nel remote URL:

`powershell
# Setup iniziale (una tantum)
cd "C:\Users\mirco\Desktop\onepage site\cmspush-wallscript"
git init
git remote add origin https://ghp_XXXX@github.com/cialdecompatibili-netizen/cmspush-wallscript.git
git branch -M main

# Push normale
cd "C:\Users\mirco\Desktop\onepage site\cmspush-wallscript"
git add .
git commit -m "Update landing page"
git push -u origin main

# Push se repo ha già commit remoti (es. file creati via API)
cd "C:\Users\mirco\Desktop\onepage site\cmspush-wallscript"
git pull --rebase
git push
`

**⚠️ PROBLEMA FREQUENTE — push rejected:** se la repo ha commit remoti (es. file caricati via API)
il push fallisce con "rejected: fetch first". Soluzione:
`powershell
git pull --rebase; git push
`
MAI usare solo git push senza pull prima — fallirà quasi sempre.

---

## Struttura file

`
cmspush-wallscript/
├── index.html      ← tutto qui — HTML + CSS Tailwind + JS tracker
└── CLAUDE.md       ← questo file
`

Il progetto è intenzionalmente minimal: 1 solo file HTML.
Non aggiungere dipendenze, build step, o file extra senza motivo.

---

## Struttura index.html

| Sezione | Descrizione |
|---------|-------------|
| NAV | Logo + menu + bottone Scarica |
| HERO | Titolo + CTA "Live Demo" e "Acquista €19" |
| ABOUT | Descrizione CmsPush + feature list |
| COME FUNZIONA | 3 step (Paga / Carica / Gestisci) |
| FEATURES | 6 card (Editor, Tema, Aggiornamenti, Pagine, Shop, Impostazioni) |
| CONFRONTO | Tabella v1 (€19) vs v2 (€49 in sviluppo) |
| PREZZI | 2 colonne pricing |
| FOOTER | Copyright + link GitHub |
| SCRIPT | Tracker click (Google Apps Script) |

---

## Tracker carrello

Il file usa un pixel tracker su ogni bottone acquisto:

`javascript
const TRACKER_ENDPOINT = "https://script.google.com/macros/s/AKfycbz.../exec";
function trackClick(sito, bottone, prodotto, prezzo) { ... }
`

I bottoni acquisto chiamano 	rackClick('cmspush-wallscript', 'acquista-hero', 'CmsPush v1', '19').
NON rimuovere o modificare questo sistema senza aggiornare anche il Google Apps Script.

---

## Link utili del progetto padre (cmspush)

| Cosa | Link |
|------|------|
| Repo sorgente CMS | https://github.com/cialdecompatibili-netizen/cmspush |
| Sito vetrina CMS | https://cialdecompatibili-netizen.github.io/cmspush/ |
| Admin CMS | https://cialdecompatibili-netizen.github.io/cmspush/admin/ |
| File locale CMS | C:\Users\mirco\Desktop\cmspush |
| CLAUDE.md CMS | C:\Users\mirco\Desktop\cmspush\.claude.md |

---

## Regole operative per Claude

- **MAI riscrivere index.html intero** — usare edit_block con old_string+new_string mirato
- **Dopo ogni modifica → ricarica via API** (script "Come aggiornare" qui sopra)
- **Leggi sempre lo SHA prima di sovrascrivere** — altrimenti l'API dà 422
- **Token dal remote cmspush** — non chiedere a Mirco, estrailo con git remote get-url
- **GitHub Pages già attivo** — non serve riattivarlo ad ogni sessione
- **1 solo file HTML** — non aggiungere CSS/JS separati senza necessità

---

## Sequenza di boot per Claude (ogni nuova sessione)

1. Leggi questo CLAUDE.md
2. Estrai token: git -C "C:\Users\mirco\Desktop\cmspush" remote get-url origin
3. Leggi stato attuale repo: Invoke-RestMethod https://api.github.com/repos/cialdecompatibili-netizen/cmspush-wallscript/contents/
4. Leggi index.html locale per capire lo stato
5. Chiedi a Mirco cosa vuole fare — veloce, chirurgico

---

## Storico modifiche

| Data | Modifica | Metodo |
|------|----------|--------|
| 2026-06-18 | Prima pubblicazione su GitHub Pages | API GitHub (via Claude) |
| 2026-06-18 | Creato CLAUDE.md con istruzioni | PowerShell write |

