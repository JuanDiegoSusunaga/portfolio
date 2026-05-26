# Sitio Web · Consorcio de Infraestructura Vial (cliente)

> Sitio oficial de un consorcio de construcción, rehabilitación y mantenimiento de infraestructura vial urbana en una ciudad intermedia de Colombia.

| | |
|---|---|
| **Estado** | En producción |
| **Tipo** | Cliente (institucional · bajo NDA) |
| **Repo** | Privado |
| **Mi rol** | Desarrollo full-stack y despliegue |

## Problema

El cliente necesitaba un sitio institucional que:
- Comunique avances de obra a la ciudadanía (sectores, fases, fotos).
- Publique noticias y convocatorias laborales de forma autónoma.
- Sea **rápido en móvil** (la mayoría del tráfico es desde celular) y **SEO-friendly** (Google indexa los avances).

## Secciones

| Ruta | Qué muestra |
|---|---|
| `/` | Hero institucional. |
| `/quienes-somos` | Ficha técnica del consorcio + mapa. |
| `/<proyecto-emblematico>` | Proyecto destacado: fases, galería, mapa. |
| `/malla-vial` | Sectores con fotos por barrio + galería. |
| `/noticias` · `/noticias/[slug]` | Listado y detalle de noticias dinámicas. |
| `/convocatoria` | Vacantes laborales activas. |

## Stack

- **Framework:** Next.js 15 (App Router)
- **Estilos:** Tailwind CSS v4
- **Media:** Cloudinary (transformaciones on-the-fly, CDN global)
- **Deploy:** Vercel (auto-deploy en `main`)
- **SEO:** `sitemap.ts` dinámico (incluye noticias auto-generadas) + `robots.ts`
- **Custom 404:** página propia con navegación de regreso

## Decisiones técnicas destacadas

- **Cloudinary** en vez de `next/image` puro: el cliente sube fotos directamente sin pasar por re-deploy, y las transformaciones de tamaño/calidad son automáticas.
- **App Router + slugs dinámicos**: noticias generadas sin tocar código. El sitemap se actualiza solo con cada nota nueva.
- **Tailwind 4** desde el inicio para evitar la migración después.

## Lecciones

- Cliente institucional = priorizar **claridad y velocidad de carga** sobre interactividad.
- Cloudinary > self-hosted assets cuando el cliente publica contenido continuamente.
