---
layout: single
title: Nmap - Guia de uso nmap
excerpt: "Guia basica de uso de la herramienta nmap."
date: 2021-08-11
classes: wide
header:
  teaser: nmap.png
categories:
  -nmap
tags:  
  - nmap
  - enumeration
---
Esta es una guia basica de la herramienta nmap.
Pagina de github oficial de nmap:
https://github.com/nmap/nmap



## Uso de nmap 
### Escaneo de puertos basico

Este escaneo no es recomendado solo es un ejemplo basico del uso de nmap 
(todos estos comandos se usan desde la terminal)
```
nmap (ip)
```
Ejemplo:
```
nmap 143.0.13.4
```
Estos escaneos pueden usarse con mas de una ip
```
nmap 143.0.13.5 192.168.1.136
```
Uso de subred
```
nmap 192.167.9.*
```
Escaneo de puertos mediante un fichero .txt con las ips anotadas
```
nmap -iL objetivos.txt
```
fichero objetivos.txt:
```
192.168.1.124
192.168.1.129
192.168.1.123
192.168.1.125
```
Escaneo de rango de ip 
```
nmap 192.168.1.123-140
```
Uso de nmap para conocer el sistema operativo del objetivo
```
nmap -o 192.168.1.143
```
Uso de nmap para listar equipos dentro de una red (Solo activos).
```
nmap -sP 192.157.1.*
```
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
Podemos filtrar con el parametro open para decirle a nmap que solo necesitamos los puertos con estatus open 
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

Esta guia esta hecha con fines educativos y realizacion de CTFs.
