# Algoritmos Avanzados · Suffix Array

> Implementación de un **Suffix Array** en C++ como ejercicio del curso de Algoritmos Avanzados durante mi intercambio en el Tec de Monterrey.

| | |
|---|---|
| **Estado** | Archivado (actividad entregada) |
| **Tipo** | Académico · ejercicio individual (Actividad 2.2) |
| **Curso** | Algoritmos Avanzados · Tec de Monterrey (intercambio) |
| **Repo** | Privado |
| **Mi rol** | Implementación individual |

## Qué resuelve

Construir el **suffix array** de una cadena `S`: arreglo de enteros que indica las posiciones iniciales (1-indexed) de todos los sufijos de `S` ordenados lexicográficamente.

Es una estructura clave para indexar texto: con ella se pueden hacer búsquedas de patrones, encontrar substrings repetidos o la subcadena común más larga en tiempo logarítmico (con LCP array adicional).

## Implementación

Versión **naive** O(n² log n):

1. Generar todos los sufijos de `S` (con un sentinel `$` para garantizar orden total).
2. Ordenarlos lexicográficamente (sort estándar de C++).
3. Devolver las posiciones iniciales en el orden ordenado.

El programa también imprime las posiciones de **todos los substrings** de `S` ordenados — variante del problema 2.2.

## Stack

- **Lenguaje:** C++ (std `<vector>`, `<algorithm>`, `<string>`)
- **Análisis estático:** SonarQube (`sonar-project.properties` incluido)
- **Build:** un único `main.cpp`, sin dependencias externas.

## Trade-offs

La implementación es naive (O(n² log n) por comparación de strings completos durante el sort).
Versiones más eficientes:
- **Manber-Myers / DC3** → O(n log n) o O(n).
- **SA-IS** (Suffix Array - Induced Sorting) → O(n) y se usa en producción (e.g. bzip2 derivados).

Para el alcance del ejercicio (cadenas pequeñas, foco en entender la estructura), naive era suficiente.

## Lecciones

- Entender el suffix array como una **forma comprimida del suffix trie**: misma información, menos memoria.
- Apreciar por qué algoritmos como SA-IS son tan citados — la diferencia es masiva en strings grandes.
- Primera vez configurando **SonarQube** sobre un proyecto C++ — útil para ver coverage y code smells en ejercicios académicos.
