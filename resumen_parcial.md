# Resumen para el Parcial — AyPI Pascal
### Ciencia de Datos — UNLP

---

## 1. Estructuras de Control

### IF-THEN-ELSE
```pascal
if condicion then
  instruccion_si_verdad
else
  instruccion_si_falso;

{ Con bloque de instrucciones: }
if condicion then
begin
  instruccion1;
  instruccion2;
end
else
begin
  instruccion3;
end;
```

### FOR — cantidad de iteraciones conocida
```pascal
for i := 1 to n do
begin
  { se ejecuta exactamente n veces }
end;
```

### WHILE — cantidad de iteraciones desconocida (con centinela)
```pascal
{ ⚠️ El primer read va AFUERA del while }
read(valor);
while valor <> CENTINELA do
begin
  { procesar valor }
  read(valor);  { ⚠️ siempre al final del ciclo }
end;
```

---

## 2. Modularización

### Función — devuelve UN valor
```pascal
function nombreFuncion(param1: tipo1; param2: tipo2): tipoRetorno;
var
  { variables locales }
begin
  { ... }
  nombreFuncion := valorDeRetorno;  { ← así se devuelve el valor }
end;
```

### Procedimiento — realiza una tarea, puede devolver por VAR
```pascal
procedure nombreProcedimiento(entradaValor: tipo1; var salidaRef: tipo2);
begin
  { entradaValor: solo lectura (copia) }
  { salidaRef: se puede modificar (afecta al original) }
end;
```

### Cuándo usar VAR:
| ¿El módulo necesita...? | Usar |
|---|---|
| Solo leer el dato | Sin `var` (por valor) |
| Modificar el dato o devolverlo | Con `var` (por referencia) |

### Regla de oro: NO variables globales
Toda comunicación entre el programa y los módulos: **solo por parámetros**.

---

## 3. Registros y Vectores

### Registro (record)
```pascal
type
  TInmueble = record
    tipo: string;
    precio: real;
    localidad: string;
  end;

var
  casa: TInmueble;
begin
  casa.tipo := 'casa';       { acceso con punto }
  casa.precio := 150000;
end.
```

### Vector (array) — con dimensión lógica
```pascal
const MAX = 100;
type
  TVector = array[1..MAX] of integer;
var
  v: TVector;
  n: integer;  { dimensión LÓGICA: cuántos se usan realmente }
```
⚠️ Siempre pasar `n` a los módulos, no `MAX`.

### Vector de registros
```pascal
type
  TEmpleados = array[1..MAX] of TEmpleado;
var
  empleados: TEmpleados;
  n: integer;
begin
  empleados[1].nombre := 'Ana';      { acceso: índice + campo }
  empleados[1].salario := 50000;
end.
```

---

## 4. Algoritmos sobre Vectores

### Búsqueda sin orden (recorre todo)
```pascal
i := 1;
encontrado := false;
while (i <= n) and (not encontrado) do
begin
  if v[i] = x then
    encontrado := true
  else
    i := i + 1;
end;
```

### Búsqueda con orden (eficiente ★)
```pascal
{ Se detiene ni bien supera el valor buscado }
i := 1;
encontrado := false;
superado := false;
while (i <= n) and (not encontrado) and (not superado) do
begin
  if v[i] = x then
    encontrado := true
  else if v[i] > x then
    superado := true         { ya no tiene sentido seguir }
  else
    i := i + 1;
end;
```

### Ordenación por Selección
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
    { intercambiar v[i] con v[iMin] }
    aux := v[i];
    v[i] := v[iMin];
    v[iMin] := aux;
  end;
end;
```

---

## 5. Listas Enlazadas

### Declaración (siempre en este orden)
```pascal
type
  PNodo = ^TNodo;      { primero el puntero }
  TNodo = record
    dato: TipoDato;
    siguiente: PNodo;
  end;
var
  lista: PNodo;
begin
  lista := nil;        { lista vacía }
end.
```

### Agregar adelante (addFirst)
```pascal
procedure addFirst(var lista: PNodo; dato: TipoDato);
var nuevo: PNodo;
begin
  new(nuevo);                    { crear nodo }
  nuevo^.dato := dato;
  nuevo^.siguiente := lista;     { el nuevo apunta al anterior primero }
  lista := nuevo;                { el nuevo es la nueva cabecera }
end;
```

### Agregar al final (add)
```pascal
procedure add(var lista: PNodo; dato: TipoDato);
var nuevo, actual: PNodo;
begin
  new(nuevo);
  nuevo^.dato := dato;
  nuevo^.siguiente := nil;
  if lista = nil then
    lista := nuevo
  else
  begin
    actual := lista;
    while actual^.siguiente <> nil do
      actual := actual^.siguiente;
    actual^.siguiente := nuevo;
  end;
end;
```

### Recorrer una lista
```pascal
actual := lista;
while actual <> nil do
begin
  writeln(actual^.dato);
  actual := actual^.siguiente;
end;
```

### Cuándo usar VAR en listas:
- `lista` con `var`: si el módulo **modifica la cabecera** (addFirst, add a lista vacía, borrar primero)
- `lista` sin `var`: si el módulo solo **recorre** (mostrar, buscar, contar)

---

## 6. Corte de Control ★

### ¿Cuándo usarlo?
Cuando los datos vienen **ordenados por un criterio** y hay que procesar por grupos.

### Patrón (memorizar)
```pascal
{ 1. Leer el primer dato }
readln(criterio, valor);
criterioActual := criterio;
acumulador := 0;
totalGeneral := 0;

while criterio <> 'FIN' do
begin
  if criterio = criterioActual then
    { mismo grupo: acumular }
    acumulador := acumulador + valor
  else
  begin
    { cambio de grupo }
    writeln(criterioActual, ': ', acumulador);   { a) informar el anterior }
    totalGeneral := totalGeneral + acumulador;
    criterioActual := criterio;                  { b) actualizar criterio }
    acumulador := valor;                         { c) el dato actual → nuevo grupo }
  end;
  readln(criterio, valor);
end;

{ ⚠️ Informar el ÚLTIMO grupo (siempre, nunca olvidar) }
writeln(criterioActual, ': ', acumulador);
totalGeneral := totalGeneral + acumulador;
writeln('Total general: ', totalGeneral);
```

---

## Errores típicos del parcial

| Error | Corrección |
|---|---|
| Única lectura dentro del `while` | Leer antes del `while` y al final de cada iteración |
| Olvidar informar el último grupo en corte de control | Siempre informar después del `while` |
| Poner `acumulador := 0` al cambiar de grupo | Debe ser `acumulador := valor` (el dato actual es del nuevo grupo) |
| Variables globales | Todo por parámetros |
| `lista` sin `var` cuando se modifica la cabecera | Agregar `var` cuando se cambia quién es el primer nodo |
| Búsqueda en vector ordenado sin aprovechar el orden | Agregar condición `not superado` al `while` |
| Declarar `TNodo` antes que `PNodo` | `PNodo = ^TNodo` siempre va primero |

---

## Recordatorios finales

- **FUNCTION** → un único valor de retorno → se usa en expresiones
- **PROCEDURE** → tarea, puede devolver varios valores por `var`
- **VAR en parámetros** → cuando el módulo modifica el dato o lo devuelve
- **Dimensión lógica `n`** → siempre pasarla a los módulos, no `MAX`
- **Corte de control** → informar el último grupo siempre
- **Lista vacía** → `lista := nil`
- **Nuevo nodo** → `new(nodo)`
- **Acceso a campo de registro por puntero** → `nodo^.campo`
