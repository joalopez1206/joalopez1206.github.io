---
layout: page
title: "Suchai CMake tutorial"
permalink: /suchai-cmake
---

Suponiendo que vieron lo anterior de cmake ahora la pregunta es

> ¿y de que me sirve esto para el suchai flight software(sfs)?

En efecto, es muy util ya que, si quieres agregar alguna libreria que maneje algun dispoisitivo (por ejemplo WiringPi si es de gpio o i2c) entonces con lo del tutorial anterior tenemos una base para logar eso!

Pero primero **¿Como se estructura el cmake del sfs?**

Veamos el codigo! Este es el primer CMake del directorio principal
```cmake
cmake_minimum_required(VERSION 3.16...3.19)

# Project name and a few useful settings. Other commands can pick up the results
project(
  SUCHAI-Flight-Software
  DESCRIPTION "SUCHAI Nanosatellite Flight Software Framework"
  LANGUAGES C)

# Only do these if this is the main project, and not if it is included through add_subdirectory
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
  set(CMAKE_C_STANDARD 99)
  set_property(GLOBAL PROPERTY USE_FOLDERS ON)
endif()

# Select app or test subproject
set(APP simple CACHE STRING "Application name")
set(TEST off CACHE BOOL "Enable testing")
if(TEST)
  set(BASE_DIR test)
else()
  set(BASE_DIR apps)
endif()
message("Application to build is: ${BASE_DIR}/${APP}")

# The compiled library code is here
include_directories(${BASE_DIR}/${APP}/include)
add_subdirectory(${BASE_DIR}/${APP})
add_subdirectory(src)
```

Identifiquemos cosas importantes primero:
1. Las primeras lineas indican el minimo del cmake y de que trata el proyecto
2. El primer `if` indica que si tenemos de proyecto principal al sfs, entonces que el C estandar esa el 99' y que usemos un sistema de folders(es por compatibilidad)
3. Le decimos que `APP` sea un `STRING` que pueda recordar su valor anterior y que `TEST` sea un booleano para saber si activamos el testing o no.
4. Luego de todo el preambulo viene __lo mas importante!__
    * Le decimos a cmake que un directorio de `includes` sea el directorio `include` que esta en nuestra aplicacion
    * Y añadimos dos subdirectorios (Esto implica que tiene que haber un cmake en cada uno de estos)
        1. La de nuestra aplicacion a buildear
        2. La de src que contiene el sistema del suchai 

Entonces ahora hay que analizar los cmakes de cada uno de estos, como nuestra meta primero es intentar meter una libreria, veremos el cmake de la aplicacion, despues veremos como podemos ver el cmake de la carpeta src, ya que ese tiene un tema un poco mas complicado.