# Ejercicios — AyPI Pascal

---

## Ejercicio 1 — Estructuras de Control

Escribir un programa en Pascal que lea legajos y notas de alumnos.
El programa debe:
- Pedir el legajo del alumno (terminar cuando el legajo sea **0**)
- Para cada alumno, leer sus **3 notas** (valores reales entre 0 y 10)
- Calcular e imprimir el promedio de ese alumno

**Ejemplo de ejecución:**
```
Legajo (0 para terminar): 12345
Nota 1: 7
Nota 2: 8
Nota 3: 6
Promedio alumno 12345: 7.00
Legajo (0 para terminar): 0
```

---

## Ejercicio 2 — Modularización

Modelar la corrección de exámenes de alumnos usando módulos separados.

Implementar:
- Una **función** `calcularPromedio` que reciba 3 notas reales y devuelva su promedio.
- Una **función** `estaAprobado` que reciba un promedio real y devuelva `true` si es >= 6.
- Un **procedimiento** `informarResultado` que reciba el nombre del alumno y sus 3 notas, y que utilice las dos funciones anteriores para imprimir: nombre, promedio y si aprobó o no.

Luego escribir un programa principal que lea datos de 5 alumnos y llame al procedimiento para cada uno.

> **Pensar:** ¿qué parámetros necesitan `var` y cuáles no? ¿Por qué?

---

## Ejercicio 3 — Vectores y Registros

Definir un tipo `TEmpleado` con los campos: `nombre` (string), `categoria` (char: A, B, C, D o E) y `salario` (real).

Escribir un programa que:
1. Lea hasta 100 empleados (terminar cuando el nombre sea `'FIN'`)
2. Implemente un **procedimiento** `contarPorCategoria` que reciba el vector de empleados y la cantidad, y llene un **vector contador** con cuántos empleados hay por cada categoría (A, B, C, D, E)
3. Imprima los contadores por cada categoría

> **Tip:** un vector contador puede indexarse con caracteres: `cont['A']`, `cont['B']`, etc.

---

## Ejercicio 4 — Listas Enlazadas

Dado el siguiente tipo:

```pascal
type
  PNodo = ^TNodo;
  TNodo = record
    valor: integer;
    siguiente: PNodo;
  end;
```

Y asumiendo que ya existe una lista cargada con enteros:

1. Implementar un **procedimiento** `contarElementos` que reciba la lista y devuelva (por referencia) la cantidad de nodos.
2. Implementar un **procedimiento** `mostrarLista` que imprima todos los valores de la lista.
3. Implementar una **función** `buscarEnLista` que reciba la lista y un valor entero, y devuelva `true` si el valor está en la lista.

> **Pensar:** ¿cuáles de estos módulos necesitan `lista` con `var` y cuáles no?

---

## Ejercicio 5 ★ — Corte de Control

Se dispone de un archivo de inmuebles ordenados por **localidad**.
Cada línea tiene: `localidad` (string) y `precio` (real).
La entrada termina con la localidad `'FIN'`.

Escribir un programa que:
1. Para cada localidad, imprima la **suma total de precios** de esa localidad
2. Al final, imprima el **total general** de todos los inmuebles

**Ejemplo de datos de entrada:**
```
La Plata  150000
La Plata  200000
Berisso   90000
Berisso   110000
Berisso   75000
Ensenada  130000
FIN       0
```

**Salida esperada:**
```
Localidad: La Plata  - Total: $ 350000.00
Localidad: Berisso   - Total: $ 275000.00
Localidad: Ensenada  - Total: $ 130000.00
Total general:        $ 755000.00
```

> **Tip:** ¿Qué pasa con la última localidad después de que termina el while?
