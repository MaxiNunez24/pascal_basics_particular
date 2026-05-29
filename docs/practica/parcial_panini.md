# 🎯 Práctica de Parcial — Panini Mundial 2026

Ejercicio al estilo del parcial real. Intentá resolver cada ítem antes de abrir las pistas.

---

## Contexto

La empresa **Panini** dispone de la información de los álbumes del mundial de fútbol 2026. De cada álbum se conoce: **código**, **nombre del dueño**, **país de origen**, **categoría de álbum** (1..5) y la **cantidad de figuritas faltantes**.

La información está almacenada en una **lista ordenada por país de origen**.

El programa principal disponible es:

```pascal
begin { prog. principal }
  cargarLista(...);              { se dispone }
  generarNuevaLista(...);
  Writeln(categoriaMenor(...));
  readln(valor);
  cantidadEliminados(...);
end.
```

!!! tip "Antes de codear: leé el programa principal"
    - ¿Los datos están en un vector o en una lista?
    - ¿Qué tipo devuelve `categoriaMenor`? ¿Cómo lo sabés?
    - `cantidadEliminados` aparece solo, sin `Writeln`. ¿Qué deberías agregar al completar el principal?

---

## Declarar los tipos — empezá por acá

Definí todos los tipos necesarios antes de arrancar.

??? question "💡 Pista — tipos"
    - ¿Cuántos registros necesitás? ¿Uno o dos?
    - La lista del ítem a, ¿almacena toda la información del álbum o solo parte?
    - ¿Qué estructura conocés que se indexe con un rango fijo como 1..5?

---

## Ítem a — generarNuevaLista ⭐⭐⭐

Implementá un módulo que reciba la lista de álbumes y genere una nueva lista de aquellos álbumes con país de origen `"Argentina"`. La nueva lista debe almacenar **solamente el nombre del dueño y la cantidad de figuritas faltantes**, ordenada por nombre del dueño de manera **ascendente**.

!!! question "Para pensar antes de codear"
    - La fuente de datos es una lista, no un vector. ¿Cómo recorrés una lista?
    - ¿El módulo modifica la lista fuente? ¿Necesita `var`?
    - ¿El módulo crea la nueva lista? ¿Ese parámetro necesita `var`?
    - ¿Cómo comparás strings con `<` y `>`? ¿Es diferente a comparar enteros?

??? question "💡 Pista — ordenar por nombre"
    - Ya sabés ordenar enteros con selección. ¿La comparación `aux[j].duenio < aux[iMin].duenio` funcionaría?
    - Si la fuente es una lista y GenericLinkedList no tiene inserción ordenada, ¿qué estructura auxiliar podrías usar para filtrar primero y ordenar después?

??? success "✅ Solución"
    ```pascal
    procedure generarNuevaLista(L: ListaAlbumes; var nueva: ListaInfo);
    var
      aux     : array[1..MAX] of TInfoAlbum;
      m, i, j : integer;
      iMin    : integer;
      temp    : TInfoAlbum;
      album   : TAlbum;
    begin
      { Filtrar de la lista }
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

      { Ordenar ascendente por duenio }
      for i := 1 to m - 1 do
      begin
        iMin := i;
        for j := i + 1 to m do
          if aux[j].duenio < aux[iMin].duenio then
            iMin := j;
        temp      := aux[i];
        aux[i]    := aux[iMin];
        aux[iMin] := temp;
      end;

      { Construir nueva lista }
      nueva := ListaInfo.create();
      for i := 1 to m do
        nueva.add(aux[i]);
    end;
    ```

---

## Ítem b — categoriaMenor ⭐⭐⭐

Implementá un módulo que reciba la **lista original de álbumes** y retorne la **categoría** (número entre 1 y 5) que tiene menor cantidad de álbumes.

!!! question "Para pensar antes de codear"
    - ¿Es función o procedimiento? ¿Qué retorna exactamente?
    - ¿Qué estructura usás para contar álbumes por categoría si las categorías son 1..5?
    - Una vez que tenés los totales, ¿buscás el máximo o el mínimo? ¿Qué cambia respecto a un ejercicio de máximo?
    - ¿Retornás el número de categoría o la cantidad de álbumes de esa categoría?

??? question "💡 Pista — mínimo"
    - Si buscar el máximo es `if cont[c] > maxVal`, ¿cómo sería buscar el mínimo?
    - ¿Con qué valor inicializás `minVal` antes de empezar a comparar?

??? success "✅ Solución"
    ```pascal
    function categoriaMenor(L: ListaAlbumes): integer;
    var
      cont   : TContador;
      c      : integer;
      album  : TAlbum;
      minVal : integer;
      minCat : integer;
    begin
      for c := 1 to 5 do cont[c] := 0;
      L.reset();
      while not L.eol() do
      begin
        album := L.current();
        cont[album.categoria] := cont[album.categoria] + 1;
        L.next();
      end;

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

---

## Ítem c — cantidadEliminados ⭐⭐⭐⭐

Implementá un módulo que reciba la **lista creada en el ítem a** y un **valor entero**. Eliminá todos los elementos donde la cantidad de figuritas faltantes sea igual al valor recibido. Retorná la cantidad de elementos que se eliminaron.

!!! question "Para pensar antes de codear"
    - ¿Es función o procedimiento? ¿Qué retorna?
    - ¿Necesita `var L`? ¿Por qué?
    - ¿Podés eliminar un elemento "en el lugar" con GenericLinkedList? Si no, ¿qué hacés?
    - ¿Cuál es la diferencia entre este ítem y el de "modificar" del parcial anterior?

??? question "💡 Pista — eliminar elementos"
    - Cuando reconstruís la lista, ¿qué elementos agregás a la nueva lista: todos, o solo los que **no** coinciden con `valor`?
    - ¿Qué hacés con los que sí coinciden? ¿Los agregás o solo sumás el contador?

??? success "✅ Solución"
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
          cant := cant + 1       { contar — NO agregar a la nueva lista }
        else
          nueva.add(info);       { solo agregar los que quedan }
        L.next();
      end;
      L := nueva;
      cantidadEliminados := cant;
    end;
    ```

---

## Programa principal completo

Completá los puntos suspensivos y lo que haga falta:

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
  Writeln(cantidadEliminados(listaArg, valor));
end.
```

---

## ¿En qué se diferencia este parcial del de StreamMusic?

| | StreamMusic | Panini |
|---|---|---|
| Datos originales | Vector | **Lista** |
| Filtrar | `for i := 1 to n` | `reset + while not eol` |
| Ordenar | Enteros descendente | **Strings ascendente** |
| Ítem de categorías | Buscar top 2 máximos | Buscar **mínimo** |
| Ítem de modificación | Sumar valor al campo | **Eliminar** el elemento |

!!! danger "Checklist antes de entregar"
    1. **Fuente es lista** → filtrar con `reset + while not eol`, no con `for i := 1 to n`
    2. **Ordenar strings** → `<` funciona igual que con enteros
    3. **Eliminar ≠ modificar** → los que coinciden **no se agregan** a la nueva lista
    4. **categoriaMenor retorna la categoría**, no la cantidad de álbumes
    5. **`var L` cuando reasignás** → `L := nueva` necesita `var`
