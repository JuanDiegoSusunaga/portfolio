# SECOP SUSU

> Plataforma personal de análisis de contratación pública colombiana: inteligencia de mercado de obra vial, due diligence de contratistas y búsqueda de oportunidades sobre datos abiertos de SECOP / datos.gov.co.

| | |
|---|---|
| **Estado** | Activo · pipeline reutilizable |
| **Tipo** | Personal / datos abiertos |
| **Repo** | Privado (los expedientes contienen PII — Ley 1581/2012) |
| **Mi rol** | Diseño y desarrollo completo |

## Problema

Los datos de contratación pública de Colombia están abiertos (SECOP II vía Socrata en datos.gov.co), pero responder preguntas reales exige cruzar más de una docena de datasets con esquemas distintos, una API que degrada con paginación por offset, y fuentes de antecedentes dispersas (Procuraduría, Contraloría, RUES, PEP). SECOP SUSU consolida todo en una base local consultable.

## Qué responde

**Inteligencia de mercado (obra vial)**
- Concentración del mercado: HHI, cuota top-5/top-10 por proveedor.
- Flujo del dinero: estimado → adjudicado → firmado → facturado → pagado, y tasa de pago por entidad.
- Señales de alerta: procesos con oferente único, descripciones clonadas entre entidades, prórrogas extremas, procesos represados >120 días sin adjudicar.
- Redes de consorcios/UT: quiénes se alían recurrentemente y quiénes actúan como "hubs".

**Due diligence de actores**
- Un comando perfila cualquier NIT cruzando ~10 fuentes en 30–60 s: RUES, sanciones (SIRI Procuraduría, multas SECOP, responsabilidad fiscal Contraloría), PEP, histórico contractual SECOP I+II, consorcios y aportes a campañas.
- Genera dosieres Markdown/PDF **citables**, separando dato VERIFICADO de DERIVADO y distinguiendo "0 verificado" de "fuente no disponible" — el output puede tener implicaciones legales.

**Oportunidades**
- Búsqueda filtrada (keywords, territorio, modalidad, valor, fechas) con export a Excel/CSV y enlace directo al portal.

## Arquitectura

```
API Socrata (SoQL) → normalización → UPSERT idempotente → Postgres 16 + pgvector → notebooks Polars/Plotly → XLSX / CSV / MD / PDF
```

- **Ingesta:** cliente Socrata propio sobre `httpx` con backoff exponencial, respeto de `Retry-After` y **keyset paging** (seek method) en lugar de `$offset` — el portal devuelve 503/504 con offsets grandes. 13 datasets ingestados a tablas locales.
- **Persistencia:** Postgres 16 + pgvector en Docker Compose, 24 migraciones SQL versionadas, índices trigram y HNSW.
- **Búsqueda semántica:** embeddings (768 dims) sobre descripciones de procesos, comando `similares "<texto>"`.
- **Documentos:** descarga y parsing de pliegos PDF (pypdf + regex, con fallback LLM) para extraer atributos técnicos.
- **Interfaz:** CLI única con ~25 subcomandos; refresh semanal orquestado en 12 pasos con validación de salud de tablas al final.
- **Tests:** 142 tests, suite 100 % offline (transporte HTTP mockeado, sin depender de Postgres ni de la API).

## Números (ventana de 24 meses, mercado vial)

| Métrica | Valor |
|---|---|
| Procesos ingestados | 64.854 |
| Contratos firmados | 49.196 |
| Valor contratado | ~$23,6 billones COP |
| Proveedores activos | 27.880 |
| Procesos represados (>120 días) | 19.913 |
| HHI del mercado | 150 (competitivo) · top-10 = 31,1 % |

## Decisiones técnicas destacadas

- **Keyset paging contra API hostil**: la decisión no salió de un manual sino del comportamiento observado del portal — offsets >40K devuelven 503/504.
- **Polars + Postgres local en vez de pandas + API en vivo**: los análisis iteran sobre 65K procesos sin re-descargar ni depender de la disponibilidad del portal.
- **Auto-auditoría**: un informe propio clasificó 63 hallazgos del sistema (5 high) — incluido un bug de matching de NIT sin dígito de verificación que producía falsos "sin antecedentes" — y derivó en un módulo de identidad con 28 tests y migraciones correctivas.
- **PII fuera de git por diseño**: expedientes, notebooks nominales y scratch están excluidos del versionado (Ley 1581/2012); lo publicable son los KPIs agregados.

## Lecciones

- En datos abiertos el trabajo duro no es el análisis sino la **ingesta confiable e idempotente**: retry, paginación correcta y UPSERT hacen la diferencia entre un notebook frágil y un pipeline.
- Cuando el output puede señalar a personas reales, la trazabilidad (verificado vs. derivado) es un requisito de producto, no un lujo.
