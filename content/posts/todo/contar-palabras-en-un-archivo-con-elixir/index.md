---
title: "Contar Palabras en Un Archivo Con Elixir"
date: 2024-05-15T20:08:01-06:00
draft: true # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - programming
tags:
  - elixir
---

# Contar palabras en un archivo con Elixir

```shell
$ mix new count
```

File tree<<

File content:

```shell
$ cat hello.txt
Hola mundo
```

```shell
$ iex
Erlang/OTP 26 [erts-14.2] [source] [64-bit] [smp:16:4] [ds:16:4:10] [async-threads:1] [jit:ns]

Interactive Elixir (1.16.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)>
{:ok, content} = File.read("hello.txt")
{:ok, "Hola mundo\n"}
```

Fail read

```shell
iex(2)> {:ok, content} = File.read("foo.txt")
** (MatchError) no match of right hand side value: {:error, :enoent}
    (stdlib 5.2) erl_eval.erl:498: :erl_eval.expr/6
    iex:5: (file)
```

Means: `{:error, :enoent}` 

```shell
iex(2)> h File.read
                             def read(path)                              

  @spec read(Path.t()) :: {:ok, binary()} | {:error, posix()}

Returns {:ok, binary}, where binary is a binary data object that
contains the contents of path, or {:error, reason} if an error occurs.
```

Transform string into list


