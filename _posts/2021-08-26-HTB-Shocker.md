---
layout: single
title: Hack the box-Shocker 
excerpt: "Maquina Shocker hack the box"
date: 2021-08-26
classes: wide
header:
  teaser: https://i.ytimg.com/vi/JGLoEts33tI/mqdefault.jpg
categories:
  -HTB -CTF
tags:  
  - HTB
  - enumeration
  - explotacion
---
Esta es una resolucion completa de la maquina shocker de hack the box una maquina linux de 64 bits.
## Nivel de dificultad:
intrusion = facil.
escala de privilegios = facil. 

# Reconocimiento de puertos
Realizamos el escaneo con nmap 
```
nmap -p- --open -n -T5 -v 10.10.10.56
```
Esta es la salida de nmap: 


Vemos que tiene un servidor http por el puerto 80 al entrar a esa pagina web vemos la suigente imagen:



## Uso de nmap con parametros
### Parametros de nmap 
Nmap ocupa parametros para funcionar existe una gran variedad de parametros en esta guia se veran algunos de ellos.
### Parametro -p-
Este parametro escanea todo el rango de puertos disponibles (65536 puertos).
```
nmap -p- (ip)
```
### Parametro -p
Este parametro se usa para "apuntar" a un puerto en especifico 
```
nmap -p 80 (ip)
```
Tambien se puede usar para un rango de puertos en especifico
```
nmap -p 10-80 (ip)
```
### Parametro -v
Este parametro va mostrando a tiempo real los resultados de los escaneos por pantalla
```
nmap -v (ip)
```
### Parametro -n 
Este parametro quita la resolucion dns lo cual agiliza el escaneo
```
nmap -n (ip)
```
### Parametro -T(numero)
Este parametro puede ser usado con 5 niveles (T1, T2, T3, T4, T5) 
```
nmap -T5 (ip)
```
Advertencia: este parametro hace que nuestro escaneo sea mas ruidoso/visible mientras mas alto sea el parametro mas ruidoso suele ser.
### Parametro -sC
Este parametro sirve para ver el servicio el cual este corriendo en un puerto 
``` 
nmap -sV (ip) 
```
### Parametro -sV 
Este parametro Sirve para ver la version del servicio el cual este corriendo en un puerto este parametro suele venir acompañado de el parametro -sC
```
nmap -sV (ip)
```
### Parametro -oG 
Este parametro sirve para poder exportar nuestro escaneo a un fichero grepeable
```
nmap (ip) -oG Puertos
```
### Otros
Podemos filtrar con el parametro open para decirle a nmap que solo necesitamos los puertos con estatus open,

este comando tambien puede servir para filtrar por cerrados o filtrados 
```
nmap --open (ip)
```
## Junta de varios parametros
Todos estos parametros pueden ir juntos en un solo comando lo cual agiliza mucho mas el escaneo en si

Ej:
```
nmap -p- -n --open -T5 -v (ip) -oG puertos
```
Imaginemos que este escaneo nos da como resultado los puertos 80, 50 abiertos

Lo que se podria hacer es hacer un escaneo de el servicio que este corriendo en estos puertos y la version de este mismo
``` 
nmap -sV -sC -p 80, 50 (ip)
```
En base a esto ya se podria ver si ese servicio/version es vurnerable y buscar o programar algun exploit para posteriormente atacar ese servicio.

Esta guia recorre algunos de los comandos y parametros posibles de nmap pero existen muchos mas puedes ver el manual de instrucciones de nmap escribiendo esto en tu terminal
```
man nmap
```

Esta guia esta hecha con fines educativos y realizacion de CTFs.
