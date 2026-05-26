# SusuTwin

> Un "gemelo digital" de tu memoria personal — busca semánticamente entre lo que viviste, sentiste y reflexionaste.

| | |
|---|---|
| **Estado** | Side project · prototipo |
| **Tipo** | Experimental / personal |
| **Repo** | Privado |
| **Mi rol** | Diseño y desarrollo |

## Idea

> *"Transformando un Search Engine en un Life Engine."*

Las apps tradicionales de notas ven el texto como archivos. SusuTwin lo trata como **memoria episódica**: cada entrada está conectada a otras por valores, emociones y temas. Una búsqueda semántica devuelve no sólo "qué dije sobre X" sino "qué siento sobre X a través del tiempo".

## Pilares

1. **Memoria Episódica** — guarda lo que viviste y sentiste hoy, no sólo lo que pensaste.
2. **ADN de Valores** — tú defines tus principios innegociables; el sistema los usa como ancla.
3. **Reflexión Nocturna** — un job batch consolida aprendizajes y descubre patrones mientras duermes.

## Flujo

```bash
# Cargar notas del día
pnpm ingest --file ./mis_notas.md

# Reflexión nocturna: descubre patrones, genera resúmenes
pnpm reflect
```

## Stack

- **DB:** Supabase con pgvector (búsqueda semántica)
- **Embeddings:** modelo de OpenAI (text-embedding-3-*)
- **Migraciones:** SQL versionado en `supabase/migrations/`

## Estado

Prototipo funcional. Sigue siendo un side project — sin planes inmediatos de productizarlo, pero la arquitectura sirve como playground para experimentos con RAG personal y memoria de largo plazo.

## Inspiración

Surge de la frustración con apps tipo Notion/Obsidian: ofrecen estructura pero no entienden el **flujo emocional** de quien escribe. SusuTwin parte del supuesto opuesto — primero el sentido, después el archivo.
