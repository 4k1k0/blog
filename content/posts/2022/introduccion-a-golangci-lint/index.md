---
title: "Introduccion a Golangci Lint"
date: 2022-11-27T03:28:01-06:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: 'Linter para lenguaje de programación Go'
categories:
  - programming
tags:
  - go
---

Cuando comencé a aprender Go por allá en el 2017 me fascinó la cantidad de herramientas de desarrollo que Go nos brinda en su instalación. Una de estas herramientas siendo gofmt, la que más llamó mi atención en ese momento. Una herramienta CLI que reescribía nuestro código fuente en el formato estándard aprobado y recomendado por la comunidad. A diferencia de otros lenguajes Go nunca tuvo debates sobre una guía de estilo, siendo fmt la norma recomendada y diseñada por las mismas personsas que diseñan el lenguaje.

Sin embargo, aunque gofmt nos permite formatear nuestro código utilizando un diseño estandard aún así necesitamos una herramienta externa para poder establecer otras normas para nuestro código. Chequeos como: número de líneas o argumentos por funciones, elementos nos utilizado o casteos innecesarios son algunos ejemplos que necesitan una herramienta más personalizable.

Ahí es donde entra [golangci-lint](https://golangci-lint.run/usage/linters/). Golangci-lint es por si mismo un ejecutor de una colección de diferentes linteres. Es decir, no es un linter por si mismo, sino que utiliza diferentes linters escritos por la comunidad de Go para poder ofrecer en un solo paquete el uso de estas herramientas.

Cuenta con diferentes opciones de instalación para macOS, Linux y Windows; así como scripts de instalación en herramienta de CI/CD para utilizarlas en nuestros pipelines. Además de esto también cuenta con un plugin de [asdf](https://asdf-vm.com/) para poder configurar diferentes versiones con esta herramienta.

Una vez con la herramienta instalada podemos confirmarlo con:

```shell
$ golangci-lint version
golangci-lint has version 1.49.0 built from cc2d97f3 on 2022-08-24T10:24:37Z
```

## Configuración

Esta herramienta puede configurarse con archivos escritos en documentos de tipo yml, taml, toml o json. Lo que nos permite tener configuraciones personalizadas por proyectos así como globales. Simplemente tendremos que colocar estos archivos en la raíz de nuestros proyectos o definir un path durante la ejecución; de otro modo golangci-lint buscará un archivo de configuración global en el directorio del usuario. Si un archivo de configuración no es encontrado se utiliza la configuración por defecto.

Podemos configurar cosas como timeout, ejecución concurrente, revisión de archivos de tests, archivos o directorios a ignorar (muy útil para ignorar cosas como vendor o archivos generados), formato de salida, etc. Además de estas opciones sobre la configuración de la herramienta podemos configurar en este archivo el conjunto de linters a ejecutar para analizar nuestro código. Esto es muy importante en caso de que querramos agregar o quitar linters a la configuración y ejecución por defecto.


## Ejemplo

Para este ejemplo utilizaremos el siguiente archivo de configuración:

```yml
linters:
  enable:
    - errcheck
    - funlen
    - gofmt
    - gosimple
    - ifshort
    - predeclared

linters-settings:
  funlen:
    lines: 20
    statements: 20
  gofmt:
    simplify: true

run:
  skip-dirs:
    - .git
    - vendor
```

Y analizaremos el siguiente programa:

```golang
package main

import (
	"fmt"
	"strconv"
)

func Foo() {
	fmt.Println("foo")
	fmt.Println("foo")
	fmt.Println("foo")
}

func Bar() {
	fmt.Println("foo")
	fmt.Println("foo")
	fmt.Println("foo")
}

func Example() (string, error) {

	n := 1

	n = 100

	n = 10

	if n < 100 {

		res, _ := strconv.ParseComplex("", 1)

		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.
		// Lorem.

		fmt.Println(res)

	}

	return "Example", nil
}

func main() {
	Foo()
	Bar()
	Example()
}

```

En este ejemplo contamos con tres funciones de prueba las cuales tienen diferentes problemas. El código compila y si verificamos con gofmt está escrito de manera correcta siguiendo la guía de estilo de Golang. Sin embargo con golangci-lint podemos verificar alguna cosas de las cuales puede que nos querramos hacer cargo antes de hacer commit de nuestros cambios.

```shell
$ golangci-lint run
main.go:58:9: Error return value is not checked (errcheck)
        Example()
               ^
main.go:20: Function 'Example' is too long (32 > 20) (funlen)
func Example() (string, error) {
main.go:22:2: ineffectual assignment to n (ineffassign)
        n := 1
        ^
main.go:24:2: ineffectual assignment to n (ineffassign)
        n = 100
        ^
main.go:30:38: SA1030: 'bitSize' argument is invalid, must be either 64 or 128 (staticcheck)
                res, _ := strconv.ParseComplex("", 1)
```

La ejecución nos da diferente información sobre los errores encontrados como: nombre del archivo y línea de código, descripción del error y nombre del linter. La línea `main.go:20: Function 'Example' is too long (32 > 20) (funlen)` nos da un error interesante: `(32 > 20)` es la comparación del número total líneas de código en nuestra función contra el número permitido en el archivo de configuración. Si cambiamos nuestra configuración podremos ver como este mensaje desaparece de la lista de errores.

```yml
linters-settings:
  funlen:
    lines: 100
    statements: 100
```

```shell
$ golangci-lint run
main.go:58:9: Error return value is not checked (errcheck)
        Example()
               ^
main.go:22:2: ineffectual assignment to n (ineffassign)
        n := 1
        ^
main.go:24:2: ineffectual assignment to n (ineffassign)
        n = 100
        ^
main.go:30:38: SA1030: 'bitSize' argument is invalid, must be either 64 or 128 (staticcheck)
                res, _ := strconv.ParseComplex("", 1)
```

Pero lo ideal sería cambiar nuestro código para que se adapte a nuestras reglas, no al revés. Así que regresaré la configuración a la original de 20 líneas.

```yml
linters-settings:
  funlen:
    lines: 20
    statements: 20
```

El código fuente tiene que ser modificado para poder cumplir con los estándares ya establecidos. Después del cambio podemos ver el nuevo archivo:

```golang
package main

import (
	"fmt"
	"log"
	"strconv"
)

func Foo() {
	fmt.Println("foo")
	fmt.Println("foo")
	fmt.Println("foo")
}

func Bar() {
	fmt.Println("foo")
	fmt.Println("foo")
	fmt.Println("foo")
}

func Example() (string, error) {
	if n := 10; n < 100 {
		res, _ := strconv.ParseComplex("", 64)
		fmt.Println(res)
	}

	return "Example", nil
}

func main() {
	Foo()
	Bar()

	if _, err := Example(); err != nil {
		log.Fatal(err)
	}
}
```

Con el archivo modificado y cumpliendo con todas las configuraciones golancgi-lint ya no debe de mostrar ningún error en su ejecución.

```shell
$ golangci-lint run
$ 
```

## Más automatización

Algunos linters son más simples que otros. Algunos reglas de configuración como de archivos en el formato de gofmt, comentarios y líneas en blanco pueden ser reparados fácilmente con el flag `--fix` durante la ejecución de la herramienta. Esto es principalmente útil si es la primera vez que ejecutas el linter en un codebase amplio, permitiendo poder reparar todos esos errores durante la primera ejecución sin tener que hacerlo a mano modificando los archivos nosotros mismos.

## Conclusión

Contar con estándares establecidos para nuestro código es muy importante, sobre todo para cuando trabajamos con más personas. De esta manera las organizaciones pueden mantener un mismo estilo de codificación siguiendo tanto las normas y guías de la comunidad oficial así como personalizar las propias. Para más información sobre golangci-lint pueden consultar su [sitio oficial.](https://golangci-lint.run)
