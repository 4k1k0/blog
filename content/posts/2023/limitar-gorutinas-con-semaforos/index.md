---
title: "Limitar Gorutinas Con Semáforos"
date: 2023-08-10T05:56:08-06:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: 'Limitar tareas concurrentes'
categories:
  - programming
tags:
  - go
  - concurrency
---

## Problema

Tenemos cierto número de tareas que podemos manejar de manera concurrente. Dichas tareas podrían ser el envío de correos electrónicos, procesar archivos o consultas a bases de datos. Da igual. Son tareas que requieren un tiempo considerable para su ejecución y para poder hacer más rápido el proceso decidimos escribir código concurrente para manejar dichas tareas. Este código concurrente espera poder ejecutar `n` cantidad de tareas de manera paralela.

Para este ejemplo revisaremos el problema al que me enfrenté en mi proyecto [togray](https://github.com/4k1k0/togray). El cual es una herramienta CLI para poder editar múltiples imágenes para aplicar filtros como escala de grises y saturación.

Después del proceso de pedir información al usuario por la línea de comandos, verificar los directorios y archivos tenemos la problematica del procesamiento de imágenes de manera concurrente.

```golang
func Run(flagPath string) {
	path := getBasePath(flagPath)
	pictures := getImagesNames(path)
	numberOfPictures := len(pictures)
	if numberOfPictures == 0 {
		log.Printf("There are no images to process")
		return
	}

	wg := sync.WaitGroup{}
	wg.Add(numberOfPictures)

	for _, picture := range pictures {
		go func(picture *Picture) {
			process(&wg, picture)
		}(picture)
	}

	wg.Wait()

}
```

En el proceso declaramos un `waitGroup` con el número total de gorutinas a ejecutar. Esto en su momento funcionó muy bien. Ejecuté el programa con algunas imágenes pequeñas en una computadora con procesador M1 y no hubo mayor inconveniente. El problema vino cuando ejecuté el proceso con una gran cantidad de imágenes de gran tamaño y peso, todo en una computadora con un procesador AMD bastante modesto. Ahí fue cuando noté que el proceso consumía todos los recursos de mi sistema operativo y tuve que reiniciar mi PC.


### Semáforos

Un semáforo ayuda a limitar el número de tareas concurrentes que puede manejar nuestro equipo de cómputo. Es decir, limitamos el número de tareas que podrán realizarse en paralelo permitiendo así que ciertas tareas se completen antes de iniciar el resto. 


## Solución

Golang nos proveé de primitivos para poder manejar patrones y programación concurrente más fácilmente. Entre ellos están los `channels`, que se encargan de la comunicación entre gorutinas.

Con los `channels` básicamente tenemos una gorutina dueña, la cual se encarga de crear el `channel`, darle información o pasar el `channel` a otra gorutina. Y una gorutina que consuma el `channel` la cual sólo debe preocuparse por saber cuando es que el `channel` está cerrado.

```golang
waitChan := make(chan struct{}, n)
```

Con esta instrucción podremos crear un `channel` de una `struct` vacía la cual permitirá `n` gorutinas ejecutándose a la vez. Este semáforo será responsable y permitirá controlar la ejecución de nuestras gorutinas, permitiendo ejecutar `n` a la vez. De esta manera el resto tendrá que esperar que el `channel` tenga espacio para una tarea más.

```golang
for _, picture := range pictures {
    waitChan <- struct{}{}
    go func(picture *Picture) {
        process(&wg, picture)
        <-waitChan
    }(picture)
}
```

Notar como es que antes de ejecutar nuestra gorutina nuestro `channel` recibe información de una `struct` vacía. Y antes de terminar la gorutina nuestro `channel` libera el espacio previamente asignado.

## Solución completa

```golang
func Run(flagPath string) {
	path := getBasePath(flagPath)
	pictures := getImagesNames(path)
	numberOfPictures := len(pictures)
	if numberOfPictures == 0 {
		log.Printf("There are no images to process")
		return
	}

	maxProcess := getMaxProcess()
	waitChan := make(chan struct{}, maxProcess)
	wg := sync.WaitGroup{}
	wg.Add(numberOfPictures)

	for _, picture := range pictures {
		waitChan <- struct{}{}
		go func(picture *Picture) {
			process(&wg, picture)
			<-waitChan
		}(picture)
	}

	wg.Wait()
}
```

## Conclusión

Limitar el número de goruinas que podrá ejecutar tu equipo nos brinda la oportunidad de poder tener más control sobre nuestro hardware y el cómo procesará nuestra información. De esta manera podemos encontrar un buen balance y performance sin correr riesgos de agotar nuestros recursos. Si notas que en tu entorno hay problemas de performance puedes ir probando el número adecuado de tareas para realizar de forma concurrente.

Puedes encontrar otro ejemplo de semáforos [aquí](https://github.com/4k1k0/examplesGo/tree/main/semaphore).

