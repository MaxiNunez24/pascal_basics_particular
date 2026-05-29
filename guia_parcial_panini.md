# Guía Docente — Práctica Parcial Panini
**Examen de referencia:** AyPI Comisión 2 – Primera Fecha – 28-05-2026  
**Duración de la clase:** 60 minutos

---

## ANTES DE EMPEZAR (5 min) — Leer el enunciado juntos

**Programa principal dado:**
```pascal
begin { prog. principal }
  cargarLista(...);           { dado }
  generarNuevaLista(...);     { ítem a }
  Writeln(categoriaMenor(...));  { ítem b — función }
  readln(valor);
  cantidadEliminados(...);    { ítem c — completar }
end.
```

**Preguntas de arranque para el alumno:**
1. "¿Los datos están en un vector o en una lista?" → Lista (dice "almacenada en una lista")
2. "¿Qué tipo devuelve `categoriaMenor`?" → Entero (está en `Writeln`)
3. "`cantidadEliminados` aparece solo, sin `Writeln`. ¿Qué harías vos?" → Agregar `Writeln` al completar el principal

**Diferencias clave respecto al parcial anterior (StreamMusic):**
| | StreamMusic | Panini |
|---|---|---|
| Fuente de datos | Vector | **Lista** |
| Para filtrar | `for i := 1 to n` | `reset + while not eol` |
| Ordenar | Strings no, enteros | **Strings** (`<` funciona igual) |
| Ítem de modificación | Sumar valor a campo | **Eliminar** elementos |
| Categorías | Tipos 1..8, buscar top 2 | Categorías 1..5, buscar **mínimo** |

---

## PASO 1 — Declarar los tipos (8 min)

```pascal
uses GenericLinkedList;

const MAX = 500;

type
  { Lista principal — datos del enunciado }
  TAlbum = record
    codigo   : integer;
    duenio   : string;
    pais     : string;
    categoria: integer;   { 1..5 }
    figuritas: integer;
  end;
  ListaAlbumes = specialize LinkedList<TAlbum>;

  { Ítem a — solo duenio y figuritas }
  TInfoAlbum = record
    duenio   : string;
    figuritas: integer;
  end;
  ListaInfo = specialize LinkedList<TInfoAlbum>;

  { Ítem b — vector contador }
  TContador = array[1..5] of integer;
```

**Pregunta:** "¿Por qué `TInfoAlbum` no tiene `codigo`, `pais` ni `categoria`?"
→ Porque el enunciado dice que la nueva lista almacena **solamente** el nombre del dueño y la cantidad de figuritas.

---

## PASO 2 — Ítem a: generarNuevaLista (15 min)

### Diferencia principal con el parcial anterior
La fuente **no es un vector** — es una lista. El filtrado usa el patrón de recorrido de lista.

### Estrategia: recorrer lista → filtrar a vector aux → ordenar → construir nueva lista

```pascal
procedure generarNuevaLista(L: ListaAlbumes; var nueva: ListaInfo);
var
  aux     : array[1..MAX] of TInfoAlbum;
  m, i, j : integer;
  iMin    : integer;
  temp    : TInfoAlbum;
  album   : TAlbum;
begin
  { 1. Filtrar de la lista original }
  m := 0;
  L.reset();
  while not L.eol() do
  begin
    album := L.current();
    if album.pais = 'Argentina' then
    begin
      m := m + 1;
      aux[m].duenio    := album.duenio;
      aux[m].figuritas := album.figuritas;
    end;
    L.next();
  end;

  { 2. Ordenar ASCENDENTE por duenio — selección con mínimo de string }
  for i := 1 to m - 1 do
  begin
    iMin := i;
    for j := i + 1 to m do
      if aux[j].duenio < aux[iMin].duenio then   { < compara strings alfabéticamente }
        iMin := j;
    temp      := aux[i];
    aux[i]    := aux[iMin];
    aux[iMin] := temp;
  end;

  { 3. Construir nueva lista }
  nueva := ListaInfo.create();
  for i := 1 to m do
    nueva.add(aux[i]);
end;
```

### Trampas

| Trampa | Corrección |
|---|---|
| Usar `for i := 1 to n` para filtrar | La fuente es una **lista** → `reset + while not eol` |
| `var L` en la lista fuente | No modifica `L`, solo la lee → sin `var` |
| No saber comparar strings | `<` y `>` funcionan igual que con enteros en Pascal |
| Ordenar descendente (confundir con ítem b del anterior) | Este pide **ascendente** → buscar el **mínimo** |

---

## PASO 3 — Ítem b: categoriaMenor (10 min)

### Análisis
- Recibe la lista **original** (no la del ítem a)
- Devuelve el **número de categoría** (1..5) que tiene menos álbumes
- Está en `Writeln(...)` → es una **función** que devuelve `integer`

### Estrategia: vector contador [1..5] → encontrar el mínimo

```pascal
function categoriaMenor(L: ListaAlbumes): integer;
var
  cont   : TContador;
  c      : integer;
  album  : TAlbum;
  minVal : integer;
  minCat : integer;
begin
  { Inicializar y contar }
  for c := 1 to 5 do cont[c] := 0;
  L.reset();
  while not L.eol() do
  begin
    album := L.current();
    cont[album.categoria] := cont[album.categoria] + 1;
    L.next();
  end;

  { Encontrar el mínimo }
  minVal := cont[1];
  minCat := 1;
  for c := 2 to 5 do
    if cont[c] < minVal then
    begin
      minVal := cont[c];
      minCat := c;
    end;

  categoriaMenor := minCat;
end;
```

### Pregunta para el alumno
"En el parcial anterior buscabas el máximo con `>`. ¿Qué cambia acá?"
→ Solo el operador: `<` en vez de `>`. El resto de la estructura es idéntica.

### Trampa principal
Retornar `minVal` (el total de álbumes) en vez de `minCat` (el número de categoría). El enunciado pide **la categoría**, no la cantidad.

---

## PASO 4 — Ítem c: cantidadEliminados (12 min)

### Análisis
- Recibe la lista del **ítem a** y un valor entero
- **Elimina** los elementos donde `figuritas = valor`
- Retorna la cantidad eliminada

### Diferencia clave con elementosModificados (parcial anterior)
| elementosModificados | cantidadEliminados |
|---|---|
| Recorre y **modifica** algunos elementos | Recorre y **omite** algunos elementos |
| Agrega todos a la nueva lista | Agrega **solo los que no coinciden** |

```pascal
function cantidadEliminados(var L: ListaInfo; valor: integer): integer;
var
  nueva : ListaInfo;
  info  : TInfoAlbum;
  cant  : integer;
begin
  nueva := ListaInfo.create();
  cant  := 0;
  L.reset();
  while not L.eol() do
  begin
    info := L.current();
    if info.figuritas = valor then
      cant := cant + 1          { contar pero NO agregar a nueva }
    else
      nueva.add(info);          { agregar solo los que quedan }
    L.next();
  end;
  L := nueva;
  cantidadEliminados := cant;
end;
```

### Trampas

| Trampa | Corrección |
|---|---|
| Agregar todos y solo sumar el contador | Los que coinciden con `valor` **no van** a la nueva lista |
| Olvidar `L := nueva` | La lista original quedaría intacta |
| Sin `var L` | No podría reasignar `L` al llamador |

---

## PASO 5 — Completar el programa principal (5 min)

```pascal
var
  listaAlbumes: ListaAlbumes;
  listaArg    : ListaInfo;
  valor       : integer;
begin
  cargarLista(listaAlbumes);
  generarNuevaLista(listaAlbumes, listaArg);
  Writeln(categoriaMenor(listaAlbumes));
  readln(valor);
  Writeln(cantidadEliminados(listaArg, valor));  { agregar Writeln }
end.
```

**Punto para discutir:** el enunciado muestra `cantidadEliminados(...)` sin `Writeln`. La nota dice "complete lo que haga falta" → el alumno debe agregar el `Writeln` para que el resultado se muestre.

---

## Los 5 puntos clave para el parcial

1. **Fuente es una lista** → filtrar con `reset + while not eol`, no con `for`
2. **Ordenar strings** → `<` y `>` funcionan igual que con enteros
3. **Eliminar = reconstruir sin los que coinciden** → agregar a nueva lista solo los que NO matchean
4. **categoriaMenor retorna la categoría, no el total** → guardar `minCat`, no `minVal`
5. **`var L` cuando reasignás** → `L := ListaInfo.create()` o `L := nueva` → necesitás `var`
