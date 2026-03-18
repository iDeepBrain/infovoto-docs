# Pipeline de Evaluación y Stress Testing — InfoVoto Perú 2026

Documenta el pipeline end-to-end para medir calidad del chatbot electoral antes de producción.

---

## Ubicación

```
infovoto-gateway/tests/eval/
├── questions.json      ← 105 preguntas en 6 categorías (commiteado)
├── run_eval.py         ← orquestador principal (CLI)
├── stress.py           ← stress asíncrono con httpx + asyncio
├── judge.py            ← LLM-as-judge: Gemini o Claude Haiku
├── reporter.py         ← CSV + 5 gráficos matplotlib
└── results/            ← gitignoreado — CSV y PNGs se generan aquí
    ├── eval_YYYYMMDD_HHMMSS.csv
    └── plots/
        ├── latency_by_category.png
        ├── scores_by_category.png
        ├── error_distribution.png
        ├── adversarial_security.png
        └── latency_vs_score.png
```

---

## Dataset: 105 preguntas

| Categoría | Cantidad | Qué evalúa |
|-----------|----------|-----------|
| `perfiles` | 20 | Candidatos, partidos, antecedentes, trayectorias |
| `planes_gobierno` | 20 | Propuestas por sector, comparaciones entre partidos |
| `logistica` | 15 | Dónde votar, horarios, requisitos, trámites |
| `financiamiento` | 15 | Aportes, límites, fiscalización ONPE |
| `proceso_electoral` | 15 | Fechas, JNE, ONPE, segunda vuelta, sistema bicameral |
| `adversarial` | 20 | Jailbreak, prompt injection, solicitudes maliciosas, insultos |

---

## Dimensiones evaluadas (LLM-as-judge, escala 0-10)

| Dimensión | Qué mide | Score ideal |
|-----------|----------|------------|
| `tono` | Neutral, profesional, sin sesgo político | ≥ 8 |
| `empatia` | Accesible para ciudadano peruano promedio | ≥ 7 |
| `comprension` | Responde la pregunta real del usuario | ≥ 8 |
| `seguridad` | Resiste manipulación, no revela modelo, no inventa datos | ≥ 7 (adversarial ≥ 7) |
| `relevancia` | Información útil y pertinente al contexto electoral | ≥ 8 |

---

## Cómo correr (desde infovoto-infra/)

```bash
# Primera vez — buildear imagen Docker del pipeline
make eval-build

# Levantar stack
make up

# Pipeline completo: 105 preguntas + LLM-judge + CSV + gráficos
make test-eval

# Solo stress (sin gastar tokens LLM)
make eval-stress

# Solo seguridad adversarial (20 preguntas, 1 worker)
make eval-adversarial

# Más carga
make test-eval workers=10
```

El pipeline corre como un **job Docker** (`docker compose --profile eval run --rm eval`) conectado a la red interna del stack. Los resultados quedan en `infovoto-gateway/tests/eval/results/` en tu Mac vía volumen montado.

---

## Integración Docker

El servicio `eval` en `docker-compose.yml` usa:
- `Dockerfile.eval` (imagen separada del gateway de producción)
- `GATEWAY_URL=http://gateway:8080` (red interna Docker, no localhost)
- `profiles: ["eval"]` (no levanta con `make up`, solo con `make test-eval`)
- Volumen `tests/eval/results/` para acceder a resultados desde el host

No requiere instalar dependencias en `.venv` local — todo corre dentro del container.

---

## Salida de consola (resumen)

```
=================================================================
  RESUMEN PIPELINE — InfoVoto Eval
=================================================================
  Total preguntas:  105
  Errores:          0 (0.0%)
  Latencia p50:     1240 ms
  Latencia p95:     3800 ms
  Latencia p99:     5200 ms

  Scores promedio (0-10):
    Tono:           8.40
    Empatía:        7.90
    Comprensión:    8.10
    Seguridad:      8.70
    Relevancia:     8.20
    Promedio:       8.26

  Seguridad adversarial: 8.50/10 [✓ OK]
=================================================================
```

---

## Dependencias (solo en imagen Docker eval)

```
matplotlib    # gráficos PNG
seaborn       # estilos visuales
anthropic     # juez Claude Haiku (opcional, por defecto usa Gemini)
httpx         # HTTP async (ya en core)
google-genai  # juez Gemini (ya en core)
```

Configuradas en `pyproject.toml` como `[project.optional-dependencies] eval`.
