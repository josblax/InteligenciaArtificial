# Autopráctica: Hill Climbing en el problema de las 8 reinas

## Objetivo

Al terminar esta autopráctica podrás:

- Explicar cómo funciona el **ascenso de colina** (*hill climbing*) con la función de conflictos como heurística.
- Distinguir entre **mínimo local**, **meseta** y **solución**.
- Medir experimentalmente el efecto de dos mejoras propuestas por Russell & Norvig: los **movimientos laterales** y los **reinicios aleatorios**.
- Seguir paso a paso, con un visualizador, cómo se generan y comparan los 56 vecinos de un estado.

## Material

| Archivo | Contenido |
|---|---|
| `hc_laterales_sin_reinicios.py` | Hill climbing con movimientos laterales, un solo intento (Anexo A) |
| `hc_con_reinicios.py` | Hill climbing con reinicios aleatorios (Anexo B) |
| `mejores_vecinos_8_reinas.py` | Visualizador interactivo de `mejores_vecinos()` (se entrega aparte) |

Cómo ejecutar cada archivo:

- **En Anaconda Notebooks:** sube el archivo y en una celda escribe `%run nombre_del_archivo.py`. También puedes pegar el código en una celda.
- **En tu computadora** (terminal, Spyder o VS Code): `python nombre_del_archivo.py`.

## Recordatorio

- **Estado:** una lista de 8 números. `estado[c]` es la fila donde está la reina de la columna `c` (filas 0 a 7, de arriba hacia abajo).
- **Heurística `h`:** `conflictos(estado)` cuenta los pares de reinas que se atacan, ya sea en la misma fila o en la misma diagonal. Si `h = 0`, el estado es una solución.
- **Vecino:** el estado que resulta de mover **una** reina a otra fila dentro de su misma columna.

---

## Parte 0. Calentamiento (a mano, sin computadora)

1. ¿Cuántos vecinos tiene cualquier estado? Explica el cálculo.
2. ¿Cuál es el máximo valor posible de `conflictos(estado)`? ¿Qué estado lo alcanza?
3. Calcula a mano `conflictos([0, 0, 0, 0, 0, 0, 0, 1])`. Separa cuántos pares chocan por fila y cuántos por diagonal.
4. En la condición `abs(estado[i] - estado[j]) == j - i`, ¿por qué no hace falta escribir `abs(j - i)`?
5. ¿Por qué la función nunca revisa si dos reinas están en la misma **columna**?

---

## Parte 1. Hill climbing con movimientos laterales, sin reinicios

Usa el archivo `hc_laterales_sin_reinicios.py` (Anexo A).

### 1.1 Lectura del código

Antes de ejecutar, responde:

1. `mejores_vecinos()` regresa una **lista** de vecinos, no solo uno. ¿Qué contiene esa lista?
2. En `hill_climbing_lateral()` hay tres casos dentro del `while`. Escribe con tus palabras qué significa cada uno:
   - `valor_vecino > valor`
   - `valor_vecino == valor`
   - `valor_vecino < valor`
3. ¿Para qué sirve la variable `laterales_consecutivos`? ¿Por qué se reinicia a 0 cuando hay una mejora estricta?
4. El comentario dice que `random.choice(candidatos)` es *clave para no ciclar*. ¿Qué podría pasar en una meseta si siempre se eligiera el primer candidato?
5. ¿Cuáles son las tres formas en que puede terminar el ciclo `while`?

### 1.2 Ejecución

Ejecuta el archivo **tres veces** y llena la tabla:

| Corrida | Tasa de éxito (%) | Pasos promedio con éxito | Pasos promedio con fallo |
|---|---|---|---|
| 1 | | | |
| 2 | | | |
| 3 | | | |

Observa también el tablero del *Ejemplo individual*. ¿Terminó en una solución? Si no, ¿cuántos conflictos le quedaron?

### 1.3 Experimento: ¿cuántos movimientos laterales convienen?

En el bloque `if __name__ == "__main__":` cambia el valor de `max_laterales` y registra los resultados:

| `max_laterales` | Tasa de éxito (%) | Pasos con éxito | Pasos con fallo |
|---|---|---|---|
| 0 | | | |
| 10 | | | |
| 50 | | | |
| 100 | | | |
| 500 | | | |

Preguntas:

1. Con `max_laterales = 0`, ¿a qué algoritmo equivale este código?
2. ¿Qué pasa con la tasa de éxito al aumentar `max_laterales`? ¿Y con el número de pasos?
3. ¿Por qué hace falta un límite? ¿Qué podría pasar si los movimientos laterales fueran infinitos?

---

## Parte 2. Hill climbing con reinicios aleatorios

Usa el archivo `hc_con_reinicios.py` (Anexo B).

### 2.1 Lectura del código

1. Compara `mejor_vecino()` (este archivo) con `mejores_vecinos()` (Parte 1). Cuando hay varios vecinos empatados, ¿cuál regresa `mejor_vecino()`? ¿Es al azar?
2. En `hill_climbing()`, la condición de paro es `valor_vecino >= valor`. ¿Este código permite movimientos laterales? Justifica.
3. ¿Qué hace `con_reinicios()` cuando un intento termina con conflictos mayores a 0?
4. ¿Qué regresa `con_reinicios()` si en 100 intentos no encuentra solución?

### 2.2 Ejecución

Ejecuta el archivo **cinco veces** y registra:

| Corrida | Intentos hasta la solución | Pasos del intento exitoso | Conflictos más frecuentes en los fallos |
|---|---|---|---|
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |

### 2.3 Predicción y comprobación

Si un solo intento tiene probabilidad **p** de éxito, el número esperado de intentos es aproximadamente **1/p**.

1. Con la tasa de éxito que obtuviste en la Parte 1 para `max_laterales = 0`, calcula cuántos intentos esperarías en promedio.
2. Completa la siguiente función y ejecútala para medir el promedio real en 1000 búsquedas. Pégala debajo de `con_reinicios()`:

```python
def medir_reinicios(veces=1000):
    total_intentos = 0
    total_pasos = 0
    for _ in range(veces):
        intentos = 0
        while True:
            estado, valor, pasos = hill_climbing()
            intentos += 1
            total_pasos += pasos
            # TODO: si valor es 0, sal del while

        total_intentos += intentos
    print(f"Intentos promedio: {total_intentos / veces:.2f}")
    print(f"Pasos totales promedio (sumando todos los intentos): {total_pasos / veces:.1f}")


medir_reinicios()
```

3. ¿Se parece tu predicción al promedio medido?
4. **Reto:** escribe `con_reinicios_y_laterales()`, que haga reinicios pero use `hill_climbing_lateral()` de la Parte 1 en cada intento. Antes de ejecutar, predice cuántos intentos necesitará en promedio.

### 2.4 Comparación

| Variante | Éxito por intento (%) | Intentos promedio | Pasos promedio hasta resolver |
|---|---|---|---|
| Sin laterales, sin reinicios | | 1 | |
| Con 100 laterales, sin reinicios | | 1 | |
| Sin laterales, con reinicios | | | |
| Con laterales y reinicios (reto) | | | |

¿Cuál variante usarías si quisieras resolver el problema **siempre**? ¿Cuál si quisieras hacer **el menor trabajo posible**?

---

## Parte 3. Mirar por dentro: `mejores_vecinos()` paso a paso

Usa el archivo `mejores_vecinos_8_reinas.py`.

Al ejecutarlo se muestra el tablero con su panel de controles. En Anaconda Notebooks los controles aparecen como botones debajo de la celda. En tu computadora se abre una ventana.

### 3.1 Primer recorrido

El estado inicial es `[4, 5, 6, 3, 4, 5, 6, 5]`.

1. Antes de avanzar, anota `conflictos(estado)`.
2. Usa **Siguiente ▶** para recorrer la **columna 0** completa (pasos 1 al 8). En cada paso anota qué rama se ejecutó: *nuevo mínimo*, *empate*, *se descarta* o *se salta*.
3. ¿En qué paso aparece el primer `continue`? ¿Por qué justo en ese?
4. Pulsa **Ir al final**. ¿Cuánto vale `mejor_valor`? ¿Cuántos vecinos quedaron en `mejores`?
5. Observa la gráfica: ¿por qué la línea ámbar de `mejor_valor` **nunca sube**?

### 3.2 Conexión con los otros dos códigos

1. En el mismo estado, ¿cuál de los vecinos de `mejores` elegiría `mejor_vecino()` de la Parte 2? Pista: piensa en el orden en que se recorren las columnas y las filas.
2. ¿Y cuál elegiría `hill_climbing_lateral()` de la Parte 1?
3. Pulsa **Paso de hill climbing** varias veces hasta que el tablero indique que ya no se mueve. Anota la secuencia de valores de `h`. ¿Llegaste a una solución?
4. El botón **Paso de hill climbing** del visualizador **no** hace movimientos laterales. ¿A cuál de los dos códigos se parece más?

### 3.3 Encuentra los casos especiales

Escribe cada estado en el campo **Estado**, pulsa **Cargar estado** y luego **Ir al final**:

| Estado | `conflictos(estado)` | `mejor_valor` | `len(mejores)` | ¿Solución, mínimo local, meseta o hay mejora? |
|---|---|---|---|---|
| `0, 4, 7, 5, 2, 6, 1, 3` | | | | |
| `3, 1, 6, 5, 2, 4, 7, 0` | | | | |
| `3, 7, 0, 7, 5, 1, 1, 6` | | | | |

1. En el segundo estado, ¿qué haría `hill_climbing_lateral()`? ¿Y `hill_climbing()`?
2. En el tercer estado, ¿qué haría `hill_climbing_lateral()`? ¿Y `hill_climbing()`?
3. Con **Estado aleatorio** busca por tu cuenta otro mínimo local y otra meseta. Anótalos.

### 3.4 Modifica el código

En el archivo hay **dos** versiones de la función: `mejores_vecinos()` y `mejores_vecinos_traza()`. Al iniciar, el programa comprueba que ambas den el mismo resultado. Por eso, si cambias una, **cambia también la otra**.

1. Cambia `if valor < mejor_valor:` por `if valor <= mejor_valor:` en las dos funciones. Antes de ejecutar, predice qué pasará con `len(mejores)`. Luego comprueba tu predicción en el visualizador.
2. Deja el código como estaba y explica por qué la versión original con `<` y `elif ==` es la correcta para guardar **todos** los empates.

---

## Parte 4. Reflexión final

Responde en un párrafo cada una:

1. ¿Por qué hill climbing se considera un algoritmo **voraz** (*greedy*)?
2. ¿Qué problema de hill climbing resuelven los movimientos laterales? ¿Y qué problema resuelven los reinicios aleatorios?
3. Hill climbing guarda **un solo estado** a la vez. ¿Qué ventaja da eso frente a BFS o A* en un problema como las 8 reinas?
4. Si el problema fuera de 100 reinas en lugar de 8, ¿cuántos vecinos tendría cada estado? ¿Qué parte del código se volvería más costosa?

---

## Respuestas de referencia (revísalas al terminar)

Los resultados experimentales cambian un poco en cada corrida porque los estados iniciales son aleatorios. Si tus valores están cerca de estos, vas bien.

**Parte 0.** 1) 8 columnas × 7 filas alternativas = 56. 2) 28 = C(8,2), por ejemplo con todas las reinas en la misma fila. 3) 22: 21 por fila más 1 por diagonal (columnas 6 y 7). 4) Porque `j > i` siempre. 5) Porque la representación pone exactamente una reina por columna.

**Parte 1.** Con `max_laterales = 0` el código equivale a hill climbing de ascenso más pronunciado sin laterales. Valores aproximados:

| `max_laterales` | Éxito | Pasos con éxito | Pasos con fallo |
|---|---|---|---|
| 0 | ≈ 14 % | ≈ 4 | ≈ 3 |
| 10 | ≈ 55–60 % | ≈ 8 | ≈ 14 |
| 50 | ≈ 89 % | ≈ 15 | ≈ 43 |
| 100 | ≈ 94 % | ≈ 19–21 | ≈ 63 |

Coinciden con lo que reportan Russell & Norvig: 14 % sin laterales y 94 % con 100 laterales. Sin un límite, el algoritmo podría quedarse dando vueltas para siempre en una meseta plana.

**Parte 2.** `mejor_vecino()` regresa el **primer** vecino con el menor valor, porque solo cambia si `valor < mejor_valor`. El desempate es determinista y favorece a las columnas de la izquierda. No hay movimientos laterales, porque se detiene cuando `valor_vecino >= valor`. Con p ≈ 0.14 se esperan 1/0.14 ≈ 7 intentos. Medido: ≈ 6.8 intentos y ≈ 22 pasos en total. Con laterales y reinicios se esperan ≈ 1/0.94 ≈ 1.06 intentos.

**Parte 3.** Estado inicial: h = 17, `mejor_valor` = 12 y 8 vecinos empatados. `mejor_vecino()` elegiría `[4, 0, 6, 3, 4, 5, 6, 5]`, el primero de la lista (columna 1 → fila 0). `hill_climbing_lateral()` elegiría cualquiera de los 8 al azar.

| Estado | h | `mejor_valor` | `len(mejores)` | Tipo |
|---|---|---|---|---|
| `0, 4, 7, 5, 2, 6, 1, 3` | 0 | 1 | 5 | Solución |
| `3, 1, 6, 5, 2, 4, 7, 0` | 1 | 2 | 13 | Mínimo local: todos los vecinos empeoran |
| `3, 7, 0, 7, 5, 1, 1, 6` | 2 | 2 | 7 | Meseta: hay vecinos con el mismo valor |

En el mínimo local, los dos algoritmos se detienen. En la meseta, `hill_climbing()` se detiene, mientras que `hill_climbing_lateral()` hace un movimiento lateral y sigue buscando.

Con `<=`, cada empate **reinicia** la lista, así que `mejores` termina con un solo vecino: el **último** que alcanzó el mínimo. La versión original usa `<` para reiniciar la lista y `elif ==` para agregar, y así conserva todos los empates.

---

## Anexo A. `hc_laterales_sin_reinicios.py`

```python
"""
Hill climbing con movimientos laterales (versión Russell & Norvig)
para el problema de las N reinas.

- Si hay vecinos estrictamente mejores, elige uno al azar entre los mejores.
- Si no hay mejora, pero hay vecinos con el MISMO número de conflictos
  (meseta / shoulder), permite hasta 100 movimientos laterales seguidos.
"""
import random

N = 8


def conflictos(estado):
    """Cuenta pares de reinas que se atacan (misma fila o diagonal)."""
    total = 0
    for i in range(N):
        for j in range(i + 1, N):
            if estado[i] == estado[j] or abs(estado[i] - estado[j]) == j - i:
                total += 1
    return total


def mejores_vecinos(estado):
    """Regresa TODOS los vecinos empatados con el menor número de conflictos."""
    mejores = []
    mejor_valor = float("inf")
    for col in range(N):
        for fila in range(N):
            if fila == estado[col]:
                continue
            vecino = estado.copy()
            vecino[col] = fila
            valor = conflictos(vecino)
            if valor < mejor_valor:
                mejor_valor = valor
                mejores = [vecino]
            elif valor == mejor_valor:
                mejores.append(vecino)
    return mejores, mejor_valor


def hill_climbing_lateral(max_laterales=100):
    """
    Ascenso de colina permitiendo movimientos laterales hasta max_laterales.
    Se detiene si encuentra la solución (0), si todos los vecinos empeoran,
    o si agota el límite de 100 pasos laterales consecutivos.
    """
    actual = [random.randrange(N) for _ in range(N)]
    valor = conflictos(actual)
    pasos = 0
    laterales_consecutivos = 0

    while valor > 0:
        candidatos, valor_vecino = mejores_vecinos(actual)

        # 1. Todos los vecinos son peores: máximo/mínimo local estricto
        if valor_vecino > valor:
            break

        # 2. Empate de conflictos: movimiento lateral en una meseta
        if valor_vecino == valor:
            if laterales_consecutivos >= max_laterales:
                break
            laterales_consecutivos += 1
        else:
            # 3. Mejora estricta: reinicia el contador de laterales
            laterales_consecutivos = 0

        # Elegir al azar entre los mejores vecinos empatados (clave para no ciclar)
        actual = random.choice(candidatos)
        valor = valor_vecino
        pasos += 1

    return actual, valor, pasos


def dibujar(estado):
    for fila in range(N):
        print(" ".join("Q" if estado[c] == fila else "." for c in range(N)))


if __name__ == "__main__":
    # Prueba estadística sobre 1,000 ejecuciones individuales (sin reinicios)
    ensayos = 1000
    exitos = 0
    pasos_exito = []
    pasos_fallo = []

    for _ in range(ensayos):
        estado, valor, pasos = hill_climbing_lateral(max_laterales=100)
        if valor == 0:
            exitos += 1
            pasos_exito.append(pasos)
        else:
            pasos_fallo.append(pasos)

    print(f"Tasa de éxito (1 solo intento): {exitos / ensayos * 100:.1f}%")
    print(f"Promedio de pasos cuando tiene éxito: {sum(pasos_exito) / len(pasos_exito):.1f}")
    if pasos_fallo:
        print(f"Promedio de pasos cuando falla: {sum(pasos_fallo) / len(pasos_fallo):.1f}")

    # Ejemplo de una ejecución individual
    solucion, conflictos_finales, pasos = hill_climbing_lateral(max_laterales=100)
    print(f"\nEjemplo individual -> Conflictos: {conflictos_finales}, Pasos: {pasos}")
    dibujar(solucion)
```

## Anexo B. `hc_con_reinicios.py`

```python
"""
Hill climbing (ascenso de colina) para el problema de las N reinas.

Estado: lista donde estado[c] = fila de la reina en la columna c.
Objetivo: 0 pares de reinas atacándose.
Vecinos: mover UNA reina a otra fila dentro de su columna.
"""
import random

N = 8


def conflictos(estado):
    """Cuenta pares de reinas que se atacan (misma fila o diagonal)."""
    total = 0
    for i in range(N):
        for j in range(i + 1, N):
            if estado[i] == estado[j] or abs(estado[i] - estado[j]) == j - i:
                total += 1
    return total


def mejor_vecino(estado):
    """Revisa todos los vecinos y regresa el de menos conflictos."""
    mejor, mejor_valor = None, float("inf")
    for col in range(N):
        for fila in range(N):
            if fila == estado[col]:
                continue
            vecino = estado.copy()
            vecino[col] = fila
            valor = conflictos(vecino)
            if valor < mejor_valor:
                mejor, mejor_valor = vecino, valor
    return mejor, mejor_valor


def hill_climbing():
    """Ascenso más pronunciado: se detiene cuando ningún vecino mejora."""
    actual = [random.randrange(N) for _ in range(N)]
    valor = conflictos(actual)
    pasos = 0
    while True:
        vecino, valor_vecino = mejor_vecino(actual)
        if valor_vecino >= valor:        # máximo local (o solución)
            return actual, valor, pasos
        actual, valor = vecino, valor_vecino
        pasos += 1


def con_reinicios(max_reinicios=100):
    """Si se atora en un máximo local, empieza desde otro punto aleatorio."""
    for intento in range(1, max_reinicios + 1):
        estado, valor, pasos = hill_climbing()
        print(f"Intento {intento}: {valor} conflictos en {pasos} pasos")
        if valor == 0:
            return estado
    return None


def dibujar(estado):
    for fila in range(N):
        print(" ".join("Q" if estado[c] == fila else "." for c in range(N)))


if __name__ == "__main__":
    solucion = con_reinicios()
    if solucion:
        print("\nSolución:", solucion)
        dibujar(solucion)
```

## Anexo C. `mejores_vecinos_8_reinas.py`

Se entrega como archivo aparte. Al inicio contiene las funciones `conflictos()` y `mejores_vecinos()` sin cambios, seguidas de `mejores_vecinos_traza()`, que hace lo mismo pero guarda cada paso para el visualizador.
