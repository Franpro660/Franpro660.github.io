---
layout: single
title: Hack the box-Shocker 
excerpt: "Maquina Shocker hack the box"
date: 2021-08-26
classes: wide
header:
  teaser: "/assets/images/shocker/shocker-portada.png"
  teaser_home_page: true
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
## Reconocimiento general de la maquina 
### Reconocimiento de puertos
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
wfuzz -c -w /usr/share/dirb/wordlists/common.txt http://10.10.10.56/FUZZ
```
Ocupe el diccionario /usr/share/dirb/wordlists/common.txt de dirbuster pero otros diccionarios pueden dar los mismos resultados
### Salida de wfuzz:
<p align="left">
<img src="/assets/images/shocker/wfuzz.png">
</p>
Podemos ver un directorio cgi-bin/

### cgi-bin/
Esta carpeta se usa principalmente para guardar archivos con la extension cgi pero no es una obligacion que tengan esta extension para ser guardados aqui
### Archivos .cgi 
Estos archivos son utilizados principalmente para realizar tareas que no son soportadas por el estandar html 

Ej:

Base de datos

Contadores 

### Uso de wfuzz para encontrar archivos con diferentes extensiones:
Creamos un diccionario con extensiones variadas para luego usarlo

extensiones.txt
```
html 
php
pl
cgi 
txt
sh 
```
Uso de wfuzz con dos diccionarios 
```
wfuzz -c --hc=440 -t 500 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -w extensiones.txt http://10.10.10.56/cgi-bin/FUZZ.FUZ2Z
```
Encontramos un archivo user-sh
<p align="left">
<img src="/assets/images/shocker/user-sh.png">
</p>
Entramos al recurso /cgi-bin/user.sh
<p align="left">
<img src="/assets/images/shocker/recurso.png">
</p>

## Intrusion a la maquina 
para porder entrar a la maquina es necesario un ataque shellshock
