---
layout: page
title: "Suchai CMake tutorial"
permalink: /suchai-cmake
---
> Esta parte del tutorial Supone que uds ya vieron como funciona el sfs en terminos de hacer aplicaciones y comandos, si no saben [aqui](./suchai_tutorial.md) está el tutorial.

Suponiendo que vieron lo anterior de cmake ahora la pregunta es:

>> ¿Y de que (chota) me sirve esto para el suchai flight software _(sfs)_?

Como veremos, es muy util ya que, si quieres agregar alguna libreria que maneje algun dispoisitivo (por ejemplo WiringPi, si es de gpio o i2c) entonces con lo del tutorial anterior tenemos una base para logar eso!

Pero primero **¿Como se estructura el cmake del sfs?**

Primero veamos la estructura un poco del directorio principal del flight software
```bash
suchai-flight-software
├── apps # <- Carpeta de las aplicaciones
├── build
├── cmake
├── CMakeLists.txt #<- este es el primer CMake que veremos!
├── CODE_OF_CONDUCT.md
├── compile.py
├── COPYING
├── docs
├── include
├── LICENSE
├── README.md
├── run_tests.sh
├── sandbox
├── src #<- Carpeta importante 
└── test
```
Notemos que tenemos el Top CMake que le llamaremos 'primer CMake'. Veamos su codigo! Este es el primer CMake del directorio principal 
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
    * Le decimos a cmake que un directorio de `includes` sea el directorio `include` que esta en nuestra aplicacion (¿Porque hacemos esto? Lo veremos en el cmake de src.)
    * Y añadimos dos subdirectorios (Esto implica que tiene que haber un cmake en cada uno de estos)
        1. La de nuestra aplicacion a buildear
        2. La de src que contiene el sistema del suchai 

Entonces ahora hay que analizar los cmakes de cada uno de estos, como nuestra meta primero es intentar usar una libreria para una aplicacion, veremos el cmake de la aplicacion, despues veremos como podemos ver el cmake de la carpeta src, ya que ese tiene un tema un poco mas complicado.

Veamos el cmake de el projecto simple, igual antes no esta demas recordar donde estamos!
en la carpeta `/apps`
```bash
apps
├── simple
│   ├── CMakeLists.txt #<- CMake lists de la aplicacion simple
│   ├── include
│   │   └── app
│   │       └── system
│   │           ├── cmdAPP.h
│   │           ├── repoDataSchema.h
│   │           └── taskHousekeeping.h
│   └── src
│       └── system
│           ├── cmdAPP.c
│           ├── main.c
│           └── taskHousekeeping.c
└── suchai
    ├── CMakeLists.txt
    ├── include
    │   └── app
    │       ├── cmdADCS.h
    │       ├── cmdEPS.h
    │       ├── cmdGSSB.h
    │       ├── cmdRW.h
    │       ├── cmdSensors.h
    │       ├── taskADCS.h
    │       └── taskSensors.h
    └── src
        ├── cmdADCS.c
        ├── cmdGSSB.c
        ├── cmdRW.c
        ├── cmdSensors.c
        ├── initApp.c
        ├── taskADCS.c
        └── taskSensors.c
```
¿y como luce el CMake del la app `simple`?

```cmake
###
# SIMPLE APP CMAKE FILE
##
set(SOURCE_FILES
        src/system/main.c
        src/system/taskHousekeeping.c
        src/system/cmdAPP.c
)

add_executable(suchai-app ${SOURCE_FILES})
target_include_directories(suchai-app PUBLIC include)
target_link_libraries(suchai-app PUBLIC suchai-fs-core)
```
Hagamos el mismo analisis anterior
1. Noten que aqui usamos el estandar que vimos antes en el tutorial de cmake, dejamos todos los `.c` en una variable `SOURCE_FILES`
2. Aqui añadimos el ejecutable con los que usa los sourcefiles que asignamos recien
3. Aqui le decimos que los includes para el ejecutable `suchai-app` estan en la carpeta include (esto explica porque cuando en nuestro codigo de alguna aplicacion del suchai, tenemos que incluir las cosas con la forma `#include "app/system/"...".h"`)
4. Finalmente, le decimos que linkee nuestro ejecutable con la libreria suchai-fs-core,que es la maquinaria del suchai!

Dicho esto, hagamos un ejemplo rapido de como prender un LED con una raspberry pi usando wiringPi, pero lo haremos de dos formas, siguiendo lo dicho en el tutorial anterior
1. Usando solo los archivos necesarios de wiringPi y importando de la forma `#include "...".h`

2. Usando wiringPi suponiendo que esta preinstalado ie, Colocandolo como dependencia en CMake y importandolo de la forma `#include <...>.h`
