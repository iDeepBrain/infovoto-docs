# Variables de Entorno — InfoVoto

## Arquitectura: dos archivos separados

| Archivo | Propósito | Claude puede leer | Scripts auto-gestionan |
|---------|-----------|:-----------------:|:----------------------:|
| `.env.secrets` | Credenciales de pago y servicios externos | ❌ Nunca | ❌ Nunca |
| `.env.config` | Configuración local sin secretos | ✅ Keys (no values) | ✅ Sí |
| `.env.secrets.example` | Plantilla de secrets (commiteable) | ✅ Sí | ✅ Sí |
| `.env.config.example` | Plantilla de config (commiteable) | ✅ Sí | ✅ Sí |

**Regla simple:** Si un servicio cobra dinero o es una credencial OAuth → `.env.secrets`. Si es configuración local (URLs, puertos, flags) → `.env.config`.

---

## .env.secrets — qué va aquí

```bash
# Google AI (Gemini) — de pago
GEMINI_API_KEY=

# Google OAuth (login web)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Azure Document Intelligence (OCR de PDFs) — de pago
AZURE_DOCUMENT_INTELLIGENCE_ENDPOINT=
AZURE_DOCUMENT_INTELLIGENCE_KEY=

# NextAuth (firma de JWT)
NEXTAUTH_SECRET=

# WhatsApp Business API (Fase 4) — de pago
WHATSAPP_VERIFY_TOKEN=
WHATSAPP_APP_SECRET=
WHATSAPP_ACCESS_TOKEN=
WHATSAPP_PHONE_NUMBER_ID=

# Admin
ADMIN_TOKEN=
```

## .env.config — qué va aquí

```bash
# App
ENVIRONMENT=development
LOG_LEVEL=DEBUG
GEMINI_MODEL=gemini-2.0-flash

# Base de datos local
DATABASE_URL=postgresql+asyncpg://infovoto:localdev@postgres:5432/infovoto

# Redis local
REDIS_URL=redis://redis:6379/0

# MCPs activos (gateway carga estos al iniciar)
# Solo demo por defecto. Agregar ,http://infovoto-mcp:8080/planes cuando SC-P1 esté listo.
MCP_URLS=http://infovoto-mcp:8080/demo

# NextAuth URL
NEXTAUTH_URL=http://localhost:2300

# ChromaDB
CHROMA_PERSIST_DIR=./data/chroma
```

---

## Cómo usar

```bash
# Primera vez (setup)
cp .env.secrets.example .env.secrets   # completar con keys reales
cp .env.config.example .env.config     # defaults ya correctos

# Verificar que .env.config tiene todas las keys
cd infovoto-infra && make env-check

# Agregar keys faltantes automáticamente a .env.config
make env-sync

# Activar MCP de planes cuando los datos estén cargados (SC-P1)
# Editar .env.config:
MCP_URLS=http://infovoto-mcp:8080/demo,http://infovoto-mcp:8080/planes
# Luego:
docker compose restart gateway
```

---

## Docker Compose

Los tres servicios que necesitan credenciales cargan ambos archivos:

```yaml
env_file:
  - ../.env.secrets   # credenciales reales
  - ../.env.config    # configuración local
```

Servicios: `gateway`, `infovoto-mcp`, `scraper`.
El servicio `web` usa `infovoto-web/.env.local` (NextAuth convention).

---

## 5 capas de protección contra commit accidental

| Capa | .env.secrets | .env.config |
|------|:---:|:---:|
| `.gitignore` (todos los repos) | ❌ bloqueado | ❌ bloqueado |
| `.claudeignore` (raíz) | ❌ bloqueado | ✅ keys visibles |
| `gitleaks` pre-commit | ❌ escanea contenido | ❌ escanea contenido |
| `detect-private-key` pre-commit | ❌ detecta | ❌ detecta |
| `check-env.sh` / `sync-env.sh` | ❌ nunca toca | ✅ gestiona keys |

Si intentas hacer `git add .env.secrets`, gitleaks bloquea el commit antes de que llegue a GitHub.

---

## Scripts disponibles (desde infovoto-infra/)

```bash
make env-check   # verifica que .env.config tiene todas las keys necesarias
make env-sync    # agrega keys faltantes a .env.config automáticamente
```

Los scripts están en `infovoto-infra/scripts/env/`:
- `check-env.sh` — compara keys de `.env.config.example` vs `.env.config` real
- `sync-env.sh` — appende keys faltantes (nunca modifica valores existentes)
