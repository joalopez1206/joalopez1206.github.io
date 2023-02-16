---
layout: page
title: "Cmake tutorial piola"
permalink: /cmake
---
### SI SABEN INGLES, RECOMIENDO MUCHO VER EL SIGUIENTE [TUTORIAL](https://hsf-training.github.io/hsf-training-cmake-webpage/01-intro/index.html) SI QUIEREN PROFUNDIZAR 

### Esto es un pequeño tutorial de como funciona cmake en el flight software, pero antes necesitamos saber que es cmake.

Cmake es un sistema para buildear proyectos (Buildear se refiere a contruir el proyecto, osea compilaciones y linkeos de librerias que vamos a usar), lo que nos ahorra el paso de realizar linkeos de liberias via gcc que uno piensa que puede "llegar y usar" pero no es así.
Por ejemplo si queremos compilar algo con threads de POSIX  usariamos el siguiente comando 
```bash
    gcc -o target_exec target_file.c -lpthread
```
Entonces compilamos nuestro `.c` con pthread, pero ahora que pasa si, nuestro `.c` lo queremos usar como una libreria y solo queremos un `.a` (estatica) o un `.so` (dinamica). Se puede hacer con gcc (y es muy WEBIADO, un link de ayuda [aqui](https://www.cprogramming.com/tutorial/shared-libraries-linux-gcc.html)) pero ahora tenemos que ver que ocurre con si tenemos un archivo que requiere esa libreria, si esto ocurre tendriamos que
1. Compilar primero la libreria 
2. Despues compilar nuestro archivo con la libreria ya compilada (ie es un `.a` o un `.so`)

Pero ahora esto se vuelve un problema si tenemos varios archivos con varias dependencias, por lo tanto se vuelve importante un sistema para buildear.

Ahora que tenemos un poco de motivacion ¿de que trata CMake?
Cmake intenta abstraer un poco este problema usando un concepto importante, __targets__.
Los __targets__ son objetivos que le decimos a cmake que tiene que construir, para fines practicos estos son 2
1. Executables (ejecutables)
2. Libraries (librerias)
Ahora nosotros le podemos decir a cmake que haga una libreria o ejecutable con el siguiente comando 

```cmake 
add_library(<LIB_NAME> <SRC_FILES>)
add_executable(<EXEC_NAME> <SRC_FILES>)
```