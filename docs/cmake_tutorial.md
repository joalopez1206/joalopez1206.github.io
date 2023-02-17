---
layout: page
title: "Cmake tutorial "
permalink: /cmake
---
> **Warning**
> texto
### SI SABEN INGLES, RECOMIENDO MUCHO VER EL SIGUIENTE [TUTORIAL](https://hsf-training.github.io/hsf-training-cmake-webpage/01-intro/index.html) SI QUIEREN PROFUNDIZAR. ESTA ES UNA VERSION ADAPTADA DE ESE TUTORIAL

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

le podemos decir a cmake que haga una libreria o ejecutable con el siguiente comando 

```cmake 
add_library(<LIB_NAME> <SRC_FILES>)
add_executable(<EXEC_NAME> <SRC_FILES>)
```

Pero antes hay un poco de boilerplate, si queremos usar cmake, tenemos que crear un __`CMakeLists.txt`__ ___¡OJO, TIENE QUE SER ESE NOMBRE!___ 

Le tenemos que decir primero como se llama el proyecto y setear el minimo requerido para usar ese cmake.

Por ejemplo, en un archivo `hello_world.c`
```c
#include <stdio.h>

int main(){
    printf("Hello world!\n");
    return 0;
}
```
Un cmakelists para este proyecto simple se veria de esta forma:
```cmake 
cmake_minimum_required(VERSION 3.15)
project(Tutorial)
add_executable(hello_world hello_world.c)
```
Expliquemos esto rapidamente

* `cmake_minimum_required(VERSION 3.15)`  le indica a cmake la minima version que se requiere para usar este cmake
* `project(Tutorial)` le indica que el nombre de este projecto se llama Tutorial
* `add_executable(hello_world hello_world.c)` le indica que estamos añadiendo un __target__ que es ejecutable, llamado hello_world de el source file `hello_world.c`

¿Y donde está este cmake en mi proyecto? Para este caso puede estar al mismo nivel de nuestro `.c` pero veremos mas adelante que el suchai flight software, los source files estan en otras carpetas y sus includes tambien. aqui hay un ejemplo de como se veria este Proyecto:
```
.
├── CMakeLists.txt
└── hello_world.c
```

Podemos incluso agregar otros parametros en project y tambien es de buena costumbre (y se vera mucho en este proyecto) que los archivos quedan en una variable llamada `SOURCE_FILES` (¿Variables? sí ! `cmake` es un lenguaje de programacion tambien, y para usar una variable tiene una syntax paracida a la de bash ie usamos `${VAR}` para referenciar lo que esta dentro de `VAR`) y podemos usar `set(<NAME_VAR> <OBJECT>)` para asignar un valor a la variable. todo lo dicho anteriormente podemos aplicarlo y queda así
```cmake 
cmake_minimum_required(VERSION 3.15)
project(Tutorial
    VERSION
        1.0
    DESCRIPTION
        "Tutorial para suchai flight software"
    LANGUAGES
        C)

set(SOURCE_FILES hello_world.c)
add_executable(hello_world ${SOURCE_FILES})
```
Ahora este `CMakeLists.txt` lo podemos usar para buildear nuestro proyecto, pero cabe destacar algo muy importante: CMake sigue una filosofia de "Buildear fuera de la fuente"(build out of source) lo que significa que, en una carpeta que nosotros escojamos, tendremos nuestro ejecutable, pero esto tiene que estar fuera de nuestro codigo fuente.

Dicho esto entonces hay que configurar nuestro cmake para que buildee fuera de nuestro source, esto se logra con el comando 
```bash
$ cmake -B build
```

Aqui le estamos diciendo a cmake que queremos que el resultado de buildear el proyecto quede en la carpeta build y va a realizar toda la configuracion automaticamente. Ahora para buildear podemos usar el siguiente comando y luego podemos cambiar de directorio y ejecutar
```bash
$ cmake --build build
$ cd build
$ ./hello_world
Hello world!
```

Ahora supongamos que yo quiero usar una funcion de libreria en un archivo llamado `mylib.c` y `mylib.h` que haga lo sigiente
```h
//este es mylib.h
#include <stdio.h>

int hola(char *name);
```
```c
//este es mylib.c
#include "mylib.h"

int hola(char *name){
    printf("hola %s\n",name);
    return 0;
}
```
Y mi archivo `hello_world.c` lo cambiamos a lo siguiente
```c
#include "mylib.h"

int main(){
  hola("matias");
  hola("profe marcos");
  hola("chiquito");
  hola("vicho");
  hola("pato");
  hola("felipe");
  return 0;
}
```

Entonces ahora nuestro problema es, queremos usarlo como funcion de libreria y no como ejecutable, entonces ¿como hacemos eso? Añadiendo un nuevo target, pero en este caso una libreria, lo normal es que esto se ubique en una carpeta aparte del main, en este caso se veria así
```bash
.
├── CMakeLists.txt
├── hello_world.c
└── MiLibreria
    ├── CMakeLists.txt
    ├── mylib.c
    └── mylib.h
```
¿Pero como CMake sabe que esta esa libreria? Se lo decimos! El cmakelists que esta en la carpeta MiLibreria, se encarga de crear la liberia. pero que hay en este cmake? Veamoslo:
```cmake
#CMakeLists de MiLibreria
add_library(mylib mylib.c mylib.h)
```
Solo esa linea, que añade un __target__ libreria llamado mylib donde estan los source_files(mylib.c) y los headers(mylib.h este paso es opcional pero deja mas claro lo que hay en mylib). Ahora queda registrada nuestra libreria, pero hay que decirle al otro cmake (el de hello_world) que incluya la libreria.

Veamos como queda:
```cmake
#Top CmakeLists
cmake_minimum_required(VERSION 3.15)

project(Tutorial
    VERSION
        1.0
    DESCRIPTION
        "Tutorial para suchai flight software"
    LANGUAGES
        C)
#Asignamos source_files a nuestro .c
set(SOURCE_FILES hello_world.c)
#Añadimos la libreria
add_subdirectory(MiLibreria)
#Añadimos el ejecutable
add_executable(hello_world ${SOURCE_FILES})
#Añadimos la libreria al include del hello_world
target_include_directories(hello_world PUBLIC MiLibreria)
#Añadimos la libreria al ejecutable
target_link_libraries(hello_world PUBLIC mylib)
```
Aqui introducimos nuevas funciones( VER ESTA PARTE):

1. `add_subdirectory(MiLibreria)` añade un subdirectorio que contenga un cmake al proyecto, en este caso estamos usando el cmake de MiLibreria, que crea la libreria `mylib`

2. `target_include_directories(hello_world PUBLIC MiLibreria)` le dice añadimos el directorio que contiene los .h a un target, así sabe donde buscar los .h, en este caso busca los punto h en MiLibreria, donde se ubica mylib.h
3. `target_link_libraries(hello_world PUBLIC mylib)` aqui le decimos que estamos linkeando la libreria mylib con hello_world, para que así el __target__ ejecutable hello_world pueda usar el __target__  mylib.

Luego de esto tenemos que reconfigurar el cmake 
```bash
$ cmake -B build
```
y buildeamos igual que siempre
```bash
$ cmake --build build
$ cd build
$ ./hello_world
hola matias
hola profe marcos
hola chiquito
hola vicho
hola pato
hola felipe
$
```
Y listo! ahora tenemos un ejecutable que usa una libreria que nosotros creamos.

Ahora por ejemplo, supongamos que una libreria esta dentro de nuestro sistema, es como una dependencia, por ejemplo Threads, entonces veamos una funcion simple de threads y veamos que onda

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
void *hola(void *args){
    char *nombre = (char*) args;
    sleep(4);
    printf("Hola desde el thread %s\n",nombre);
    return NULL;
}

int main(){
    pthread_t t1;
    pthread_create(&t1, NULL, hola,"2");
    sleep(2);
    hola("1");
    pthread_join(t1,NULL);
    return 0;
}
```
si intentamos hacer lo de toda la vida 
``` bash 
gcc -o target_exec target_file.c 
```
tendremos el siguiente error ...