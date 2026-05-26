# Groupay

> App móvil para dividir gastos compartidos en grupos (estilo Splitwise) construida sobre Expo.

| | |
|---|---|
| **Estado** | Prototipo inicial · pausado |
| **Tipo** | Side project · mobile |
| **Repo** | Privado |
| **Mi rol** | Desarrollo |

## Idea

Las apps de "split bills" existentes (Splitwise, Tricount) cobran suscripción o tienen UX heredada. Groupay parte de un MVP móvil simple:

1. Crear un grupo (viaje, casa, comida del finde).
2. Anotar gastos con foto y descripción.
3. Algoritmo de saldos calcula quién debe a quién.
4. Botón "pagar" abre Nequi/Daviplata con el monto pre-llenado.

## Stack

- **Framework:** Expo SDK + React Native
- **Routing:** Expo Router (file-based)
- **Estado:** prototipo inicial — auth, modelado de grupos, UI de gastos.

## Por qué Expo

- Single codebase iOS + Android.
- OTA updates sin pasar por review de App Store.
- Pago integrado con apps colombianas (Nequi/Daviplata) vía URL schemes.

## Estado

Pausado. El MVP corre en simulador pero quedó sin pulir el flujo de payout. Posible retomar cuando salga tiempo entre QNexus y CVArmenia.

> *Esta ficha es un placeholder — actualizar con descripción más precisa cuando se retome el proyecto.*
