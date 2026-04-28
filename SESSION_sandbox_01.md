# SESSION_sandbox_01 — 28. april 2026.

## Kontekst

Flavio je otvorio novu temu: **Transformers.js** — fasciniran demo primjerima na HuggingFace.
Odluka: napraviti zaseban sandbox (`sandbox.opik.net`) kao pravo igralište, nezavisno od
thelab-a, X-Ray knjige i svih ostalih projekata. Bez redoslijeda, smisla, koristi — čista
eksploracija.

---

## Infrastruktura

### sandbox.opik.net — novo postavljanje

**Flavio (sudo):**
- `mkdir -p /var/www/vhosts/sandbox/html`
- Kreirao `/etc/apache2/sites-available/sandbox.opik.net.conf` (iz Claude generisanog fajla)
- `a2ensite sandbox.opik.net.conf && systemctl reload apache2`
- `certbot --apache -d sandbox.opik.net` — TLS aktivan

**GitHub:** `fladroid/sandbox` repo kreiran. Commit historija:
- `eae31b5` — Initial: landing + embeddings
- `07522f5` — diff stats fix
- `b4221e7` — v1.1 + no-cache headers
- `3c7e6bb` — v1.2 renderDiffStats fix
- `aee03b4` — v1.3 svih 384 dims
- `af0d31d` — v1.4 3-col grid, diff grey scale
- `f44360d` — v1.5 diff cells 5x20
- `5632fb9` — v1.6 diff cells 8x13
- `d3f8457` — v1.7 inner div hack (nije dobro)
- `45f30c3` — v1.8 diff cells 10x10 (finalno)
- `7bd0dd0` — v2.0 multilingual model switcher fix
- `6b17c6d` — v1.9 (redoslijed u gitu, logično v2.0 je fix)
- `850d0ac` — arxiv corpus loader
- `76f3622` — translation quality experiment

**Web root:** `/var/www/vhosts/sandbox/html/`
**Balsam local:** `/home/balsam/sandbox/`
**Deploy:** `cp -r /var/www/vhosts/sandbox/html/* /home/balsam/sandbox/`

---

## Eksperimenti

### 01 — Embeddings (`/embeddings/`)

**Model:** `Xenova/all-MiniLM-L6-v2` (384 dims, ~23MB)
**Što radi:** Upiši dvije fraze → cosine similarity + heat mapa vektora (384 ćelije) +
difference vizualizacija (siva skala)

**Verzije:**
- v1.0 — inicijalna verzija (dark theme)
- v1.1 — light theme, diff stats (mean Δ, max Δ, top dims)
- v1.2 — fix: renderDiffStats nije bio pozvan (module scope bug)
- v1.3 — svih 384 dimenzija (bilo 192)
- v1.4 — 3-col grid (A, B, Difference u jednom redu), diff grey scale
- v1.5–v1.7 — iteracije visine diff kartice (nisu dobro riješile)
- v1.8 — diff cells 10×10px identično A i B (finalno)
- v2.0 — **model switcher**: all-MiniLM-L6-v2 (EN) i multilingual-MiniLM-L12-v2 (50 jezika)
  Fix: `onclick` ne radi u `<script type="module">` scope → event listeners

**Ključni nalazi iz eksperimenta:**
- `king/queen` (EN): 0.685 MiniLM → 0.679 multilingual (EN gubi malo prostora)
- `könig/königin` (DE): 0.881 → 0.946 (DE dobro zastupljen u oba modela)
- `kralj/kraljica` (BHS): 0.678 → 0.725 (multilingual pomaže)
- `re/regina` (IT): 0.230 → 0.528 (polisemija `re` razriješena kontekstom)
- `il re / la regina`: 0.746 (član pomaže modelu da jednoznačno identificira značenje)
- `man/woman` = `muškarac/žena` = 0.414 (distribucijska opozicija, ne srodnost)
- `muž/žena`: 0.497 vs `marito/moglie` (IT): 0.791 (bračni par IT > BHS)
- `Ehemann/Ehefrau`: 0.383 (birokratski registar, svaki termin zasebno u formularima)

**Zaključak koji prirodno iskrsava (X-Ray tema):**
Cosine similarity mjeri distribucijsku ko-pojavu, ne semantičku srodnost kakvu bi čovjek
intuitivno nazvao "srodnom". Površina (broj) izgleda kao mjera "koliko su slični", ali
ispod je samo statistika ko-pojave u tekstu.

---

### 02 — Semantic Search (`/search/`)

**Stack:** PGlite (Postgres u WebAssembly) + pgvector + Transformers.js
**Model:** `Xenova/all-MiniLM-L6-v2`
**Perzistencija:** IndexedDB — preživljava reload

**Što radi:**
- Lijeva strana: kolekcija fraza (ručno ili 20 seed chips)
- Svaka fraza se encodira → sprema u PGlite (`vector(384)` kolona)
- Desna strana: upiši query → SQL `ORDER BY embedding <=> query_vector LIMIT 10`
- Vraća N najbližih s cosine similarity

**arXiv corpus loader (v1.1):**
- Python skripta na Balsamu povukla 300 papira s arXiv API
- Filter: oba pojma "thought" AND "experiment" moraju biti u abstractu
- 281 papira prošlo filter → JSON na `/search/arxiv_thought_experiments.json`
- Browser učita JSON, encodira title + prvih 500 znakova abstracta, sprema u PGlite
- IndexedDB pamti → drugi put ne encodira ponovo

**Flavio's ideja o ovome:** Inspiriran čitanjem o Tatoeba/Helsinki parovima —
htio je obrnuto pretraživanje: dam vektor, dobim najbliže fraze.

---

### 03 — Translation Quality (`/translation/`)

**Flaviova ideja (precizno definisana):**
```
X  = originalna rečenica (ne-engleski)
E  = tačan engleski prijevod (Tatoeba — referenca)
M  = translation model (Helsinki-NLP opus-mt)
ME = M prevede X → engleski
MX = M prevede E → nazad na X jezik

Mjerimo:
  cos(E, ME)  — koliko dobro model prevodi X na engleski
  cos(X, MX)  — koliko dobro model prevodi E nazad na X jezik
```

**Stack:**
- Helsinki-NLP opus-mt modeli (ONNX, ~30-45MB/model, cached u browser)
- multilingual-MiniLM-L12-v2 za embeddings (cross-lingual prostor)
- 15 jezika: DE, FR, ES, IT, RU, CS, ZH, JA, NL, PL, UK, AR, FI, HU, KO
- Tatoeba primjeri ugrađeni (Descartes, Goethe, kineski klasici, japanske izreke...)

**Rezultati koje je Flavio dobio:**
- DE, FR, IT, ES: perfektni (opus-mt odličan za te parove)
- RU, CS: nisu tako dobri
- ZH, JA: katastrofa (ONNX modeli za te jezike slabi, tokenizacija kompleksna)

**Ograničenje:** BHS (srpski/hrvatski) nema direktnog Xenova modela. Potencijalno
rješenje za sljedeću sesiju: koristiti `cs` kao aproksimaciju ili external LLM API.

---

## Dogovoreno za nastavak

1. **Translation experiment** — ostajemo ovdje, nastavljamo sutra
2. Ideje za proširenje:
   - Automatski loader Tatoeba parova (CSV ili API)
   - Usporedba više modela za isti par (Helsinki vs nllb-200 vs neki LLM API)
   - BHS podrška — pronaći rješenje
   - Više jezika, više parova, statistike po jeziku
3. Sandbox landing page ima još placeholdere: Briscola, Pong, Genetic Algorithms

## Sandbox struktura

```
sandbox.opik.net/
├── index.html              — landing page
├── embeddings/
│   └── index.html          — v2.0
├── search/
│   ├── index.html          — v1.1
│   └── arxiv_thought_experiments.json  — 281 papira
└── translation/
    └── index.html          — v1.0
```

## Tehnički detalji

- `.htaccess`: `Cache-Control: no-cache` na HTML fajlovima
- Sve JS u `<script type="module">` — event listeneri umjesto onclick atributa
- PGlite: `idb://sandbox-search-v1` (IndexedDB namespace)
- Git: `/home/balsam/sandbox/`, deploy manual copy iz web roota

---

*Session dokumentovao Claude, 28. april 2026.*
