---
title: Creando un laboratorio para threat hunting
author: Rafa de Vega
date: 2021-08-11 11:33:00 +0800
categories: [Threat Hunting, Laboratorio]
tags: [threat hunting,hunting,laboratorio]
---

En esta  serie de posts, nos centraremos en crear un pequeño laboratorio para nuestros ejercicios de threat hunting, existen varias soluciones ya automatizadas. como pueden ser [DetectionLab](https://github.com/clong/DetectionLab) o [adaz](https://github.com/christophetd/Adaz), que mediante un pequeño fichero de configuración te permiten desplegar un laboratorio completo en distintas plataformas de virtualización. Estas soluciones son geniales, pero lo que buscaremos montando nuestro propio laboratorio de forma manual, será aprender, y tener las herramientas justas que vamos emplear. En caso de necesitar más, las integraremos más adelante, no necesitamos desplegar un montón de herramientas que no vamos usar al principio.

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

#  Desplegando pfSense.
El primer paso, será desplegar nuestro firewall. La máquina virtual de pfsense, deberá tener dos interfaces de red, una para la LAN, que se conectará a la misma red que el resto de máquinas del laboratio (la red interna) y otra para la WAN, que servirá para conectar el firewall, y por tanto todo el laboratorio, a nuestra red local y a internet. 

Como lo vamos a desplegar en un ESXI, nuestra interfaz de WAN, irá conectada a la red VM Network, que es la red que hace de puente con la interfaz física real. En el caso de vmware o virtualbox podrán crearse esta interfaz como adaptador puente o red NAT. Lo recomendable será emplear adaptador puente, para que se vea como una máquina más de nuestra red local, en caso de usar un adaptador NAT, solo será visible por la máquina anfitriona y puede que tengamos que crear una tercera interfaz "solo anfitrión" para poder acceder desde el equipo anfitrión.

En cuanto a la interfaz LAN, en VMWare Workstation o Virtual Box usaremos una red interna, la cual aplicaremos al resto de máquinas igualmente. En ESXI, deberemos crear antes esta red interna para poder aplicarla, para ello:

Redes ->conmutadores virtuales->agregar conmutador virtual estándar

![img1](/images/pfsense/img1.png){: width="1086" height="542"}

A continuación:

Pestaña grupo de puertos -> Agregar grupo de puertos, le ponemos el nombre deseado y seleccionamos el switch creado anteriormente:

![img2](/images/pfsense/img2.png){: width="1086" height="542"}

Una vez creada la interfaz de red, ya podemos crear la máquina virtual y asignarle la iso que nos descarguemos de pfSense:

![img3](/images/pfsense/img3.png){: width="1086" height="542"}

Como hemos explicado anteriormente, en la interfaz 1 asignaremos VM Network, quer será nuestra WAN, y en la interfaz 2 la red creada anteriormente y que será la red interna de nuestro laboratorio. Ahora ya podemos arrancar la máquina y proceder con la instalación de pfsense. La instalación la realizaremos con todas las opciones por defecto, y cuando nos de la opción, reiniciaremos y ya arrancará pfsense instalado. Después de la instalación nos realizará una serie de preguntas, que marcaremos como sigue en la imagen:

![img](/images/pfsense/img4.png){: width="1086" height="542"}

Ahora configuraremos las interfaces pulsando la opción 2. Empezaremos por la interfaz LAN, en esta aplicaremos la red interna que queramos. En mi caso voy usar el rango 10.0.89.1/24, pero se podría usar cualquier rango interno que no entre en conflicto con ninguna otra subred que tengamos. Quedaría como sigue en las imágenes:

![img](/images/pfsense/img5.bmp){: width="1086" height="542"}

![img](/images/pfsense/img6.bmp){: width="1086" height="542"}

Luego, repetimos los pasos pero para configurar la WAN. En este caso, podríamos dejarla por DHCP, pero personalmente prefiero que sea una ip estática para que no ande variando, y saber en qué ip tengo acceso desde fuera de la red al laboratorio. Así pues, mi rango local es el 192.168.89.1/24, así que le asignaré una ip de este rango que sepa que no va ser ocupada, por ejemplo la 193.168.89.13. La configuración ser haría como sigue en las imágenes:

![img](/images/pfsense/img7.bmp){: width="1086" height="542"}

Ya estarían listas las interfaces de nuestro FW:

![img](/images/pfsense/img8.bmp){: width="1086" height="542"}

Ahora mismo no tenemos acceso a la interfaz de configuración del mismo, ya que por políticas de seguridad solamente se puede acceder desde la red interna a la consola de administración, es decir, desde la red 10.0.89.1 que hemos creado pero en la que todavía no tenemos ninguna máquina. Podemos esperar a tener creada alguna máquina conectada a esta red, o deshabilitar esta política (sólo porque se trata de un laboratorio y por comodidad) pudiendo así acceder desde la WAN. Para ello, seleccionamos la opción 8 y escribimos el comando:

```bash
pfctl -d
```

![img](/images/pfsense/img9.bmp){: width="1086" height="542"}

Y podemos observar como tenemos acceso ya desde la ip que configuramos en la WAN:

![img](/images/pfsense/img10.bmp){: width="1086" height="542"}

Las credenciales por defecto para acceder son admin:pfsense, las cuales deberíamos cambiar.

![img](/images/pfsense/img11.bmp){: width="1086" height="542"}


Ya tenemos nuestro FW listo, en el próximo post, instalaremos el directorio activo y configuraremos su estructura y auditoría.
