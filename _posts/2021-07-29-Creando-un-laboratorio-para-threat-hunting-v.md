---
title: Creando un laboratorio para threat hunting IV. Configurando la auditoría de los sistemas operativos
author: Rafa de Vega
date: 2021-07-29 11:33:00 +0800
categories: [Threat Hunting, Laboratorio]
tags: [threat hunting,hunting,laboratorio, windows, redhat, auditoría]
---

# Configurando la auditoría de los sistemas operativos

## Auditoría de Windows

La auditoría de Windows la configuraremos mediante una GPO, para que aplique tanto al DC como a las máquinas que añadamos al dominio. Para ello, buscaremos el "Group Policy Management" y crearemos una política en "Group Policy Objects" de forma idéntica a como lo hicimos en posts anteriores. 

![img](/assets/img/auditWindows/img1.bmp){: width="1086" height="542"}

Editamos la política, y en el editor de políticas, abrimos el árbol tal y como se muestra en la imagen y veremos como configurar las políticas generales:

![img](/assets/img/auditWindows/img2.bmp){: width="1086" height="542"}

En cada política, podemos configurar la auditoría de fallos y aciertos:

![img](/assets/img/auditWindows/img3.bmp){: width="1086" height="542"}

Para configurar estas políticas, podemos seguir las [recomendaciones de microsoft](https://docs.microsoft.com/es-es/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations). En este caso las dejaremos como sigue:

![img](/assets/img/auditWindows/img4.bmp){: width="1086" height="542"}

Continuamos modificando las políticas de auditoría avanzadas, dejándolas como sigue en las capturas siguientes:

![img](/assets/img/auditWindows/img5.bmp){: width="1086" height="542"}

![img](/assets/img/auditWindows/img6.bmp){: width="1086" height="542"}

![img](/assets/img/auditWindows/img7.bmp){: width="1086" height="542"}

![img](/assets/img/auditWindows/img8.bmp){: width="1086" height="542"}

![img](/assets/img/auditWindows/img9.bmp){: width="1086" height="542"}

Con esto, tendremos auditadas las creaciones de procesos, pero no veremos el comando usado para lanzarlos. Para esto habitaremos la siguiente política:

![img](/assets/img/auditWindows/img10.bmp){: width="1086" height="542"}

Por último, linkaremos la política al dominio:

![img](/assets/img/auditWindows/img11.bmp){: width="1086" height="542"}

Una vez realizado esto, podemos listar las auditorías de la máquina con el comando:

```bash
auditpol /get /category:*
```

y deberíamos ver lo siguiente:

![img](/assets/img/auditWindows/img12.bmp){: width="1086" height="542"}

En cualquier máquina del dominio, deberíamos tener también estas políticas configuradas.

## Sysmon

Además de esta auditoría de windows, será interesante tener desplegado Sysmon, que aunque se pisa bastante con la configuración que hemos realizado, nos dará mayores posibilidades de detección. Para instalar sysmon, descargaremos su binario de la [web de sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon). También necesitaremos crear una configuración para sysmon. Podemos empezar la configuración del siguiente [enlace](https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml)

Le añadiremos una linea para evitar que nos llegue las ejecuciones del agente de Splunk que instalaremos más tarde:

```xml
<Image condition="is">C:\Program Files\SplunkUniversalForwarder\bin\*</Image>
```
Deberemos hacerlo en la sección indicada tal y como se ve en la imagen:

![img](/assets/img/auditWindows/img13.bmp){: width="1086" height="542"}

Por último, incluimos en el mismo directorio el binario de sysmon y el fichero de configuración, instalaremos sysmon con el siguiente comando:

```bash
.\Sysmon64.exe -i
```

Y aplicaremos la configuración con el siguiente comando:

```bash
.\Sysmon64.exe -c .\sysmonconfig-export.xml
```

![img](/assets/img/auditWindows/img15.bmp){: width="1086" height="542"}

Desde el visor de Eventos de windows, podemos observar que ya tenemos eventos de Sysmon:

![img](/assets/img/auditWindows/img16.bmp){: width="1086" height="542"}

## Auditd

En el caso de Linux, tendremos que usar Auditd para la auditoría. Nuestro RedHat ya debería venir con el servicio instalado, en otras distribuciones quizá haya que instalarlo.

Aunque está instalado, si ejecutamos el siguiente comando:

```bash
auditctl -l
```

Veremos que no tenemos ninguna regla configurada. Para configurar reglas de audit, podemos empezar a usar [esta lista pública de reglas](https://github.com/bfuzzy/auditd-attack/blob/master/auditd-attack.rules) que está mapeada con el framwork Mitre ATT&CK. La descargamos y la copiamos en la ruta /etc/auditd/rules.d/

![img](/assets/img/auditd/img2.bmp){: width="1086" height="542"}

Deberemos borrar el otro fichero que hay en esa ruta, o quitarle el parámetro -D al mismo. Una vez realizado esto, reiniciaremos el servicio:

```bash
sudo service auditd restart
```

Y si volvemos listar las reglas, ahora sí deberíamos ver la lista completa:

![img](/assets/img/auditd/img3.bmp){: width="1086" height="542"}


Con esto, nuestra auditoría debería estar completa, sólo queda llevarla a nuestro Splunk.