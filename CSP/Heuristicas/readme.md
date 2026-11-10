# Práctica: Heurísticas para Problemas de Satisfacción de Restricciones (CSP)
## Coloreo del mapa de Australia

#### ==========================================================
### Heurísticas para CSP: MRV, grado, LCV y forward checking
### Autopráctica guiada
#### ==========================================================

**Caso:** hay que colorear los 7 estados de Australia con 3 colores (rojo, verde y azul) de modo que **dos estados vecinos nunca tengan el mismo color**. El problema es pequeño, pero sirve para ver cómo cada heurística le ahorra trabajo a la búsqueda con *backtracking*.

| Elemento del CSP | En este problema |
|---|---|
| Variables | WA, NT, Q, NSW, V, SA, T |
| Dominio | {rojo, verde, azul} para cada variable |
| Restricciones | si dos estados son vecinos, sus colores deben ser distintos |

Ten a la mano las **imágenes del mapa** vistas en clase: cada parte de esta práctica reproduce una de ellas.

**Cómo usar este notebook**

1. Lee las **instrucciones de Python** de cada parte: qué hace cada instrucción y si transforma los datos o solo te da información.
2. Completa las celdas de código donde aparece `___` (guion bajo triple).
3. Si te atoras, abre la **Pista**. Si aun así no sale, compara con **Ver solución**.
4. Revisa la sección **Verifica tu resultado** para comprobar que vas bien.
5. Lee los **Hints** y escribe tu interpretación en la celda **Tu interpretación**. Esa es la parte más importante: el código solo es el medio.

> **Importante:** ejecuta las partes en orden. Las funciones que completes en una parte se usan en las siguientes.

---
## Parte 0. Definición del problema

En esta parte no hay nada que completar: solo ejecuta la celda y revisa cómo se representa el CSP en Python.

#### Instrucciones de Python que usarás

* **`VARIABLES = ["WA", "NT", ...]`**
    * *Tipo:* Definición.
    * *Explicación:* Lista con las variables del problema (los estados).
* **`VECINOS = {"WA": ["NT", "SA"], ...}`**
    * *Tipo:* Definición.
    * *Explicación:* Diccionario que guarda las restricciones: para cada estado, la lista de estados con los que no puede compartir color.
* **`asignacion = {"WA": "rojo"}`**
    * *Tipo:* Definición.
    * *Explicación:* Diccionario con los estados que ya tienen color. Los estados que no aparecen aún no están asignados.
* **`asignacion.get(clave)`**
    * *Tipo:* Información.
    * *Explicación:* Devuelve el valor de la clave, o `None` si la clave no existe. Sirve para preguntar el color de un estado sin provocar un error si todavía no tiene color.
* **`[x for x in lista if condición]`**
    * *Tipo:* Transformación.
    * *Explicación:* *List comprehension*: crea una lista nueva solo con los elementos que cumplen la condición.

```python
VARIABLES = ["WA", "NT", "Q", "NSW", "V", "SA", "T"]
COLORES = ["rojo", "verde", "azul"]

# Restricciones: quién es vecino de quién
VECINOS = {
    "WA":  ["NT", "SA"],
    "NT":  ["WA", "SA", "Q"],
    "SA":  ["WA", "NT", "Q", "NSW", "V"],
    "Q":   ["NT", "SA", "NSW"],
    "NSW": ["Q", "SA", "V"],
    "V":   ["SA", "NSW"],
    "T":   [],                  # Tasmania es una isla: no tiene vecinos
}

def es_consistente(var, valor, asignacion):
    """True si 'var' puede tomar 'valor' sin chocar con un vecino ya coloreado."""
    for vecino in VECINOS[var]:
        if asignacion.get(vecino) == valor:
            return False
    return True

def valores_legales(var, asignacion):
    """Colores que todavía le quedan disponibles a 'var'."""
    return [c for c in COLORES if es_consistente(var, c, asignacion)]

def mostrar_dominios(dominios):
    """Tabla de dominios como la de la imagen de clase.
    R = rojo, V = verde, A = azul, . = valor eliminado."""
    print(" | ".join(f"{v:^5}" for v in VARIABLES))
    letras = {"rojo": "R", "verde": "V", "azul": "A"}
    print(" | ".join(f"{' '.join(letras[c] if c in dominios[v] else '.' for c in COLORES):^5}"
                     for v in VARIABLES))

def dominios_actuales(asignacion):
    """Dominios de todas las variables a partir de una asignación parcial."""
    return {v: [asignacion[v]] if v in asignacion else valores_legales(v, asignacion)
            for v in VARIABLES}

for var in VARIABLES:
    print(f"{var:>4}: {len(VECINOS[var])} vecinos -> {VECINOS[var]}")
print()
mostrar_dominios(dominios_actuales({}))
```

**Verifica tu resultado:** SA tiene 5 vecinos (el máximo) y T tiene 0. La tabla muestra `R V A` en los 7 estados: al inicio todos pueden tomar los 3 colores.

> **Cómo leer las tablas:** R = rojo, V = verde, A = azul y el punto `.` indica un color eliminado. No confundas la letra V de *verde* con el estado V (*Victoria*).

---
## Parte 1. Heurística 1: Mínimo de Valores Remanentes (MRV)

**Pregunta que responde:** ¿qué variable coloreo ahora?

**Idea:** elegir la variable con **menos colores disponibles**. Es la que tiene más probabilidad de fallar, así que conviene atenderla primero (*fail first*): si va a fallar, es mejor saberlo pronto.

**Imagen 1:** WA = rojo, luego NT = verde, luego SA = azul. ¿Por qué SA y no otra?

#### Instrucciones de Python que usarás

* **`len(lista)`**
    * *Tipo:* Información.
    * *Explicación:* Número de elementos de una lista. Aplicado a `valores_legales(...)`, da cuántos colores le quedan a una variable.
* **`v not in asignacion`**
    * *Tipo:* Información.
    * *Explicación:* Es verdadero si la variable `v` todavía no tiene color.
* **`min(lista, key=función)`**
    * *Tipo:* Cálculo.
    * *Explicación:* Devuelve el elemento de la lista para el que la función da el valor **más pequeño**. Si hay empate, devuelve el primero que encuentra.
* **`lambda v: expresión`**
    * *Tipo:* Definición.
    * *Explicación:* Función corta sin nombre. `lambda v: len(valores_legales(v, asignacion))` recibe una variable y devuelve cuántos colores le quedan.

### 1.1 Contar los valores remanentes

Con WA = rojo y NT = verde, cuenta cuántos colores le quedan a cada variable sin asignar.

```python
asignacion = {"WA": "rojo", "NT": ___}

no_asignadas = [v for v in VARIABLES if v ___ asignacion]
for v in no_asignadas:
    legales = valores_legales(v, asignacion)
    print(f"{v:>4}: {___(legales)} valor(es) -> {legales}")
print()
mostrar_dominios(dominios_actuales(asignacion))
```

<details><summary><b>Pista</b></summary>

Para filtrar las variables sin color se usa el operador `not in`. Para contar elementos de una lista se usa `len`.

</details>

<details><summary><b>Ver solución</b></summary>

```python
asignacion = {"WA": "rojo", "NT": "verde"}

no_asignadas = [v for v in VARIABLES if v not in asignacion]
for v in no_asignadas:
    legales = valores_legales(v, asignacion)
    print(f"{v:>4}: {len(legales)} valor(es) -> {legales}")
print()
mostrar_dominios(dominios_actuales(asignacion))
```

</details>

**Verifica tu resultado:** SA tiene **1** valor (`['azul']`), Q tiene 2 (`['rojo', 'azul']`) y NSW, V y T tienen 3.

### 1.2 Programar la heurística MRV

Completa la función para que devuelva la variable sin asignar con **menos** valores legales.

```python
def mrv(asignacion):
    no_asignadas = [v for v in VARIABLES if v not in asignacion]
    return ___(no_asignadas, key=lambda v: len(___(v, asignacion)))

print("Con WA = rojo y NT = verde, MRV elige:", mrv({"WA": "rojo", "NT": "verde"}))
print("Con el mapa vacío, MRV elige:", mrv({}))
```

<details><summary><b>Pista</b></summary>

Buscas el **mínimo** número de valores legales. La función que da los colores disponibles se definió en la Parte 0.

</details>

<details><summary><b>Ver solución</b></summary>

```python
def mrv(asignacion):
    no_asignadas = [v for v in VARIABLES if v not in asignacion]
    return min(no_asignadas, key=lambda v: len(valores_legales(v, asignacion)))

print("Con WA = rojo y NT = verde, MRV elige:", mrv({"WA": "rojo", "NT": "verde"}))
print("Con el mapa vacío, MRV elige:", mrv({}))
```

</details>

**Verifica tu resultado:** Con WA = rojo y NT = verde, MRV elige **SA**. Con el mapa vacío elige **WA**: todas empatan con 3 valores y `min` devuelve la primera de la lista.

#### Hints para interpretar tus resultados

* **Hint de fail first:** A SA solo le queda el azul. *Si dejaras SA para el final y a esas alturas el azul ya estuviera ocupado por un vecino, ¿cuánto trabajo tendrías que deshacer?*
* **Hint de empates:** Con el mapa vacío, MRV eligió WA solo porque es la primera de la lista. *¿Te parece una buena razón? ¿Qué información adicional usarías para desempatar?*
* **Hint de costo:** *Para aplicar MRV hay que contar los valores legales de todas las variables en cada paso. ¿Vale la pena ese cálculo extra?*

**Tu interpretación:**

*(Escribe aquí tus conclusiones de esta parte.)*

---
## Parte 2. Heurística 2: Variable con más restricciones (heurística de grado)

**Pregunta que responde:** ¿qué variable coloreo ahora, cuando MRV no puede decidir?

**Idea:** elegir la variable con **más restricciones con variables todavía sin asignar**. Al resolverla primero, se reducen de golpe las opciones de muchas otras.

#### Instrucciones de Python que usarás

* **`sum(1 for v in lista if condición)`**
    * *Tipo:* Cálculo.
    * *Explicación:* Cuenta cuántos elementos de la lista cumplen la condición (suma un 1 por cada uno).
* **`max(lista, key=función)`**
    * *Tipo:* Cálculo.
    * *Explicación:* Igual que `min`, pero devuelve el elemento con el valor **más grande**.
* **`key=lambda v: (a, b)`**
    * *Tipo:* Cálculo.
    * *Explicación:* Si la función devuelve una tupla, Python compara primero `a` y, solo si hay empate, compara `b`. Sirve para combinar dos criterios.
* **`-numero`**
    * *Tipo:* Cálculo.
    * *Explicación:* Cambiar el signo convierte un "mayor es mejor" en "menor es mejor". Así se puede usar `min` con un criterio que se quiere maximizar.

### 2.1 Calcular el grado de cada variable

El grado es el número de vecinos **sin asignar**.

```python
def grado(var, asignacion):
    return sum(1 for v in VECINOS[___] if v not in ___)

for v in VARIABLES:
    print(f"{v:>4}: grado {grado(v, {})}")
```

<details><summary><b>Pista</b></summary>

Recorre los vecinos de la variable `var` y cuenta solo los que no están en la asignación.

</details>

<details><summary><b>Ver solución</b></summary>

```python
def grado(var, asignacion):
    return sum(1 for v in VECINOS[var] if v not in asignacion)

for v in VARIABLES:
    print(f"{v:>4}: grado {grado(v, {})}")
```

</details>

**Verifica tu resultado:** Con el mapa vacío: WA 2, NT 3, Q 3, NSW 3, V 2, **SA 5** y T 0.

### 2.2 Programar la heurística de grado

```python
def mayor_grado(asignacion):
    no_asignadas = [v for v in VARIABLES if v not in asignacion]
    return ___(no_asignadas, key=lambda v: ___(v, asignacion))

print("Con el mapa vacío, la heurística de grado elige:", mayor_grado({}))
```

<details><summary><b>Ver solución</b></summary>

```python
def mayor_grado(asignacion):
    no_asignadas = [v for v in VARIABLES if v not in asignacion]
    return max(no_asignadas, key=lambda v: grado(v, asignacion))

print("Con el mapa vacío, la heurística de grado elige:", mayor_grado({}))
```

</details>

**Verifica tu resultado:** Elige **SA**, con 5 restricciones. Compara con MRV, que en la misma situación eligió WA por empate.

### 2.3 Combinar MRV con el grado como desempate

En la práctica las dos heurísticas se usan juntas: primero MRV y, **solo si hay empate**, gana la variable de mayor grado.

Con WA = rojo, NT y SA quedan con 2 colores cada una (empate en MRV).

```python
def mrv_con_desempate(asignacion):
    no_asignadas = [v for v in VARIABLES if v not in asignacion]
    return min(no_asignadas,
               key=lambda v: (len(valores_legales(v, asignacion)), ___grado(v, asignacion)))

asignacion = {"WA": "rojo"}
for v in ["NT", "SA"]:
    print(f"{v}: {len(valores_legales(v, asignacion))} valores, grado {grado(v, asignacion)}")
print("MRV sola elige:", mrv(asignacion))
print("MRV con desempate elige:", ___(asignacion))
```

<details><summary><b>Pista</b></summary>

Se usa `min` porque MRV busca el menor número de valores. Pero para el grado queremos el **mayor**: ¿qué signo debes ponerle?

</details>

<details><summary><b>Ver solución</b></summary>

```python
def mrv_con_desempate(asignacion):
    no_asignadas = [v for v in VARIABLES if v not in asignacion]
    return min(no_asignadas,
               key=lambda v: (len(valores_legales(v, asignacion)), -grado(v, asignacion)))

asignacion = {"WA": "rojo"}
for v in ["NT", "SA"]:
    print(f"{v}: {len(valores_legales(v, asignacion))} valores, grado {grado(v, asignacion)}")
print("MRV sola elige:", mrv(asignacion))
print("MRV con desempate elige:", mrv_con_desempate(asignacion))
```

</details>

**Verifica tu resultado:** NT: 2 valores, grado 2. SA: 2 valores, grado 4. MRV sola elige **NT** (aparece primero en la lista); con desempate elige **SA**.

#### Hints para interpretar tus resultados

* **Hint de "cuello de botella":** SA toca a 5 de los 6 estados del continente. *¿Por qué conviene decidir primero la variable que está en el centro de todas las restricciones?*
* **Hint de Tasmania:** T tiene grado 0. *¿En qué momento de la búsqueda conviene colorear T? ¿Importa qué color le toque?*
* **Hint de combinación:** *¿Por qué el grado se usa como desempate y no como criterio principal? Piensa en una variable con grado alto pero con los 3 colores todavía libres, frente a otra con grado bajo y un solo color.*

**Tu interpretación:**

*(Escribe aquí tus conclusiones de esta parte.)*

---
## Parte 3. Heurística 3: Valor menos restrictivo (LCV)

**Pregunta que responde:** ya elegí la variable, ¿qué valor pruebo primero?

**Idea:** probar el color que le quite **menos opciones a los vecinos**. Así se deja la mayor libertad posible al resto del problema.

**Imágenes 2 y 3:** con WA = rojo y NT = verde, toca colorear Q. Si Q = rojo, a SA le queda 1 valor; si Q = azul, a SA le quedan 0.

#### Instrucciones de Python que usarás

* **`{**diccionario, clave: valor}`**
    * *Tipo:* Transformación.
    * *Explicación:* Crea un diccionario **nuevo** con todo lo del original más (o reemplazando) una clave. Sirve para probar una asignación hipotética sin modificar la real.
* **`valor in lista`**
    * *Tipo:* Información.
    * *Explicación:* Es verdadero si el valor aparece en la lista.
* **`sorted(lista, key=función)`**
    * *Tipo:* Transformación.
    * *Explicación:* Devuelve una lista nueva ordenada de menor a mayor según la función. No modifica la lista original.

### 3.1 Probar una asignación hipotética

Para cada color legal de Q, construye la asignación hipotética y revisa cuántos colores le quedan a SA.

```python
asignacion = {"WA": "rojo", "NT": "verde"}
print("Valores legales de Q:", valores_legales("Q", asignacion))

for valor in valores_legales("Q", asignacion):
    prueba = {**asignacion, "Q": ___}
    sa = valores_legales(___, prueba)
    print(f"Si Q = {valor}: a SA le quedan {len(sa)} valor(es) -> {sa}")
```

<details><summary><b>Ver solución</b></summary>

```python
asignacion = {"WA": "rojo", "NT": "verde"}
print("Valores legales de Q:", valores_legales("Q", asignacion))

for valor in valores_legales("Q", asignacion):
    prueba = {**asignacion, "Q": valor}
    sa = valores_legales("SA", prueba)
    print(f"Si Q = {valor}: a SA le quedan {len(sa)} valor(es) -> {sa}")
```

</details>

**Verifica tu resultado:** Valores legales de Q: `['rojo', 'azul']`. Si Q = rojo, a SA le queda 1 valor (`['azul']`); si Q = azul, le quedan **0** (`[]`).

### 3.2 Contar cuántas opciones elimina cada valor

Un valor elimina una opción por cada vecino sin asignar que **todavía tenía** ese color disponible.

```python
def opciones_eliminadas(var, valor, asignacion):
    eliminadas = 0
    for vecino in VECINOS[var]:
        if vecino not in asignacion and valor ___ valores_legales(vecino, asignacion):
            eliminadas += ___
    return eliminadas

for valor in valores_legales("Q", asignacion):
    print(f"Q = {valor}: elimina {opciones_eliminadas('Q', valor, asignacion)} opción(es)")
```

<details><summary><b>Pista</b></summary>

Pregunta si el valor **está** en la lista de colores legales del vecino. Si está, al asignarlo se le quita una opción.

</details>

<details><summary><b>Ver solución</b></summary>

```python
def opciones_eliminadas(var, valor, asignacion):
    eliminadas = 0
    for vecino in VECINOS[var]:
        if vecino not in asignacion and valor in valores_legales(vecino, asignacion):
            eliminadas += 1
    return eliminadas

for valor in valores_legales("Q", asignacion):
    print(f"Q = {valor}: elimina {opciones_eliminadas('Q', valor, asignacion)} opción(es)")
```

</details>

**Verifica tu resultado:** Q = rojo elimina **1** opción (el rojo de NSW). Q = azul elimina **2** (el azul de SA y de NSW).

### 3.3 Ordenar los valores con LCV

```python
def lcv(var, asignacion):
    return ___(valores_legales(var, asignacion),
                  key=lambda valor: ___(var, valor, asignacion))

print("Orden en que LCV prueba los colores de Q:", lcv("Q", asignacion))
```

<details><summary><b>Ver solución</b></summary>

```python
def lcv(var, asignacion):
    return sorted(valores_legales(var, asignacion),
                  key=lambda valor: opciones_eliminadas(var, valor, asignacion))

print("Orden en que LCV prueba los colores de Q:", lcv("Q", asignacion))
```

</details>

**Verifica tu resultado:** `['rojo', 'azul']`: primero el rojo, que deja libre a SA.

#### Hints para interpretar tus resultados

* **Hint de variable vs. valor:** MRV elige la variable más restringida, pero LCV elige el valor **menos** restrictivo. *¿Por qué una heurística busca "lo más difícil" y la otra "lo más fácil"? Piensa en qué se gana al fallar pronto y en qué se gana al acertar pronto.*
* **Hint de callejón sin salida:** Con Q = azul, SA se queda sin colores. *Sin LCV, ¿en qué momento de la búsqueda descubrirías ese error?*
* **Hint de todas las soluciones:** *Si quisieras encontrar **todas** las soluciones del mapa y no solo una, ¿seguiría sirviendo LCV? ¿Por qué?*

**Tu interpretación:**

*(Escribe aquí tus conclusiones de esta parte.)*

---
## Parte 4. Heurística 4: Forward checking

**Pregunta que responde:** ¿puedo detectar un fallo antes de llegar a él?

**Idea:** cada vez que se asigna un color, se **borra del dominio de los vecinos**. Si algún vecino se queda sin colores, se sabe de inmediato que el camino no funciona y se regresa (*backtrack*), sin esperar a intentar colorear ese vecino.

**Imagen 4:** al asignar WA = rojo, el rojo desaparece del dominio de NT y de SA.

#### Instrucciones de Python que usarás

* **`{v: list(d) for v, d in dominios.items()}`**
    * *Tipo:* Transformación.
    * *Explicación:* Copia el diccionario de dominios, incluyendo una copia de cada lista. Así, si hay que regresar, los dominios originales siguen intactos.
* **`lista.remove(valor)`**
    * *Tipo:* Transformación.
    * *Explicación:* Elimina el valor de la lista. **Modifica** la lista; marca error si el valor no está, por eso antes se revisa con `in`.
* **`return a, b`**
    * *Tipo:* Definición.
    * *Explicación:* Una función puede devolver dos resultados a la vez; se reciben con `x, y = funcion(...)`.

### 4.1 Programar forward checking

La función devuelve los nuevos dominios y el nombre del vecino que se quedó vacío (o `None` si ninguno).

```python
def forward_checking(var, valor, dominios):
    nuevos = {v: list(d) for v, d in dominios.items()}    # copia
    nuevos[var] = [valor]
    vacia = None
    for vecino in VECINOS[___]:
        if valor in nuevos[vecino]:
            nuevos[vecino].___(valor)
            if len(nuevos[vecino]) == ___:
                vacia = vecino
    return nuevos, vacia

dominios = {v: list(COLORES) for v in VARIABLES}
print("Dominios iniciales:")
mostrar_dominios(dominios)

dominios, vacia = forward_checking("WA", ___, dominios)
print("\nDespués de WA = rojo:")
mostrar_dominios(dominios)
print("Variable sin valores:", vacia)
```

<details><summary><b>Pista</b></summary>

Solo se revisan los vecinos de la variable que se acaba de asignar. Un vecino falla cuando su lista de colores queda con **cero** elementos.

</details>

<details><summary><b>Ver solución</b></summary>

```python
def forward_checking(var, valor, dominios):
    nuevos = {v: list(d) for v, d in dominios.items()}    # copia
    nuevos[var] = [valor]
    vacia = None
    for vecino in VECINOS[var]:
        if valor in nuevos[vecino]:
            nuevos[vecino].remove(valor)
            if len(nuevos[vecino]) == 0:
                vacia = vecino
    return nuevos, vacia

dominios = {v: list(COLORES) for v in VARIABLES}
print("Dominios iniciales:")
mostrar_dominios(dominios)

dominios, vacia = forward_checking("WA", "rojo", dominios)
print("\nDespués de WA = rojo:")
mostrar_dominios(dominios)
print("Variable sin valores:", vacia)
```

</details>

**Verifica tu resultado:** Después de WA = rojo, la tabla queda igual que la segunda fila de la imagen 4:

```
 WA   |  NT   |   Q   |  NSW  |   V   |  SA   |   T
R . . | . V A | R V A | R V A | R V A | . V A | R V A
```

Variable sin valores: `None`.

### 4.2 Detectar un fallo antes de tiempo

Continúa la secuencia con Q = verde y luego V = azul. Q y V no son vecinos de WA ni entre sí, pero **SA es vecino de los tres**.

```python
for var, valor in [("Q", "verde"), (___, "azul")]:
    dominios, vacia = forward_checking(var, valor, ___)
    print(f"\nDespués de {var} = {valor}:")
    mostrar_dominios(dominios)
    if vacia is not ___:
        print(f"{vacia} se quedó sin valores: hay que regresar.")
        break
```

<details><summary><b>Ver solución</b></summary>

```python
for var, valor in [("Q", "verde"), ("V", "azul")]:
    dominios, vacia = forward_checking(var, valor, dominios)
    print(f"\nDespués de {var} = {valor}:")
    mostrar_dominios(dominios)
    if vacia is not None:
        print(f"{vacia} se quedó sin valores: hay que regresar.")
        break
```

</details>

**Verifica tu resultado:** Después de Q = verde, a NT y a SA solo les queda `A`. Después de V = azul:

```
 WA   |  NT   |   Q   |  NSW  |   V   |  SA   |   T
R . . | . . A | . V . | R . . | . . A | . . . | R V A
```

Aparece el mensaje **SA se quedó sin valores: hay que regresar.**

#### Hints para interpretar tus resultados

* **Hint de anticipación:** Al asignar V = azul todavía nadie había intentado colorear SA. *Sin forward checking, ¿cuántas variables más podrías haber asignado antes de descubrir el problema?*
* **Hint de copia:** La función trabaja sobre una **copia** de los dominios. *¿Qué pasaría al hacer backtracking si modificaras directamente el diccionario original?*
* **Hint de límites:** Después de Q = verde, NT y SA quedaron ambos con solo el azul, y son vecinos entre sí. *Forward checking no detectó ese conflicto. ¿Por qué? ¿Qué tendría que revisar adicionalmente para detectarlo?* (Investiga qué es **consistencia de arcos** o AC-3.)

**Tu interpretación:**

*(Escribe aquí tus conclusiones de esta parte.)*

---
## Parte 5. Todo junto: backtracking con las 4 heurísticas

Ahora se resuelve el mapa completo combinando todo:

* **¿Qué variable?** MRV y, si hay empate, la de mayor grado (heurísticas 1 y 2).
* **¿Qué valor?** El menos restrictivo (heurística 3).
* **¿Cómo detectar fallos pronto?** Forward checking (heurística 4).

#### Instrucciones de Python que usarás

* **`global pasos`**
    * *Tipo:* Definición.
    * *Explicación:* Permite que la función modifique una variable definida fuera de ella. Aquí se usa para contar cuántos valores se prueban en total.
* **Función recursiva**
    * *Tipo:* Definición.
    * *Explicación:* Una función que se llama a sí misma. Cada llamada asigna una variable más; si un camino falla, la función regresa (`return None`) y la llamada anterior prueba el siguiente valor. Eso es el *backtracking*.

### 5.1 Backtracking con heurísticas

Completa las líneas marcadas con el número de heurística.

```python
pasos = 0

def backtracking(asignacion, dominios):
    global pasos
    if len(asignacion) == len(VARIABLES):
        return asignacion

    # Heurísticas 1 y 2: elegir la variable
    no_asignadas = [v for v in VARIABLES if v not in asignacion]
    var = min(no_asignadas, key=lambda v: (len(dominios[v]), -___(v, asignacion)))

    # Heurística 3: ordenar los valores (menos restrictivo primero)
    valores = sorted(dominios[var],
                     key=lambda val: sum(val in dominios[n] for n in VECINOS[var] if n not in asignacion))

    for valor in valores:
        pasos += 1
        # Heurística 4: forward checking
        nuevos, vacia = ___(var, valor, dominios)
        print(f"Paso {pasos}: {var} = {valor}")
        if vacia is None:
            resultado = backtracking({**asignacion, var: valor}, ___)
            if resultado is not None:
                return resultado
        print(f"   Regreso: {var} = {valor} no funciona")
    return None

solucion = backtracking({}, {v: list(COLORES) for v in VARIABLES})
print(f"\nSolución en {pasos} pasos:")
for v in VARIABLES:
    print(f"{v:>4}: {solucion[v]}")
```

<details><summary><b>Pista</b></summary>

Al llamar a la recursión hay que pasarle los dominios **ya reducidos** por forward checking, no los originales.

</details>

<details><summary><b>Ver solución</b></summary>

```python
pasos = 0

def backtracking(asignacion, dominios):
    global pasos
    if len(asignacion) == len(VARIABLES):
        return asignacion

    # Heurísticas 1 y 2: elegir la variable
    no_asignadas = [v for v in VARIABLES if v not in asignacion]
    var = min(no_asignadas, key=lambda v: (len(dominios[v]), -grado(v, asignacion)))

    # Heurística 3: ordenar los valores (menos restrictivo primero)
    valores = sorted(dominios[var],
                     key=lambda val: sum(val in dominios[n] for n in VECINOS[var] if n not in asignacion))

    for valor in valores:
        pasos += 1
        # Heurística 4: forward checking
        nuevos, vacia = forward_checking(var, valor, dominios)
        print(f"Paso {pasos}: {var} = {valor}")
        if vacia is None:
            resultado = backtracking({**asignacion, var: valor}, nuevos)
            if resultado is not None:
                return resultado
        print(f"   Regreso: {var} = {valor} no funciona")
    return None

solucion = backtracking({}, {v: list(COLORES) for v in VARIABLES})
print(f"\nSolución en {pasos} pasos:")
for v in VARIABLES:
    print(f"{v:>4}: {solucion[v]}")
```

</details>

**Verifica tu resultado:** El primer paso es **SA = rojo**. La solución se encuentra en **7 pasos**, sin ningún regreso: WA azul, NT verde, Q azul, NSW verde, V azul, SA rojo y T rojo.

### 5.2 Comparar contra un backtracking sin heurísticas

Esta versión ya está completa: recorre las variables en un **orden fijo**, prueba los colores siempre en el mismo orden y no hace forward checking. Ejecútala con dos órdenes distintos y compara el número de pasos.

```python
def backtracking_simple(asignacion, orden):
    global pasos
    if len(asignacion) == len(orden):
        return asignacion
    var = [v for v in orden if v not in asignacion][0]     # siguiente en el orden fijo
    for valor in COLORES:                                    # colores siempre en el mismo orden
        pasos += 1
        if es_consistente(var, valor, asignacion):
            resultado = backtracking_simple({**asignacion, var: valor}, orden)
            if resultado is not None:
                return resultado
    return None

for orden in [VARIABLES, ["T", "WA", "NSW", "NT", "V", "Q", "SA"]]:
    pasos = 0
    backtracking_simple({}, orden)
    print(f"Orden {orden}: {pasos} pasos")
```

**Verifica tu resultado:** con el orden de la lista `VARIABLES` se necesitan **11 pasos**; con el orden `T, WA, NSW, NT, V, Q, SA` se necesitan **44 pasos**. Con las heurísticas fueron 7.

#### Hints para interpretar tus resultados

* **Hint de orden:** Con el mismo algoritmo, cambiar solo el orden de las variables cuadruplicó los pasos (de 11 a 44). *¿Qué tiene de malo dejar SA hasta el final? ¿Y empezar por T?*
* **Hint de escala:** El mapa de Australia tiene 7 variables y 3^7 = 2,187 asignaciones posibles. *¿Cuántas habría en un mapa de los 32 estados de México con 4 colores? ¿Qué tan importante se vuelve una buena heurística?*
* **Hint de costo por paso:** Un paso con heurísticas hace más cálculos que un paso sin ellas. *¿Siempre conviene usarlas? ¿En qué tipo de problemas crees que el ahorro es mayor?*

**Tu interpretación:**

*(Escribe aquí tus conclusiones de esta parte.)*

---
## Cierre de la práctica

Antes de redactar tu reporte, responde:

* ¿Cuáles de las cuatro heurísticas deciden **qué variable** asignar, cuál decide **qué valor** probar y cuál sirve para **detectar fallos**?
* ¿Por qué MRV busca la variable más restringida y LCV el valor menos restrictivo? ¿Se contradicen?
* ¿Qué limitación tiene forward checking que se vio en la Parte 4?
* ¿Cómo cambió el número de pasos al usar las heurísticas frente al backtracking sin ellas?
* ¿Qué otros problemas reales se pueden modelar como un CSP? (Piensa en horarios de clases, asignación de salones o turnos de trabajo.)

**Ejercicios adicionales:**

1. Agrega al diccionario `VECINOS` un estado nuevo, `"ACT"`, vecino solo de `"NSW"` (no olvides agregarlo también a la lista de vecinos de NSW y a `VARIABLES`). ¿Cómo cambia el grado de NSW? ¿Cambia el número de pasos de la Parte 5?
2. Repite la Parte 3 con la asignación `{"WA": "rojo", "SA": "azul"}` y aplica LCV a la variable NT.
3. Cambia `COLORES` a solo dos colores. ¿Qué devuelve el backtracking? ¿En qué paso lo detecta forward checking?

**Lista de evidencias:** este notebook con todas las celdas completas y ejecutadas, las interpretaciones de cada parte y las respuestas del cierre.
