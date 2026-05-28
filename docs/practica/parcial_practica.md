# 🎯 Práctica de Parcial — StreamMusic

Ejercicio al estilo del parcial real. Intentá resolverlo completo antes de ver las pistas.

---

## Contexto

La empresa **StreamMusic** dispone de información de las 500 playlists creadas por sus usuarios. De cada playlist se conoce: **código** (entero), **nombre de usuario**, **país de origen**, **tipo de playlist** (1..8) y **cantidad de canciones favoritas**.

La información está almacenada en un vector.

El programa principal disponible es:

```pascal
begin { prog. principal }
  cargarVector(...);
  generarLista(...);
  tiposMayores(...);
  readln(valor); readln(nombre);
  writeln(elementosModificados(...));
end.
```

!!! tip "Antes de codear: leé el programa principal"
    Las firmas de los módulos están implícitas en cómo se usan. Por ejemplo:
    `writeln(elementosModificados(...))` → `elementosModificados` es una **función** que devuelve un entero.

---

## Declarar los tipos — empezá por acá

Definí todos los tipos necesarios. ¿Cuántos registros distintos hacen falta? ¿Cuántas listas?

??? tip "💡 Pista — tipos"
    ```pascal
    uses GenericLinkedList;
    const MAX = 500;
    type
      TPlaylist = record
        codigo, tipo, canciones: integer;
        usuario, pais: string;
      end;
      TVectorP = array[1..MAX] of TPlaylist;

      { Para la lista del ítem a }
      TInfoPlaylist = record
        usuario  : string;
        canciones: integer;
      end;
      ListaInfo = specialize LinkedList<TInfoPlaylist>;

      { Para el ítem b }
      TTipoTotal = record
        tipo, total: integer;
      end;
      ListaTipos = specialize LinkedList<TTipoTotal>;

      TContador = array[1..8] of integer;
    ```

---

## Ítem a — generarLista ⭐⭐⭐

Implementá un módulo que reciba el vector y genere una lista que contenga el **nombre de usuario** y la **cantidad de canciones favoritas** de todas las playlists donde:
- el **país de origen** sea `'Argentina'`
- el **código de playlist** sea **par**

La lista debe quedar ordenada por cantidad de canciones favoritas de manera **descendente**.

!!! question "Para pensar antes de codear"
    - ¿Va `var` en el vector? ¿Y en la lista?
    - ¿Cómo expresás "código par" en Pascal?
    - ¿Cómo ordenás descendente con selección? ¿Buscás el mínimo o el máximo?
    - ¿GenericLinkedList te deja insertar en el medio? ¿Qué estrategia usás entonces?

??? tip "💡 Estrategia"
    1. Recorrer el vector y copiar los que cumplen la condición a un **vector auxiliar**
    2. Ordenar ese vector auxiliar **descendente** con selección (encontrar el máximo en cada pasada)
    3. Construir la lista recorriendo el vector auxiliar ordenado con `add`

??? success "✅ Solución"
    ```pascal
    procedure generarLista(v: TVectorP; n: integer; var L: ListaInfo);
    var
      aux     : array[1..MAX] of TInfoPlaylist;
      m, i, j : integer;
      iMax    : integer;
      temp    : TInfoPlaylist;
    begin
      { Filtrar }
      m := 0;
      for i := 1 to n do
        if (v[i].pais = 'Argentina') and (v[i].codigo mod 2 = 0) then
        begin
          m := m + 1;
          aux[m].usuario   := v[i].usuario;
          aux[m].canciones := v[i].canciones;
        end;

      { Ordenar descendente — buscar el MÁXIMO en cada pasada }
      for i := 1 to m - 1 do
      begin
        iMax := i;
        for j := i + 1 to m do
          if aux[j].canciones > aux[iMax].canciones then
            iMax := j;
        temp      := aux[i];
        aux[i]    := aux[iMax];
        aux[iMax] := temp;
      end;

      { Construir lista }
      L := ListaInfo.create();
      for i := 1 to m do
        L.add(aux[i]);
    end;
    ```

---

## Ítem b — tiposMayores ⭐⭐⭐

Implementá un módulo que reciba el **vector original** y encuentre los **2 tipos de playlist** (valores 1..8) que acumulan la mayor cantidad de canciones favoritas totales. El resultado debe quedar ordenado descendentemente.

!!! question "Para pensar"
    - ¿Qué estructura usás para acumular canciones por tipo? ¿Un vector de 8 posiciones?
    - ¿Cómo encontrás los 2 mayores en un solo recorrido?

??? tip "💡 Estrategia"
    - Vector contador `cont[1..8]`: acumular `v[i].canciones` en `cont[v[i].tipo]`
    - Un `for` de 1 a 8 con dos variables (`max1`/`max2`) para encontrar el top 2

??? success "✅ Solución"
    ```pascal
    procedure tiposMayores(v: TVectorP; n: integer; var L: ListaTipos);
    var
      cont        : TContador;
      t, i        : integer;
      max1, max2  : integer;
      tipo1, tipo2: integer;
      dato        : TTipoTotal;
    begin
      for t := 1 to 8 do cont[t] := 0;
      for i := 1 to n do
        cont[v[i].tipo] := cont[v[i].tipo] + v[i].canciones;

      max1 := -1; tipo1 := 0;
      max2 := -1; tipo2 := 0;
      for t := 1 to 8 do
      begin
        if cont[t] > max1 then
        begin
          max2 := max1; tipo2 := tipo1;
          max1 := cont[t]; tipo1 := t;
        end
        else if cont[t] > max2 then
        begin
          max2 := cont[t]; tipo2 := t;
        end;
      end;

      L := ListaTipos.create();
      dato.tipo := tipo1; dato.total := max1; L.add(dato);
      dato.tipo := tipo2; dato.total := max2; L.add(dato);
    end;
    ```

---

## Ítem c — elementosModificados ⭐⭐⭐⭐

Implementá un módulo que reciba la **lista del ítem a**, un **nombre de usuario** y un **valor entero**.  
Para cada playlist de la lista cuyo usuario coincida con el nombre recibido, sumá el valor a la cantidad de canciones.  
**Retorná** la cantidad de elementos modificados.

!!! danger "Trampa — GenericLinkedList no tiene replace"
    `L.current()` devuelve una **copia** del elemento. Modificar esa copia no cambia la lista.

    ```pascal
    { ❌ Esto NO modifica la lista }
    info := L.current();
    info.canciones := info.canciones + valor;  { solo modifica la copia local }
    ```

    Solución: **reconstruir la lista** con los elementos ya modificados.

!!! question "Para pensar"
    - ¿Es función o procedimiento? ¿Cómo lo sabés mirando el programa principal?
    - ¿Necesita `var L`? ¿Por qué?

??? success "✅ Solución"
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
      L := nueva;
      elementosModificados := cant;
    end;
    ```

---

## Programa principal completo

Completá los puntos suspensivos:

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

---

## Los 5 puntos que no podés olvidar mañana

!!! danger "Checklist antes de entregar"
    1. **Código par** → `codigo mod 2 = 0`, no `= 1`
    2. **Ordenar descendente** → selección busca el **máximo**, no el mínimo
    3. **GenericLinkedList sin replace** → si modificás elementos, reconstruí la lista
    4. **Top 2 en un solo recorrido** → `max1`/`max2` con un `for` de 1 a 8
    5. **`var L` cuando reasignás** → si hacés `L := ListaTipo.create()` o `L := nueva`, necesitás `var`
