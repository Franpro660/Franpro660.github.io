---
layout: single
title: Hack the box-Shocker 
excerpt: "Maquina Shocker hack the box"
date: 2021-08-26
classes: wide
header:
  teaser: "/assets/images/shocker/shocker-portada.png"
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
<img src="/assets/images/shocker/wapalyzer.png">
</p>
### Uso de wfuzz en busca de rutas potenciales:
```
wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.10.56/
```
Ocupe el diccionario directory-list-2.3-medium.txt de dirbuster pero otros diccionarios pueden llegar a dar los mismos resultados
