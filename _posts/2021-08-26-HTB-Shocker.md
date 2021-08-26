---
layout: single
title: Hack the box-Shocker 
excerpt: "Maquina Shocker hack the box"
date: 2021-08-26
classes: wide
header:
  teaser: /assets/images/shocker/shocker-portada.png
categories:
  -HTB -CTF
tags:  
  - HTB
  - enumeration
  - explotacion
---
Esta es una resolucion completa de la maquina shocker de hack the box una maquina linux de 64 bits.
### Nivel de dificultad:

Intrusion = facil.

Escala de privilegios = facil. 
## Reconocimiento de puertos
Realizamos el escaneo con nmap 
```
nmap -p- --open -n -T5 -v 10.10.10.56
```
### Salida de nmap: 
<p align="left">
<img src="/assets/images/shocker/salida-shocker.png">
</p>

## Vemos que tiene un servidor http por el puerto 80 al entrar a esa pagina web vemos la suigente imagen:
<p align="left">
<img src="/assets/images/shocker/web-shocker.png">
</p>

### El codigo fuente muestra esto:
<p align="left">
<img src="/assets/images/shocker/codigofuente-shocker.png">
</p>
### Wappalyzer:
<p align="left">
<img src="/assets/images/shocker/wapalyzer.png.png">
</p>
No se ve nada interesante en la pagina por ahora, hare un escaneo de los servicios y versiones que esten corriendo en los puertos  



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
