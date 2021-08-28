---
title: Creando un laboratorio para threat hunting III. Desplegando Controlador de dominio
author: Rafa de Vega
date: 2021-07-15 11:33:00 +0800
categories: [Threat Hunting, Laboratorio]
tags: [threat hunting,hunting,laboratorio, windows, domain controller, dc]
---


#  Desplegando el Controlador de Dominio.

## Instalación del servidor
El siguiente paso en nuestro despliegue, será desplegar el controlador de dominio. Para ello, emplearemos una versión de evaluación de un Windows Server 2019, que podemos descargar del [Evaluation Center](https://www.microsoft.com/es-es/evalcenter/) de Microsoft. Una máquina virtual con las siguientes características debería ser suficiente:

![img1](/assets/img/domainController/img1.bmp){: width="1086" height="542"}

Como podemos ver, la conectaremos unicamente a la red que hemos creado anteriormente, por lo que si queremos que la máquina tenga conexión a Internet, deberá estar levantada la máquina del pfsense. Seleccionamos la versión Standard, con interfaz gráfica:

![img](/assets/img/domainController/img2.bmp){: width="1086" height="542"}

Seleccionamos "custom installation" y el disco completo que hemos asignado a la máquina virtual:

![img](/assets/img/domainController/img3.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img4.bmp){: width="1086" height="542"}

Configuramos la contraseña del usuario Administrator:

![img](/assets/img/domainController/img5.bmp){: width="1086" height="542"}

Una vez termine la instalación, es recomendable instalar las VMWare tools (o las correspondientes si es virtual box):

![img](/assets/img/domainController/img6.bmp){: width="1086" height="542"}

Si comprobamos nuestra interfaz de red, vemos que ya tiene asignada una ip en el rango que le hemos configurado al pfSense.

![img](/assets/img/domainController/img7.bmp){: width="1086" height="542"}

Sin embargo, en nuestra red, será el "Domain Controller" el que se encargará de asignar ips, además, debe estar en una ip estática, por lo que se la configuraremos siguiendo los pasos que se muestran en las imágenes:

![img](/assets/img/domainController/img8.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img9.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img10.bmp){: width="1086" height="542"}

A continuación, en el Server Manager que se abre cuando arrancamos la máquina, cambiaremos el nombre de host en "Local Server":

![img](/assets/img/domainController/img11.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img12.bmp){: width="1086" height="542"}

El siguiente paso será promocionar el servidor Windows a controlador de dominio, para lo que abriremos "add Roles and features":

![img](/assets/img/domainController/img13.bmp){: width="1086" height="542"}

## Instalación de los roles del servidor

Seguimos las instrucciones paso a paso, y seleccionamos como roles para instalar:

- Active Directory Domain Service
- DHCP Server
- DNS Server

Y finalmente pulsamos en instalar:

![img](/assets/img/domainController/img14.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img15.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img16.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img17.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img18.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img19.bmp){: width="1086" height="542"}

Una vez finalizada la instalación, pulsaremos sobre "Promote this server to a domain controller":


![img](/assets/img/domainController/img20.bmp){: width="1086" height="542"}

Seleccionamos "Add new Forest" y pondremos el nombre de nuestro dominio:

![img](/assets/img/domainController/img21.bmp){: width="1086" height="542"}

Elegimos windos Server 2012 R2 en ambos combobox y la password que elijamos:

![img](/assets/img/domainController/img22.bmp){: width="1086" height="542"}

Y los siguientes paneles los dejaremos por defecto hasta poder darle al botón de instalar:

![img](/assets/img/domainController/img23.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img24.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img25.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img26.bmp){: width="1086" height="542"}

Una vez instalado, el servidor se reiniciará:

![img](/assets/img/domainController/img27.bmp){: width="1086" height="542"}

Una vez arrancado de nuevo el servidor, en la parte de notificaciones, tendremos la opción de completar la configuración del servidor de DHCP:

![img](/assets/img/domainController/img28.bmp){: width="1086" height="542"}

y seguimos los pasos dejando todo por defecto hasta el final:

![img](/assets/img/domainController/img29.bmp){: width="1086" height="542"}

A continuación clickamos en el siguiente sitio:

![img](/assets/img/domainController/img30.bmp){: width="1086" height="542"}

Aquí, botón derecho sobre IPv4 y pulsamos "New Scope"

![img](/assets/img/domainController/img31.bmp){: width="1086" height="542"}

Configuramos el nombre y el rango de ips que queramos que asigne nuestro DC, por ejemplo, siguiendo el rango de red que estamos usando, se puede seleccionar desde la 10.0.89.50 hasta la 10.0.89.200:

![img](/assets/img/domainController/img32.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img33.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img34.bmp){: width="1086" height="542"}

Ponemos la ip de nuestro gateway:

![img](/assets/img/domainController/img35.bmp){: width="1086" height="542"}

Y los DNS que servirá el servidor DHCP. En este caso pondremos la propia ip de nuestro DC, y un servidor secundario, por ejemplo el 1.1.1.1 y finalizamos esta configuración:

![img](/assets/img/domainController/img36.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img37.bmp){: width="1086" height="542"}

## Creación de los usuarios del dominio

El siguiente paso será crear el esquema de nuestra organización, para ello pulsaremos sobre "Active Directory Users and Computers":

![img](/assets/img/domainController/img38.bmp){: width="1086" height="542"}

Haciendo click derecho en nuestro dominio añadiremos una "organizational unit" que nombraremos con el mismo nombre, y partir de esto, seguiremos creando unidades organizacionales hasta tener una arquitectura a nuestro gusto (la forma en que lo organicemos, será a nuestro gusto, en este caso como los usuarios ficticios serán personajes de mario bros, lo dejo como sigue en la siguientes capturas).

![img](/assets/img/domainController/img39.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img40.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img41.bmp){: width="1086" height="542"}

Ahora, dentro de la unidad de usuarios, crearé mis usuarios ficticios siguiendo los pasos que se describen en la imagen:

![img](/assets/img/domainController/img42.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img43.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img44.bmp){: width="1086" height="542"}

Creamos cuantos usuarios queramos, por ejemplo, para este laboratorio lo dejamos así:

![img](/assets/img/domainController/img45.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img46.bmp){: width="1086" height="542"}


Ahora crearemos también un grupo de usuarios en la unidad de grupos:

![img](/assets/img/domainController/img47.bmp){: width="1086" height="542"}

Este grupo lo emplearemos para configurar qué usuario serán administradores locales de los equipos. Así que posteriormente, seleccionaremos los usuarios deseados (en este caso mario y luigi) y les asignaremos ese grupo haciendo click derecho y seleccionando la opción adecuada:

![img](/assets/img/domainController/img48.bmp){: width="1086" height="542"}

Si revisamos el grupo creado, veremos ya los dos usuarios añadidos. También podremos añadir usuarios desde aquí:

![img](/assets/img/domainController/img49.bmp){: width="1086" height="542"}

## Configurando GPOs

Para que los usuarios de este grupo sean administradores locales, usaremos una configuración mediante GPO. Para ello en Tools -> "Group Policy Management":

![img](/assets/img/domainController/img50.bmp){: width="1086" height="542"}

Buscaremos la GPO en el directorio correspondiente, le daremos al bóton derecho del ratón y seleccioanmos "edit".

![img](/assets/img/domainController/img51.bmp){: width="1086" height="542"}

Esto nos abrirá el editor de GPOs, en donde seleccionaremos "Local Users and Groups", y sobre esto pulsaremos botón derecho y New -> Local Group:

![img](/assets/img/domainController/img52.bmp){: width="1086" height="542"}

Ahí seleccionaremos "Administrators" y añadiremos al grupo creado:

![img](/assets/img/domainController/img53.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img54.bmp){: width="1086" height="542"}

Y estos usuarios ya son administradores locales en cualquier equipo que se agregue al dominio. Para

Como usaremos este laboratorio para hacer pruebas con software muchas veces "no legítimo", una buena opción puede ser desactivar el antivirus. Una buena opción para esto será crear una GPO, así que, una vez creada otra política de forma idéntica al anterior caso, pero ahora la editaremos como se ve en las imágenes, deshabilitando el antivirus:

![img](/assets/img/domainController/img55.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img56.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img57.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img58.bmp){: width="1086" height="542"}


## Creando el administrador de dominio

Una cosa más que podemos hacer, es asignar uno de los usuaros al grupo de administradores de dominio, para no tener que estar utilizando nuestro "Administrator". Para esto, seleccionamos cualquier usuario y lo editamos:

![img](/assets/img/domainController/img59.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img60.bmp){: width="1086" height="542"}

![img](/assets/img/domainController/img61.bmp){: width="1086" height="542"}


Y nuestro dominio ya está listo!



