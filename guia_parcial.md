# Guía Docente — Práctica Parcial StreamMusic
**Examen de referencia:** AyPI Comisión 3 – Primera Fecha – 28-05-2026  
**Duración de la clase:** 60 minutos

---

## ANTES DE EMPEZAR (5 min) — Leer el enunciado juntos

Leer en voz alta y subrayar:
1. ¿Qué datos tiene cada playlist? → campos del registro
2. ¿Qué pide cada ítem? → módulo, parámetros, qué retorna
3. ¿Qué está dado en el programa principal? → firmas que hay que respetar

**Programa principal dado:**
```pascal
begin { prog. principal }
  cargarVector(...);          { dado — carga el vector de playlists }
  generarLista(...);          { ítem a }
  tiposMayores(...);          { ítem b }
  readln(valor); readln(nombre);
  writeln(elementosModificados(...));  { ítem c — función }
end.
```

**Preguntarle al alumno:** "¿Qué podemos deducir de la firma de `elementosModificados` solo con mirarlo en el `writeln`?"
→ Es una **función** que devuelve un entero.

---

## PASO 1 — Declarar los tipos (10 min)

Hacerlo juntos en papel antes de codear. Los tipos salen directamente del enunciado.

```pascal
uses GenericLinkedList;

const MAX = 500;

type
  { Vector principal — datos del enunciado }
  TPlaylist = record
    codigo   : integer;
    usuario  : string;
    pais     : string;
    tipo     : integer;   { 1..8 }
    canciones: integer;
  end;
  TVectorP = array[1..MAX] of TPlaylist;

  { Ítem a — lo que va en la lista filtrada }
  TInfoPlaylist = record
    usuario  : string;
    canciones: integer;
  end;
  ListaInfo = specialize LinkedList<TInfoPlaylist>;

  { Ítem b — tipo + total acumulado }
  TTipoTotal = record
    tipo  : integer;
    total : integer;
  end;
  ListaTipos = specialize LinkedList<TTipoTotal>;

  { Ítem b — vector contador interno }
  TContador = array[1..8] of integer;
```

**Pregunta para el alumno:** "¿Por qué `TInfoPlaylist` no tiene `pais` ni `tipo`?"
→ Porque el enunciado dice que la lista solo contiene `usuario` y `canciones`.

---

## PASO 2 — Ítem a: generarLista (15 min)

### Análisis
- Recibe el vector → **sin `var`** (solo lo lee)
- Crea la lista → **`var L`** (reasigna con `create()`)
- Condiciones: `pais = 'Argentina'` Y `codigo mod 2 = 0` (par)
- La lista debe quedar **ordenada descendente** por `canciones`

### Estrategia: filtrar → ordenar → construir lista

```
⚠️ GenericLinkedList no tiene insertarOrdenado.
Solución: vector auxiliar → selección descendente → add en orden.
```

```pascal
procedure generarLista(v: TVectorP; n: integer; var L: ListaInfo);
var
  aux  : array[1..MAX] of TInfoPlaylist;
  m    : integer;
  i, j : integer;
  iMax : integer;
  temp : TInfoPlaylist;
begin
  { 1. Filtrar — solo Argentina y código par }
  m := 0;
  for i := 1 to n do
  begin
    if (v[i].pais = 'Argentina') and (v[i].codigo mod 2 = 0) then
    begin
      m := m + 1;
      aux[m].usuario   := v[i].usuario;
      aux[m].canciones := v[i].canciones;
    end;
  end;

  { 2. Ordenar DESCENDENTE por canciones — selección con máximo }
  for i := 1 to m - 1 do
  begin
    iMax := i;
    for j := i + 1 to m do
      if aux[j].canciones > aux[iMax].canciones then  { > para descender }
        iMax := j;
    temp      := aux[i];
    aux[i]    := aux[iMax];
    aux[iMax] := temp;
  end;

  { 3. Construir lista en orden }
  L := ListaInfo.create();
  for i := 1 to m do
    L.add(aux[i]);
end;
```

### Trampas comunes — señalárselas antes de que las cometa

| Trampa | Corrección |
|---|---|
| `codigo mod 2 = 1` para "par" | Par = resto 0: `codigo mod 2 = 0` |
| Selección ascendente (find min) | Para descender: buscar el **máximo** (`>`) |
| No usar vector auxiliar, intentar insertar en orden en la lista | GenericLinkedList no tiene `insertarDespuesDe` — usar vector auxiliar |
| `var v` en la firma | El vector solo se lee — sin `var` |

---

## PASO 3 — Ítem b: tiposMayores (12 min)

### Análisis
- Recibe el vector original (no la lista)
- Encuentra los **2 tipos** (1..8) con mayor total acumulado de canciones
- Se llama como procedimiento: el resultado puede guardarse en var parámetro o imprimirse

### Estrategia: vector contador [1..8] → encontrar top 2

```pascal
procedure tiposMayores(v: TVectorP; n: integer; var L: ListaTipos);
var
  cont       : TContador;
  t, i       : integer;
  max1, max2 : integer;
  tipo1, tipo2: integer;
  dato       : TTipoTotal;
begin
  { 1. Inicializar y acumular }
  for t := 1 to 8 do
    cont[t] := 0;
  for i := 1 to n do
    cont[v[i].tipo] := cont[v[i].tipo] + v[i].canciones;

  { 2. Encontrar los 2 tipos con más canciones }
  max1 := -1; tipo1 := 0;
  max2 := -1; tipo2 := 0;
  for t := 1 to 8 do
  begin
    if cont[t] > max1 then
    begin
      max2  := max1;  tipo2 := tipo1;
      max1  := cont[t]; tipo1 := t;
    end
    else if cont[t] > max2 then
    begin
      max2  := cont[t]; tipo2 := t;
    end;
  end;

  { 3. Construir lista de 2 elementos, descendente }
  L := ListaTipos.create();
  dato.tipo := tipo1; dato.total := max1;
  L.add(dato);
  dato.tipo := tipo2; dato.total := max2;
  L.add(dato);
end;
```

### Pregunta para el alumno
"¿Por qué inicializamos `max1` y `max2` en `-1` y no en `0`?"
→ Porque podría haber tipos con 0 canciones. Con `-1` cualquier tipo real lo supera.

### Trampa principal
El "top 2" se confunde con "recorrer 2 veces". Mostrarle que un solo `for` con dos variables (`max1`/`max2`) alcanza.

---

## PASO 4 — Ítem c: elementosModificados (10 min)

### Análisis
- Recibe la lista del ítem a, un nombre y un valor entero
- Para cada elemento donde `usuario = nombre`: sumar `valor` a `canciones`
- **Retorna** la cantidad de modificados → es una **función** que devuelve integer
- Modifica la lista → necesita **`var L`**

### El problema clave: GenericLinkedList no tiene "replace"
`L.current()` devuelve una **copia** del elemento. Modificarla no cambia la lista.
→ Solución: **reconstruir la lista** con los elementos modificados.

```pascal
function elementosModificados(var L: ListaInfo; nombre: string; valor: integer): integer;
var
  nueva: ListaInfo;
  info : TInfoPlaylist;
  cant : integer;
begin
  nueva := ListaInfo.create();
  cant  := 0;
  L.reset();
  while not L.eol() do
  begin
    info := L.current();
    if info.usuario = nombre then
    begin
      info.canciones := info.canciones + valor;
      cant := cant + 1;
    end;
    nueva.add(info);
    L.next();
  end;
  L := nueva;                    { reemplazar la lista original }
  elementosModificados := cant;
end;
```

### Trampas comunes

| Trampa | Corrección |
|---|---|
| Hacer `L.current().canciones := ...` | No funciona — `current()` devuelve copia. Guardar en variable local, modificar, y agregar a la nueva lista |
| Olvidar `L := nueva` al final | La lista original quedaría sin cambios |
| Sin `var L` | La reasignación `L := nueva` no se propagaría al llamador |
| Inicializar `cant` después del while | Inicializar en 0 ANTES del while |

---

## PASO 5 — Completar el programa principal (5 min)

Con todo lo anterior, completar los puntos suspensivos:

```pascal
var
  v      : TVectorP;
  n      : integer;
  lista  : ListaInfo;
  tipos  : ListaTipos;
  valor  : integer;
  nombre : string;
begin
  cargarVector(v, n);
  generarLista(v, n, lista);
  tiposMayores(v, n, tipos);
  readln(valor);
  readln(nombre);
  writeln(elementosModificados(lista, nombre, valor));
end.
```

**Pregunta de cierre:** "¿Por qué `lista` en el `writeln` puede ser modificada por `elementosModificados`?"
→ Porque se pasa con `var`. La función puede reasignar `L` y eso afecta a `lista` en el principal.

---

## Los 5 puntos clave para el parcial de mañana

1. **Leer el programa principal dado** — las firmas están implícitas ahí.
2. **Código par** = `codigo mod 2 = 0`, no `= 1`.
3. **Selección descendente** = buscar el máximo, no el mínimo.
4. **GenericLinkedList no tiene replace** → reconstruir la lista si hay que modificar elementos.
5. **Top 2** = un solo `for` con `max1`/`max2` — no recorrer dos veces.
