# QNexus

> Plataforma SaaS para digitalizar la gestión de proyectos de construcción: asistencia, inventario, SST, finanzas y personal.

| | |
|---|---|
| **Estado** | En construcción · v10.4.0 (pre-launch) |
| **Tipo** | Producto SaaS (PWA) — en desarrollo |
| **Repo** | Privado (proprietary) |
| **Mi rol** | Desarrollo full-stack |

## Problema

La gestión de obra en construcción colombiana sigue corriendo en Excel, WhatsApp y cuadernos. Esto genera:
- Asistencia inflada (cuadrillas "fantasma").
- Pérdida de herramientas y materiales sin trazabilidad.
- Riesgo legal por falta de checklists preoperacionales.
- Cálculo manual de nómina (HED/HEN/recargos) propenso a errores.

QNexus consolida todo en una sola PWA con motor laboral colombiano integrado.

## Funcionalidades

**Gestión de Personal**
- Asistencia por **QR/NFC** con carnés digitales y Tótem anti-fraude (TOTP).
- Marcaje híbrido offline-first: el residente marca la cuadrilla en campo sin señal, con **reconocimiento facial opcional** (modelo servido como asset público, cache en Service Worker).
- Control de cuadrillas y asignación dinámica por frente de obra.
- Horas extra: cálculo automático HED / HEN / recargos / dominicales.
- Vacaciones, permisos, licencias con flujo de aprobación.
- Nómina mensual: liquidación de devengados, deducciones y neto a pagar.

**Inventario Total**
- Trazabilidad de herramientas, maquinaria, consumibles, materiales y pétreos.
- Transacciones atómicas vía RPCs PostgreSQL.
- Control de flota: horómetros, combustible, alertas de mantenimiento.

**Seguridad y Salud (SST)**
- Checklists preoperacionales con bloqueo automático de equipos con fallas críticas.
- Botón de pánico SOS con alertas GPS en tiempo real.

**Presupuesto y control de obra**
- Cronograma probabilístico: simulación **Monte Carlo** (P50/P80/P90 con intervalos de confianza) y optimizador por **algoritmo genético** (simheuristics).
- Factor clima con datos reales (Open-Meteo) cruzados con novedades de obra para medir impacto en plazos.
- Control EVM (valor ganado) con CPI/SPI a la fecha y Actas Parciales de Cobro acumuladas.

**Financiero y adquisiciones**
- Flujo completo de compra: requerimiento → OC con matriz de aprobación por monto y guard presupuestal → recepción parcial → CxP con match factura↔OC↔recibido → asiento contable automático.
- Contabilidad alineada a **NIIF 15** (reconocimiento de ingreso), costeo PPP y conciliación multimoneda.

## Stack

- **Frontend:** Next.js 15.1 · React 19.2.3 · TypeScript 5.7 · PWA
- **Backend / DB:** PostgreSQL / Supabase (RPCs atómicos para transacciones críticas, RLS, stack local reproducible)
- **Tests:** 3.681 tests pasando (Vitest)
- **Build:** Passing

## Decisiones técnicas destacadas

- **PWA en vez de app nativa**: instalación sin App Store, actualizaciones automáticas, single codebase.
- **RPCs PostgreSQL para inventario**: las transacciones de stock necesitan atomicidad real — mover lógica al servidor de DB elimina race conditions.
- **TOTP en tótem de asistencia**: rotación de códigos cada N segundos hace imposible compartir credenciales fuera del sitio físico.
- **Simheuristics para el cronograma**: Monte Carlo estima la distribución real de plazos (no una fecha "optimista" única) y un GA busca la secuencia de actividades que minimiza el P80.
- **Hardening de seguridad server-side**: REVOKE de `anon` sobre funciones `SECURITY DEFINER`, guards presupuestales y máquinas de estado en RPCs — las reglas de negocio críticas no viven en el cliente.
- **Desarrollo RAG-First**: todo el monorepo se indexa en [SusuRAG](susurag.md), un motor de memoria semántica propio que los agentes de código consultan por MCP antes de escribir.

## Estado actual

**En construcción** — todavía no lanzado al mercado. Versión 10.4.0 con base estable: 3.681 tests pasando, build verde. Desde mediados de 2026 el foco pasó de personal/inventario a cerrar el ciclo financiero completo (compras → contabilidad NIIF) y al cronograma probabilístico, antes de habilitar pilotos.
