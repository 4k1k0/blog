---
title: "Encriptar archivos con GPG"
date: "2015-05-16"
categories: 
  - "linux"
tags: 
  - "encriptacion"
  - "gpg"
coverImage: "2015-05-16-020339_1366x768_scrot.png"
---

Encriptar información con GPG es mucho más simple de lo que parece. Lo primero que tendremos que hacer es asegurarnos de tener instalado el paquete en nuestra distribución GNU/Linux, en caso de no tenerlo lo podemos encontrar en los repositorios como **gnupg**.

```shell
# apt-get install gnupg
```

Ya con la herramienta instalada procedemos a crear la pareja de claves:

```shell
$ gpg -gen-key
```

A lo cual nos pedirá cierta información (Tipo de llave, periodo de validez, nombre, comentario, correo electrónico y contraseña). Al terminar podemos verificar la existencia de las llaves con:

```shell
$ gpg -list-keys
```

Podemos ver las llaves privadas registradas con el comando:

```shell
$ gpg -list-secret-keys
```

Ya estamos listos para encriptar un archivo, en este ejemplo vamos a contar con un archivo de prueba llamado prueba.txt y crearemos una versión encriptada de éste llamada archivo\_encriptado.gpg

```shell
$ gpg -c -o archivo\_encriptado.gpg prueba.txt
```

![Encriptar archivos con gpg](images/screenshot-160515-020110.png)

Acto seguido nos pedirá una contraseña, la cual será la contraseña que servirá para poder desencriptar el archivo resultante.

Ya tenemos un nuevo archivo con extensión gpg, este archivo está completamente encriptado. Para poder desencriptarlo usamos:

```shell
$ gpg -o nuevo\_archivo.txt -d archivo\_encriptado.gpg
```

Lo cual nos pedirá la contraseña del archivo que escribimos anteriormente.

![Encriptar archivos con gpg](images/2015-05-16-020339_1366x768_scrot.png)

Para encriptar carpetas es un proceso similar, pero usaremos el comando gpg-zip. En este ejemplo vamos a tener una carpeta llamada carpeta\_de\_prueba la cual contiene algunos archivos de texto. Al igual que los archivos, el encriptar directorios nos pedirá una contraseña para poder desencriptarlos.

```shell
$ gpg-zip -c -o carpeta\_encriptada.gpg carpeta\_de\_prueba
```

Esto nos generará un archivo gpg que contiene nuestro directorio encriptado. Para desencriptarlo utilizamos:

```shell
$ gpg-zip -d carpeta\_encriptada.gpg
```

Ahora ya saben como encriptar sus archivos y directorios de manera sencilla.
