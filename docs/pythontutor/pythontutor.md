# 🔬 Python Tutor — Visualización en C

Python Tutor no soporta Pascal, pero el modelo de memoria de **C es idéntico al de Pascal**. Los snippets de esta página pueden pegarse directamente en [pythontutor.com/c.html](https://pythontutor.com/c.html) para visualizar paso a paso.

| Pascal | C equivalente |
|---|---|
| `record` | `struct` |
| `^TNodo` (puntero) | `struct Nodo*` |
| `var x: integer` (VAR param) | `int* x` |
| `new(p)` | `p = malloc(sizeof(*p))` |
| `p^.campo` | `p->campo` |
| `nil` | `NULL` |

---

## VAR vs Valor {#var-vs-valor}

Pegá este código en [pythontutor.com/c.html](https://pythontutor.com/c.html) y ejecutalo paso a paso. Observá cómo el stack frame de `porValor` tiene su propia copia de `x`, mientras que `porReferencia` trabaja directamente sobre la variable original.

```c
#include <stdio.h>

/* Por VALOR — recibe una copia */
void porValor(int x) {
    x = 99;  /* solo cambia la copia local */
}

/* Por REFERENCIA — recibe la dirección (equivalente a VAR en Pascal) */
void porReferencia(int* x) {
    *x = 99;  /* cambia el original */
}

int main() {
    int a = 5;
    int b = 5;

    porValor(a);        /* a sigue siendo 5 */
    porReferencia(&b);  /* b pasa a ser 99  */

    printf("a = %d\n", a);  /* 5  */
    printf("b = %d\n", b);  /* 99 */
    return 0;
}
```

!!! info "Equivalencia Pascal → C"
    ```pascal
    { Pascal }
    procedure porValor(x: integer);
    procedure porReferencia(var x: integer);

    { C equivalente }
    void porValor(int x);
    void porReferencia(int* x);
    ```

---

## Listas Enlazadas — addFirst {#listas-enlazadas}

Visualiza cómo cada `addFirst` crea un nuevo nodo en el heap y actualiza el puntero cabecera. Es el mismo comportamiento que en Pascal.

```c
#include <stdio.h>
#include <stdlib.h>

struct Nodo {
    int valor;
    struct Nodo* siguiente;
};

/* Equivalente a: procedure addFirst(var lista: PNodo; valor: integer) */
void addFirst(struct Nodo** lista, int valor) {
    struct Nodo* nuevo = (struct Nodo*)malloc(sizeof(struct Nodo));
    nuevo->valor     = valor;
    nuevo->siguiente = *lista;
    *lista           = nuevo;
}

void mostrar(struct Nodo* lista) {
    struct Nodo* actual = lista;
    while (actual != NULL) {
        printf("%d\n", actual->valor);
        actual = actual->siguiente;
    }
}

int main() {
    struct Nodo* lista = NULL;   /* lista := nil */

    addFirst(&lista, 30);
    addFirst(&lista, 20);
    addFirst(&lista, 10);

    /* La lista quedó: 10 -> 20 -> 30 -> NULL */
    mostrar(lista);
    return 0;
}
```

!!! tip "Qué observar en Python Tutor"
    - Cada `addFirst` agrega un frame nuevo en el **stack** y un nodo en el **heap**
    - El doble puntero `**lista` en C corresponde al `var lista: PNodo` en Pascal
    - Al salir de `addFirst`, el heap conserva el nodo — la memoria dinámica persiste

---

## Listas Enlazadas — Insertar en el medio {#insertar-en-el-medio}

Este snippet muestra el paso crítico: el **orden** en que se reasignan los punteros.  
Ejecutalo paso a paso y observá cómo el heap cambia en cada asignación.

```c
#include <stdio.h>
#include <stdlib.h>

struct Nodo {
    int valor;
    struct Nodo* siguiente;
};

void addFirst(struct Nodo** lista, int valor) {
    struct Nodo* nuevo = (struct Nodo*)malloc(sizeof(struct Nodo));
    nuevo->valor     = valor;
    nuevo->siguiente = *lista;
    *lista           = nuevo;
}

/*
 * Equivalente Pascal:
 *   procedure insertarDespuesDe(lista: PNodo; valorBuscar, valorNuevo: integer)
 *
 * lista NO es doble puntero porque no cambia la cabecera.
 */
void insertarDespuesDe(struct Nodo* lista, int valorBuscar, int valorNuevo) {
    /* 1. Buscar el nodo de referencia */
    struct Nodo* actual = lista;
    while (actual != NULL && actual->valor != valorBuscar)
        actual = actual->siguiente;

    /* 2. Insertar después si se encontró */
    if (actual != NULL) {
        struct Nodo* nuevo = (struct Nodo*)malloc(sizeof(struct Nodo));
        nuevo->valor      = valorNuevo;
        nuevo->siguiente  = actual->siguiente; /* paso 1: NUEVO apunta al siguiente */
        actual->siguiente = nuevo;             /* paso 2: anterior apunta al NUEVO  */
    }
}

void mostrar(struct Nodo* lista) {
    struct Nodo* actual = lista;
    while (actual != NULL) {
        printf("%d ", actual->valor);
        actual = actual->siguiente;
    }
    printf("\n");
}

int main() {
    struct Nodo* lista = NULL;

    /* Armar lista: 10 -> 30 -> NULL */
    addFirst(&lista, 30);
    addFirst(&lista, 10);
    mostrar(lista);   /* 10 30 */

    /* Insertar 20 después del 10: 10 -> 20 -> 30 -> NULL */
    insertarDespuesDe(lista, 10, 20);
    mostrar(lista);   /* 10 20 30 */

    return 0;
}
```

!!! tip "Qué observar en Python Tutor"
    - Cuando `insertarDespuesDe` empieza, `lista` **no es doble puntero** — la cabecera no cambia
    - Fijate el estado del heap **entre el paso 1 y el paso 2**: `nuevo->siguiente` ya apunta a `30`, pero `10->siguiente` todavía apunta a `30` también — ambos apuntan al mismo nodo momentáneamente
    - Después del paso 2: `10->siguiente` pasa a apuntar a `NUEVO (20)`, y la cadena queda correcta
    - Si invirtieras los pasos, en el paso 1 `actual->siguiente` ya sería `nuevo`, y `nuevo->siguiente = actual->siguiente` asignaría `nuevo` a sí mismo — bucle infinito

!!! warning "Equivalencia Pascal → C para el doble puntero"
    ```
    Pascal: var lista: PNodo      →  C: struct Nodo** lista
    Pascal: lista (sin var)       →  C: struct Nodo*  lista
    ```
    En `insertarDespuesDe` la cabecera no cambia, así que en C alcanza con `struct Nodo*` (un solo nivel de indirección).

---

## Corte de Control {#corte-de-control}

Los datos están **hardcodeados en un array** para poder steppear sin tener que ingresar input.
Pegá el código en [pythontutor.com/c.html](https://pythontutor.com/c.html) y avanzá de a un paso.

```c
#include <stdio.h>
#include <string.h>

/*
 * Equivalente Pascal:
 *   while localidad <> 'FIN' do
 *     if localidad = localidadActual then acumular
 *     else informar + reiniciar
 *   { informar último grupo }
 */

typedef struct {
    char localidad[20];
    double precio;
} Inmueble;

int main() {
    /* Datos ordenados por localidad — simulan la entrada del programa */
    Inmueble datos[] = {
        {"La Plata",  150000},
        {"La Plata",  200000},
        {"Berisso",    90000},
        {"Berisso",   110000},
        {"Ensenada",  130000},
        {"FIN",            0}   /* centinela */
    };

    int i = 0;
    char localidadActual[20];
    double totalLocalidad = 0;
    double totalGeneral   = 0;

    /* Primer dato — antes del while */
    strcpy(localidadActual, datos[i].localidad);
    totalLocalidad = 0;

    while (strcmp(datos[i].localidad, "FIN") != 0) {
        if (strcmp(datos[i].localidad, localidadActual) == 0) {
            /* Mismo grupo: acumular */
            totalLocalidad += datos[i].precio;
        } else {
            /* Cambio de grupo: informar el anterior */
            printf("%s: %.2f\n", localidadActual, totalLocalidad);
            totalGeneral += totalLocalidad;
            /* Actualizar criterio y arrancar el nuevo grupo */
            strcpy(localidadActual, datos[i].localidad);
            totalLocalidad = datos[i].precio;  /* ← el dato actual ya es del nuevo grupo */
        }
        i++;
    }

    /* Último grupo — nunca olvidar */
    printf("%s: %.2f\n", localidadActual, totalLocalidad);
    totalGeneral += totalLocalidad;
    printf("Total general: %.2f\n", totalGeneral);

    return 0;
}
```

!!! tip "Qué observar en Python Tutor"
    - Stepear hasta la **primera vez que entra al `else`** (cuando `i=2`, pasa de La Plata a Berisso): observá que `printf` imprime La Plata **antes** de que `localidadActual` cambie
    - Fijate el valor de `totalLocalidad` justo antes y después del `else`: se imprime el acumulado anterior y luego se asigna `datos[i].precio` — si pusiera `0` en vez del precio, se perdería el primer elemento del nuevo grupo
    - Al llegar a `"FIN"`, el `while` **no entra** — el último `printf` (fuera del loop) es el único que imprime Ensenada

!!! warning "Equivalencia Pascal → C para strings"
    ```
    Pascal: localidad = localidadActual   →  C: strcmp(a, b) == 0
    Pascal: localidadActual := localidad  →  C: strcpy(dest, src)
    ```
    En Pascal los strings se comparan con `=` directamente; en C hay que usar `strcmp`.

---

## Registros — struct con campos {#registros}

```c
#include <stdio.h>

/* Pascal: TAlumno = record nombre: string; promedio: real; end */
struct Alumno {
    char nombre[50];
    double promedio;
};

double calcularPromedio(double n1, double n2, double n3) {
    return (n1 + n2 + n3) / 3.0;
}

int main() {
    struct Alumno a;
    double n1 = 7.0, n2 = 8.0, n3 = 6.0;

    a.promedio = calcularPromedio(n1, n2, n3);

    printf("Promedio: %.2f\n", a.promedio);
    return 0;
}
```
