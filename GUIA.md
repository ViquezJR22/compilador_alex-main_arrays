# Compilador de ALEX: guía paso a paso

Estudiante: Alexis Viquez Terranova · Compiladores · Prof. William Martinez

## 0. Cómo correrlo

```
python alex.py ejemplos/ok.alex              # programa correcto: AST, AST Table, símbolos y ejecución
python alex.py ejemplos/ok.alex --tokens     # igual, mostrando también la tabla de tokens
python alex.py ejemplos/err_lexico.alex      # errores léxicos
python alex.py ejemplos/err_sintactico.alex  # errores sintácticos
python alex.py ejemplos/err_semantico.alex   # errores semánticos
python alex.py ejemplos/funciones.alex       # funciones propias y predefinidas
python alex.py ejemplos/err_func_sintactico.alex  # errores sintácticos en funciones
python alex.py ejemplos/err_func_semantico.alex   # errores semánticos en funciones
python alex.py ejemplos/condicionales.alex   # si / sino, comparaciones y recursión
python alex.py ejemplos/err_si_sintactico.alex    # errores sintácticos en si / sino
python alex.py ejemplos/err_si_semantico.alex     # errores semánticos en si / sino
python alex.py ejemplos/arreglos.alex        # arreglos: crear, leer, modificar, agregar, quitar, funciones
python alex.py ejemplos/err_arr_sintactico.alex   # errores sintácticos en arreglos
python alex.py ejemplos/err_arr_semantico.alex    # errores semánticos en arreglos
python alex.py ejemplos/err_arr_ejecucion.alex    # error de ejecución: índice fuera de rango
```

Solo necesita Python 3 (sin librerías externas).

## 1. Las reglas de ALEX (lo que explicas en la evaluación)

| Regla | Detalle |
|---|---|
| Declarar | `dato nombre: tipo;` o `dato nombre: tipo = valor;` |
| Asignar | `nombre = expresion;` (la variable debe existir) |
| Fin de instrucción | Siempre `;` |
| Tipos | `entero` (7, -15), `decimal` (3.14), `texto` ("Hola"), `logico` (verdadero / falso) |
| Nombres | Empiezan con letra o `_`; distinguen mayúsculas; no pueden ser reservadas; no se repiten |
| Reservadas | dato, entero, decimal, texto, logico, verdadero, falso |
| Valor por defecto | 0, 0.0, "" y falso |
| Compatibilidad | entero → decimal sí; decimal → entero no |
| Prohibido | `a = b = 3;` (asignación encadenada) |
| Comentarios | `// hasta el final de la línea` |
| Funciones propias | `funcion nombre(p: tipo, ...): tipo { ... retornar valor; }`; el tipo de retorno es opcional |
| Ámbito | Cada función tiene su propio ámbito: ve sus parámetros y sus variables locales, no las globales |
| Llamadas | `nombre(argumentos)` como instrucción o dentro de una expresión; los argumentos deben coincidir en cantidad y tipo |
| Funciones predefinidas | `mostrar`, `longitud`, `mayusculas`, `redondear`, `absoluto` |
| Prohibido en funciones | Código después de un `retornar` que siempre se ejecuta, definir una función dentro de otra o dentro de un bloque |
| Recursión | Una función puede llamarse a sí misma; el `si` sirve de caso base (límite de 200 llamadas anidadas) |
| Comparaciones | `==  !=  <  >  <=  >=`; el resultado es `logico`. Los de orden solo comparan números; `==` y `!=` comparan dos del mismo tipo |
| Condicionales | `si (condicion) { ... } sino { ... }` y `sino si (condicion) { ... }`; la condición debe ser `logico` |
| Ámbito de bloque | Una variable declarada dentro de un bloque `si` o `sino` solo existe ahí; el bloque ve las variables que lo rodean |
| Arreglos | `dato notas: entero[] = [80, 95, 70];` Tipos `entero[]`, `decimal[]`, `texto[]`, `logico[]`; sin valor inicial empiezan vacíos (`[]`) |
| Elementos | Todos del mismo tipo; un decimal[] acepta enteros mezclados (`[1.5, 2]`). No hay arreglos de arreglos |
| Índices | `lista[i]` para leer y `lista[i] = valor;` para modificar. Empiezan en 0 y deben ser `entero` |
| Métodos | `lista.agregar(v);` añade al final, `lista.quitar()` quita y devuelve el último, `lista.longitud()` da la cantidad (también `longitud(lista)`) |
| Referencia | Un arreglo no se copia al asignarlo ni al pasarlo a una función: ambos nombres apuntan al mismo arreglo |
| Errores de ejecución | Índice fuera de rango (de `0` a `longitud - 1`) y `quitar()` en un arreglo vacío |

Operadores `+ - * /` y paréntesis: los incluí porque tus propios ejemplos usan `precio + 2.5` y `edad + 1`. `+` también concatena texto. `entero / entero` da decimal. Si el profesor pide solo lo de la actividad anterior, puedes decir que son una extensión mínima.

## 2. Las 4 fases (y dónde está cada una en `alex.py`)

| Fase | Entregable | Dónde está |
|---|---|---|
| 1. Léxica | Tokens | `analizar_lexico()` |
| 2. Sintáctica | Analizador sintáctico + árbol AST | `class Parser` |
| 2b. | AST Table | `numerar()` + `compilar()` (tabla "AST TABLE") |
| 3. Semántica | Analizador semántico + Tabla de Símbolos | `class Semantico` y `class Simbolo` |
| 4. Ejecución | Correr el código fuente | `ejecutar()` |

### Fase 1: léxico
Lee el texto carácter por carácter y lo convierte en tokens: `DATO`, `TIPO`, `IDENT`, `ENTERO`, `DECIMAL`, `TEXTO`, `VERDADERO`, `FALSO`, `:`, `;`, `=`, `+ - * /`, `( )`, y para arreglos `[` (`LCORCH`), `]` (`RCORCH`) y `.` (`PUNTO`). Detecta: carácter no válido (`@`), decimal mal formado (`3.`), texto sin cerrar.

### Fase 2: sintáctico
Descendente recursivo: una función por regla de la gramática.

```
programa    ::= { sentencia }
sentencia   ::= declaracion | asignacion | asigna_elem | funcion | retorno | si
              | llamada ";" | metodo ";"
tipo        ::= TIPO [ "[" "]" ]
declaracion ::= "dato" IDENT ":" tipo [ "=" expresion ] ";"
asignacion  ::= IDENT "=" expresion ";"
asigna_elem ::= IDENT "[" expresion "]" "=" expresion ";"
metodo      ::= postfijo "." IDENT "(" [ expresion { "," expresion } ] ")"
funcion     ::= "funcion" IDENT "(" [ parametros ] ")" [ ":" tipo ] "{" { sentencia } "}"
parametros  ::= IDENT ":" tipo { "," IDENT ":" tipo }
retorno     ::= "retornar" [ expresion ] ";"
si          ::= "si" "(" expresion ")" bloque [ "sino" ( si | bloque ) ]
bloque      ::= "{" { sentencia } "}"
llamada     ::= IDENT "(" [ expresion { "," expresion } ] ")"
expresion   ::= suma { ("==" | "!=" | "<" | ">" | "<=" | ">=") suma }
suma        ::= termino { ("+" | "-") termino }
termino     ::= factor { ("*" | "/") factor }
factor      ::= "-" factor | postfijo
postfijo    ::= primario { "[" expresion "]" | "." IDENT "(" [ expresion { "," expresion } ] ")" }
primario    ::= ENTERO | DECIMAL | TEXTO | "verdadero" | "falso" | arreglo | llamada | IDENT | "(" expresion ")"
arreglo     ::= "[" [ expresion { "," expresion } ] "]"
```

La precedencia sale de la gramática: `*` y `/` están un nivel más abajo que `+` y `-`, así que se evalúan primero. Los corchetes `[ ]` y el punto `.` de los arreglos están en `postfijo`, el nivel más profundo: `a[0] + 1` toma primero el elemento y después suma.

Los arreglos se reconocen mirando el token que sigue a un identificador: `nombre(` es una llamada, `nombre[` o `nombre.` es un elemento o un método, y `nombre =` es una asignación normal. Así el parser sigue sin retroceder. Cuando hay un error, el parser lo reporta y salta hasta el siguiente `;` para seguir revisando (por eso puede mostrar varios errores en una corrida).

### AST Table
Cada nodo del árbol se numera (ID), con su tipo de nodo, valor, tipo de dato (lo llena el semántico), padre, hijos y línea. Sale justo después del árbol dibujado.

### Fase 3: semántico y Tabla de Símbolos
Recorre el AST en orden y lleva la tabla de símbolos por ámbito (`global` y uno por función): `nombre → tipo, clase, ámbito, línea, asignaciones, lecturas`. También lleva una tabla de funciones con su firma y cuántas veces se llamó. Verifica:
1. Variable declarada antes de usarse.
2. No declarada dos veces.
3. El valor coincide con el tipo (entero → decimal permitido, decimal → entero no).
4. Operadores válidos para los tipos (no se puede `"a" - "b"`).
5. Funciones: no se redefinen ni se llaman antes de definirlas, y los parámetros no se repiten.
6. Llamadas: la cantidad y el tipo de los argumentos coinciden con los parámetros, y una función sin valor no se usa dentro de una expresión.
7. Retornos: una función con tipo debe tener `retornar` en todos sus caminos con un valor de ese tipo (un `si` solo cuenta si su `sino` también retorna), no hay código después de un `retornar` que siempre se ejecuta, y `retornar` solo existe dentro de funciones.
8. Ámbitos: una función no ve las variables globales, y las variables de una función o de un bloque no existen afuera.
9. Condiciones: la condición de un `si` debe ser `logico`.
10. Comparaciones: `<  >  <=  >=` solo entre números; `==` y `!=` entre dos números o dos del mismo tipo.
11. Arreglos: todos los elementos de un literal tienen el mismo tipo (entero con decimal da `decimal[]`); el valor guardado en un arreglo o en un elemento coincide con el tipo de los elementos; `entero[]` no cabe en `decimal[]`.
12. Índices: `[ ]` solo se aplica a un arreglo, el índice es `entero` y un índice literal negativo se rechaza.
13. Métodos: `agregar` (1 argumento del tipo de los elementos, no devuelve valor), `quitar` (0 argumentos, devuelve un elemento) y `longitud` (0 argumentos, devuelve entero); otro nombre o una cantidad distinta de argumentos es un error.

Si hay errores semánticos no se ejecuta nada.

### Fase 4: ejecución
Recorre el AST, calcula las expresiones y guarda los valores. Cada llamada a una función crea un ámbito nuevo con sus parámetros; `retornar` sale de la función con su valor. Un `si` evalúa su condición y ejecuta solo uno de sus bloques, cada uno con su propio ámbito; en la ejecución se muestra cuál rama se tomó. Lo que imprime `mostrar` aparece en la ejecución y en el bloque "SALIDA DEL PROGRAMA". Al final la Tabla de Símbolos muestra valor inicial y valor final de cada variable global. Un arreglo se guarda como una lista; `lista[i]` revisa que `i` esté entre 0 y la longitud menos 1, y `quitar()` revisa que el arreglo no esté vacío. En la ejecución, cada cambio a un arreglo muestra cómo quedó (`notas = [80, 100, 70, 60]`). También detecta división entre cero.

## 3. Guion para la evaluación (5 minutos)

1. Mostrar `ok.alex` y explicar las reglas de la sección 1 con 2 o 3 líneas del código.
2. Correr `python alex.py ejemplos/ok.alex --tokens`: señalar los tokens, el árbol, la AST Table, la ejecución y la Tabla de Símbolos.
3. Correr `err_sintactico.alex`: «falta el `;`», «asignación encadenada», «palabra reservada».
4. Correr `err_semantico.alex`: «no declarada», «redeclarada», «decimal en entero».
5. Correr `arreglos.alex`: señalar `Arreglo`, `Indice` y `LlamadaMetodo` en el árbol, cómo cambia `notas` en la ejecución y la columna de tipo `entero[]` en la tabla de símbolos. Después `err_arr_semantico.alex` y `err_arr_ejecucion.alex` (índice fuera de rango).
6. Cerrar con el siguiente paso: agregar ciclos (`mientras`) para recorrer los arreglos, que serían nuevas reglas en el parser y nuevos casos en el semántico.

## 4. Si el profesor pide un cambio en vivo

- Palabra reservada nueva: agregarla a `RESERVADAS` y en `analizar_lexico()`.
- Tipo nuevo: agregarlo a `TIPOS`, a `defecto()` y a `fmt()`.
- Operador nuevo: añadirlo a `SIMPLES`, al parser (`expresion` o `termino`) y a `evaluar()`.
- Función predefinida nueva: agregarla a `PREDEFINIDAS` (parámetros y tipo de retorno) y escribir su comportamiento en `llamar_predef()`.
- Comparador nuevo: agregarlo a `DOBLES` o `SIMPLES` y a `COMPARADORES`, y darle su regla de tipos en `_tipo()` (rama `BinOp`) y su cálculo en `evaluar()`.
- Método nuevo de arreglo (por ejemplo `vaciar`): agregarlo a `METODOS_ARREGLO` (con su cantidad de argumentos), darle su tipo de retorno en `metodo()` del semántico y su comportamiento en `llamar_metodo()`.
- Condicional nuevo (por ejemplo un ciclo `mientras`): reutiliza `bloque()` en el parser, `bloque()` en el semántico y `ejecutar_bloque()` en la ejecución.
