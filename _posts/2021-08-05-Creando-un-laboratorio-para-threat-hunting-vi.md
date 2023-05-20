---
title: Creando un laboratorio para threat hunting VI. Desplegando Splunk
author: Rafa de Vega
date: 2021-08-05 11:33:00 +0800
categories: [Threat Hunting, Laboratorio]
tags: [threat hunting,hunting,laboratorio, splunk, SIEM]
---

# Desplegando Splunk

## Instalación del servidor

Comenzaremos creando la máquina virtual, en este caso usaremos una Debian 11, pero como podemos ver en la documentación, las opciones son múltiples:

![img](/assets/img/splunk/img1.bmp){: width="1086" height="542"}

Como podemos ver, incluimos dos interfaces de red, una conectada a la subred de nuestro laboratorio, y otra en modo bridge. Esta segunda, es para hacer un poco de trampa, y poder conectar a nuestro splunk directamente desde nuestro equipo. Otra opción sería exponer el servicio a través del nuestro pfSense, o conectarnos desde una máquina dentro del dominio.

Una vez instalado el sistema operativo, deberemos configurar una ip estática dentro de la subred del laboratorio, y de paso también lo podemos hacer con la interfaz que conecta con nuestra red local para saber siempre donde estará disponible Splunk. Para ello editaremos el fichero "/etc/network/interfaces" con el siguiente contenido (ojo con los nombres de la insterfaces de red, que puede variar, para verlos podemos ejecutar un "ip addr"):

```yaml
auto <nombreInterfaz1>
iface <nombreInterfaz1> inet static
	addresss <ip rango laboratorio>
	netmask <máscara de nuestro rango>
	gateway <ip del pfsense>

auto <nombreInterfaz2>
iface <nombreInterfaz2> inet static
	addresss <ip rango local>
	netmask <máscara de nuestro rango local>
	gateway <ip del gateway de nuestra red local>
```

![img](/assets/img/splunk/img2.bmp){: width="1086" height="542"}


El siguiente paso es descargar el binario de splunk desde su web (será necesario registrarse):

![img](/assets/img/splunk/img3.bmp){: width="1086" height="542"}

En nuestro caso elegiremos el .deb, pero elegiremos el correspondiente de nuestro SO.

Para descargarlo directamente en nuestra máquina, podemos usar el comando wget que nos facilita splunk:

![img](/assets/img/splunk/img4.bmp){: width="1086" height="542"}

![img](/assets/img/splunk/img5.bmp){: width="1086" height="542"}

Ejecutaremos el siguiente comando para instalarlo:

```bash
dpkg -i nombre_del_paquete_de_splunk.deb
```
![img](/assets/img/splunk/img6.bmp){: width="1086" height="542"}

A continuación lo arrancaremos con el comando:

```bash
splunk start --accept-license --answer-yes
```
![img](/assets/img/splunk/img7.bmp){: width="1086" height="542"}

Y ya tendremos el servicio de splunk listo en el puerto 8000:

![img](/assets/img/splunk/img8.bmp){: width="1086" height="542"}

![img](/assets/img/splunk/img9.bmp){: width="1086" height="542"}

Nos logeamos con la contraseña anteriormente introducida, y lo primero que haremos será configurar la licencia. La licencia de prueba solamente durará 30 días, pero la licencia gratuita es indefinida. Careceremos de configuración de alertas, autenticación, etc, y limitado a una ingesta de 500MB al día de logs. Todo esto es más que suficiente para nuestro laboratorio. Así pues, iremos a Settings->licensing. Aquí pulsaremos "Add license" y seleccionaremos "Free":

![img](/assets/img/splunk/img10.bmp){: width="1086" height="542"}

![img](/assets/img/splunk/img11.bmp){: width="1086" height="542"}

![img](/assets/img/splunk/img12.bmp){: width="1086" height="542"}

![img](/assets/img/splunk/img13.bmp){: width="1086" height="542"}

Por último, con el siguiente comando, habilitaremos la ejecución automática de splunk cada vez que se inicie la máquina:

```bash
splunk enable boot-start
```

![img](/assets/img/splunk/img14.bmp){: width="1086" height="542"}

El siguiente paso será instalar en splunk los "Add-on" que nos parsearan nuestros eventos para una búsqeuda y visualización más sencilla. Seleccionaremos "Manage APPs" y buscaremos "Windows". La primera que aparece es la que instalaremos:

![img](/assets/img/splunk/img15.bmp){: width="1086" height="542"}

Continuaremos con la de sysmon y auditd:

![img](/assets/img/splunk/img16.bmp){: width="1086" height="542"}

A continuación debemos crear el "listener" que escuchará para recibir los eventos de los agentes. Para esto desplegaremos los setttings, y pincharemos en "Forwarding and receiving":

![img](/assets/img/splunk/img22.bmp){: width="1086" height="542"}

Ĺuego "Configure receiving", y elegiremos un puerto:

![img](/assets/img/splunk/img23.bmp){: width="1086" height="542"}

Y por último, debemos crear los index dónde guardaremos nuestros logs. Crearemos dos:

- wineventlog: para los eventos de windows
- os: para los de linux

Para crearlos, iremos a Settings->Indexes->New Index:

![img](/assets/img/splunk/img25.bmp){: width="1086" height="542"}

Repetimos lo mismo para el index os ya tenemos listo nuestro servidor con Splunk, pasemos a nuestro DC para instalar el agente. 


## Instalación agente Windows

Volveremos a la web de splunk para descargarnos el "Splunk Universal Forwarder" para la correspondiente plataforma:

![img](/assets/img/splunk/img17.bmp){: width="1086" height="542"}

Seguimos las instrucciones para la instalación, seleccionando el puerto antes configurado (para el deployment server es indiferente, ya que no lo usaremos):

![img](/assets/img/splunk/img18.bmp){: width="1086" height="542"}

![img](/assets/img/splunk/img19.bmp){: width="1086" height="542"}

A continuación deberemos definir en el equipo el fichero outputs.conf(configuración de a dónde envía los eventos) e inputs.conf (configuración de qué eventos se lleva el agente de la máquina). Ambos se encuentran en el directorio "C:\Program Files\SplunkUniversalForwarder\etc\system\local\" Tras la instalación indicada, el outputs debería haberse generado automáticamente con algo como lo que sigue:

![img](/assets/img/splunk/img20.bmp){: width="1086" height="542"}

En cuanto al inputs.conf, lo crearemos en el mismo directorio, con el siguiente contenido:

```yaml
[default]
host=$decideOnStartup

[WinEventLog://Security]
disabled=0
index = wineventlog
source = XmlWinEventLog:Security
sourcetype = XmlWinEventLog
evt_resolve_ad_obj=1
renderXml = true
suppress_text = true
suppress_sourcename = true
suppress_keywords = true
suppress_task = true
suppress_opcode = true
blacklist = 4703
blacklist1 = EventCode="4688" $XmlRegex="(<Data Name='NewProcessName'>(C:\\Program Files\\SplunkUniversalForwarder\\bin\\*)<\/Data>)"

[WinEventLog://Microsoft-Windows-WinRM/Operational]
renderXml = true
suppress_text = true
suppress_sourcename = true
suppress_keywords = true
suppress_task = true
suppress_opcode = true
disabled = 0
index = wineventlog
sourcetype = XmlWinEventLog
source = XmlWinEventLog:WinRM
evt_resolve_ad_obj=1

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
renderXml = true
suppress_text = true
suppress_sourcename = true
suppress_keywords = true
suppress_task = true
suppress_opcode = true
index = wineventlog
sourcetype = XmlWinEventLog
source = XmlWinEventLog:Powershell
evt_resolve_ad_obj=1


[WinEventLog://Windows PowerShell]
disabled = 0
renderXml = true
suppress_text = true
suppress_sourcename = true
suppress_keywords = true
suppress_task = true
suppress_opcode = true
index = wineventlog
sourcetype = XmlWinEventLog
source = XmlWinEventLog:Powershell
evt_resolve_ad_obj=1

[WinEventLog://Microsoft-Windows-TaskScheduler/Operational]
disabled = 0
renderXml = true
suppress_text = true
suppress_sourcename = true
suppress_keywords = true
suppress_task = true
suppress_opcode = true
index = wineventlog
sourcetype = XmlWinEventLog
source = XmlWinEventLog:TaskScheduler
evt_resolve_ad_obj=1

[WinEventLog://Microsoft-Windows-TerminalServices-LocalSessionManager/Operational]
disabled = 0
renderXml = true
suppress_text = true
suppress_sourcename = true
suppress_keywords = true
suppress_task = true
suppress_opcode = true
index = wineventlog
sourcetype = XmlWinEventLog
source = XmlWinEventLog:TerminalServices
evt_resolve_ad_obj=1

[WinEventLog://Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational]
disabled = 0
index = wineventlog
renderXml = true
suppress_text = true
suppress_sourcename = true
suppress_keywords = true
suppress_task = true
suppress_opcode = true
sourcetype = XmlWinEventLog
source = XmlWinEventLog:TerminalServices
evt_resolve_ad_obj=1

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
renderXml = true
index = wineventlog
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```
Una vez guardado, reiniciamos el splunk forwarder, por ejemplo desde powershell:

![img](/assets/img/splunk/img21.bmp){: width="1086" height="542"}

Y si comprobamos nuestro Splunk, ya deberíamos tener eventos:

![img](/assets/img/splunk/img24.bmp){: width="1086" height="542"}

Repetiremos estos pasos para todas las máquinas windows que hayamos desplegado.

## Instalación agente Linux

Para nuestra máquina RedHat, los pasos son muy parecidos. Descargamos el agente:

![img](/assets/img/splunk/img26.bmp){: width="1086" height="542"}

Instalamos el agente:

![img](/assets/img/splunk/img27.bmp){: width="1086" height="542"}

Lo iniciamos con el comando:

```bash
./splunk start --accept-license
```
![img](/assets/img/splunk/img28.bmp){: width="1086" height="542"}

![img](/assets/img/splunk/img19.bmp){: width="1086" height="542"}

En este caso, el outputs.conf será idéntico al de windows, pero lo deberemos configurar a mano:

```yaml
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 10.0.89.9:9997

[tcpout-server://10.0.89.9:9997]
```

Y en el inputs.conf usaremos la siguiente configuración:

```yaml
[default]
host = LINUXSERVER1

[monitor:///var/log/audit/*]
disabled = 0
sourcetype = linux:audit
index = os

[monitor:///var/log/secure]
disabled = 0
sourcetype = linux:auth
index = os
```

Por último comprobamos que los logs están llegando a Splunk y...

NUESTRO LABORATORIO ESTÁ LISTO PARA SU USO!!!!
