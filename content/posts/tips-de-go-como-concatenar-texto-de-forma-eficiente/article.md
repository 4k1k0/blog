---
title: "Tips de Go: Cómo concatenar texto de forma eficiente"
date: 2022-09-17T02:09:29-06:00
draft: true # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: 'Solución más óptima para concatenar texto utilizando Go'
categories:
  - programming
tags:
  - go
---

En ocasiones es necesario unir dos o más cadenas de texto en una sola. Qué mejor que hacerlo de una manera eficiente. La manera "tradicional" o como haríamos en otros lenguajes es concatenar cada elemento a una cadena de texto. Para esto utilizaríamos el operador `+=` en una cadena de texto para añadir cada elemento de nuestra colección.

```golang
res := ""
for _, f := range fields {
  res += f
}
```

Es un código simple, pero no muy eficiente. Crearemos un benchmark para demostrarlo. Para esto crearemos en un archivo de pruebas una función que nos ayudará a tener data de prueba. Utilizaremos [faker](https://github.com/jaswdr/faker) para generar datos falsos.

```golang
func createFakeDataForBenchmark(b *testing.B) []string {
	b.Helper()
	faker := faker.New()
	n := 100_000
	res := make([]string, n)

	for i := 0; i < n; i++ {
		line := faker.Person().Name()
		res[i] = line
	}
	return res
}
```

Nuestra función de data falsa nos creará un slice de 100,000 nombres. El cual será utilizado en nuestro benchmark.

```golang
func BenchmarkConcat(b *testing.B) {
	fields := createFakeDataForBenchmark(b)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		res := ""
		for _, f := range fields {
			res += f
		}
	}
}
```

Ejecutaremos nuestro benchmark y guardaremos los resultados en un archivo txt.

```shell
$ go test -bench=. -count=20 > old.txt
```

De esta manera nos aseguramos de que ejecutemos 20 veces nuestro benchmark y cada resultado lo almacenamos en nuestro archivo `old.txt`.

```shell
$ cat old.txt
goos: darwin
goarch: arm64
pkg: testGo
BenchmarkConcat-8   	       1	10799521250 ns/op
BenchmarkConcat-8   	       1	8960573042 ns/op
BenchmarkConcat-8   	       1	8948590917 ns/op
BenchmarkConcat-8   	       1	9240871750 ns/op
BenchmarkConcat-8   	       1	9568445417 ns/op
BenchmarkConcat-8   	       1	9483034167 ns/op
BenchmarkConcat-8   	       1	9359205584 ns/op
BenchmarkConcat-8   	       1	9431272208 ns/op
BenchmarkConcat-8   	       1	9284579833 ns/op
BenchmarkConcat-8   	       1	9633018708 ns/op
BenchmarkConcat-8   	       1	9615555208 ns/op
BenchmarkConcat-8   	       1	9435577625 ns/op
BenchmarkConcat-8   	       1	9472732667 ns/op
BenchmarkConcat-8   	       1	9526675042 ns/op
BenchmarkConcat-8   	       1	9425694959 ns/op
BenchmarkConcat-8   	       1	9336684417 ns/op
BenchmarkConcat-8   	       1	9773708583 ns/op
BenchmarkConcat-8   	       1	9535284291 ns/op
BenchmarkConcat-8   	       1	9922773833 ns/op
BenchmarkConcat-8   	       1	9593992500 ns/op
PASS
ok  	testGo	191.381s
```

Podemos notar que nuestro benchmark fue ejecutado 20 veces y cada una de ellas la función pudo realizarse una única vez. Demorando una cantidad de tiempo considerable para concatenar unas 100,000 cadenas de texto.

Ahora haremos unos cambios a nuestra solución para hacerla más óptima. Para esto utilizaremos un [string builder](https://pkg.go.dev/strings#Builder). La cual es una estructura de la biblioteca estándar la cual está optimizada para el manejo de cadenas de texto.

Nuestro benchmark utiliza esta estructura para concatenar eficientemente cada uno de los elementos de nuestra data falsa. Para esto se vale del método `WriteString`.

```golang
func BenchmarkConcat(b *testing.B) {
	fields := createFakeDataForBenchmark(b)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		var res strings.Builder
		for _, f := range fields {
			res.WriteString(f)
		}
	}
}
```

De igual manera ejecutamos y almacenamos el resultado de nuestro benchmark en un archivo de texto.

```shell
$ go test -bench=. -count=20 > new.txt
```

A simple vista podemos ver una diferencia significativa entre ambos archivos.

```shell
$ cat new.txt
goos: darwin
goarch: arm64
pkg: testGo
BenchmarkConcat-8   	     627	   1920271 ns/op
BenchmarkConcat-8   	     621	   1924366 ns/op
BenchmarkConcat-8   	     619	   1936682 ns/op
BenchmarkConcat-8   	     610	   1938242 ns/op
BenchmarkConcat-8   	     613	   1950456 ns/op
BenchmarkConcat-8   	     680	   1554438 ns/op
BenchmarkConcat-8   	     868	   1580795 ns/op
BenchmarkConcat-8   	     794	   1563124 ns/op
BenchmarkConcat-8   	     747	   1529450 ns/op
BenchmarkConcat-8   	     733	   1536551 ns/op
BenchmarkConcat-8   	     732	   1532616 ns/op
BenchmarkConcat-8   	     781	   1568152 ns/op
BenchmarkConcat-8   	     790	   1535016 ns/op
BenchmarkConcat-8   	     745	   1536268 ns/op
BenchmarkConcat-8   	     843	   1746608 ns/op
BenchmarkConcat-8   	     736	   1533825 ns/op
BenchmarkConcat-8   	     786	   1583720 ns/op
BenchmarkConcat-8   	     619	   1712188 ns/op
BenchmarkConcat-8   	     802	   1550521 ns/op
BenchmarkConcat-8   	     751	   1589650 ns/op
PASS
ok  	testGo	30.768s
```

Donde en el benchmark anterior nuestros resultados no pasaban de una ejecución en este caso tenemos hasta un total de 868 ejecuciones. Pero para poder comprobar de una manera más óptima el resultado de nuestros benchmarks utilizaremos la herramienta [benchstat](https://github.com/golang/perf).

```shell
$ benchstat old.txt new.txt

name      old time/op  new time/op  delta
Concat-8   9.48s ± 5%   0.00s ±17%  -99.98%  (p=0.000 n=18+20)
```

Benchstat nos da información acerca del promedio de cada uno de nuestros benchmarks. La columna `delta` nos indica una dismunución en tiempo de ejecución entre nuestra prueba `old` y nuestra prueba `new`. Siendo una reducción de -99.98% de tiempo bastante significativo entre nuestras dos soluciones. Siendo así `strings.Builder` la solución más óptima para este tipo de situaciones.

Así que ya lo sabes. La próxima vez que tengas que crear una cadena de texto concatenando varios elementos no utilices la manera "clásica" y utiliza en su lugar un `string builder`.

