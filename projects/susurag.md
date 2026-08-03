# SusuRAG

> Motor de memoria semántica para agentes de código: indexa un monorepo completo (código + docs + reglas) en pgvector y lo expone como herramientas MCP para que el agente busque semánticamente antes de escribir — "RAG-First Development".

| | |
|---|---|
| **Estado** | v11 · en uso activo diario |
| **Tipo** | Herramienta de desarrollo / infra de agentes |
| **Repo** | Privado (recién extraído de QNexus a proyecto independiente) |
| **Mi rol** | Diseño y desarrollo completo |

## Problema

En un monorepo grande, `grep` y `find` no le alcanzan a un agente de IA: encuentra texto, no *significado*, y no sabe qué archivo es el canónico ni dónde se han cometido bugs antes. SusuRAG nació dentro de QNexus como su "cerebro": todo el codebase indexado con embeddings, un grafo de relaciones entre archivos, y memoria de errores pasados — consultable por el agente vía MCP.

El usuario final no es una persona: es **Claude Code**. Una regla del monorepo prohíbe `grep`/`find` cuando el MCP puede resolverlo semánticamente, y hooks del agente marcan cada archivo leído/editado para alimentar los "hot paths".

## Arquitectura

```
Ingesta: descubrimiento → chunking AST (fronteras de export/función, 80–250 líneas)
       → resumen de contexto por archivo (LLM) → chunk enriquecido [CONTEXT][METADATA][FILE]
       → embedding 3072D → pgvector (halfvec + HNSW) + grafo de sinapsis (imports, RPCs, tests)

Búsqueda: MCP server (25 tools, stdio) → caché exacto SHA-256 (~5 ms) / semántico (~50 ms)
       → Edge Function → RRF híbrido (vectorial + full-text en español) → scoring generacional
```

Mecanismos que lo separan de un RAG genérico:

- **Contextual retrieval**: ningún chunk se embebe "desnudo" — cada uno lleva un resumen del archivo y metadata antepuestos, lo que cambia radicalmente la calidad del recall.
- **Scoring generacional**: cuando un chunk es reemplazado por una versión nueva, el viejo no se borra — decae (×0,70 por generación). Un A/B mostró que este decay funciona mejor que el decay por edad, que se eliminó.
- **Bug DNA / cicatrices**: los archivos donde hubo bugs quedan marcados con nivel de peligro; la búsqueda devuelve el warning automáticamente. Las cicatrices pueden propagarse entre proyectos.
- **Sinapsis con doble score**: cada arista del grafo tiene `confidence` (estructural: imports, llamadas a RPC) y `strength` (aprendida por co-acceso del agente).
- **Consolidación nocturna**: un cron resume los archivos más accedidos (−60 % tokens), refuerza sinapsis co-accedidas, debilita las inactivas y purga chunks zombis — mantenimiento del índice sin intervención.
- **Multi-proyecto**: aislamiento total por proyecto con búsqueda scoped, más inteligencia cruzada (detectar patrones repetidos entre proyectos, propagar cicatrices).

## Evaluación — la parte que más me importa

Retrieval sin harness de evaluación es fe. SusuRAG tiene un golden set de 30 casos con control negativo (queries fuera de dominio) y gate de regresión (falla el build si recall@3 baja de 0,60):

| Corrida | Recall@1 | Recall@3 | Recall@5 | MRR |
|---|---|---|---|---|
| Baseline | 53,8 % | 69,2 % | 84,6 % | 0,650 |
| Tras iterar chunking + contexto | **67,9 %** | **96,4 %** | **100 %** | **0,813** |
| Con reranker cross-encoder (descartado) | 57,1 % | 82,1 % | 96,4 % | 0,705 |

El reranker de Vertex AI se prototipó, se midió, **empeoró todas las métricas** y quedó apagado por defecto con fail-safe — documentado en el propio repo. Había crédito de GCP de sobra para usarlo; el A/B negativo pesó más.

## Stack

- **TypeScript 5.7** sobre Node (tsx) · MCP SDK (transporte stdio) · Vitest (93 tests, incl. 21 E2E multi-proyecto)
- **Embeddings:** Gemini embedding a 3.072 dims (`RETRIEVAL_DOCUMENT`/`RETRIEVAL_QUERY`) · resúmenes con Gemini Flash
- **Vector store:** Supabase Postgres + pgvector (`halfvec(3072)`, HNSW), proyecto Supabase dedicado, separado del de negocio
- **Edge:** función Deno para la búsqueda híbrida (RRF vectorial + FTS español) · 37 migraciones SQL · 29 RPCs
- **Extra:** visualización 3D del grafo de conocimiento (three.js + force-graph) con control por gestos vía webcam (MediaPipe)

## Números

| Métrica | Valor |
|---|---|
| Código | ~12.100 líneas TS · ~10.800 líneas SQL |
| Herramientas MCP | 25 |
| Índice | ~1.500 chunks activos sobre ~1.260 archivos |
| Grafo | 3.312 sinapsis (imports, RPCs, tests↔fuente, docs) |
| Latencia | búsqueda híbrida ~300 ms · caché ~5–50 ms |
| Costo | < $0,15 USD/año a 1.000 búsquedas/mes |

## Lecciones

- **El contexto del chunk importa más que el modelo**: pasar de recall@3 69 % → 96 % no vino de cambiar el embedding sino de chunking AST + prefijos de contexto.
- **Medir antes de adoptar**: el reranker era la mejora "obvia" y los datos dijeron que no. El harness de evaluación pagó su costo en un solo experimento.
- **La memoria necesita olvido**: sin scoring generacional ni consolidación nocturna, el índice acumula versiones muertas y el retrieval se degrada — un RAG sobre código vivo es un problema de *mantenimiento*, no solo de indexación.
