# Consultas sobre Candidatos

## MCP: `infovoto-perfiles`

### Buscar candidato por DNI

**Tool:** `buscar_candidato_por_dni`

```
¿Quién es el candidato con DNI 12345678?
Búscame información sobre el postulante con DNI 87654321
```

**Retorna:** nombre completo, cargo (presidente/senador/diputado/parlamento andino),
partido político, región/circunscripción, posición en lista, URL de foto.

---

### Listar candidatos por región y cargo

**Tool:** `listar_candidatos_region`

```
¿Quiénes son los candidatos a diputados por Cusco?
Lista los senadores del partido Fuerza Popular
¿Quiénes postulan a la presidencia?
```

**Parámetros:**
- `region`: nombre de la región (ej: "Lima", "Arequipa", "Junín") o "Distrito Único" para presidenciales
- `cargo` (opcional): presidente, vicepresidente_1, vicepresidente_2, senador, diputado, parlamento_andino
- `partido` (opcional): nombre del partido para filtrar

---

### Verificar antecedentes penales

**Tool:** `verificar_antecedentes`

```
¿Tiene antecedentes penales el candidato con DNI 12345678?
¿El postulante X tiene sentencias declaradas?
```

**Fuente:** Declaración Jurada de Hoja de Vida (DJHV) presentada ante el JNE.
Los datos son los que el propio candidato declaró — no es una consulta al INPE.

---

## Cargos disponibles

| Cargo | Descripción |
|-------|-------------|
| `presidente` | Presidente de la República |
| `vicepresidente_1` | Primer vicepresidente |
| `vicepresidente_2` | Segundo vicepresidente |
| `senador` | Senador nacional o regional |
| `diputado` | Diputado por región |
| `parlamento_andino` | Representante al Parlamento Andino |

## Regiones disponibles (26)

Amazonas, Áncash, Apurímac, Arequipa, Ayacucho, Cajamarca, Callao, Cusco,
Huancavelica, Huánuco, Ica, Junín, La Libertad, Lambayeque, Lima,
Lima Metropolitana, Lima Provincias, Loreto, Madre de Dios, Moquegua,
Pasco, Piura, Puno, San Martín, Tacna, Tumbes, Ucayali.
