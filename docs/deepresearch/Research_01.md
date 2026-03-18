# Chatbot electoral en WhatsApp por menos de $300: guía completa

**Sí es posible, y el costo real puede ser tan bajo como $40–$70 para todo el ciclo electoral.** La clave está en un cambio de precios de WhatsApp vigente desde noviembre 2024: todas las conversaciones iniciadas por el usuario (service conversations) son **100% gratuitas e ilimitadas**. Un chatbot informativo donde los ciudadanos escriben primero para consultar sobre candidatos tiene costo de mensajería cercano a cero. Combinado con la Meta Cloud API directa (sin costo de plataforma), Google Cloud Run free tier ($0), y Gemini 2.5 Flash-Lite (~$5 por cada 10,000 conversaciones), el presupuesto de $300 alcanza para atender potencialmente **más de 500,000 conversaciones**.

El contexto electoral es extraordinariamente favorable: Perú enfrenta las elecciones más complejas de su historia el **12 de abril de 2026**, con **39 partidos en una sola cédula**, **48% de votantes indecisos**, y **2.5 millones de votantes primerizos**. No existe un chatbot público con IA que compare propuestas de candidatos — el proyecto "Revisa tu Candidato" de Transparencia solo revisa antecedentes, no políticas. Este vacío es tu oportunidad.

---

## La arquitectura de costo cero que lo hace viable

La revolución está en los cambios de precios de WhatsApp de 2024-2025. Desde el **1 de noviembre de 2024**, Meta eliminó el cobro por conversaciones de servicio (cuando el usuario escribe primero). Antes existía un límite de 1,000 conversaciones gratuitas al mes; ahora son **ilimitadas y permanentemente gratuitas**. Adicionalmente, desde el **1 de abril de 2025**, los mensajes de plantilla tipo "utility" dentro de la ventana de 24 horas también son gratuitos.

Para un chatbot electoral informativo, esto significa que el flujo principal — ciudadano escribe pregunta → chatbot responde — tiene **costo $0 en mensajería**, sin importar si son 1,000 o 50,000 conversaciones. Solo se paga cuando tú inicias el contacto con mensajes de plantilla fuera de la ventana de servicio: **$0.0703 por mensaje de marketing** o **$0.02 por mensaje de utilidad** en Perú.

| Componente | Servicio recomendado | Costo |
|---|---|---|
| API de WhatsApp | Meta Cloud API directa | **$0/mes** |
| Hosting del backend | Google Cloud Run (free tier) | **$0/mes** |
| Modelo de IA | Gemini 2.5 Flash-Lite | **~$5 por 10K conversaciones** |
| Base de datos | Supabase Free o Neon Free | **$0/mes** |
| Dominio .com | Cloudflare Registrar | **$12/año** |
| SSL/CDN | Cloudflare Free | **$0/mes** |
| Mensajería reactiva (ilimitada) | WhatsApp service conversations | **$0** |
| Broadcasts opcionales (5,000 msgs) | Utility templates a $0.02 | **$100** |
| **Total campaña 3 meses** | | **$40–$117** |

Con $300 de presupuesto, el remanente de **~$183–$260** queda como colchón para Gemini API (~$5.10 por cada 10,000 conversaciones con Flash-Lite), lo que cubre entre **360,000 y 510,000 conversaciones** adicionales. El presupuesto no es la limitante; la distribución viral sí lo es.

---

## WhatsApp Business API: la ruta directa vs los intermediarios

Existen tres opciones para acceder a WhatsApp programáticamente, y la diferencia importa enormemente para el presupuesto.

**La WhatsApp Business App** (gratuita, la que usa cualquier negocio pequeño) no sirve para este proyecto: no permite automatización, no tiene API, y los broadcasts solo llegan a contactos que guardaron tu número. **La API On-Premises** está en proceso de cierre total para octubre 2025. La **Cloud API de Meta** es la opción correcta: alojada directamente por Meta, sin costo de infraestructura, acceso directo desde developers.facebook.com, y cobro exclusivo por mensajes de plantilla enviados.

Entre los BSP (Business Solution Providers) que actúan como intermediarios, la diferencia de costos es dramática:

| Proveedor | Costo mensual | Markup por mensaje | Ideal para |
|---|---|---|---|
| **Meta Cloud API directa** | $0 | $0 | Desarrolladores que construyen todo |
| **YCloud** | $0 | $0 | Desarrolladores que quieren zero markup |
| **360dialog** | ~$39/mes | $0 (reclaman) | Devs que quieren BSP sin markup |
| **WATI** | $59–69/mes | ~20% sobre tarifas Meta | No-code, pero caro para $300 |
| **Twilio** | $0 | $0.005/mensaje adicional | Documentación excelente, markup moderado |
| **Ultramsg** | ~$39/mes | N/A (API no oficial) | **NO RECOMENDADO — riesgo de baneo** |

La recomendación clara: **Meta Cloud API directa o YCloud**. Ambos tienen cero markup. La Cloud API requiere construir tu propio webhook server (lo cual harías de todas formas con Claude Code), pero eso es exactamente lo que un desarrollador independiente necesita.

**Sobre el riesgo de baneo con APIs no oficiales**: esto **no es marketing de miedo**, es un riesgo real y documentado. Proveedores como Ultramsg, Wassenger (modo QR), y Green-API conectan mediante WhatsApp Web scraping, violando los términos de servicio de Meta. WhatsApp banea ~2 millones de cuentas por mes por mensajería no solicitada, y el riesgo se multiplica con contenido electoral (tema sensible que Meta monitorea activamente). Si tu número es baneado a una semana de las elecciones, pierdes todo el canal. Un repositorio público en GitHub lista específicamente a estos proveedores como blacklisted. Para un proyecto cívico con reputación en juego, la API oficial es la única opción sensata.

---

## Gemini 2.5 Flash es absurdamente barato para este caso

Los costos de inferencia con modelos de Google hacen que el componente de IA sea casi insignificante dentro del presupuesto total. **Gemini 2.5 Flash** cuesta $0.30 por millón de tokens de entrada y $2.50 por millón de tokens de salida. Su variante **Flash-Lite** baja a $0.10 y $0.40 respectivamente — efectivamente **5 veces más barato**.

Para 10,000 conversaciones (asumiendo 3 interacciones por conversación, 500 tokens de input y 300 de output por interacción):

- **Gemini 2.5 Flash**: $4.50 (input) + $22.50 (output) = **$27 total**
- **Gemini 2.5 Flash-Lite**: $1.50 (input) + $3.60 (output) = **$5.10 total**

La recomendación es usar **Flash-Lite para el 90% de consultas** (preguntas simples sobre candidatos, ubicación de mesas de votación) y reservar Flash completo para consultas complejas que requieran razonamiento más profundo. Con esta estrategia, 50,000 conversaciones cuestan aproximadamente **$25–$135** en API de Gemini.

**Advertencia sobre el free tier de Gemini**: Google redujo significativamente los límites gratuitos en diciembre 2025. Flash quedó en ~250 requests por día (algunos reportan hasta 20 RPD). Esto **no es viable para producción** — necesitas el paid tier (Tier 1), que solo requiere habilitar billing y ofrece 2,000 RPM y 10,000 RPD. El costo es mínimo con los precios por token mencionados.

**Google Cloud Run free tier** es la pieza complementaria ideal: **2 millones de requests al mes**, 180,000 vCPU-seconds, escala a cero cuando no hay tráfico, y auto-escala a miles de usuarios concurrentes. Es suficiente para un chatbot electoral sin pagar un centavo en infraestructura. Las alternativas (Render, Railway, Fly.io) son inferiores: Render tiene cold starts de 15+ minutos, Railway solo da $5 de crédito trial, y Fly.io eliminó su free tier en 2024.

**¿Jetson Nano Orin local?** No tiene sentido para este proyecto. Cuesta **$249** (83% del presupuesto) para algo que Cloud Run hace gratis. El Jetson solo haría HTTP calls a la API de Gemini — cualquier servidor gratuito en la nube hace lo mismo con mejor uptime, escalabilidad y sin dependencia de tu conexión de internet doméstica.

---

## Telegram y web: canales complementarios, no sustitutos

**Telegram tiene penetración de ~3–5% en Perú** (~839,000 usuarios activos) contra el ~90% de WhatsApp (~20.9 millones). Los peruanos abren WhatsApp unas **1,350 veces al mes** — es una utilidad básica, no una app. Telegram simplemente no puede ser el canal principal para alcanzar votantes masivamente.

Sin embargo, el Bot API de Telegram es **100% gratuito** (sin límite de mensajes, sin cobro por conversación, 30 mensajes/segundo), y los canales admiten suscriptores ilimitados. Como canal secundario para audiencia tech-savvy y urbana, es una adición de costo cero que vale la pena implementar.

**Un chatbot web** usando **Typebot** (open source, self-hosted en el mismo VPS) o **Botpress** (free tier: 500 mensajes/mes) sirve como punto de entrada inmediato. La ventaja clave: genera un **link compartible** (tipo tudominio.com/chat) que puede circular en WhatsApp groups, Facebook, TikTok bios, y códigos QR impresos — todo sin costo. La estrategia óptima es "web + Telegram primero para validar, WhatsApp como canal principal":

- **Semana 1**: Deploy web chatbot con link compartible + bot de Telegram ($0)
- **Semana 2**: Activar WhatsApp Cloud API (una vez verificado el número business)
- **Semana 3–5**: Distribución masiva en los tres canales simultáneamente

El widget web también permite embedderse en sitios de medios y ONGs aliadas, multiplicando el alcance sin costo adicional.

---

## El terreno electoral peruano favorece exactamente este proyecto

Las elecciones del **12 de abril de 2026** presentan una convergencia única de factores. La ONPE las calificó como "las más complejas en la historia del Perú": **39 partidos** en una sola cédula de votación, más de **34 candidatos presidenciales**, y por primera vez en décadas se elige Senado (60 senadores) además de 130 diputados.

Los datos de opinión pública son reveladores: **48% de los votantes no sabe por quién votar**, **64.8% de jóvenes de 18–29 años** dice que no votaría por ningún candidato, y **49% obtiene sus noticias de redes sociales**. Existe un vacío informativo real. El proyecto "Revisa tu Candidato" de la Asociación Civil Transparencia (lanzado el 12 de marzo de 2026) usa IA para revisar **antecedentes** de candidatos en 18+ bases de datos públicas — pero no compara **propuestas ni políticas**. Tu chatbot llena precisamente ese hueco.

Precedentes regionales validan el modelo: el **JNE lanzó un chatbot "Voto Informado"** en WhatsApp para las elecciones de 2021 (cobertura mediática nacional automática por ser institucional). Argentina desplegó **"Vot-A"** en WhatsApp para las PASO 2023, en colaboración directa con Meta. España tuvo **PolitiBot** en Telegram durante las elecciones de 2016, logrando más de 70% de satisfacción. Ninguno de estos usó IA generativa moderna — tu chatbot sería el primero con esa capacidad en Perú.

**Consideraciones legales**: un chatbot informativo y no partidario es completamente legal en Perú. La Ley N° 28094 reconoce el derecho a difundir propuestas políticas durante el período electoral. La Ley N° 29733 de Protección de Datos Personales aplica, pero como los usuarios inician la conversación voluntariamente, no hay problema de consentimiento. La única restricción operativa: **no difundir encuestas en la semana previa a la votación** (después del ~5 de abril).

---

## La estrategia de cobertura mediática gratuita que funciona en Perú

La ruta más eficiente hacia cobertura mediática en Perú no es contactar periodistas directamente — es **asociarse con una institución establecida**. Todos los proyectos cívico-tecnológicos que lograron cobertura masiva en Perú siguieron este patrón: "Revisa tu Candidato" se alió con Transparencia + IPYS + Proética; "eMonitor+" se respaldó en PNUD + 4 universidades; "Tejiendo Ciudadanía" unió a IEP + La República + la Unión Europea. La institucionalidad peruana abre puertas mediáticas que un emprendedor solo no puede abrir.

**Contactos prioritarios para alianzas**: Álvaro Henzler (presidente de Transparencia), el equipo de IPAE Acción Empresarial (organiza CADE Universitario con 600+ líderes estudiantiles), y los departamentos de comunicación de PUCP, UNMSM y USIL — las tres universidades ya involucradas en eMonitor+ del PNUD. Incluso una mención de respaldo de una de estas instituciones transforma la narrativa de "emprendedor lanza chatbot" a "alianza cívica despliega herramienta de IA para los votantes".

**Periodistas tech específicos para pitchear**: Daniel Augusto Bedoya Ramos (El Comercio, nombrado entre los mejores periodistas tech emergentes de LATAM por The Sociable en 2024), Patricia del Río (RPP Noticias, también escribe para El Comercio y Político.pe), y Marco Sifuentes (fundador de La Encerrona, medio digital independiente con audiencia masiva).

**El pitch que funciona**: "El primer chatbot con IA que ayuda a 27.5 millones de votantes peruanos a navegar las elecciones más complejas de la historia — 39 partidos en una sola cédula." El anzuelo noticioso combina tres elementos irresistibles para los medios: **novedad tecnológica** (IA generativa), **relevancia electoral inmediata** (faltan X semanas), y **problema sentido** (48% no sabe por quién votar). RPP (97% de cobertura radial nacional), El Comercio, e Infobae Perú están activamente publicando sobre IA y elecciones — tu proyecto encaja en una narrativa que ya están cubriendo.

---

## Plan de ejecución en 5 semanas con $300

**Semana 1 (8–14 marzo)**: Construcción con Claude Code. Deploy del backend en Cloud Run, integración con Gemini 2.5 Flash-Lite, web chatbot con Typebot, y bot de Telegram. Alimentar la base de conocimiento con planes de gobierno de los candidatos principales (disponibles en la web de JNE/ONPE). Registrar el número en Meta Cloud API. Costo: $12 (dominio).

**Semana 2 (15–21 marzo)**: Activar WhatsApp Cloud API (verificación puede tomar 2–5 días). Crear media kit digital (1 página web con demo, screenshots, stats del problema electoral). Enviar pitch a Daniel Bedoya (El Comercio), mesa de elecciones de RPP, e Infobae Perú. Contactar a Transparencia/IPAE para explorar endorsement. Publicar en r/peru. Costo: $0.

**Semana 3 (22–28 marzo)**: Lanzamiento público. Contenido TikTok/Reels tipo "39 partidos en UNA cédula — ¿ya sabes por quién votar?" con link al chatbot. Contactar consejos estudiantiles de PUCP, San Marcos, UNI, USIL. Reclutar 10–20 micro-influencers universitarios. Distribuir en grupos de Facebook regionales. Activar mecánica de "comparte con 3 amigos". Costo: ~$5–15 (API Gemini para primeros usuarios).

**Semana 4–5 (29 marzo – 12 abril)**: Máxima intensidad. Contenido reactivo a debates y noticias. WhatsApp Status templates para que usuarios compartan. Broadcasts de utility a usuarios que ya interactuaron ($0.02/mensaje). Push final: "Faltan X días — comparte con alguien que no ha decidido." Costo: $20–100 (broadcasts + API).

**Presupuesto total estimado: $37–$127**. Queda un colchón significativo dentro de los $300 para escalar si el chatbot gana tracción viral.

---

## Conclusión

El obstáculo para este proyecto nunca fue el dinero — es la ejecución y la distribución. **El cambio de precios de WhatsApp de noviembre 2024 eliminó la barrera económica** para chatbots reactivos: cero costo por conversación de servicio, ilimitadas. La combinación de Meta Cloud API directa + Cloud Run free tier + Gemini Flash-Lite produce un stack de producción funcional por menos de $50 para todo el ciclo electoral, dejando más de $250 de los $300 como reserva.

Las tres decisiones críticas son: primero, usar **exclusivamente la API oficial de Meta** (nunca APIs no oficiales — el riesgo de baneo durante la campaña es catastrófico e irreversible). Segundo, diseñar el chatbot como **primariamente reactivo** (usuarios escriben primero = gratis) y reservar los broadcasts pagados solo para recordatorios electorales de alto impacto. Tercero, buscar **una alianza institucional** (Transparencia, IPAE, o una universidad) antes de lanzar — esto multiplica la cobertura mediática y la credibilidad de formas que ningún presupuesto de marketing podría comprar.

El vacío informativo es real: no existe un chatbot con IA que compare propuestas de los 34+ candidatos. Con 2.5 millones de votantes primerizos buscando orientación y 48% de indecisos, el producto tiene demanda orgánica. Lo que importa ahora es velocidad. Un chatbot funcional compartido masivamente vale infinitamente más que uno perfecto lanzado tarde. Las elecciones son el 12 de abril — el reloj corre.