# Consultas sobre Logística Electoral

## MCP: `infovoto-logistica`

---

### Consultar local de votación

**Tool:** `consultar_local_votacion`

```
¿Dónde me toca votar? Mi DNI es 12345678
¿Cuál es mi local de votación con DNI 87654321?
¿En qué mesa voto?
```

**Retorna:** nombre del colegio/local, dirección completa, pabellón, aula y número de mesa.

> Si la base de datos local no tiene el dato, se consulta directamente la API de ONPE.
> Consulta directa: https://consultaelectoral.onpe.gob.pe

---

### Consultar si fui sorteado como miembro de mesa

**Tool:** `consultar_miembro_mesa`

```
¿Soy miembro de mesa? DNI: 12345678
¿Me tocó ser miembro de mesa titular o suplente?
```

**Roles posibles:**
- **Titular:** Obligación de presentarse el día de elecciones desde las 7:00 a.m.
- **Suplente:** Debe presentarse si el titular no asiste.
- **Ninguno:** Solo votar.

> La inasistencia como miembro de mesa titular genera multa electoral.

---

### Consultar multas electorales

**Tool:** `consultar_multas`

```
¿Tengo multas electorales? DNI: 12345678
¿Debo alguna multa por no votar?
```

> Pago de multas: https://pagos.onpe.gob.pe

---

### Información general del día de elecciones

**Resource:** `elecciones://info`

```
¿Cuándo son las elecciones?
¿Qué horario tienen las mesas de votación?
¿Qué documentos necesito para votar?
¿Cómo votan los peruanos en el exterior?
```

---

## Resumen del día de elecciones

| Dato | Información |
|------|-------------|
| **Fecha** | Domingo 12 de abril de 2026 |
| **Apertura de mesas** | 7:00 a.m. |
| **Cierre de mesas** | 5:00 p.m. |
| **Documento requerido** | DNI vigente |
| **Miembros de mesa** | Se presentan a las 7:00 a.m. |

## Voto en el exterior

El voto se rige por la **dirección del DNI**. Los peruanos en el extranjero votan en
la embajada o consulado más cercano, en el horario local del país de residencia.
Más información: [gob.pe - voto exterior](https://www.gob.pe/votoenelexterior)
