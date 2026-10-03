\# MaxiRest — Instalación de Comandera Térmica por Red



Guía práctica para instalar y configurar una \*\*comandera térmica Ethernet\*\* para trabajar con \*\*MaxiRest\*\*, utilizando una PC principal como servidor de impresión y compartiendo la impresora con los demás puestos de la red.



Esta guía está basada en una instalación real con una \*\*Elitronic SOL801\*\*, un router MikroTik y varios puestos de MaxiRest.



---



\## 📌 Esquema de funcionamiento



```text

&nbsp;                        RED LAN

&nbsp;                   192.168.88.0/24

&nbsp;                           │

&nbsp;                   ┌───────┴───────┐

&nbsp;                   │    MikroTik   │

&nbsp;                   │ 192.168.88.1  │

&nbsp;                   └───────┬───────┘

&nbsp;                           │

&nbsp;                    Ethernet / LAN

&nbsp;                           │

&nbsp;                   ┌───────▼────────┐

&nbsp;                   │ Comandera      │

&nbsp;                   │ SOL801         │

&nbsp;                   │ 192.168.88.210 │

&nbsp;                   │ TCP 9100       │

&nbsp;                   └────────────────┘





&nbsp;                 PC PRINCIPAL

&nbsp;                 192.168.88.65

&nbsp;                      │

&nbsp;            Impresora de Windows

&nbsp;               "cocinasonido"

&nbsp;                      │

&nbsp;            Compartida por Windows

&nbsp;                      │

&nbsp;         \\\\PC-PRINCIPAL\\cocinasonido

&nbsp;                      │

&nbsp;         ┌────────────┴────────────┐

&nbsp;         │                         │

&nbsp;      MOZOS 1                   MOZOS 2

&nbsp;         │                         │

&nbsp;   Puerto Local              Puerto Local

&nbsp;         │                         │

&nbsp;         └────────────┬────────────┘

&nbsp;                      │

&nbsp;                   MaxiRest

&nbsp;                      │

&nbsp;                   COMANDA

```



La cadena completa es:



```text

COMANDERA

&nbsp;   ↓

RED TCP/IP

&nbsp;   ↓

IMPRESORA DE WINDOWS

&nbsp;   ↓

RECURSO COMPARTIDO

&nbsp;   ↓

PUERTO LOCAL EN LOS PUESTOS

&nbsp;   ↓

MAXIREST

&nbsp;   ↓

COMANDA

```



---



\# 1. Requisitos



\## Hardware



\* Comandera térmica con conexión Ethernet.

\* Router o switch con red LAN.

\* Cable de red.

\* PC principal con Windows.

\* Uno o más puestos de MaxiRest.



\## Software



\* Windows.

\* MaxiRest.

\* Controlador compatible con impresión térmica.

\* Acceso administrativo a Windows.

\* Acceso a la configuración de red de la impresora.



---



\# 2. Elegir una dirección IP



La comandera debe tener una IP dentro de la misma red LAN que utiliza la PC principal.



Ejemplo utilizado en esta instalación:



```text

Red:       192.168.88.0/24

Router:    192.168.88.1

PC:        192.168.88.65

Comandera: 192.168.88.210

```



Se recomienda utilizar una IP que no esté siendo utilizada por otro dispositivo.



Por ejemplo:



```text

IP:        192.168.88.210

Máscara:   255.255.255.0

Gateway:   192.168.88.1

Puerto:    9100

```



---



\# 3. Conectar físicamente la comandera



Conectar el puerto Ethernet de la comandera al router o switch de la red LAN.



En este caso:



```text

Comandera

&nbsp;  │

&nbsp;  └── Ethernet

&nbsp;         │

&nbsp;         ▼

&nbsp;     MikroTik

```



No es necesario conectar la comandera directamente a la PC.



---



\# 4. Configurar la red de la comandera



Ingresar a la configuración de red de la impresora y establecer una dirección IP estática.



Ejemplo:



```text

IP Address: 192.168.88.210

Subnet Mask: 255.255.255.0

Gateway: 192.168.88.1

Port: 9100

```



La mayoría de las comandas térmicas de red utilizan impresión RAW mediante TCP/IP en el puerto:



```text

9100

```



---



\# 5. Comprobar conectividad



Antes de configurar MaxiRest hay que comprobar que la PC pueda comunicarse con la impresora.



Desde PowerShell:



```powershell

ping 192.168.88.210

```



El resultado esperado es algo similar a:



```text

Reply from 192.168.88.210

Reply from 192.168.88.210

Reply from 192.168.88.210

Reply from 192.168.88.210

```



Si el ping falla, \*\*no continuar todavía con MaxiRest\*\*.



Primero solucionar la conectividad de red.



---



\# 6. Crear la impresora en Windows — PC principal



En la PC principal:



```text

Configuración

→ Bluetooth y dispositivos

→ Impresoras y escáneres

→ Agregar dispositivo

```



Si Windows no encuentra automáticamente la impresora, seleccionar:



```text

Agregar manualmente

```



Elegir:



```text

Agregar una impresora local o de red con configuración manual

```



---



\# 7. Crear un puerto TCP/IP estándar



Seleccionar:



```text

Crear un puerto nuevo

```



Tipo:



```text

Standard TCP/IP Port

```



Ingresar:



```text

Hostname o IP:

192.168.88.210

```



Cuando Windows consulte el tipo de dispositivo, utilizar una configuración TCP/IP estándar.



Para impresión RAW:



```text

Protocol: RAW

Port: 9100

```



---



\# 8. Seleccionar el controlador



Para comandas térmicas ESC/POS puede utilizarse:



```text

Generic / Text Only

```



si la aplicación y la impresora son compatibles con este método.



En la instalación utilizada para esta guía se empleó:



```text

Generic / Text Only

```



Esto permite que MaxiRest envíe texto directamente a la impresora.



---



\# 9. Nombrar la impresora



Utilizar un nombre descriptivo.



Ejemplo:



```text

cocinasonido

```



Es importante mantener un nombre sencillo porque posteriormente será utilizado como recurso compartido.



---



\# 10. Compartir la impresora



En las propiedades de la impresora:



```text

Propiedades de impresora

→ Compartir

```



Activar:



```text

Compartir esta impresora

```



Utilizar como nombre del recurso:



```text

cocinasonido

```



Desde otra PC debería ser accesible mediante:



```text

\\\\PC-PRINCIPAL\\cocinasonido

```



---



\# 11. Probar la impresión desde la PC principal



Antes de involucrar MaxiRest, realizar una prueba desde Windows.



La prueba debe imprimir correctamente en la comandera.



Si la página de prueba funciona:



```text

Windows

&nbsp;  ↓

TCP/IP

&nbsp;  ↓

192.168.88.210:9100

&nbsp;  ↓

Comandera

```



la parte física y de red está funcionando.



---



\# 12. Configurar MaxiRest en la PC principal



Abrir la configuración de MaxiRest relacionada con:



```text

Comprobantes del sistema

→ Comanda Cocina

```



Seleccionar la impresora creada anteriormente:



```text

COCINASONIDO

```



La idea es que MaxiRest envíe las comandas de cocina a esta impresora.



Realizar una comanda real de prueba.



Debe ocurrir:



```text

MaxiRest

&nbsp;  ↓

COCINASONIDO

&nbsp;  ↓

Windows

&nbsp;  ↓

TCP/IP

&nbsp;  ↓

SOL801

```



---



\# 13. Configurar los puestos de mozos



Este punto es especialmente importante cuando existen varias PCs utilizando MaxiRest.



Desde un puesto de mozos, comprobar primero que la PC pueda acceder al recurso compartido:



```text

\\\\PC-PRINCIPAL\\cocinasonido

```



También se puede comprobar desde:



```text

Red

→ PC-PRINCIPAL

```



La impresora compartida debería aparecer allí.



---



\# 14. Probar la impresora desde Windows en el puesto de mozos



Antes de revisar MaxiRest, probar la impresión desde Windows.



Si la prueba de Windows funciona, significa que:



\* La red funciona.

\* La PC principal es accesible.

\* El recurso compartido funciona.

\* La impresora está compartida correctamente.

\* La comandera funciona.



En ese caso, el problema probablemente esté en la asociación entre \*\*Windows y MaxiRest\*\*.



---



\# 15. ⚠️ Problema importante: el puerto local



Durante la instalación real apareció el siguiente problema:



La PC de mozos podía ver la impresora:



```text

\\\\PC-PRINCIPAL\\cocinasonido

```



y Windows podía imprimir correctamente.



Sin embargo, MaxiRest seguía enviando las comandas de cocina a:



```text

BARRA

```



en lugar de:



```text

COCINASONIDO

```



La solución fue crear un \*\*Puerto local\*\* apuntando al recurso compartido.



---



\# 16. Crear el Puerto Local



En el puesto de mozos:



```text

Panel de control

→ Dispositivos e impresoras

→ Propiedades de la impresora

→ Puertos

```



Seleccionar:



```text

Agregar puerto...

```



Elegir:



```text

Local Port

```



Crear un puerto con el siguiente nombre:



```text

\\\\PC-PRINCIPAL\\cocinasonido

```



Aceptar los cambios.



---



\# 17. ¿Por qué funciona el Puerto Local?



El puerto local funciona como una ruta de Windows hacia la impresora compartida.



```text

MaxiRest

&nbsp;   │

&nbsp;   ▼

Impresora de Windows

&nbsp;   │

&nbsp;   ▼

Puerto Local

\\\\PC-PRINCIPAL\\cocinasonido

&nbsp;   │

&nbsp;   ▼

PC PRINCIPAL

&nbsp;   │

&nbsp;   ▼

Impresora cocinasonido

&nbsp;   │

&nbsp;   ▼

192.168.88.210:9100

&nbsp;   │

&nbsp;   ▼

COMANDERA

```



Esto permite que MaxiRest trabaje con una impresora de Windows aunque la impresora física esté conectada por red a otra PC.



---



\# 18. No eliminar las impresoras existentes



Si el sistema ya tiene impresoras configuradas como:



```text

BARRA

COCINA

CONTROL

```



no es necesario eliminarlas.



En particular, una impresora antigua puede ser útil como respaldo.



La instalación nueva puede convivir con las existentes.



Ejemplo:



```text

COCINA

&nbsp;   ↓

Impresora anterior / USB



COCINASONIDO

&nbsp;   ↓

Nueva comandera Ethernet

```



Esto permite volver temporalmente a la impresora anterior si existe algún problema.



---



\# 19. Repetir la configuración en los demás puestos



Cada puesto de MaxiRest puede necesitar su propia configuración de Windows.



Para cada PC:



```text

1\. Comprobar acceso a PC-PRINCIPAL

2\. Comprobar \\\\PC-PRINCIPAL\\cocinasonido

3\. Probar impresión desde Windows

4\. Revisar la impresora utilizada por MaxiRest

5\. Revisar la pestaña Puertos

6\. Crear Puerto Local si es necesario

7\. Probar una comanda real

```



---



\# 20. Diagnóstico rápido



\## El ping no responde



Problema probable:



```text

Red / IP / cable / VLAN / firewall

```



Comprobar:



```powershell

ping 192.168.88.210

```



---



\## Windows no imprime desde la PC principal



Revisar:



```text

IP de la impresora

Puerto TCP

9100

Driver

Standard TCP/IP Port

```



---



\## Windows imprime pero MaxiRest no



Revisar:



```text

Configuración de impresoras de MaxiRest

```



y comprobar que la comanda de cocina esté asociada a la impresora correcta.



---



\## MaxiRest imprime en BARRA



Revisar en el puesto de mozos:



```text

Propiedades de impresora

→ Puertos

```



Crear:



```text

Local Port

\\\\PC-PRINCIPAL\\cocinasonido

```



Este fue el problema encontrado durante la instalación real.



---



\## Windows ve la impresora pero no imprime



Comprobar primero:



```text

\\\\PC-PRINCIPAL\\cocinasonido

```



y realizar una prueba desde Windows.



Si tampoco imprime desde Windows, el problema todavía no es MaxiRest.



---



\# 21. Ejemplo real utilizado



\## Comandera



```text

Modelo:       Elitronic SOL801

IP:           192.168.88.210

Máscara:      255.255.255.0

Gateway:      192.168.88.1

Puerto:       9100

MAC:          00-2E-8F-AA-7F-2F

Protocolo:    ESC/POS

```



\## PC principal



```text

Hostname:     PC-PRINCIPAL

IP:           192.168.88.65

```



\## Impresora Windows



```text

Nombre:       cocinasonido

Driver:       Generic / Text Only

Puerto:       Standard TCP/IP

IP:           192.168.88.210

RAW:          9100

Compartida:   Sí

```



\## Recurso compartido



```text

\\\\PC-PRINCIPAL\\cocinasonido

```



\## Puerto local utilizado en los puestos



```text

\\\\PC-PRINCIPAL\\cocinasonido

```



---



\# 22. Orden recomendado para solucionar problemas



Cuando algo no funciona, seguir siempre este orden:



```text

1\. ¿La comandera está encendida?

&nbsp;          ↓

2\. ¿Tiene la IP correcta?

&nbsp;          ↓

3\. ¿Responde al ping?

&nbsp;          ↓

4\. ¿Windows imprime desde la PC principal?

&nbsp;          ↓

5\. ¿La impresora está compartida?

&nbsp;          ↓

6\. ¿El puesto de mozos accede al recurso?

&nbsp;          ↓

7\. ¿Windows imprime desde el puesto?

&nbsp;          ↓

8\. ¿MaxiRest tiene asociada la impresora correcta?

&nbsp;          ↓

9\. ¿El puerto de Windows es correcto?

&nbsp;          ↓

10\. ¿La comanda real llega a la cocina?

```



No conviene empezar modificando MaxiRest si todavía no se comprobó que Windows pueda imprimir.



---



\# 23. Resumen de la instalación



```text

&nbsp;                   ┌─────────────────────┐

&nbsp;                   │      MAXIREST       │

&nbsp;                   └──────────┬──────────┘

&nbsp;                              │

&nbsp;                              ▼

&nbsp;                   ┌─────────────────────┐

&nbsp;                   │ IMPRESORA WINDOWS   │

&nbsp;                   │    cocinasonido     │

&nbsp;                   └──────────┬──────────┘

&nbsp;                              │

&nbsp;                              ▼

&nbsp;                   ┌─────────────────────┐

&nbsp;                   │   PUERTO LOCAL      │

&nbsp;                   │ \\\\PC-PRINCIPAL\\...  │

&nbsp;                   └──────────┬──────────┘

&nbsp;                              │

&nbsp;                              ▼

&nbsp;                   ┌─────────────────────┐

&nbsp;                   │    PC PRINCIPAL     │

&nbsp;                   │    192.168.88.65    │

&nbsp;                   └──────────┬──────────┘

&nbsp;                              │

&nbsp;                              ▼

&nbsp;                   ┌─────────────────────┐

&nbsp;                   │   TCP/IP - RAW 9100 │

&nbsp;                   └──────────┬──────────┘

&nbsp;                              │

&nbsp;                              ▼

&nbsp;                   ┌─────────────────────┐

&nbsp;                   │     SOL801          │

&nbsp;                   │  192.168.88.210     │

&nbsp;                   └─────────────────────┘

```



---



\# 💡 Idea clave



Una comandera de red no es simplemente:



```text

IMPRESORA → IP

```



En una instalación con MaxiRest y varios puestos intervienen varias capas:



```text

COMANDERA FÍSICA

&nbsp;      ↓

RED TCP/IP

&nbsp;      ↓

IMPRESORA DE WINDOWS

&nbsp;      ↓

RECURSO COMPARTIDO

&nbsp;      ↓

PUERTO LOCAL

&nbsp;      ↓

MAXIREST

&nbsp;      ↓

COMANDA

```



Cuando una capa funciona y la siguiente no, es importante identificar exactamente dónde se corta el recorrido.



En esta instalación, la red y la impresora funcionaban correctamente. El problema estaba en la asociación del puesto de mozos con la impresora compartida de Windows.



La solución fue agregar:



```text

\\\\PC-PRINCIPAL\\cocinasonido

```



como \*\*Puerto Local\*\*.



---



\## Licencia



Esta documentación puede utilizarse, modificarse y adaptarse libremente para instalaciones propias.



