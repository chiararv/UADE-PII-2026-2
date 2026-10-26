## Parte 2 — Implementación

El grafo se implementa con una matriz de adyacencia `boolean[n][n]`, donde `n` es la capacidad indicada en `crear(n)`. La posición `[i][j]` vale `true` si hay una arista entre el vértice de índice `i` y el de índice `j`. Como el grafo es no dirigido, `agregarArista` escribe `[i][j]` y `[j][i]`, así que la matriz siempre queda simétrica.

Para traducir etiquetas a índices se usa el TDA Diccionario (implementación dinámica): clave = etiqueta, valor = índice en la matriz. Para el camino inverso (índice → etiqueta, necesario para mostrar resultados) el grafo guarda además un arreglo `etiquetas[]`, donde `etiquetas[i]` es la etiqueta del vértice de índice `i`.

### 2.1 Análisis de complejidad espacial

La matriz tiene `n × n` celdas, así que ocupa **O(n²)** espacio. A eso se suman el arreglo `etiquetas[]` (n posiciones) y el Diccionario (un nodo por vértice), que son O(n) cada uno. El total queda en **O(n²)**, dominado por la matriz.

Ese espacio no depende de la cantidad de aristas `m`: se reserva completo en `crear(n)`. Un grafo con n vértices y ninguna arista ocupa lo mismo que uno donde todos están conectados con todos. Tampoco depende de cuántos vértices se agregaron realmente: la matriz se dimensiona por la capacidad, no por `cantidadVertices`.

**Grafo disperso.** En un grafo no dirigido sin aristas de un vértice consigo mismo hay como máximo `n(n−1)/2` aristas, y cada arista ocupa dos celdas en `true` (`[i][j]` y `[j][i]`). Si el grafo tiene muchos vértices pero pocas aristas, casi toda la matriz queda en `false`: de las `n²` celdas solo `2m` tienen información. Por ejemplo, con 1.000 vértices y 1.000 aristas la matriz tiene 1.000.000 de celdas y solo 2.000 en `true` (el 0,2 %). El resto es espacio reservado que no aporta nada. Además, como la matriz es simétrica, las `(n² − n)/2` celdas de un lado de la diagonal repiten la información del otro lado, y las `n` celdas de la diagonal quedan siempre en `false`.

### 2.2 Matriz vs. listas de adyacencia

**Listas de adyacencia.** En lugar de una matriz, cada vértice tiene su propia lista (por ejemplo, un TDA Lista o un Diccionario) con los vértices adyacentes. En un grafo no dirigido, la arista v1–v2 aparece dos veces: v2 en la lista de v1 y v1 en la lista de v2. Un vértice sin aristas tiene una lista vacía.

Sea `n` la cantidad de vértices y `m` la cantidad de aristas:

| Aspecto | Matriz de adyacencia | Listas de adyacencia |
|---|---|---|
| Espacio | O(n²), siempre | O(n + m): una lista por vértice y 2m entradas en total |
| `existeArista(g, v1, v2)` | O(1): se mira la celda `[i][j]` | O(grado(v1)): hay que recorrer la lista de v1 buscando v2 |
| `vecinos(g, v)` | O(n): hay que recorrer la fila completa de v, aunque tenga pocos vecinos | O(grado(v)): la lista ya tiene exactamente los vecinos |

Estos costos suponen que ya se conocen los índices de los vértices. En esta implementación, traducir cada etiqueta a su índice con el Diccionario dinámico es una búsqueda lineal, O(n). Por eso `existeArista` y `vecinos` terminan siendo O(n) en la práctica. Con listas de adyacencia también habría que encontrar primero el vértice por su etiqueta, con un costo similar.

**¿Cuándo conviene cada una?**

- **Matriz de adyacencia:** conviene en grafos densos, donde `m` es del orden de `n²` (cerca del máximo `n(n−1)/2`) y casi todas las celdas se usan. También cuando la operación más frecuente es preguntar si existe una arista entre dos vértices, y cuando la cantidad máxima de vértices se conoce de antemano y es chica, de modo que reservar `n²` celdas no es un problema.
- **Listas de adyacencia:** conviene en grafos dispersos, donde `m` es mucho menor que `n²`, porque solo se guarda lo que existe. También cuando se recorren los vecinos de cada vértice muchas veces, como en BFS y DFS: con listas, recorrer todos los vecinos de todos los vértices cuesta O(n + m); con matriz cuesta O(n²), porque cada fila se recorre completa. Además, las listas pueden crecer sin una capacidad fija.
