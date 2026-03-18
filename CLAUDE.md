# infovoto-docs

Documentación global del sistema InfoVoto. No se despliega.

## Estructura actual

```
docs/
├── deepresearch/       ← Research_01, Research_02, Debates_Arquitectura,
│   │                     Plan_Arquitectura_Tecnica, PMBOK, diagramas SVG/PNG
│   └── Images/
├── entendimiento/      ← estructura_de_elecciones.md (contexto electoral peruano)
├── technical/          ← arquitectura cross-repo, ADRs, runbooks
├── contracts/          ← contratos de interoperabilidad entre repos
└── user/               ← guías para usuarios finales (consultas_candidatos, planes, logistica)
```

## Regla
Docs sobre cómo funciona un repo específico → van en `<repo>/docs/`.
Docs sobre dominio electoral, arquitectura global o UX del sistema → van aquí.

## Git (CRÍTICO)

- SSH: `github.com-personal` (NUNCA `github.com` — esa es de marvik)
- User: `CristianLazoQuispe` / Email: `mecatronico.lazo@gmail.com`
- Org: `iDeepBrain`

### Configurar en repo nuevo
```bash
git config user.name "CristianLazoQuispe"
git config user.email "mecatronico.lazo@gmail.com"
```

### Scripts globales (desde infovoto-infra/)
```bash
make git-status              # estado de todos los repos
make git-commit m="mensaje"  # commitea y pushea todos los repos con cambios
```

## Reglas .gitignore (NUNCA commitear)

- Credenciales: `.env`, `*.pem`, `*.key`, `*.p12`, `*credentials*.json`, `*service_account*.json`
- Datos/binarios: `*.pdf`, `*.csv`, `*.xlsx`, `*.xls`, `*.parquet`, `*.db`, `*.sqlite`
- Solo commitear `.env.example` (sin valores reales)
