---
title: "Erlang: Nifs"
date: 2025-10-30T22:41:40-06:00
draft: true # Set 'false' to publish
tableOfContents: true # Enable/disable Table of Contents
description: ''
categories:
  - programming
tags:
  - erlang
  - elixir
  - nif
---

* **Interoperabilidad**
  : 1. Gral. Habilidad de dos o más sistemas o de sus componentes para utilizarse de forma conjunta e intercambiable.
  : 2. Adm. Capacidad de los sistemas de información, y por ende de los procedimientos a los que estos dan soporte, de compartir datos y posibilitar el intercambio de información y conocimiento entre ellos.

## Descripción

Erlang ofrece algunas ventajas como programación funcional, inmutabilidad, concurrencia y **OTP**. Todo esto es algo sensacional al momento de diseñar software concurrente y con múltiples procesos. A pesar de todas las ventajas que Erlang nos ofrece; y lenguajes similares que se ejecuten en **BEAM**, sigue siendo demasiado lento para tareas que requieren alto poder de cómputo. Como son; por ejemplo, operaciones complejas, manipulación de imágenes y multimedia, procesamiento de video y audio, etc. Para resolver esto el equipo de Erlang ofrece la posibilidad de integrar **interoperabilidad**.

La interoperabilidad es la capacidad de poder ejecutar código de un lenguaje de programación diferente al que estamos ejecutando. Por ejemplo, la JVM permite ejecutar diferentes lenguajes de programación compatibles en conjunto. Go permite ejecutar código de C. Hasta lenguajes como Elixir o Gleam son capaces de utilizar código de Erlang. Pero hay ..


## Interoperabilidad en la BEAM

Una de las formas de interoperabilidad que nos ofrece Erlang son los NIF (Native Implemented Funcion). Esto nos permite ejecutar código de otros lenguajes de programación como C o Java para tareas demandantes. El NIF se compila y se liga a la **BEAM**, así que es la manera más rápida de ejecutar código externo, a diferencia de los **Ports**.

Hay que recalcar que un NIF puede hacer que la **BEAM** tenga un fallo de ejecución. Así que las funciones NIF son más rápidas, pero con riesgo de provocar un error en la ejecución de Erlang y su máquina virtual.

Esta característica no es única de Erlang, el resto de lenguajes del entorno **BEAM** pueden beneficiarse de esto. Tanto Elixir como Gleam pueden utilizar código de C para procesos demandantes, y aún mejor, las NIF no están límitadas a código escrito en C, sino que pueden ser escritas en otros lenguajes de bajo nivel como [Zig](https://ziglang.org) y [Rust](https://rust-lang.org).

## Referencias

- [NIFs - Erlang System Documentation](https://www.erlang.org/doc/system/nif.html)
- [Interoperability Tutorial](https://erlang.org/documentation/doc-5.6/pdf/tutorial.pdf)
- []()
