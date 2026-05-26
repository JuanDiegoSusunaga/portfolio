# QNexus

> Plataforma SaaS para digitalizar la gestión de proyectos de construcción: asistencia, inventario, SST, finanzas y personal.

| | |
|---|---|
| **Estado** | En construcción · v10.3.0 (pre-launch) |
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

## Stack

- **Frontend:** Next.js 15.1 · React 19.2.3 · TypeScript 5.7 · PWA
- **Backend / DB:** PostgreSQL (RPCs atómicos para transacciones críticas)
- **Tests:** 1,888 tests pasando
- **Build:** Passing

## Decisiones técnicas destacadas

- **PWA en vez de app nativa**: instalación sin App Store, actualizaciones automáticas, single codebase.
- **RPCs PostgreSQL para inventario**: las transacciones de stock necesitan atomicidad real — mover lógica al servidor de DB elimina race conditions.
- **TOTP en tótem de asistencia**: rotación de códigos cada N segundos hace imposible compartir credenciales fuera del sitio físico.

## Estado actual

**En construcción** — todavía no lanzado al mercado. Versión 10.3.0 con base estable: 1,888 tests pasando, build verde. Trabajando en endurecer el motor laboral colombiano, pulir la PWA en móvil y dejar listos los flujos de onboarding antes de habilitar pilotos.
