### Depuración Paso a Paso (Trace) del Algoritmo DFS

El algoritmo utiliza **Búsqueda en Profundidad (DFS)** de forma recursiva, manteniendo un registro del `camino_actual` para evitar ciclos. 

Vamos a seguir el flujo del código paso a paso desde el inicio **'S'** hasta el destino **'G'**.

---

#### **Paso 1: Llamada inicial**
* `dfs_todos_los_caminos(G, 'S', 'G', ['S'])`
* Los vecinos de `'S'` en el grafo son: `['A', 'D']`.
* El algoritmo va a iterar primero sobre **`'A'`**.

---

#### **Rama A: Explorando el camino a través de 'A'**

* **Paso 1.1**: Llama a `dfs_todos_los_caminos(G, 'A', 'G', ['S', 'A'])`
  * Vecinos de `'A'`: `['S', 'B', 'D']`.
  * `'S'` ya está en el camino actual, así que se ignora.
  * Se procesa el primer vecino válido: **`'B'`**.

* **Paso 1.2**: Llama a `dfs_todos_los_caminos(G, 'B', 'G', ['S', 'A', 'B'])`
  * Vecinos de `'B'`: `['A', 'C', 'E']`.
  * `'A'` ya está en el camino, se ignora.
  * Vecino **`'C'`**: Llama a `dfs_todos_los_caminos(G, 'C', 'G', ['S', 'A', 'B', 'C'])`.
    * Vecinos de `'C'`: `['B']` (ya visitado). Fin de esta rama (retorna vacío).
  * Vecino **`'E'`**: Llama a `dfs_todos_los_caminos(G, 'E', 'G', ['S', 'A', 'B', 'E'])`.
    * Vecinos de `'E'`: `['B', 'D', 'F']`.
    * `'B'` ya visitado.
    * Vecino `'D'`: Llama con `['S', 'A', 'B', 'E', 'D']`. Vecinos de `'D'` son `'S'`, `'A'`, `'E'` (todos ya visitados). Fin de esta rama.
    * Vecino **`'F'`**: Llama a `dfs_todos_los_caminos(G, 'F', 'G', ['S', 'A', 'B', 'E', 'F'])`.
      * Vecinos de `'F'`: `['E', 'G']`.
      * Vecino **`'G'`**: ¡Destino alcanzado!
      * **Camino Guardado:** `['S', 'A', 'B', 'E', 'F', 'G']` (Solución 1).

* **Paso 1.3**: Volviendo a la llamada del **Paso 1.1** (en el nodo `'A'`), ahora procesa el siguiente vecino: **`'D'`**.
  * Llama a `dfs_todos_los_caminos(G, 'D', 'G', ['S', 'A', 'D'])`
  * Vecinos de `'D'`: `['S', 'A', 'E']` ( `'S'` y `'A'` ya visitados).
  * Vecino **`'E'`**: Llama a `dfs_todos_los_caminos(G, 'E', 'G', ['S', 'A', 'D', 'E'])`
    * Vecinos de `'E'`: `['B', 'D', 'F']`.
    * Vecino `'B'`: Llama con `['S', 'A', 'D', 'E', 'B']` -> Vecinos de `'B'` ya visitados. Fin de rama.
    * Vecino **`'F'`**: Llama a `dfs_todos_los_caminos(G, 'F', 'G', ['S', 'A', 'D', 'E', 'F'])`
      * Vecino **`'G'`**: ¡Destino alcanzado!
      * **Camino Guardado:** `['S', 'A', 'D', 'E', 'F', 'G']` (Solución 2).

---

#### **Rama D: Explorando el camino a través de 'D'**

* De vuelta en el **Paso 1**, ahora se procesa el segundo vecino de `'S'`: **`'D'`**.
* **Paso 2.1**: Llama a `dfs_todos_los_caminos(G, 'D', 'G', ['S', 'D'])`
  * Vecinos de `'D'`: `['S', 'A', 'E']`.
  * Vecino **`'A'`**: Llama a `dfs_todos_los_caminos(G, 'A', 'G', ['S', 'D', 'A'])`
    * Vecinos de `'A'`: `['S', 'B', 'D']`.
    * Vecino **`'B'`**: Llama con `['S', 'D', 'A', 'B']`
      * Sigue hacia **`'E'`** -> luego **`'F'`** -> luego **`'G'`**.
      * **Camino Guardado:** `['S', 'D', 'A', 'B', 'E', 'F', 'G']` (Solución 3).
  * Vecino **`'E'`**: Llama a `dfs_todos_los_caminos(G, 'E', 'G', ['S', 'D', 'E'])`
    * Vecinos de `'E'`: `['B', 'D', 'F']`.
    * Vecino `'B'`: Llama con `['S', 'D', 'E', 'B']` -> de `'B'` va a `'A'` o `'C'`, pero no logran llegar a `'G'` sin repetir.
    * Vecino **`'F'`**: Llama a `dfs_todos_los_caminos(G, 'F', 'G', ['S', 'D', 'E', 'F'])`
      * Vecino **`'G'`**: ¡Destino alcanzado!
      * **Camino Guardado:** `['S', 'D', 'E', 'F', 'G']` (Solución 4).
