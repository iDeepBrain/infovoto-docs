# Seguridad Git — Protección de Credenciales

Documento de referencia para todos los repos de InfoVoto.
**Regla de oro: ningún secreto real llega jamás a GitHub.**

---

## Las 5 capas de protección

```
INTENTO DE COMMIT
       │
       ▼
┌─────────────────────────────────────────────────┐
│ 1. .gitignore                                   │
│    .env, *.pem, *.key, *credentials*.json       │
│    git add ni los ve — nunca entran al staging  │
└─────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│ 2. pre-commit: detect-private-key               │
│    Bloquea llaves SSH/SSL/PEM en el diff        │
└─────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│ 3. pre-commit: gitleaks + .gitleaks.toml        │
│    Escanea TODO el diff con 100+ patrones       │
│    (AWS, GCP, Google API keys, tokens, JWTs...) │
└─────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│ 4. pre-commit: check-added-large-files          │
│    Bloquea archivos >500KB (PDFs, datasets)     │
└─────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│ 5. .claudeignore + CLAUDE.md + MEMORY.md        │
│    Claude nunca lee ni sugiere commitear .env   │
└─────────────────────────────────────────────────┘
```

---

## Archivos que NUNCA se commitean

| Tipo | Patrón | Motivo |
|------|--------|--------|
| Variables de entorno | `.env`, `.env.*` | Contienen credenciales reales |
| Llaves privadas | `*.pem`, `*.key`, `*.p12` | Acceso a servicios |
| Credenciales GCP | `*credentials*.json`, `*service_account*.json` | Acceso a GCP |
| Datos electorales | `*.pdf`, `*.csv`, `*.xlsx`, `*.parquet` | Peso + datos sensibles |
| Bases de datos locales | `*.db`, `*.sqlite` | Datos locales |

**Siempre commitear solo:** `.env.example` con valores placeholder como `your-key-here`.

---

## Archivos de configuración por repo

| Archivo | Dónde vive | Qué hace |
|---------|-----------|----------|
| `.gitignore` | cada repo | Impide que `git add` tome secretos |
| `.pre-commit-config.yaml` | cada repo | Define los hooks que corren antes de cada commit |
| `.gitleaks.toml` | cada repo | Configuración de gitleaks con patrones del stack |
| `.claudeignore` | raíz del proyecto | Le dice a Claude qué nunca leer ni commitear |

---

## Hooks pre-commit activos

```yaml
# Repos Python (gateway, mcp, scraper)
- gitleaks          # detecta secretos hardcodeados (100+ patrones)
- detect-private-key # bloquea llaves SSH/SSL
- check-added-large-files --maxkb=500  # bloquea PDFs, datasets
- check-merge-conflict
- end-of-file-fixer
- trailing-whitespace
- ruff --fix --line-length=120   # linter Python
- ruff-format --line-length=120  # formatter Python

# Repos no-Python (web, infra, planning, docs)
# mismos hooks excepto ruff
```

### Instalar hooks en un repo nuevo

```bash
cd infovoto-<nombre>
cp ../infovoto-gateway/.pre-commit-config.yaml .   # o el template que corresponda
cp ../infovoto-gateway/.gitleaks.toml .
pre-commit install
pre-commit install-hooks
```

### Verificar que todo pasa

```bash
pre-commit run --all-files
```

---

## Qué hacer si un secreto llegó a git por error

1. **No hacer push** — si aún no se pusheó, el daño es local
2. Rotar el secreto inmediatamente en el servicio (GCP, Supabase, etc.)
3. Limpiar el historial con `git filter-repo`:

```bash
pip install git-filter-repo
git filter-repo --path .env --invert-paths
git push origin main --force
```

4. Si ya se pusheó a GitHub, además:
   - Usar la herramienta de GitHub: *Settings → Secret scanning → Remediate*
   - Notificar al equipo para que hagan `git pull --rebase`

> **Nota:** `git rm --cached .env` + commit NO elimina el secreto del historial. Solo `git filter-repo` lo borra definitivamente.

---

## Verificar historial limpio

```bash
# Ver si algún .env fue commiteado alguna vez
git log --all --full-history -- "*.env" ".env"

# Buscar patrones de API keys en todo el historial
git log -p --all | grep -E "AIza|sk-|Bearer [a-zA-Z0-9]{20}" | grep "^\+"
```

---

## Stack de secretos en producción

| Secreto | Dónde vive en prod | Nunca en |
|---------|-------------------|----------|
| `GOOGLE_API_KEY` | GCP Secret Manager | código, .env commiteado |
| `DATABASE_URL` | GCP Secret Manager | código, .env commiteado |
| `NEXTAUTH_SECRET` | Vercel env vars | código, .env commiteado |
| `AZURE_KEY` | GCP Secret Manager | código, .env commiteado |
| Credenciales Supabase | GCP Secret Manager | código, .env commiteado |

En local: solo `.env` (no commiteado) copiado de `.env.example`.

---

## Variables de entorno — modelo de dos archivos

Desde marzo 2026, el proyecto usa dos archivos separados en lugar de un `.env` monolítico:

- **`.env.secrets`** — credenciales de pago (Gemini, Azure, OAuth, WhatsApp). Claude y los scripts nunca lo tocan.
- **`.env.config`** — configuración local (URLs, puertos, flags). Scripts pueden gestionar keys automáticamente.

Ver documentación completa: [variables_de_entorno.md](variables_de_entorno.md)

### Pre-commit: qué detecta gitleaks en `.env.secrets`

Si alguien intenta commitear `.env.secrets` con claves reales, gitleaks detecta:
- Patrones `AIza...` (Google API keys)
- Patrones de Azure Cognitive Services keys
- AWS, GitHub tokens, Slack webhooks (reglas base de gitleaks)
- Claves privadas PEM/RSA

El commit queda bloqueado antes de llegar a GitHub.
