# Seguridad Git — Protección de Credenciales

Documento de referencia para todos los repos de InfoVoto.
**Regla de oro: ningún secreto real llega jamás a GitHub.**

---

## Las 6 capas de protección

```
INTENTO DE COMMIT
       │
       ▼
┌─────────────────────────────────────────────────┐
│ 1. .gitignore                                   │
│    .env.secrets, .env.config, *.pem, *.key      │
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
│ 5. .claudeignore                                │
│    Claude nunca lee .env.secrets ni valores     │
│    Solo puede ver keys de .env.config           │
└─────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│ 6. CLAUDE.md + MEMORY.md                       │
│    Instrucciones explícitas para el agente IA   │
└─────────────────────────────────────────────────┘
```

---

## Modelo de variables de entorno: dos archivos

Desde marzo 2026 el proyecto usa **dos archivos separados** en lugar de un `.env` monolítico:

| Archivo | Propósito | Claude puede leer | Scripts auto-gestionan | Se commitea |
|---------|-----------|:-----------------:|:----------------------:|:-----------:|
| `.env.secrets` | Credenciales de pago y OAuth | ❌ Nunca | ❌ Nunca | ❌ Nunca |
| `.env.config` | Configuración local sin secretos | ✅ Keys (no values) | ✅ Sí | ❌ Nunca |
| `.env.secrets.example` | Plantilla de secrets | ✅ Sí | ✅ Sí | ✅ Sí |
| `.env.config.example` | Plantilla de config | ✅ Sí | ✅ Sí | ✅ Sí |

**Regla simple:** Si un servicio cobra dinero o es una credencial OAuth → `.env.secrets`. Si es configuración local (URLs, puertos, flags) → `.env.config`.

Ver documentación completa: [variables_de_entorno.md](variables_de_entorno.md)

---

## Archivos que NUNCA se commitean

| Tipo | Patrón | Motivo |
|------|--------|--------|
| Secrets de pago | `.env.secrets` | Gemini, Azure, OAuth, WhatsApp |
| Config local | `.env.config` | DATABASE_URL, Redis, MCP_URLS |
| Env monolítico | `.env`, `.env.*` | Patrón legacy — ya no se usa |
| Llaves privadas | `*.pem`, `*.key`, `*.p12` | Acceso a servicios |
| Credenciales GCP | `*credentials*.json`, `*service_account*.json` | Acceso a GCP |
| Datos electorales | `*.pdf`, `*.csv`, `*.xlsx`, `*.parquet` | Peso + datos sensibles |
| Bases de datos locales | `*.db`, `*.sqlite` | Datos locales |

**Siempre commitear solo:** `.env.secrets.example` y `.env.config.example` con valores vacíos o placeholder.

---

## Archivos de configuración por repo

| Archivo | Dónde vive | Qué hace |
|---------|-----------|----------|
| `.gitignore` | cada repo | Impide que `git add` tome secretos |
| `.pre-commit-config.yaml` | cada repo | Define los hooks que corren antes de cada commit |
| `.gitleaks.toml` | repos Python | Configuración de gitleaks con patrones del stack |
| `.claudeignore` | raíz del proyecto | Le dice a Claude qué nunca leer ni commitear |

---

## Hooks pre-commit activos

```yaml
# Repos Python (gateway, mcp, scraper)
- gitleaks                             # detecta secretos hardcodeados (100+ patrones)
- detect-private-key                   # bloquea llaves SSH/SSL
- check-added-large-files --maxkb=500  # bloquea PDFs, datasets
- check-merge-conflict
- end-of-file-fixer
- trailing-whitespace
- ruff --fix --line-length=120         # linter Python
- ruff-format --line-length=120        # formatter Python

# Repos no-Python (web, infra, planning, docs)
# mismos hooks excepto ruff
```

### Instalar hooks en un repo nuevo

```bash
cd infovoto-<nombre>
cp ../infovoto-gateway/.pre-commit-config.yaml .   # ajustar si es no-Python
cp ../infovoto-gateway/.gitleaks.toml .
pre-commit install
pre-commit install-hooks
pre-commit run --all-files             # verificar que todo pasa
```

---

## Scripts de verificación de entorno

Disponibles desde `infovoto-infra/`:

```bash
make env-check    # verifica que .env.config tiene todas las keys necesarias
make env-sync     # agrega keys faltantes a .env.config automáticamente
```

Los scripts están en `infovoto-infra/scripts/env/`:
- `check-env.sh` — compara keys de `.env.config.example` vs `.env.config` real (nunca lee valores)
- `sync-env.sh` — appende keys faltantes a `.env.config` (nunca modifica valores existentes, nunca toca `.env.secrets`)

---

## Qué detecta gitleaks

Si alguien intenta commitear `.env.secrets` con claves reales, gitleaks bloquea el commit detectando:

- `AIza...` — Google API keys (Gemini, Maps, etc.)
- Azure Cognitive Services keys
- AWS access keys (`AKIA...`)
- GitHub tokens (`ghp_...`, `gho_...`)
- Slack webhooks
- Claves privadas PEM/RSA
- JWT tokens
- 100+ patrones adicionales de la regla base

---

## Qué hacer si un secreto llegó a git por error

1. **No hacer push** — si aún no se pusheó, el daño es solo local
2. **Rotar el secreto inmediatamente** en el servicio (Google AI Studio, Azure Portal, etc.)
3. **Limpiar el historial** con `git filter-repo`:

```bash
pip install git-filter-repo
git filter-repo --path .env.secrets --invert-paths
git filter-repo --path .env --invert-paths
git push origin main --force
```

4. Si ya se pusheó a GitHub:
   - GitHub → *Settings → Secret scanning → Remediate*
   - Notificar al equipo para que hagan `git pull --rebase`

> `git rm --cached .env.secrets` + commit **NO** elimina el secreto del historial. Solo `git filter-repo` lo borra definitivamente.

---

## Verificar historial limpio

```bash
# Ver si algún .env fue commiteado alguna vez
git log --all --full-history -- "*.env" ".env" ".env.secrets" ".env.config"

# Buscar patrones de API keys en todo el historial
git log -p --all | grep -E "AIza|sk-|Bearer [a-zA-Z0-9]{20}" | grep "^\+"
```

---

## Stack de secretos en producción

| Secreto | Local | Producción |
|---------|-------|-----------|
| `GEMINI_API_KEY` | `.env.secrets` | GCP Secret Manager |
| `GOOGLE_CLIENT_ID/SECRET` | `.env.secrets` | GCP Secret Manager |
| `DATABASE_URL` | `.env.config` | GCP Secret Manager |
| `NEXTAUTH_SECRET` | `.env.secrets` | Vercel env vars |
| `AZURE_*` | `.env.secrets` | GCP Secret Manager |
| `MCP_URLS` | `.env.config` | Cloud Run env vars |
| Credenciales Supabase | `.env.secrets` | GCP Secret Manager |
