---
title: Creando un laboratorio para threat hunting I. Introducción
author: Rafa de Vega
date: 2021-07-01 11:33:00 +0800
categories: [Threat Hunting, Laboratorio]
tags: [threat hunting,hunting,laboratorio]
---

En esta  serie de posts, nos centraremos en crear un pequeño laboratorio para nuestros ejercicios de threat hunting, existen varias soluciones ya automatizadas. como pueden ser [DetectionLab](https://github.com/clong/DetectionLab) o [adaz](https://github.com/christophetd/Adaz), que mediante un pequeño fichero de configuración te permiten desplegar un laboratorio completo en distintas plataformas de virtualización. Estas soluciones son geniales, pero lo que buscaremos montando nuestro propio laboratorio de forma manual, será aprender, y tener las herramientas justas que vamos emplear. En caso de necesitar más, las integraremos más adelante, no necesitamos desplegar un montón de herramientas que no vamos usar al principio.

## Laboratorio
Así pues, pasamos a describir nuestro pequeño laboratorio. El entorno, tratará de emular un pequeño dominio windows que podría tener cualquier empresa, pero a muy pequeña escala y sin segmentación de red inicialmente, pero con lo suficiente para ejecutar nuestra emulación de adversarios y la recolecta de posibles indicadores para el desarrollo de la detección. Así pues, las máquinas que desplegaremos son:

- **pfSense**. Será nuestro firewall, que separará y conectará nuestra red local con la red interna del laboratorio.
- **Windows Server 2019**. Actuará como controlador de dominio 
- **Windows 10**. Será la máquina que simulará la estación de trabajo de cualquier empleado. Podremos crear una, o varias dependiendo de los recursos que tengamos disponibles.
- **Red Hat 8**. Crearemos un servidor linux, para tener la opción de trabajar con los eventos de este sistema. Se elije Red Hat, ya que recientemente se ha liberado una licencia para desarrolladores que nos permitirá usarla de forma gratuita, y segurmente es uno de las distribuciones de linux más usadas a nivel empresarial. No obstante podría emplearse cualquier otra distribución de linux que admita las últimas versiones de audit.
- **Debian 11**. Será la máquina linux sobre la que desplegaremos nuestro recolector de logs/SIEM o como nos guste llamarlo. En este caso elegiremos **Splunk**. Otra muy buena opción sería usar el stack de Elastic. Sin embargo, vamos optar por la licencia gratuita de splunk (que nos permite una ingesta diaria de 500MB de logs) ya que es mucho más sencilla de instalar y configurar la ingesta de logs, lo que nos permitirá centrarnos en nuestra principal tarea, desarrollar métodos de detección.

## Requisitos 
El entorno de virtualización empleado en este caso será un ESXI, pero será aplicable también a VMWare Workstation y VirtualBox simplemente teniendo en cuenta las pequeñas diferencias entre los distintos entornos en la parte de las interfaces de red y redes internas. Los requisitos recomendados para desplegar este laboratorio son:

- 6 cores
- 16GB de RAM
- 100GB de almacenamiento

Quizá, a alguna máquina se le podrá recortar algún GB de RAM para reducir estos requisitos, o incluso, el almacenamiento dedicado a cada máquina también se pueda reducir si tomamos snapshots y las restauramos cuando el disco se vaya llenando. Otra opción, si estamos escasos de recursos, será no levantar todas las máquinas a la vez, pudiendo apagar cuando no las usemos, la Windows 10 y la Red Hat 8.

El esquema del laboratorio tendría la siguiente pinta:


## Índice

Dejamos aquí los enlaces a los distintos posts que conforman las instrucciones para desplegar nuestro pequeño laboratorio.

- [Introducción](https://rafadvega.github.io/posts/Creando-un-laboratorio-para-threat-hunting-i/)
- [Desplegando pfSense](https://rafadvega.github.io/posts/Creando-un-laboratorio-para-threat-hunting-ii/)
- [Desplegando Controlador de Dominio](https://rafadvega.github.io/posts/Creando-un-laboratorio-para-threat-hunting-iii/)
- [Desplegando máquinas de dominio](https://rafadvega.github.io/posts/Creando-un-laboratorio-para-threat-hunting-iv/)
- [Configurando la auditoría de los sistemas operativos](https://rafadvega.github.io/posts/Creando-un-laboratorio-para-threat-hunting-v/)
- [Desplegando Splunk](https://rafadvega.github.io/posts/Creando-un-laboratorio-para-threat-hunting-vi/)

![img](/assets/img/intro/img1.bmp){: width="1086" height="542"}