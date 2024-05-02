---
title: "Leer un archivo de configuración para tu aplicación de Go"
date: 2022-06-18T02:12:45-06:00
draft: true # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - programming
tags:
  - go
---

## ¿Qué es un archivo de configuración?

Un archivo de configuración es un archivo de texto plano que puede ser modificado por el usuario para dar ciertos parámetros de configuración a una aplicación. Un ejemplo de estos serían los archivos de configuración de Apache o el archivo de configuración de Git.

## ¿Existe algún formato específico?

No, no existe ningún formato específico. Bien podría ser un xml, un yml, un json o incluso existen programas que crean su propia sintaxis para sus archivos de configuración. Para este ejemplo usaremos un archivo json.

## Nuestro proyecto

Tendremos un proyecto de ejemplo con el siguiente árbol de directorios:

```shell
.
├── Makefile
├── cmd
│   └── app
│       └── main.go
├── config
│   └── config.go
├── config.json
├── go.mod
└── service
    └── foo.go
```

### Definiendo nuestro paquete de configuración

Este paquete se encargará de leer nuestro archivo de configuración. Además tendrá que hacerlo una sola vez, ya que la configuración con cambiará durante la ejecución de nuestra aplicación.

Para esto tenemos que aplicar un patrón de diseño conocido como singleton. Lo cual crea una única instancia de un tipo de dato en nuestra aplicación.

Dentro de `config/config.go` creamos una estructura para almacenar la información de nuestra configuración y un puntero de la misma.

```golang
type Config struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

var config *Config
```

Exponemos una función llamada `GetConfig` la cual retorna un `*Config`. Dentro de esta función revisamos que el puntero `config` declarado anteriormente se encuentre vacío.

```golang
if config == nil {
  log.Println("Loading config file...")
}
```

Una vez asegurados que `config` no contiene información podemos abrir el archivo de configuración utilizando el path del archivo. En este ejemplo el path se pasa directamente, pero bien podría tomarse de una variable de entorno para definir otra ruta de archivo.

```golang
file, _ := os.Open("config.json")
defer file.Close()
```

Una vez abierto el archivo se crea un decoder de tipo JSON para poder leerlo. Y en caso de error se colocan valores por defecto en una variable de tipo `Conf`.

```golang
decoder := json.NewDecoder(file)
var conf Config
err := decoder.Decode(&conf)
if err != nil {
	conf.Name = "Default Name"
	conf.Age = 21
}
```

Para poder pasar los valores de la variable `conf` al puntero `config` creamos un `Sync.Once`.

```golang
var config *Config
var once sync.Once
```

Y dentro de nuestra función de `GetConfig` utilizamos `once` para poder pasar nuestra info a `config`.

```golang
once.Do(func() {
  config = &Config{
    Name: conf.Name,
    Age:  conf.Age,
  }
})
```

Y por último `GetConfig` retornará `config` después de nuestra validación.

```golang
func GetConfig() *Config {
	if config == nil {
		log.Println("Loading config file...")
		file, _ := os.Open("config.json")
		defer file.Close()

		decoder := json.NewDecoder(file)
		var conf Config
		err := decoder.Decode(&conf)
		if err != nil {
			conf.Name = "Default Name"
			conf.Age = 21
		}

		once.Do(func() {
			config = &Config{
				Name: conf.Name,
				Age:  conf.Age,
			}
		})
	}

	return config
}
```

### Utilizando nuestra configuración

Nuestro `cmd` se encargará únicamente de iniciar nuestra aplicación llamando la función `Start` dentro de `service/foo.go`.

```golang
package main

import "config/service"

func main() {
	service.Start()
}
```

Y `service/foo.go` tendrá la funcionalidad de nuestra aplicación. Donde llamaremos a `GetConfig` para obtener nuestra configuración. 

Dentro de la función `Start` llamaremos por múltiples veces a `GetConfig` y notaremos que la inicialización del puntero sólo sucede una vez.

Obtenemos la configuración y la utilizamos para imprimir un mensaje en pantalla. Además ejecutamos las funciones `foo` y `bar`.

```golang
func Start() {
	conf := config.GetConfig()
	fmt.Printf("Hello %s. You are %d years old.\n", conf.Name, conf.Age)
	foo()
	bar()
}
```

Dentro de `foo` y `bar` haremos algo similar. Obtendremos nuestra configuración y la utilizaremos para imprimir un mensaje en pantalla.

```golang
func foo() {
	conf := config.GetConfig()
	fmt.Printf("Hello from foo> %s | %d\n", conf.Name, conf.Age)
}

func bar() {
	conf := config.GetConfig()
	fmt.Printf("Hello from bar> %d | %s\n", conf.Age, conf.Name)
}
```

Compilamos el proyecto y podemos ver que el log colocado dentro de `GetConfig` sólo se muestra una vez. Además podemos notar que se están utilizando los valores por defecto colocados en el manejo del error en `GetConfig`. Esto porque no existe aún el archivo `config.json` dentro de nuestro proyecto.

```shell
$ make compile
$ ./app                
2022/06/18 16:50:40 Loading config file...
Hello Default Name. You are 21 years old.
Hello from foo> Default Name | 21
Hello from bar> 21 | Default Name
$ 
```
Creamos el archivo `config.json` con el siguiente contenido:

```json
{
  "age": 29,
  "name": "Wako"
}
```

Y ejecutamos el programa de nuevo. No es necesario compilar.

```shell
$ ./app      
2022/06/18 16:53:18 Loadingconfig file...
Hello Wako. You are 29 years old.
Hello from foo> Wako | 29
Hello from bar> 29 | Wako
$
```

De esta forma es fácil proveer de nuestros programas de Go archivos de configuración. Pueden encontrar el código completo [aquí](https://github.com/4k1k0/examplesGo/tree/main/config). Además [acá](https://github.com/4k1k0/gopher-glitch) pueden ver un ejemplo más complejo aplicado a un generador de arte digital.
