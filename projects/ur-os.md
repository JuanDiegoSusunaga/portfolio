# UR-OS Process Scheduling Simulator

> Simulador modular de planificación de procesos en Java para el curso de Sistemas Operativos de la Universidad del Rosario.

| | |
|---|---|
| **Estado** | Archivado (curso cerrado) |
| **Tipo** | Académico · proyecto de clase |
| **Curso** | Sistemas Operativos · Universidad del Rosario |
| **Repo** | Privado (cumple con política de no-republicación de material académico) |
| **Mi rol** | Implementación + extensión sobre base provista por el curso |

## Contexto

UR-OS es una plataforma educativa desarrollada en la Universidad del Rosario que reproduce los componentes principales de un sistema operativo: procesador, memoria, sistema de archivos.

La versión 0.0.3.8 sobre la que trabajé se enfoca en **planificación de procesos**.

## Mi contribución

- Implementación del algoritmo **FCFS (First Come First Served)**.
- Integración del simulador con la base provista (`UR_OS.java`, `OS.java`, `SystemOS.java`).
- Variante posterior **Planificador-de-Procesos-Predictivo-e-Interactivo-ML-para-UR-OS** (público en mi perfil): extiende UR-OS con un modelo predictivo de ML para anticipar tiempos de CPU.

## Stack

- **Lenguaje:** Java 17 (JSE estándar, sin dependencias externas)
- **IDE:** NetBeans 20
- **Patrón:** Modular — cada función del SO (scheduler, memory manager, FS) es intercambiable.

## Lecciones

- Lo más interesante no fue implementar FCFS (es trivial), sino entender **cómo se modela un SO** para que cada subsistema sea pluggable.
- La variante ML me llevó a la pregunta: *¿qué tan bien puede aprender un modelo simple a predecir burst times?* (spoiler: sorprendentemente bien con features básicos).
