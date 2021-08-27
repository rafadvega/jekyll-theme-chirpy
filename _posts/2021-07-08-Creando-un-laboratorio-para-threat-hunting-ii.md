---
title: Creando un laboratorio para threat hunting II. Desplegando pfSense
author: Rafa de Vega
date: 2021-07-08 11:33:00 +0800
categories: [Threat Hunting, Laboratorio]
tags: [threat hunting,hunting,laboratorio, pfsense, firewall]
---


#  Desplegando pfSense.
El primer paso en la preparación de nuestro laboratorio será desplegar nuestro firewall. La máquina virtual de pfsense, deberá tener dos interfaces de red, una para la LAN, que se conectará a la misma red que el resto de máquinas del laboratio (la red interna) y otra para la WAN, que servirá para conectar el firewall, y por tanto todo el laboratorio, a nuestra red local y a internet. 

Como lo vamos a desplegar en un ESXI, nuestra interfaz de WAN, irá conectada a la red VM Network, que es la red que hace de puente con la interfaz física real. En el caso de vmware o virtualbox podrán crearse esta interfaz como adaptador puente o red NAT. Lo recomendable será emplear adaptador puente, para que se vea como una máquina más de nuestra red local, en caso de usar un adaptador NAT, solo será visible por la máquina anfitriona y puede que tengamos que crear una tercera interfaz "solo anfitrión" para poder acceder desde el equipo anfitrión.

En cuanto a la interfaz LAN, en VMWare Workstation o Virtual Box usaremos una red interna, la cual aplicaremos al resto de máquinas igualmente. En ESXI, deberemos crear antes esta red interna para poder aplicarla, para ello:

Redes ->conmutadores virtuales->agregar conmutador virtual estándar

![img1](/assets/img/pfsense/img1.png){: width="1086" height="542"}

A continuación:

Pestaña grupo de puertos -> Agregar grupo de puertos, le ponemos el nombre deseado y seleccionamos el switch creado anteriormente:

![img2](/assets/img/pfsense/img2.png){: width="1086" height="542"}

Una vez creada la interfaz de red, ya podemos crear la máquina virtual y asignarle la iso que nos descarguemos de pfSense:

![img3](/assets/img/pfsense/img3.png){: width="1086" height="542"}

Como hemos explicado anteriormente, en la interfaz 1 asignaremos VM Network, quer será nuestra WAN, y en la interfaz 2 la red creada anteriormente y que será la red interna de nuestro laboratorio. Ahora ya podemos arrancar la máquina y proceder con la instalación de pfsense. La instalación la realizaremos con todas las opciones por defecto, y cuando nos de la opción, reiniciaremos y ya arrancará pfsense instalado. Después de la instalación nos realizará una serie de preguntas, que marcaremos como sigue en la imagen:

![img](/assets/img/pfsense/img4.png){: width="1086" height="542"}

Ahora configuraremos las interfaces pulsando la opción 2. Empezaremos por la interfaz LAN, en esta aplicaremos la red interna que queramos. En mi caso voy usar el rango 10.0.89.1/24, pero se podría usar cualquier rango interno que no entre en conflicto con ninguna otra subred que tengamos. Quedaría como sigue en las imágenes:

![img](/assets/img/pfsense/img5.bmp){: width="1086" height="542"}

![img](/assets/img/pfsense/img6.bmp){: width="1086" height="542"}

Como no necesitaremos que nuestro FW (es decir, nuestro laboratorio) se encuentre siempre en la misma ip, lo dejaremos en DHCP. Es importante tener en cuenta de que si le asignamos una ip estática, deberemos configurar también el gateway y los DNS de manera manual, ya que el pfsense no los podrá obtener mediante DHCP.

Ya estarían listas las interfaces de nuestro FW:

![img](/assets/img/pfsense/img.bmp){: width="1086" height="542"}

Ahora mismo no tenemos acceso a la interfaz de configuración del mismo, ya que por políticas de seguridad solamente se puede acceder desde la red interna a la consola de administración, es decir, desde la red 10.0.89.1 que hemos creado pero en la que todavía no tenemos ninguna máquina. Podemos esperar a tener creada alguna máquina conectada a esta red, o deshabilitar esta política (sólo porque se trata de un laboratorio y por comodidad) pudiendo así acceder desde la WAN. Para ello, seleccionamos la opción 8 y escribimos el comando:

```bash
pfctl -d
```

![img](/assets/img/pfsense/img9.bmp){: width="1086" height="542"}

Y podemos observar como tenemos acceso ya desde la ip que configuramos en la WAN:

![img](/assets/img/pfsense/img10.bmp){: width="1086" height="542"}

Es importante destacar que esto es provisional, y que tras un reinicio el fw volverá aplicar la política de que no se puede acceder a la consola de administración desde la WAN.

Las credenciales por defecto para acceder son admin:pfsense, las cuales deberíamos cambiar.

![img](/assets/img/pfsense/img11.bmp){: width="1086" height="542"}


Ya tenemos nuestro FW listo, en el próximo post, instalaremos el directorio activo y configuraremos su estructura y auditoría.
