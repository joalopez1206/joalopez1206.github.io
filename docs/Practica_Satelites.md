---
layout: page
title: "Notas de la practica"
permalink: /suchai
---

## Semana 1
En la primera semana, estuve familiarizandome con el raspberry pi. Lo primero fue instalar el
RaspberryPi OS y de ahí fue ver como usar el GPIO y setearme con mi ambiente de trabajo, lo cual fue installar mint en un computador del laboratorio. Tambien realize los ejercicios de los comandos del suchai flight software(sfs) [repo aqui](https://gitlab.com/spel-uchile/suchai-flight-software). Llegando a hacer comandos en C y ver el funcionamiento y la filosofia el cual usan el command pattern para ejecutar comandos enviados del ground station al obc. Esta semana me dijieron que la meta seria construir un HAL para usar en el suchai, en particular para el i2c.

## Semaana 2
 
 Despues de familiarizarme un poco con el software, en particular usar cmake lo cual me tomó un par de semanas, tambien replique el script de python de samuel en C para usar el sensor de giroscopio por i2c, tambien le pedi al profesor Diaz que me agendara unas reuniones con
 Carlos, el creador del sfs, ya que una de las formas para probar el script de python seria:

1. Pasarlo a comando de flight software y ver que funcione
2. Despues intentar hacer que funcione en un esp32

Pero tengo que portar el sfs a un esp32 y de ahí tuve que probar el [ESP idf](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/)

## Semana 3
Tuve la primera reunion con carlos para empezar a esbozar la idea sobre como portar el sfs al esp32 usando idf, y por este tiempo empece a entender bien como se buildeaba el proyecto con `cmake`, tambien empece a notar que la libreria libcsp, un submodulo del proyecto, tiene otro sistema de build `waf`, lo cual sera un inconveniente mas tarde.

Tambien en esta semana, implementé el comando para el suchai con la libreria wiringPi pero tambien en una reunion con carlos me comento que el sfs tenia implementado el i2c read y write que use en la libreria wiringPi y la idea es mantener el software autocontenido y por lo tanto lo mejor seria reescribir el comando con el i2c, el cual es el hal del sfs y por lo tanto ahora la meta seria implementar el hal para el esp32

## Semana 4

La ultima semana me dedique mas a ver como portar el sfs al esp32, ya que el hal podria ser implementado facilmente pq ESP-IDF tiene las funciones read y write i2c en sus componentes. La siguiente reunion con carlos intentamos seguir implementando el sfs al flight software al esp32 y llegamos a que
1. Falta linkear el freertos del ESP-IDF
2. Ver como podemos compilar el libCSP al esp32

Lo de linkear freertos se resolvio facilmente con un REQUIRES "freertos" en el CMake del componente, pero ahora el problema va con la libCSP, que usa un sistema de buildeo llamado `waf` y esto es un problema, ya que esp-idf funciona con CMake y si quisiesemos intentar compilar la libCSP con waf a freertos, habria que agregar cada archivo uno por uno a los includes, lo cual tomaria mucho tiempo.

La otra solución sería, las nuevas versiones de libCSP tienen disponible el CMake como sistema de buildeo, por lo tanto seria util usar esa version en el sfs, pero hace falta ver si la API es compatible con el sfs y si hay que hacer cambios, realizarlos.