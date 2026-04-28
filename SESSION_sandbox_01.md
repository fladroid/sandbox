# SESSION_sandbox_01 — 28. april 2026.

## Kontekst

Flavio je otvorio novu temu: **Transformers.js** — fasciniran demo primjerima na HuggingFace.
Odluka: napraviti zaseban sandbox (`sandbox.opik.net`) kao pravo igralište, nezavisno od
thelab-a, X-Ray knjige i svih ostalih projekata. Bez redoslijeda, smisla, koristi — čista
eksploracija.

Protokol sesije: PROTOKOL_v01.md + Poglavlje 5 — kolegijalni ton, "ti" forma, bez asistent moda.

---

## Infrastruktura

### sandbox.opik.net — novo postavljanje

**Flavio (sudo):** mkdir, vhost conf, a2ensite, certbot — TLS aktivan

**Claude:** generisao HTTP vhost conf, kreirao sve HTML stranice na Balsamu, git operacije.

**GitHub:** `fladroid/sandbox`
**Web root:** `/var/www/vhosts/sandbox/html/`
**Balsam local:** `/home/balsam/sandbox/`
**Cache:** `.htaccess` → `Cache-Control: no-cache` na HTML fajlovima

---

## Eksperimenti

### 01 — Embeddings (`sandbox.opik.net/embeddings/`) — v2.0

**Model:** `Xenova/all-MiniLM-L6-v2` (384 dims, ~23MB)

Upiši dvije fraze → cosine similarity + heat mapa vektora (384 celije 10x10px) +
difference vizualizacija (siva skala) + statistike (mean, max, min, mean D, max D, top dims).

Model switcher: all-MiniLM-L6-v2 (EN) vs multilingual-MiniLM-L12-v2 (50 jezika).
Bug fix: onclick ne radi u script type=module — koristiti event listeners.

**Nalazi:**

| Par | MiniLM | Multilingual |
|-----|--------|--------------|
| king / queen (EN) | 0.685 | 0.679 |
| koenig / koenigin (DE) | 0.881 | 0.946 |
| kralj / kraljica (BHS) | 0.678 | 0.725 |
| re / regina (IT) | 0.230 | 0.528 |
| il re / la regina (IT) | — | 0.746 |
| man/woman = muskarac/zena | 0.414 | 0.414 |
| muz/zena | 0.497 | — |
| marito/moglie (IT) | 0.791 | — |
| Ehemann/Ehefrau (DE) | 0.383 | — |

X-Ray zapazanje: Cosine similarity mjeri distribucijsku ko-pojavu u corpus-u, ne
semanticku srodnost. Bracni par koji uvijek ide zajedno = visok sim. Opozicijski par
u kontrastivnim recenicama = nizak sim, cak i kad su konceptualno srodne stvari.

---

### 02 — Semantic Search (`sandbox.opik.net/search/`) — v1.1

Stack: PGlite (Postgres u WebAssembly) + pgvector + Transformers.js
Perzistencija: IndexedDB (prezivljava reload).

Lijeva strana: kolekcija fraza → encodira → PGlite vector(384).
Desna strana: query → SQL ORDER BY embedding <=> query LIMIT 10.

arXiv corpus: 281 papira (thought AND experiment u abstractu), JSON na serveru,
browser encodira i cuva u IndexedDB — jednom, ne ponavlja.

PGlite: operator <=> vraca distance (0=identicno). Similarity = 1 - distance.

---

### 03 — Translation Quality (`sandbox.opik.net/translation/`) — v1.0

Flaviova ideja:

```
X   = originalna recenica (ne-engleski)
E   = tacan engleski prijevod (Tatoeba — referenca)
M   = Helsinki-NLP opus-mt
ME  = M prevede X → engleski
MX  = M prevede E → nazad na X jezik

Mjerimo: cos(E, ME) i cos(X, MX)
```

15 jezika: DE, FR, ES, IT, RU, CS, ZH, JA, NL, PL, UK, AR, FI, HU, KO.
Embedding model: multilingual-MiniLM-L12-v2 (isti prostor za sve jezike).

Rezultati:
- DE, FR, IT, ES: perfektni
- RU, CS: nisu tako dobri  
- ZH, JA: katastrofa (ONNX modeli slabi za azijske skripte)
- BHS/SR/HR: nema Xenova modela — otvoreno pitanje

---

## Sandbox struktura (kraj sesije)

```
sandbox.opik.net/
├── .htaccess
├── index.html                             — landing (3 live, 2 coming soon)
├── embeddings/index.html                  — v2.0
├── search/index.html                      — v1.1
├── search/arxiv_thought_experiments.json  — 281 papira
└── translation/index.html                 — v1.0
```

---

## Plan za nastavak

1. Translation experiment — nastavljamo, Flavio daje Tatoeba parove
2. BHS podrška: cs aproksimacija ili external LLM API
3. Vise modela za usporedbu: Helsinki vs nllb-200-distilled-600M (200 jezika)
4. Automatski Tatoeba loader (CSV)
5. Statistike po jeziku
6. Buducnost: Briscola, Pong, Genetic Algorithms

---

*Sesija: 28. april 2026. | Claude Sonnet 4.6 | Sandbox projekt*
