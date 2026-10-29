# Presentación del compilador ALEX

Duración sugerida: 7 a 8 minutos. Dos partes: características del lenguaje y demostración. Después, las preguntas de dominio de conceptos.

## Parte 1: Características de ALEX (2 minutos)

Decir con mis palabras:

1. **Qué es:** un lenguaje con palabras en español (`dato`, `entero`, `verdadero`) para que el código se lea fácil.
2. **Declaración:** `dato nombre: tipo;` o con valor inicial `dato nombre: tipo = valor;`. Toda instrucción termina con `;`.
3. **Asignación:** `nombre = expresion;`. La variable debe existir y el valor debe ser del mismo tipo.
4. **Cuatro tipos:** entero, decimal, texto y logico. El tipo se reconoce por cómo está escrito el valor: `5` es entero, `5.0` es decimal, `"25"` es texto.
5. **Compatibilidad:** un entero cabe en un decimal, pero un decimal no cabe en un entero.
6. **Nombres:** empiezan con letra o `_`, distinguen mayúsculas, no pueden ser palabras reservadas ni repetirse.
7. **Valores por defecto:** 0, 0.0, texto vacío y falso.
8. **Prohibido:** asignaciones encadenadas (`a = b = 3;`).
9. **Operadores:** `+ - * /` y paréntesis, que agregué porque mis ejemplos ya usaban `edad + 1`.
10. **Funciones propias:** `funcion suma(a: entero, b: entero): entero { retornar a + b; }`. El tipo de retorno es opcional: sin él, la función solo hace algo y no devuelve nada.
11. **Ámbito:** cada función tiene su propio espacio. Ve sus parámetros y sus variables locales, pero no las globales; lo que necesita lo recibe como parámetro.
12. **Funciones predefinidas:** `mostrar`, `longitud`, `mayusculas`, `redondear` y `absoluto`. Con `mostrar` el lenguaje por fin tiene salida en pantalla.
13. **Reglas de las funciones:** se definen antes de usarse, los argumentos deben coincidir en cantidad y tipo, y si declaran un tipo deben retornar un valor de ese tipo en todos sus caminos. No puede haber código después de un `retornar` que siempre se ejecuta.
14. **Condicionales:** `si (condicion) { ... } sino { ... }`. La condición va entre paréntesis y debe ser de tipo `logico`. También acepta `sino si` para varias opciones.
15. **Comparaciones:** `==  !=  <  >  <=  >=`, y siempre dan un valor `logico`. Los de orden (`<`, `>`, `<=`, `>=`) solo comparan números; `==` y `!=` comparan dos valores del mismo tipo.
16. **Ámbito de bloque:** una variable declarada dentro de un bloque `si` o `sino` solo existe ahí. El bloque ve las variables que lo rodean y puede modificarlas.
17. **Recursión:** ahora que hay `si`, una función puede llamarse a sí misma con un caso base (por ejemplo, el factorial). Hay un límite de 200 llamadas anidadas para detectar un caso base que falta.
18. **Arreglos:** un tipo seguido de corchetes, `entero[]`, `texto[]`, `decimal[]` o `logico[]`. Se crean con un literal, `dato notas: entero[] = [80, 95, 70];`, y sin valor inicial empiezan vacíos.
19. **Índices:** `notas[0]` lee un elemento y `notas[1] = 100;` lo modifica. Los índices empiezan en 0 y deben ser enteros. Todos los elementos de un arreglo tienen el mismo tipo.
20. **Métodos:** `notas.agregar(60);` añade al final, `notas.quitar()` quita y devuelve el último y `notas.longitud()` da la cantidad. También sirve `longitud(notas)`, y `mostrar(notas)` imprime `[80, 100, 70]`.
21. **Referencia:** un arreglo no se copia al asignarlo ni al pasarlo a una función, así que una función puede modificar el arreglo que recibe. Una función también puede devolver un arreglo (`: entero[]`).
22. **Errores de arreglos:** en compilación se detectan elementos de tipos distintos, índice que no es entero, métodos que no existen o con argumentos incorrectos; en ejecución, un índice fuera de rango.

## Parte 2: Demostración (4 minutos)

| Paso | Comando | Qué señalar |
|---|---|---|
| 1 | Mostrar `ejemplos/ok.alex` | Los cuatro tipos, una asignación con expresión y un comentario |
| 2 | `python alex.py ejemplos/ok.alex --tokens` | Tabla de tokens, árbol AST, AST Table, ejecución paso a paso y tabla de símbolos |
| 3 | Señalar la tabla de símbolos | `edad` pasa de 21 a 22 y `promedio` empieza en 0.0 por defecto |
| 4 | `python alex.py ejemplos/err_lexico.alex` | Carácter `@`, decimal `3.` y texto sin cerrar |
| 5 | `python alex.py ejemplos/err_sintactico.alex` | Falta `;`, falta `:`, `a = b = 3;`, palabra reservada como nombre |
| 6 | `python alex.py ejemplos/err_semantico.alex` | Variable no declarada, redeclarada, decimal en entero. No se ejecuta nada |
| 7 | Mostrar `ejemplos/funciones.alex` | Tres funciones propias (una con retorno entero, una con decimal, una sin retorno) y las cinco predefinidas |
| 8 | `python alex.py ejemplos/funciones.alex` | El nodo `Funcion` y `Llamada` en el árbol, "SALIDA DEL PROGRAMA", y la tabla de símbolos con la columna **Ámbito** (`resultado` vive solo en `suma`) junto con la tabla de funciones |
| 9 | `python alex.py ejemplos/err_func_semantico.alex` | Argumentos de más o de menos, tipo de argumento incorrecto, función no definida, `retornar` fuera de función, código inalcanzable y una función que no ve una variable global |
| 10 | `python alex.py ejemplos/err_func_sintactico.alex` | Paréntesis sin cerrar, función dentro de otra, coma sobrante y falta de `}` |
| 11 | Mostrar `ejemplos/condicionales.alex` | `si` con `sino`, `si` sin `sino`, `sino si`, una función con `si` y `sino` que retorna en los dos caminos, y el factorial recursivo |
| 12 | `python alex.py ejemplos/condicionales.alex` | En el árbol, los nodos `Si` y `Bloque`; en la ejecución, qué rama tomó cada `si` ("condición verdadera → se ejecuta el bloque «si»"); en la tabla de símbolos, `mensaje` con ámbito `global/si@49` |
| 13 | `python alex.py ejemplos/err_si_semantico.alex` | Condición que no es lógica, comparar texto con entero, variable de bloque usada afuera, función que no retorna en todos los caminos y código inalcanzable |
| 14 | `python alex.py ejemplos/err_si_sintactico.alex` | Falta de paréntesis, falta de `{`, `sino` suelto y falta de `}` |
| 15 | Mostrar `ejemplos/arreglos.alex` | Arreglos de los cuatro tipos, `notas[1] = 100;`, `agregar`, `quitar`, `longitud`, y funciones que reciben y devuelven arreglos (la suma recursiva usa el `si` como caso base) |
| 16 | `python alex.py ejemplos/arreglos.alex` | En el árbol, los nodos `Arreglo`, `Indice`, `AsignacionIndice` y `LlamadaMetodo`; en la ejecución, cómo queda `notas` después de cada cambio; en la tabla de símbolos, el tipo `entero[]` y el valor inicial y final de cada arreglo (`[80, 95, 70]` pasa a `[160, 100, 70]` porque `duplicar_primero` modificó el arreglo original) |
| 17 | `python alex.py ejemplos/err_arr_sintactico.alex` | Falta `]` en el tipo, en el literal y en el índice, arreglo de arreglos, método sin `(` y asignación encadenada |
| 18 | `python alex.py ejemplos/err_arr_semantico.alex` | Elementos de tipos distintos, índice texto, decimal o negativo, `[ ]` sobre un entero, `agregar` con el tipo equivocado, método que no existe y `agregar` dentro de una expresión |
| 19 | `python alex.py ejemplos/err_arr_ejecucion.alex` | El programa compila bien y falla al ejecutar: "Índice fuera de rango" al pedir la posición 4 de un arreglo con 4 elementos |

Para mostrar dónde vive cada fase en el código, buscar con `Ctrl+F`: `analizar_lexico`, `class Parser`, `class Semantico`, `ejecutar`.

## Parte 3: Dominio de conceptos (preguntas probables)

**¿Qué hace cada fase?**
- Léxica: lee el texto carácter por carácter y lo agrupa en tokens.
- Sintáctica: comprueba que los tokens sigan la gramática y construye el árbol.
- Semántica: comprueba el significado (declaraciones, tipos, duplicados).
- Ejecución: recorre el árbol y calcula los valores.

**¿Qué es un token, un lexema y un patrón?**
Un lexema es el texto concreto (`edad`), un token es su categoría (`IDENT`) y el patrón es la regla que lo describe (empieza con letra o `_`, sigue con letras, números o `_`).

**¿Qué es una gramática libre de contexto?**
Un conjunto de reglas de producción que describe qué programas son válidos. La mía está escrita en notación BNF en el `README.md`.

**¿Qué tipo de parser usé?**
Un descendente recursivo: una función por cada regla de la gramática. Decide con solo mirar el token actual (por ejemplo, `dato` inicia una declaración y un identificador inicia una asignación), por eso no necesita retroceder.

**¿Cómo resuelvo la precedencia de operadores?**
Con la propia gramática. `*` y `/` están en `termino`, un nivel más profundo que `+` y `-` en `suma`, y las comparaciones están en `expresion`, el nivel más alto. Por eso `a + 1 > b * 2` se calcula como `(a + 1) > (b * 2)`: primero lo aritmético y al final la comparación. Los ciclos `{ ... }` hacen que se evalúen de izquierda a derecha.

**¿Qué es el AST y en qué se diferencia del árbol de derivación?**
El AST guarda solo lo esencial. No tiene nodos para `;`, `:` ni paréntesis, porque su efecto ya está en la forma del árbol. Un árbol de derivación tendría un nodo por cada regla aplicada.

**¿Qué es la AST Table?**
Es el mismo árbol en forma de tabla: cada nodo con un ID, su tipo, valor, tipo de dato, padre, hijos y línea.

**¿Qué guarda la tabla de símbolos y cuándo se llena?**
Una entrada por variable o parámetro: nombre, tipo, clase, ámbito, línea de declaración, asignaciones, lecturas, valor inicial y valor final. Se llena durante el análisis semántico, y los valores de las globales se completan al ejecutar. Además hay una tabla de funciones con su firma y las veces que se llamaron.

**¿Qué verifica el análisis semántico que el sintáctico no puede?**
Que la variable exista, que no se declare dos veces y que los tipos sean compatibles. Una instrucción como `edad = "veinte";` está bien escrita (la gramática la acepta) pero no tiene sentido porque `edad` es entera.

**¿Qué es la coerción de tipos?**
Una conversión automática. En ALEX solo hay una: un entero se convierte a decimal (`precio = 5;` guarda `5.0`). La contraria no existe porque perdería información.

**¿Cómo manejo los errores?**
Cada error indica fase, línea, columna y la línea de código con una flecha. El parser, al fallar, salta hasta el siguiente `;` y sigue revisando (recuperación en modo pánico), por eso reporta varios errores en una sola corrida. Un error en una fase detiene las siguientes.

**¿Es un compilador o un intérprete?**
Mi proyecto cubre el *front-end* de un compilador (léxico, sintáctico y semántico) y además ejecuta el AST directamente, como un intérprete. No genera código máquina ni bytecode. Ese sería el siguiente paso: una fase de generación de código a partir del mismo AST.

**¿Qué diferencia hay entre parámetro y argumento?**
El parámetro es la variable que aparece en la definición (`a: entero` en `funcion suma(a: entero, b: entero)`). El argumento es el valor que se pasa en la llamada (`suma(2, 3)`). El semántico comprueba que coincidan en cantidad y en tipo.

**¿Qué es un ámbito (scope) y cómo lo manejo?**
Es la zona del programa donde una variable existe. En ALEX hay un ámbito global y uno por cada función. La tabla de símbolos guarda cada variable con su ámbito, así que `x` global y `x` local pueden convivir sin chocar. Una función no ve las variables globales; lo que necesita lo recibe por parámetros.

**¿Qué pasa cuando se ejecuta una llamada?**
Primero se calculan los argumentos. Luego se crea un ámbito nuevo con los parámetros ya cargados, se ejecuta el cuerpo, y `retornar` sale de la función con su valor. El valor reemplaza a la llamada dentro de la expresión.

**¿Cómo funciona un `si` / `sino` en el compilador?**
En el parser, `si` es una regla de la gramática: lee la condición entre paréntesis, un bloque y, si viene `sino`, otro bloque (o un `si` más, para `sino si`). En el AST queda un nodo `Si` con tres hijos: la condición, el bloque `entonces` y, opcionalmente, el bloque `sino`. El semántico comprueba que la condición sea `logico` y analiza cada bloque en su propio ámbito. En la ejecución se evalúa la condición y se corre solo uno de los dos bloques.

**¿Cómo sabe el compilador que una función retorna en todos sus caminos?**
Cada sentencia informa si siempre termina en un `retornar`. Un `retornar` sí; un `si` solo si tiene `sino` y los dos bloques retornan; cualquier otra instrucción no. Si la lista de instrucciones de la función no retorna siempre y la función declara un tipo, hay error. Por ejemplo, un `si` sin `sino` que retorna no alcanza, pero un `retornar` después de él sí.

**¿Qué diferencia hay entre el ámbito de una función y el de un bloque?**
Una función solo ve sus parámetros y sus variables locales; no ve las globales. Un bloque (`si` o `sino`) sí ve las variables que lo rodean y puede modificarlas, pero lo que declara dentro desaparece al terminar el bloque. En la tabla de símbolos aparece con un ámbito como `global/si@49` (el bloque del `si` de la línea 49).

**¿Qué pasa si una variable de un bloque se declara con el nombre de una externa?**
Es válido: la del bloque tapa a la externa (sombreado) solo dentro del bloque. Al salir, la externa conserva su valor. Dentro del mismo ámbito, en cambio, no se puede declarar dos veces el mismo nombre.

**¿Cómo manejo la recursión?**
Una función se registra antes de analizar su cuerpo, así que puede llamarse a sí misma. El `si` aporta el caso base, como en el factorial de `ejemplos/condicionales.alex`. Si el caso base falta, la ejecución se detiene a las 200 llamadas anidadas con el mensaje "Recursión demasiado profunda", en vez de colgarse.

**¿Cómo sabe el semántico qué tipo devuelve una llamada?**
Busca la función en la tabla de funciones y toma su tipo de retorno. En `absoluto` el retorno depende del argumento: entero si recibe entero y decimal si recibe decimal. Una función sin tipo de retorno no se puede usar dentro de una expresión.

**¿Cómo funcionan los arreglos en el compilador?**
Léxico: tres tokens nuevos, `[`, `]` y `.`. Sintáctico: un tipo puede llevar `[]` (regla `tipo`), hay un literal `[a, b, c]` y después de cualquier valor puede venir `[indice]` o `.metodo(...)` (regla `postfijo`). En el AST aparecen los nodos `Arreglo`, `Indice`, `AsignacionIndice` y `LlamadaMetodo`. Semántico: el tipo del arreglo es el tipo de sus elementos más `[]`, y se comprueba que el índice sea entero y que lo que se guarda tenga el tipo de los elementos. Ejecución: el arreglo es una lista y se revisa el rango en cada acceso.

**¿Por qué un arreglo `entero[]` no cabe en un `decimal[]` si un entero sí cabe en un decimal?**
Porque los arreglos se comparten por referencia. Si `entero[]` se pudiera guardar en un `decimal[]`, el mismo arreglo tendría que ser a la vez de enteros y de decimales, y se podría meter un decimal en un arreglo de enteros. Solo un literal como `[1, 2]` se adapta a `decimal[]`, porque se crea nuevo y se convierte elemento por elemento.

**¿Qué diferencia hay entre un error de índice en compilación y en ejecución?**
El compilador solo ve lo que está escrito: puede rechazar un índice que no es entero o un literal negativo como `lista[-1]`. En cambio no sabe cuántos elementos tendrá el arreglo cuando se ejecute (pueden cambiar con `agregar` y `quitar`), así que `lista[10]` se revisa al ejecutar y da "Índice fuera de rango" con la línea.

**¿Qué significa que los arreglos se pasan por referencia?**
Que `b = a` no crea una copia: `a` y `b` son dos nombres del mismo arreglo, y lo mismo pasa con los parámetros de una función. Por eso `duplicar_primero(notas)` cambia el arreglo original aunque la función tenga su propio ámbito; lo que no ve una función son las variables globales, pero el arreglo que recibe sí es el mismo.

**¿Cómo sabe el parser si `nombre[` o `nombre.` es una asignación o una llamada a un método?**
Parsea el valor completo con `postfijo` y mira qué viene después. Si sigue `=` y lo anterior es un elemento (`lista[0]`), es una asignación a un elemento. Si lo anterior es una llamada a un método, exige `;`. Cualquier otra cosa se reporta como "esta expresión no es una instrucción". No necesita retroceder.

**¿Qué agregaría después?**
Ciclos (`mientras`), que reutilizarían la misma idea de bloque y ámbito que ya uso para `si` y servirían para recorrer los arreglos sin recursión. También operadores lógicos (`y`, `o`, `no`) para combinar condiciones, y arreglos de varias dimensiones. Cada uno sería una regla nueva en la gramática, un nodo nuevo en el AST y un caso nuevo en el analizador semántico.

## Antes de presentar

- [ ] El link de GitHub abre en una ventana de incógnito.
- [ ] `python alex.py ejemplos/ok.alex`, `python alex.py ejemplos/funciones.alex`, `python alex.py ejemplos/condicionales.alex` y `python alex.py ejemplos/arreglos.alex` corren sin errores en la computadora de la presentación.
- [ ] La terminal de VS Code está abierta en la carpeta donde está `alex.py`.
