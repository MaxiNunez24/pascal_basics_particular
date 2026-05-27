# 🏋️ Ejercicios

Los ejercicios están ordenados del más simple al más complejo. Intentá resolver cada uno antes de ver la solución.

---

## Ejercicio 1 — Estructuras de Control ⭐

Escribí un programa que lea legajos y notas de alumnos.  
El programa termina cuando el legajo es **0**.  
Para cada alumno, leer sus **3 notas** (reales entre 0 y 10) y mostrar su promedio.

**Ejemplo:**
```
Legajo (0 para terminar): 12345
Nota 1: 7
Nota 2: 8
Nota 3: 6
Promedio alumno 12345: 7.00
Legajo (0 para terminar): 0
```

??? tip "💡 Pista"
    ¿Qué estructura de repetición corresponde usar si no sabés cuántos alumnos hay?  
    ¿Dónde va el primer `readln(legajo)`?

??? success "✅ Solución"
    ```pascal
    program PromedioAlumnos;
    var
      legajo: integer;
      nota1, nota2, nota3, promedio: real;
    begin
      write('Legajo (0 para terminar): ');
      readln(legajo);

      while legajo <> 0 do
      begin
        write('Nota 1: '); readln(nota1);
        write('Nota 2: '); readln(nota2);
        write('Nota 3: '); readln(nota3);
        promedio := (nota1 + nota2 + nota3) / 3;
        writeln('Promedio alumno ', legajo, ': ', promedio:5:2);

        write('Legajo (0 para terminar): ');
        readln(legajo);
      end;
    end.
    ```

---

## Ejercicio 2 — Modularización ⭐⭐

Implementar los siguientes módulos:

- `function calcularPromedio(n1, n2, n3: real): real` — devuelve el promedio de 3 notas
- `function estaAprobado(prom: real): boolean` — devuelve `true` si el promedio es >= 6
- `procedure informarResultado(nombre: string; n1, n2, n3: real)` — usa las dos funciones anteriores e imprime el resultado del alumno

Luego escribir un programa principal que lea datos de **5 alumnos** y llame al procedimiento para cada uno.

!!! question "Para pensar antes de escribir"
    - ¿Qué parámetros necesitan `var` y cuáles no?
    - ¿Podría `calcularPromedio` ser un `procedure`? ¿Qué tendría que cambiar?

??? success "✅ Solución"
    ```pascal
    program Modularizacion;

    function calcularPromedio(n1, n2, n3: real): real;
    begin
      calcularPromedio := (n1 + n2 + n3) / 3;
    end;

    function estaAprobado(prom: real): boolean;
    begin
      estaAprobado := prom >= 6.0;
    end;

    procedure informarResultado(nombre: string; n1, n2, n3: real);
    var prom: real;
    begin
      prom := calcularPromedio(n1, n2, n3);
      write('Alumno: ', nombre, ' — Promedio: ', prom:5:2, ' — ');
      if estaAprobado(prom) then
        writeln('APROBADO')
      else
        writeln('DESAPROBADO');
    end;

    var
      i: integer;
      nombre: string;
      n1, n2, n3: real;
    begin
      for i := 1 to 5 do
      begin
        write('Nombre: '); readln(nombre);
        write('Nota 1: '); readln(n1);
        write('Nota 2: '); readln(n2);
        write('Nota 3: '); readln(n3);
        informarResultado(nombre, n1, n2, n3);
      end;
    end.
    ```

---

## Ejercicio 3 — Vectores y Registros ⭐⭐

Definir un tipo `TEmpleado` con los campos: `nombre` (string), `categoria` (char: A..E) y `salario` (real).

1. Leer hasta 100 empleados (terminar cuando el nombre sea `'FIN'`)
2. Implementar un **procedimiento** `contarPorCategoria` que reciba el vector y la cantidad, y llene un vector contador con cuántos hay por categoría
3. Imprimir los contadores para A, B, C, D y E

??? tip "💡 Pista"
    Un vector contador puede indexarse directamente con un `char`: `cont['A']`, `cont['B']`, etc.
    ```pascal
    type TContadores = array['A'..'E'] of integer;
    ```

??? success "✅ Solución"
    ```pascal
    program Empleados;

    const MAX = 100;
    type
      TEmpleado   = record
        nombre   : string;
        categoria: char;
        salario  : real;
      end;
      TEmpleados  = array[1..MAX] of TEmpleado;
      TContadores = array['A'..'E'] of integer;

    procedure contarPorCategoria(emp: TEmpleados; n: integer; var cont: TContadores);
    var i: integer; c: char;
    begin
      for c := 'A' to 'E' do cont[c] := 0;
      for i := 1 to n do
        cont[emp[i].categoria] := cont[emp[i].categoria] + 1;
    end;

    var
      emp : TEmpleados;
      cont: TContadores;
      n   : integer;
      c   : char;
    begin
      n := 0;
      readln(emp[n+1].nombre);
      while emp[n+1].nombre <> 'FIN' do
      begin
        n := n + 1;
        readln(emp[n].categoria);
        readln(emp[n].salario);
        readln(emp[n+1].nombre);
      end;

      contarPorCategoria(emp, n, cont);

      for c := 'A' to 'E' do
        writeln('Categoría ', c, ': ', cont[c], ' empleados');
    end.
    ```

---

## Ejercicio 4 — Listas Enlazadas ⭐⭐⭐

Dado el tipo:
```pascal
type
  PNodo = ^TNodo;
  TNodo = record
    valor    : integer;
    siguiente: PNodo;
  end;
```

Implementar:

1. `procedure contarElementos(lista: PNodo; var cant: integer)` — cuenta los nodos
2. `procedure mostrarLista(lista: PNodo)` — imprime todos los valores
3. `function buscarEnLista(lista: PNodo; x: integer): boolean` — devuelve `true` si el valor está

!!! question "Para pensar"
    ¿Cuáles de estos tres módulos necesitan `lista` con `var`? ¿Por qué?

??? success "✅ Solución"
    ```pascal
    procedure contarElementos(lista: PNodo; var cant: integer);
    var actual: PNodo;
    begin
      cant   := 0;
      actual := lista;
      while actual <> nil do
      begin
        cant   := cant + 1;
        actual := actual^.siguiente;
      end;
    end;

    procedure mostrarLista(lista: PNodo);
    var actual: PNodo;
    begin
      actual := lista;
      while actual <> nil do
      begin
        writeln(actual^.valor);
        actual := actual^.siguiente;
      end;
    end;

    function buscarEnLista(lista: PNodo; x: integer): boolean;
    var actual: PNodo;
    begin
      actual := lista;
      while (actual <> nil) and (actual^.valor <> x) do
        actual := actual^.siguiente;
      buscarEnLista := actual <> nil;
    end;
    ```
    Ninguno necesita `var lista` porque ninguno modifica quién es la cabecera.
    `cant` sí necesita `var` porque es el valor que se devuelve.

---

## Ejercicio 5 — Corte de Control ⭐⭐⭐

Se dispone de inmuebles **ordenados por localidad**. Cada línea tiene: localidad (string) y precio (real). La entrada termina con `'FIN'`.

Escribir un programa que imprima el **total de precios por localidad** y al final el **total general**.

**Datos de entrada:**
```
La Plata   150000
La Plata   200000
Berisso     90000
Berisso    110000
Berisso     75000
Ensenada   130000
FIN              0
```

**Salida esperada:**
```
Localidad: La Plata  - Total: $ 350000.00
Localidad: Berisso   - Total: $ 275000.00
Localidad: Ensenada  - Total: $ 130000.00
Total general:        $ 755000.00
```

??? tip "💡 Pista"
    Aplicar el patrón de corte de control. Acordate de:
    1. Leer el primer dato **antes** del while
    2. Al cambiar de grupo: informar el anterior, actualizar criterio, y poner `acumulador := precio` (no `:= 0`)
    3. Informar el **último grupo** después del while

??? success "✅ Solución"
    ```pascal
    program CortePorLocalidad;
    var
      localidad, localidadActual: string;
      precio, totalLocalidad, totalGeneral: real;
    begin
      totalGeneral := 0;

      readln(localidad, precio);
      localidadActual := localidad;
      totalLocalidad  := 0;

      while localidad <> 'FIN' do
      begin
        if localidad = localidadActual then
          totalLocalidad := totalLocalidad + precio
        else
        begin
          writeln('Localidad: ', localidadActual, ' - Total: $', totalLocalidad:10:2);
          totalGeneral    := totalGeneral + totalLocalidad;
          localidadActual := localidad;
          totalLocalidad  := precio;
        end;
        readln(localidad, precio);
      end;

      { Último grupo }
      writeln('Localidad: ', localidadActual, ' - Total: $', totalLocalidad:10:2);
      totalGeneral := totalGeneral + totalLocalidad;
      writeln('Total general:         $', totalGeneral:10:2);
    end.
    ```
