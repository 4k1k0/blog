---
title: "Básicos Erlang / Elixir: La elegancia de match operator y aridad múltiple"
date: 2022-09-22T02:06:43-06:00
draft: true # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - programming
tags:
  - erlang
  - elixir
---

[Erlang](https://www.erlang.org/) y [Elixir](https://elixir-lang.org/) son lenguajes que me han impresionado bastante y que recientemente me he tomado más tiempo para explorar. Ambos lenguajes de programación son sumamente poderosos por sus paradigmas funcionales y concurrentes. Siendo Elixir un lenguaje de programación basado en Erlang pero con una azúcar sintáctica similar a Ruby. 

Algunos de los conceptos básicos en ambos lenguajes son [pattern matching](https://en.wikipedia.org/wiki/Pattern_matching) y [aridad](https://es.wikipedia.org/wiki/Aridad). La combinación de ambos nos permite tener código limpio y expresivo.

## Pattern Matching

Es la acción de revisar que una secuencia de identificadores siga un patrón otorgado. Esta comparación tiene que ser exacta para poder ser válida.

```elixir
iex(1)> x = 1
1
```

Este es el ejemplo más sencillo de pattern matching. En otros lenguajes esta operación sería conocida como una asignación. Donde a `x` se le asigna el valor `1`. Pero en Erlang y Elixir esto se conoce como pattern matching. Siendo `x` un identificador no declarado éste intenta hacer match con su patrón de la derecha. Este patrón al hacer match hace un vínculo entre `a` y `1`.

```elixir
iex(1)> a = 1
1
iex(2)> 1 = a
1
iex(3)> 2 = a
** (MatchError) no match of right hand side value: 1

iex(3)> 
```

Lo que está pasando es que se hizo un pattern matching entre `a` y `1`. Siendo así que `a` hace un match con este valor. En la segunda línea podemos ver `1 = a` lo cual es correcto, ya que `a` tomó este patrón. Pero en `2 = a` tenemos el error `** (MatchError) no match of right hand side value: 1` ya que al intentar hacer el pattern matching entre el valor de la izquierda con el patrón de la derecha estos no coinciden. No hay un match entre `2` y `a` ya que `a` anteriormente hizo un match con `1`.

Veamos un ejemplo más complejo. Tenemos una lista de números al cual se aplicará pattern matching para almacenarlos en identificadores:

```elixir
iex(1)> [a, b, c] = [1, 2, 3]
[1, 2, 3]
iex(2)> a
1
iex(3)> b
2
iex(4)> c
3
iex(5)> 
```

Tenemos una lista de elementos `[a, b, c]` del lado izquierdo a la cual se aplicará pattern matching contra el patrón `[1, 2, 3]`. De esta manera tenemos en cada uno de los identificadores el valor correspondiente de la derecha con el que hizo match. Pero ¿Qué pasa si el patrón de la izquierda no coincide con el de la derecha?

```elixir
iex(1)> [a, b, c, d] = [1, 2, 3]
** (MatchError) no match of right hand side value: [1, 2, 3]

iex(1)> [a, b] = [1, 2, 3]      
** (MatchError) no match of right hand side value: [1, 2, 3]

iex(1)> 
```

El primer error nos indica que no existe un cuarto elemento en el patrón de la derecha cuando intenta hacer match con el patrón de la izquierda el cual sí contiene un cuarto elemento.

El segundo error indica el patrón de la izquierda contiene únicamente dos elementos cuando el patrón de la derecha contiene tres.

Se puede escribir mucho más artículos enteros sobre pattern matching. Pero espero que con estos ejemplos haya quedado claro que nos permite comparar patrones extrictos para poder acceder a sus valores.

## Aridad

Wikipedia define aridad como:

> En el análisis matemático, la aridad de un operador matemático o de una función es el número de argumentos necesarios para que dicho operador o función se pueda calcular. 

En Erlang y Elixir la aridad de una función es parte de la firma de la misma. Esta aridad nos permite identificar funciones por el número de sus parámetros.

Por ejemplo. Una función de suma con aridad 2 y aridad 3 se ven así:

```elixir
def sum(a, b)
def sum(a, b, c)
```

Nuestras funciones se identifican como `sum/2` y `sum/3`. De esta manera podemos exportarlas o importarlas.

## El poder de pattern matching y aridad

De la misma manera a que la aridad nos permite definir una función del mismo nombre dependiendo sus parámetros, pattern matching nos permite hacer algo similar. Pudiendo así definir múltiples veces la misma función dependiendo el match que se realiza en sus parámetros.

Por ejemplo:

```elixir
def fac(0), do: 1
def fac(n), do: n * fac(n-1)
```

Gracias a la combinación de estas dos propiedades podemos separar la lógica de la función dependiendo del valor del parámetro recibido. En este caso cuando `fac` reciba como argumento el valor `0` la función retornará `1`. En cambio cuando `fac` reciba cualquier otro valor `n` se realizará el cálculo de `n *fac(n-1)` utilizando recursividad.

En un lenguaje como Go tendríamos que recurrir a algo 

```golang
func fac(n int) int64 {
	if n == 0 {
		return 1
	}

	return int64(n) * fac(n-1)
}
```

Donde tendríamos que tener un bloque `if` para poder dividir la función y así obtener dos resultados dependiendo el valor del parámetro. ¿Qué pasa con funciones más complejas? En lenguajes "tradicionales" tendríamos que meter condiciones dentro del cuerpo de nuestras funciones para así poder obtener diferentes resultados dependiendo su parámetro. Pero aprovechando la ventaja que nos da pattern matching y la aridad de las funciones en Erlang y Elixir podemos escribir código como los siguientes ejemplos.

```elixir
def eat_ghost?(true, true), do: true
def eat_ghost?(_, _), do: false

def exp(x, n), do: exp(x, n, 1)
def exp(_, 0, acc), do: acc
def exp(x, n, acc), do: exp(x, n - 1, x * acc)
```

En conclusión: Erlang y Elixir nos ofrecen desde sus características más básicas herramientas muy poderosas parar poder escribir código limpio y descriptivo.
