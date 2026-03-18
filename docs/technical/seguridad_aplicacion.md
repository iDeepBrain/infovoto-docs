# Seguridad de Aplicación — InfoVoto Perú 2026

Documento de referencia sobre las capas de seguridad del stack. Última actualización: marzo 2026.

---

## Modelo de capas (de red a aplicación)

```
Internet
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA 1 — Red / DNS                                          │
│ Google Cloud Armor  →  DDoS volumétrico, WAF, reglas por IP │
│ Cloud DNS + DNSSEC  →  protección DNS hijacking             │
│ Estado: ❌ PENDIENTE — configurar antes de producción       │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA 2 — TLS / HTTPS                                        │
│ Cloud Run provee TLS automático (certificado managed)        │
│ Estado: ✅ gratis, sin configuración adicional              │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA 3 — Rate limit por IP  [infovoto-gateway]              │
│ IPRateLimitMiddleware (Redis sliding window)                 │
│   - Anónimo: 60 req/min por IP                              │
│   - Autenticado: 300 req/min por IP                         │
│ Protege: /auth/verify, /api/chat, /health, webhooks         │
│ Estado: ✅ implementado (marzo 2026)                        │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA 4 — Autenticación  [infovoto-gateway]                  │
│ AuthMiddleware: Google ID token → Google sub (user_id)       │
│ Dev bypass: X-Test-User-Id (deshabilitado en producción)     │
│ Estado: ✅ implementado                                     │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA 5 — Rate limit por usuario  [infovoto-gateway]         │
│ RateLimiter: 30 req/hora por user_id (Redis sorted set)      │
│ Estado: ✅ implementado                                     │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA 6 — Guardrails LLM  [infovoto-gateway]                 │
│ llm-guard PromptInjection scanner en process_message()       │
│ Rechaza: jailbreak, prompt injection, instrucciones maliciosas│
│ Estado: ✅ implementado (parcial — solo input scanner)      │
│ Pendiente: output scanners (endorsement, blackout filter)    │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA 7 — Agente Gemini  [infovoto-gateway / src/agent/]     │
│ System prompt: instrucciones de dominio electoral            │
│ Cache por query hash (evita llamadas repetidas a Gemini)     │
│ Blackout de encuestas: output filter por fecha (pendiente)   │
│ Estado: ✅ base implementada, SE-003 pendiente              │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA 8 — Red interna VPC  [GCP]                             │
│ infovoto-mcp: solo accesible desde gateway (no público)      │
│ Postgres + Redis: dentro de la VPC, nunca expuestos          │
│ Estado: ❌ PENDIENTE — configurar VPC Connector en GCP      │
└─────────────────────────────────────────────────────────────┘
```

---

## CORS

Configurado en `src/gateway/main.py`:

```python
# Producción: solo el dominio propio
allow_origins=["https://infovoto.pe"]

# Desarrollo local: abierto
allow_origins=["*"]
```

El flag `settings.is_production` (variable `ENVIRONMENT=production`) controla el switch automáticamente.

---

## Webhook WhatsApp

- Verificación HMAC-SHA256 de cada request entrante (`src/gateway/webhook/hmac_verify.py`)
- Si la firma no coincide → 403 inmediato, sin procesar el mensaje
- El secret de firma vive en `.env.secrets` (`WHATSAPP_APP_SECRET`)

---

## Qué hace cada herramienta de seguridad

| Herramienta | Tipo | Qué protege | Dónde |
|-------------|------|-------------|-------|
| `IPRateLimitMiddleware` | Código | Flood/brute force por IP | `middleware/rate_limiter.py` |
| `RateLimiter` | Código | Abuso por usuario autenticado | `middleware/rate_limiter.py` |
| `AuthMiddleware` | Código | Acceso sin token válido | `middleware/auth.py` |
| `llm-guard` | Código | Prompt injection al LLM | `agent/core.py` |
| HMAC verify | Código | Webhooks WhatsApp falsos | `webhook/hmac_verify.py` |
| CORS | Código | Requests cross-origin no autorizados | `main.py` |
| Cloud Armor | GCP config | DDoS, WAF, reglas por país/IP | GCP Console |
| Cloud DNS DNSSEC | GCP config | DNS hijacking | GCP + registrador |
| VPC Connector | GCP config | Exposición de servicios internos | GCP Console |
| Secret Manager | GCP config | Secrets en producción (no `.env`) | GCP Console |

---

## Qué NO necesita repos nuevos ni herramientas adicionales

Toda la seguridad a nivel aplicación vive en `infovoto-gateway`. Las capas de red (Cloud Armor, VPC, DNS) son configuraciones de GCP que se hacen una sola vez desde la consola o scripts en `infovoto-infra/scripts/gcp/`.

---

## Pendientes antes de producción (ordenados por urgencia)

| Prioridad | Tarea | Tipo | Ref |
|-----------|-------|------|-----|
| P0 | Cloud Armor: DDoS + rate limit por IP a nivel red | GCP config | SE-GCP-01 |
| P0 | VPC Connector: infovoto-mcp y Postgres no públicos | GCP config | SE-GCP-02 |
| P0 | Secret Manager: reemplazar `.env.secrets` en prod | GCP config | SE-GCP-03 |
| P1 | LLM Guard output scanners (endorsement, blackout) | Código | SE-001 |
| P1 | Kill switch admin API | Código | SE-002 |
| P1 | Blackout automático encuestas (5 abril) | Código | SE-003 |
| P2 | Audit log a Cloud Logging | Código | SE-004 |
| P2 | DNSSEC en registrador de dominio | DNS config | SE-GCP-04 |

---

## Pipeline de evaluación de seguridad

El pipeline `tests/eval/` incluye 20 preguntas adversariales (`category: adversarial`) que prueban:
- Jailbreak y prompt injection
- Revelación del modelo de IA
- Solicitudes maliciosas (falsificación de votos, compra de votos, etc.)
- Insultos y abuso verbal

Se ejecuta con:
```bash
cd infovoto-infra
make eval-adversarial   # solo categoría adversarial, 1 worker
make test-eval          # pipeline completo (105 preguntas + LLM-judge)
```

El score de seguridad adversarial objetivo es **≥ 7/10 en promedio**.
