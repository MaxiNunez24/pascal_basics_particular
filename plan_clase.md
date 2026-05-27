# Plan de Clase — Parcial AyPI (Pascal)
**Duración estimada:** ~85 minutos  
**Nivel:** Ciencia de Datos UNLP — primer parcial

---

## ANTES DE EMPEZAR (5 min) — Diagnóstico rápido

Hacerle estas preguntas al alumno para saber dónde poner el foco:

1. "¿Cuál es el tema que más te genera dudas?"
2. "¿Podés explicarme la diferencia entre pasar un parámetro por valor y por referencia?"
3. "¿Sabés qué es el corte de control?"

**Si ya maneja bien FOR/WHILE/IF:** pasar rápido el Bloque 1 (5 min) e invertir más en Listas y Corte de Control.  
**Si duda en modularización:** ser generoso con el Bloque 2.

---

## BLOQUE 1 — Estructuras de Control (10 min)

### Explicación clave

| Estructura | Cuándo usarla |
|---|---|
| `FOR` | Cantidad de iteraciones **conocida de antemano** ("los 7 días de la semana") |
| `WHILE` | Condición variable, cantidad desconocida ("hasta que ingrese -1") |
| `IF-THEN-ELSE` | Tomar una decisión según una condición |

**Patrón WHILE con centinela** (escribirlo en papel/pizarrón):
```
leer PRIMER valor        ← ¡Este read va AFUERA del while!
WHILE valor <> CENTINELA DO
BEGIN
  procesar valor
  leer SIGUIENTE valor   ← Al final del ciclo
END
```
⚠️ **Error más común:** poner el único `read` dentro del `while` y nunca arrancar, o leer al principio y olvidar leer al final.

### Ejercicio 1 para el alumno (5 min)
*(ver `ejercicios_alumno.md` — Ejercicio 1)*

**Solución:**
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

## BLOQUE 2 — Modularización (20 min)

### Explicación clave

**FUNCTION vs PROCEDURE:**
- `function`: devuelve **un único valor**, se usa en expresiones (`x := calcularPromedio(...)`)
- `procedure`: realiza una tarea, puede devolver 0 o más valores a través de parámetros `var`

**Pasaje por VALOR vs por REFERENCIA:**

```
Por VALOR:          Por REFERENCIA (var):
┌─────────┐         ┌─────────┐
│ x = 10  │ ─copia─▶│ local=10│         │ x = 10  │ ─dirección─▶ comparten
└─────────┘         └─────────┘         └─────────┘              la misma celda
 original            copia local         original
 NO cambia           puede cambiar        SÍ cambia si el módulo la modifica
```

**Analogía:** Por valor = te anoto el número de teléfono en un papel (tengo una copia). Por referencia = te comparto mi celular (modificaciones afectan el original).

**Regla de oro para el parcial:** `var` cuando el módulo necesita *modificar* el dato o *devolverlo*. Sin `var` cuando solo lo *lee*.

⚠️ **NO variables globales.** Toda comunicación entre el programa principal y los módulos: solo por parámetros.

### Ejercicio 2 para el alumno (8 min)
*(ver `ejercicios_alumno.md` — Ejercicio 2)*

**Solución:**
```pascal
function calcularPromedio(n1, n2, n3: real): real;
begin
  calcularPromedio := (n1 + n2 + n3) / 3;
end;

function estaAprobado(prom: real): boolean;
begin
  estaAprobado := prom >= 6.0;
end;

procedure informarResultado(nombre: string; n1, n2, n3: real);
var
  prom: real;
begin
  prom := calcularPromedio(n1, n2, n3);
  write('Alumno: ', nombre, ' — Promedio: ', prom:5:2, ' — ');
  if estaAprobado(prom) then
    writeln('APROBADO')
  else
    writeln('DESAPROBADO');
end;
```
Pedir al alumno que explique por qué `nombre`, `n1`, `n2`, `n3` van sin `var`.

---

## BLOQUE 3 — Vectores, Registros y Algoritmos (20 min)

### Explicación clave

**Registro:** agrupa datos de tipos *distintos* bajo un mismo nombre (ficha de un empleado).  
**Vector:** colección de datos del *mismo tipo*, acceso por índice.

**Dimensión física vs lógica:**
```pascal
const MAX = 100;
var
  v: array[1..MAX] of integer;  { física: siempre 100 }
  n: integer;                    { lógica: cuántos realmente se usan }
```
⚠️ Siempre pasar `n` (la dimensión lógica) a los módulos, nunca `MAX`.

**Búsqueda en vector ORDENADO** (la versión eficiente que piden):
```pascal
{ Se detiene en cuanto supera el valor buscado }
i := 1;
encontrado := false;
superado := false;
while (i <= n) and (not encontrado) and (not superado) do
begin
  if v[i] = x then
    encontrado := true
  else if v[i] > x then
    superado := true
  else
    i := i + 1;
end;
{ resultado: si encontrado, el índice es i }
```

**Ordenación por Selección** (el método que enseñan):
```
Para i de 1 a n-1:
  Buscar el índice del mínimo entre posición i y n
  Intercambiar v[i] con v[iMin]
```
```pascal
procedure seleccion(var v: TVector; n: integer);
var
  i, j, iMin, aux: integer;
begin
  for i := 1 to n - 1 do
  begin
    iMin := i;
    for j := i + 1 to n do
      if v[j] < v[iMin] then
        iMin := j;
    aux := v[i];
    v[i] := v[iMin];
    v[iMin] := aux;
  end;
end;
```

### Ejercicio 3 para el alumno (10 min)
*(ver `ejercicios_alumno.md` — Ejercicio 3)*

**Solución:**
```pascal
const MAX = 100;
type
  TEmpleado = record
    nombre: string;
    categoria: char;
    salario: real;
  end;
  TEmpleados = array[1..MAX] of TEmpleado;
  TContadores = array['A'..'E'] of integer;

procedure inicializarContadores(var cont: TContadores);
var c: char;
begin
  for c := 'A' to 'E' do
    cont[c] := 0;
end;

procedure contarPorCategoria(emp: TEmpleados; n: integer; var cont: TContadores);
var i: integer;
begin
  inicializarContadores(cont);
  for i := 1 to n do
    cont[emp[i].categoria] := cont[emp[i].categoria] + 1;
end;
```

---

## BLOQUE 4 — Listas Enlazadas (15 min)

### Explicación clave

**Vector vs Lista:**
| | Vector | Lista |
|---|---|---|
| Tamaño | Fijo (MAX) | Dinámico |
| Acceso | Directo por índice | Secuencial (recorrer) |
| Inserción | Costosa (desplazar) | Barata |

**Librería GenericLinkedList** — se importa y se especializa para el tipo que necesitamos:
```pascal
uses GenericLinkedList;
type
  ListaE = specialize LinkedList<integer>;
  { Para registros: specialize LinkedList<TInmueble> }
```

**Crear + agregar:**
```pascal
var L: ListaE;
begin
  L := ListaE.create();  { lista vacía }
  L.add(10);             { agrega al final }
  L.addFirst(10);        { agrega al inicio }
end.
```

**Patrón de recorrido — los 4 métodos:**
```pascal
L.reset();
while not L.eol() do
begin
  { procesar L.current() }
  L.next();
end;
```
⚠️ Si se olvida `L.next()` → bucle infinito.

**Regla VAR para listas:**
- `var L` → el módulo ejecuta `L := ListaE.create()` (reasigna la variable)
- Sin `var` → el módulo solo llama métodos (`reset`, `add`, `current`, etc.)

**armarLista (con VAR):**
```pascal
procedure armarLista(var L: ListaE);
var num: integer;
begin
  L := ListaE.create();
  read(num);
  while num <> -1 do
  begin
    L.add(num);
    read(num);
  end;
end;
```

**sumarLista (sin VAR):**
```pascal
function sumarLista(L: ListaE): integer;
var suma: integer;
begin
  suma := 0;
  L.reset();
  while not L.eol() do
  begin
    suma := suma + L.current();
    L.next();
  end;
  sumarLista := suma;
end;
```

### Ejercicio 4 para el alumno (5 min)
*(ver ejercicios — Ejercicio 4)*

Pedir que escriba `buscarEnLista(L: ListaE; x: integer): boolean`.

**Solución:**
```pascal
function buscarEnLista(L: ListaE; x: integer): boolean;
var encontrado: boolean;
begin
  encontrado := false;
  L.reset();
  while (not L.eol()) and (not encontrado) do
  begin
    if L.current() = x then
      encontrado := true
    else
      L.next();
  end;
  buscarEnLista := encontrado;
end;
```
Preguntar: "¿Va `var L`?" → No, porque no reasigna L.

---

## BLOQUE 5 ★ — Corte de Control (15 min)

### Concepto clave
Los datos llegan **ordenados** por un criterio (ej: localidad, departamento).
Cuando ese criterio *cambia*, terminamos de procesar el grupo anterior e informamos.

### Patrón (escribirlo en papel, el alumno debe memorizarlo):
```pascal
{ 1. Leer el PRIMER dato }
readln(criterio, valor);
criterioActual := criterio;
{ 2. Inicializar acumuladores del primer grupo }
acumulador := 0;

while criterio <> CENTINELA do
begin
  if criterio = criterioActual then
    { MISMO GRUPO: acumular }
    acumulador := acumulador + valor
  else
  begin
    { CAMBIO DE GRUPO }
    { a) Informar el grupo que terminó }
    writeln('Grupo ', criterioActual, ': ', acumulador);
    totalGeneral := totalGeneral + acumulador;
    { b) Actualizar criterio y reiniciar acumulador }
    criterioActual := criterio;
    acumulador := valor;  { ← el dato actual pertenece al NUEVO grupo }
  end;
  readln(criterio, valor);
end;

{ 3. ¡NUNCA olvidar informar el ÚLTIMO grupo! }
writeln('Grupo ', criterioActual, ': ', acumulador);
totalGeneral := totalGeneral + acumulador;
```

⚠️ **Error clásico del parcial:** olvidar el último grupo después del `while`.  
⚠️ **Otro error:** no usar el dato del nuevo grupo cuando hay cambio (poner `acumulador := 0` en vez de `acumulador := valor`).

### Ejercicio 5 para el alumno (10 min)
*(ver `ejercicios_alumno.md` — Ejercicio 5)*

**Solución:**
```pascal
program CortePorLocalidad;
var
  localidad, localidadActual: string;
  precio, totalLocalidad, totalGeneral: real;
begin
  totalGeneral := 0;
  readln(localidad, precio);
  localidadActual := localidad;
  totalLocalidad := 0;

  while localidad <> 'FIN' do
  begin
    if localidad = localidadActual then
      totalLocalidad := totalLocalidad + precio
    else
    begin
      writeln('Localidad: ', localidadActual, ' - Total: $', totalLocalidad:8:2);
      totalGeneral := totalGeneral + totalLocalidad;
      localidadActual := localidad;
      totalLocalidad := precio;
    end;
    readln(localidad, precio);
  end;

  { Último grupo }
  writeln('Localidad: ', localidadActual, ' - Total: $', totalLocalidad:8:2);
  totalGeneral := totalGeneral + totalLocalidad;
  writeln('Total general: $', totalGeneral:10:2);
end.
```

---

## CIERRE (5 min)

### Los 5 recordatorios más importantes para el parcial:

1. **No variables globales** — todo por parámetros
2. **WHILE con centinela** — el primer `read` va afuera del loop
3. **Búsqueda en vector ordenado** — parar si el elemento actual ya superó al buscado
4. **Listas** — `var L` solo cuando el módulo reasigna L con `create()`, no cuando solo llama métodos
5. **Corte de control** — siempre informar el último grupo después del `while`

### Darle al alumno el archivo `resumen_parcial.md` para imprimir o abrir en pantalla.

---

## Recomendaciones pedagógicas

- **Hacelo escribir**, no solo escuchar. Que tipee cada ejercicio o lo haga en papel.
- **Preguntá antes de explicar:** "¿Por qué creés que esto va con VAR?" activa más que explicar directo.
- **No te trabés en un tema.** Si no entiende algo en 5 min, da un hint, seguí adelante y volvé al final.
- **Priorizá** si te quedás sin tiempo: Corte de Control y Listas son los temas más complejos y recurrentes.
