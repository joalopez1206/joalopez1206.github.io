---
layout: page
title: "Notas de la practica"
permalink: /suchai
---

## Semana 1
En la primera semana, estuve familiarizandome con el raspberry pi. Lo primero fue instalar el
RaspberryPi OS y de ahí fue ver como usar el GPIO y setearme con mi ambiente de trabajo, lo cual fue installar mint en un computador del laboratorio. Tambien realize los ejercicios de los comandos del suchai flight software(sfs) [repo aqui](https://gitlab.com/spel-uchile/suchai-flight-software). Llegando a hacer comandos en C y ver el funcionamiento y la filosofia el cual usan el command pattern para ejecutar comandos enviados del ground station al obc. Esta semana me dijieron que la meta seria construir un HAL para usar en el suchai, en particular para el i2c.

 ## Semana 2
 Despues de familiarizarme un poco con el software, en particular usar cmake lo cual me tomó un par de semanas, tambien replique el script de python de samuel en C para usar el sensor de giroscopio por i2c, tambien le pedi al profesor Diaz que me agendara unas reuniones con
 Carlos, el creador del sfs, ya que una de las formas para probar el script de python seria:

 1. Pasarlo a comando de flight software y ver que funcione
 2. Despues intentar hacer que funcione en un esp32

Pero tengo que portar el sfs a un esp32 y de ahí tuve que probar el [ESP idf](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/)

## Semana 3
Tuve la primera reunion con carlos para empezar a esbozar la idea sobre como portar el sfs al esp32 usando idf, y por este tiempo empece a entender bien como se buildeaba el proyecto con `cmake`, tambien empece a notar que la libreria libcsp, un submodulo del proyecto, tiene otro sistema de build `waf`, lo cual sera un inconveniente mas tarde, tambien en esta semana, implementé el comando para el suchai con la libreria wiringPi pero tambien en una reunion con carlos me comento que el sfs tenia implementado el i2c read y write que use en la libreria wiringPi y la idea es mantener el software autocontenido y por lo tanto lo mejor seria reescribir el comando con el i2c, el cual es el hal del sfs