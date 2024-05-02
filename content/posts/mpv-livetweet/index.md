---
title: "MPV-Livetweet"
date: "2015-08-08"
categories: 
  - "linux"
tags: 
  - "mpv"
  - "twitter"
---

mpv-livetweet es un script para el reproductor de video mpv el cual permite subir capturas de pantalla a Twitter directamente desde el reproductor mpv.

**Los requerimientos para que este script funcione son:**

- Lua 5.1 o 5.2 (por el momento no funciona con 5.3)
- Luarocks
- Luatwit y Luasockets (los cuales se instalan con luarocks: sudo luarocks install luatwit luasocket)
- Zenity

**Instalación**

- [Descargar el proyecto desde Github](https://github.com/steinuil/mpv-livetweet).
- Dirigirnos a la carpeta del proyecto.
- Ejecutar **_lua get-keys.lua_** y seguir las instrucciones para obtener las llaves.
- Abrir el archivo **mpv-livetweet.lua** con algún editor de texto y seguir las instrucciones para colocar las llaves del paso anterior.
- Mover el archivo **mpv-livetweet.lua** a **~/.config/mpv/scritps**

Ahora ya puedes hacer spam en Twitter directamente de tu reproductor de video.
