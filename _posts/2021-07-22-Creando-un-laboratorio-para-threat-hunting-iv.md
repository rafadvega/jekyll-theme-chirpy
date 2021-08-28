---
title: Creando un laboratorio para threat hunting IV. Desplegando Máquinas del dominio
author: Rafa de Vega
date: 2021-07-22 11:33:00 +0800
categories: [Threat Hunting, Laboratorio]
tags: [threat hunting,hunting,laboratorio, windows, redhat]
---

# Desplegando Máquinas del dominio

## Instalación de Windows 10

Antes de ponernos a instalar la máquina, ejecutaremos en el controlador de dominio el siguiente comando:

![img](/assets/img/workstation/img5.bmp){: width="1086" height="542"}

Que hará que cada vez que una máquina se dé de alta en el dominio, se creará en la unidad organizativa que le digamos.

Ahora crearemos otra máquina virtual con un Windows 10, por ejemplo con las siguientes características:

![img](/assets/img/workstation/img1.bmp){: width="1086" height="542"}

Seguimos los pasos para la instalación:

![img](/assets/img/workstation/img2.bmp){: width="1086" height="542"}

![img](/assets/img/workstation/img3.bmp){: width="1086" height="542"}

En el siguiente paso, selecionaremos "Domain join instead":

![img](/assets/img/workstation/img4.bmp){: width="1086" height="542"}

Una vez la maquina está arrancada, selecionaremos la configuración en el botón inicio, y buscaremos "Change workgroup name":

![img](/assets/img/workstation/img6.bmp){: width="1086" height="542"}

Pondremos el hostname que queramos para la máquina, seleccionaremos "Domain" y escribiremos nuestro nombre de dominio:

![img](/assets/img/workstation/img7.bmp){: width="1086" height="542"}

Una vez le demos a "OK", nos pedirá usuario y contraseña. Introduciremos las credenciales de un administrador de dominio, y nos confirmará que la máquina ha sido añadida al dominio:

![img](/assets/img/workstation/img8.bmp){: width="1086" height="542"}

Si cerramos sesión, y nos logeamos de nuevo, veremos que ya podemos usar un usuario cualquiera del dominio:

![img](/assets/img/workstation/img9.bmp){: width="1086" height="542"}

Y si comprobamos su ip, vemos que ya ha obtenido una ip del rango que hemos configurado en el servidor DHCP:

![img](/assets/img/workstation/img11.bmp){: width="1086" height="542"}

En el DC, si vamos a la correspondiente unidad organizativa, podremos ver dado de alta este nuevo equipo:

![img](/assets/img/workstation/img10.bmp){: width="1086" height="542"}

Estos pasos, los podremos repetir las veces que queramos, si queremos integrar más máquinas en nuestro laboratorio.

## Instalación Red Hat Enterprisa Linux

Ya tenemos una máquina windows en nuestro laboratorio, así que ahora desplegaremos también una linux. En este caso lo haremos con una Red Hat 8. El motivo de esta elección, es que posiblemente sea una de las distribuciones más frecuentes en entornos de producción empresariales. Además, con el nuevo programa para desarrolladores, la licencia será gratuita con solo darse de alta con un correo en su web.

Creamos una nueva máquina virtual:

![img](/assets/img/redhat/img1.bmp){: width="1086" height="542"}

Seleccionamos el idioma que deseemos:

![img](/assets/img/redhat/img2.bmp){: width="1086" height="542"}

En el siguiente panel, configuraremos la distribución de teclado, lo conectaremos con red hat para dar de alta la licencia gratuita, selecionaremos el disco de la máquina virtual, activaremos el interfaz de red y configuraremos tanto la password como un nuevo usuario:

![img](/assets/img/redhat/img3.bmp){: width="1086" height="542"}

![img](/assets/img/redhat/img4.bmp){: width="1086" height="542"}

![img](/assets/img/redhat/img9.bmp){: width="1086" height="542"}

![img](/assets/img/redhat/img11.bmp){: width="1086" height="542"}

![img](/assets/img/redhat/img10.bmp){: width="1086" height="542"}

![img](/assets/img/redhat/img12.bmp){: width="1086" height="542"}

Continamos con el proceso de instalación:

![img](/assets/img/redhat/img5.bmp){: width="1086" height="542"}

![img](/assets/img/redhat/img6.bmp){: width="1086" height="542"}

Y ya tenemos lista la máquina Linux:

![img](/assets/img/redhat/img13.bmp){: width="1086" height="542"}

En este caso, no vamos a unir la máquina al dominio, ya que con esto será suficiente para nuestras pruebas, y no necesitaremos que esté en el dominio.