---
layout: page
title: "Notas de la practica"
permalink: /suchai
---
## Semana 1
En la primera semana, estuve familiarizandome con el raspberry pi. Lo primero fue instalar el
RaspberryPi OS y de ahí fue ver como usar el GPIO y setearme con mi ambiente de trabajo, lo cual fue installar mint en un computador del laboratorio. Tambien realize los ejercicios de los comandos del suchai flight software(sfs) [repo aqui](https://gitlab.com/spel-uchile/suchai-flight-software). Llegando a hacer comandos en C y ver el funcionamiento y la filosofia el cual usan el command pattern para ejecutar comandos enviados del ground station al obc. Esta semana me dijieron que la meta seria construir un HAL para usar en el suchai, en particular para el i2c.

 ## Semana 2
 Despues de familiarizarme un poco con el software, Replique el script de python de samuel en C para usar el sensor de giroscopio por i2c, tambien le pedi al profesor Diaz que me agendara unas reuniones con
 Carlos, el creador del sfs, ya que una de las formas para probar el script de python seria:

 1. Pasarlo a comando de flight software y ver que funcione
 2. Despues intentar hacer que funcione en un esp

Pero para intentar que hacer que funcione en un esp necesito portar el sfs a un esp32 y de ahí tengo que probar el [ESP idf](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/)
