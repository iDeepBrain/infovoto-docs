# Consultas sobre Planes de Gobierno

## MCP: `infovoto-planes-gobierno`

Los planes de gobierno de los 37 partidos están procesados con OCR (Azure Document Intelligence)
y vectorizados en ChromaDB para búsqueda semántica.

---

### Buscar propuesta de un partido sobre un tema

**Tool:** `buscar_propuesta_tema`

```
¿Qué propone Fuerza Popular sobre seguridad ciudadana?
¿Cuál es la propuesta económica de Alianza para el Progreso?
¿Qué dice el Partido Morado sobre educación?
¿Cómo planea reducir la corrupción el partido Renovación Popular?
```

**Ejes temáticos sugeridos:**
- Economía, empleo, inversión
- Seguridad ciudadana
- Salud pública
- Educación
- Medio ambiente / cambio climático
- Corrupción / transparencia
- Vivienda
- Infraestructura
- Agricultura
- Tecnología / digitalización

---

### Comparar propuestas entre partidos

**Tool:** `comparar_planes_gobierno`

```
Compara las propuestas de salud de APP, Fuerza Popular y Partido Morado
¿Qué diferencias hay entre Renovación Popular y Peru Libre sobre economía?
Contrasta las propuestas ambientales de 3 partidos
```

**Recomendación:** Comparar 2 a 4 partidos por consulta para respuestas más precisas.

---

### Ver qué partidos tienen planes vectorizados

**Resource:** `planes://lista`

```
¿Qué partidos tienen planes de gobierno disponibles?
Lista los partidos con planes cargados
```

---

## Notas importantes

- Los extractos provienen directamente del texto oficial del plan de gobierno.
- La búsqueda es **semántica**: no busca palabras exactas sino conceptos relacionados.
- El campo `relevancia` indica qué tan relacionado está el fragmento con tu búsqueda (0-1).
- Si un partido no tiene plan vectorizado, puede que el PDF no esté disponible aún.
