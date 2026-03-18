# Arquitectura del Scraper — InfoVoto

## Filosofía

El scraper es un **job local** — no se despliega en GCP. Su única responsabilidad es llenar las bases de datos que luego los MCPs leen.

```
Scraper (local)  →  PostgreSQL + ChromaDB  →  MCPs (Cloud Run)  →  Gateway  →  Usuario
```

**Principio:** `raw/` y `processed/` tienen exactamente la misma estructura de carpetas. Solo cambia la extensión: `.pdf` → `.md`, `.csv` → `.md`, `.json` → `.md`.

---

## Mapa completo: dato → procesado → DB → MCP

| Fuente | raw/ | processed/ | DB | MCP |
|--------|------|------------|----|-----|
| JNE API | `jne/candidatos/*.json` | `jne/candidatos/*.md` | PostgreSQL | `perfiles` |
| JNE PDF | `jne/planes/{partido}/*.pdf` | `jne/planes/{partido}/*.md` | ChromaDB | `planes_gobierno` |
| ONPE CSV | `onpe/locales/*.csv` | `onpe/locales/*.md` | PostgreSQL | `logistica` |
| ONPE API | `onpe/financiamiento/*.json` | `onpe/financiamiento/*.md` | PostgreSQL | `financiamiento` |
| Deep Research | `deepresearch/*.md` | `deepresearch/*.md` | ChromaDB | `fiscalizacion` |
| Deep Research | `deepresearch/*.md` | `deepresearch/*.md` | ChromaDB | `proceso_electoral` |

---

## Estructura de carpetas

```
infovoto-scraper/
├── config.py                          ← Pydantic Settings
├── downloaders/                       ← solo descargan, no procesan
│   ├── jne_candidatos.py             → data/raw/jne/candidatos/
│   ├── jne_planes.py                 → data/raw/jne/planes/{partido}/
│   ├── onpe_financiamiento.py        → data/raw/onpe/financiamiento/
│   └── onpe_locales.py              → data/raw/onpe/locales/
│
├── notebooks/                         ← 1 notebook = 1 MCP = 1 fuente
│   ├── 01_candidatos.ipynb           JSON  → MD → PostgreSQL  (perfiles)
│   ├── 02_planes_ocr.ipynb           PDF   → MD → ChromaDB    (planes_gobierno) ← PRIORIDAD
│   ├── 03_locales.ipynb              CSV   → MD → PostgreSQL  (logistica)
│   ├── 04_financiamiento.ipynb       JSON  → MD → PostgreSQL  (financiamiento)
│   ├── 05_fiscalizacion.ipynb        MD    → MD → ChromaDB    (fiscalizacion)
│   └── 06_proceso_electoral.ipynb    MD    → MD → ChromaDB    (proceso_electoral)
│
└── data/
    ├── raw/                           ← datos crudos (nunca commitear)
    │   ├── jne/
    │   │   ├── candidatos/           ← 3 JSONs de API JNE
    │   │   └── planes/               ← 33 PDFs ✅ disponibles
    │   ├── onpe/
    │   └── deepresearch/             ← MDs de Gemini Deep Research
    └── processed/                    ← misma estructura que raw/
        └── (generado por notebooks)
```

---

## ChromaDB — compartir datos entre scraper y MCP

El scraper escribe en `infovoto-scraper/data/chroma/`. El MCP en docker lee el mismo directorio montado como volumen:

```yaml
# docker-compose.yml
infovoto-mcp:
  volumes:
    - ../infovoto-scraper/data:/app/scraper_data:ro
  environment:
    - CHROMA_PERSIST_DIR=/app/scraper_data/chroma
```

---

## Flujo de trabajo

```bash
# 1. Descargar datos
cd infovoto-scraper
python downloaders/jne_candidatos.py
python downloaders/jne_planes.py       # o colocar PDFs manualmente
python downloaders/onpe_financiamiento.py
python downloaders/onpe_locales.py
# + colocar MDs de Gemini Deep Research en data/raw/deepresearch/

# 2. Procesar y cargar (en orden)
jupyter notebook notebooks/
# Ejecutar: 02 → 01 → 03 → 04 → 05 → 06
# (02 primero porque ya tenemos los 33 PDFs)

# 3. Activar MCPs en gateway
# Editar .env.config → agregar MCPs disponibles a MCP_URLS
# docker compose restart gateway
```

---

## OCR: Azure vs PyMuPDF

| | Azure Document Intelligence | PyMuPDF |
|---|---|---|
| Calidad en PDFs escaneados | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Calidad en PDFs digitales | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Velocidad | ~30s/PDF | ~1s/PDF |
| Costo | ~$0.01/página | Gratis |
| Config | `AZURE_DOCUMENT_INTELLIGENCE_*` en `.env.secrets` | Sin config |

El notebook `02_planes_ocr.ipynb` usa Azure si las keys están en `.env.secrets`, PyMuPDF como fallback automático.

---

## Metadatos en ChromaDB

Los chunks de planes de gobierno se indexan con:

```python
{
    "partido": "renovacion_popular",   # nombre del directorio (slug)
    "chunk": 0,                        # índice del chunk
}
```

El MCP filtra por `partido` cuando el usuario especifica un partido.
El partido se normaliza a slug: `"Renovación Popular"` → `"renovacion_popular"`.
