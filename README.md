# MaxiRest — Instalación de Comandera Térmica por Red

Guía práctica para instalar y configurar una **comandera térmica Ethernet** para trabajar con **MaxiRest**, utilizando una PC principal como servidor de impresión y compartiendo la impresora con los demás puestos de la red.

> Basada en una instalación real con una **Elitronic SOL801**, un router MikroTik, Windows y varios puestos de MaxiRest.

---

## ⚠️ Caso importante: MaxiRest imprime en la impresora equivocada

Si la comandera:

- responde al ping,
- imprime correctamente desde la PC principal,
- está compartida por Windows,
- y también puede imprimirse desde un puesto de mozos,

pero **MaxiRest sigue enviando la comanda a otra impresora**, por ejemplo **BARRA**, el problema puede estar en el **puerto de Windows utilizado por MaxiRest**.

En la instalación documentada aquí, la solución fue crear un **Local Port** apuntando al recurso compartido:

```text
\\PC-PRINCIPAL\cocinasonido
```

Este detalle es el principal motivo de esta guía.

---

## 📋 Índice

- [Arquitectura](#-arquitectura)
- [Requisitos](#-requisitos)
- [1. Configurar la red de la comandera](#1-configurar-la-red-de-la-comandera)
- [2. Comprobar conectividad](#2-comprobar-conectividad)
- [3. Crear la impresora en Windows](#3-crear-la-impresora-en-windows)
- [4. Compartir la impresora](#4-compartir-la-impresora)
- [5. Probar Windows antes de MaxiRest](#5-probar-windows-antes-de-maxirest)
- [6. Configurar MaxiRest](#6-configurar-maxirest)
- [7. Configurar los puestos de mozos](#7-configurar-los-puestos-de-mozos)
- [8. Solución: Local Port](#8-solución-local-port)
- [9. Diagnóstico](#9-diagnóstico)
- [10. Ejemplo real](#10-ejemplo-real)
- [Resumen](#-resumen)

---

## 🖧 Arquitectura

En una instalación con varios puestos existen varias capas:

```text
                    RED LAN
                       │
                 ┌─────▼─────┐
                 │  Router   │
                 └─────┬─────┘
                       │
              ┌────────▼────────┐
              │ Comandera       │
              │ Ethernet        │
              │ IP: 192.168.88.210
              │ TCP: 9100       │
              └─────────────────┘

PC PRINCIPAL
192.168.88.65
      │
      ▼
Impresora de Windows
"cocinasonido"
      │
      ▼
Recurso compartido
\\PC-PRINCIPAL\cocinasonido
      │
      ├───────────────┐
      ▼               ▼
   MOZOS 1          MOZOS 2
      │               │
  Local Port      Local Port
      │               │
      └───────┬───────┘
              ▼
           MaxiRest
              │
              ▼
           COMANDA
```

La ruta completa es:

```text
MaxiRest
   ↓
Impresora de Windows
   ↓
Puerto / recurso compartido
   ↓
PC principal
   ↓
TCP/IP RAW 9100
   ↓
Comandera
```

### Tres conceptos que no hay que confundir

**Comandera física**

La impresora Ethernet, por ejemplo la SOL801.

**Impresora de Windows**

La impresora instalada en Windows, por ejemplo `cocinasonido`.

**Recurso compartido**

La ruta que permite acceder a esa impresora desde otra PC:

```text
\\PC-PRINCIPAL\cocinasonido
```

---

## 📦 Requisitos

### Hardware

- Comandera térmica con Ethernet.
- Router o switch.
- Cable de red.
- PC principal con Windows.
- Uno o más puestos de MaxiRest.

### Software

- Windows.
- MaxiRest.
- Driver compatible con la impresora.
- Acceso administrativo a Windows.
- Acceso a la configuración de red de la comandera.

---

# 1. Configurar la red de la comandera

La comandera debe ser **accesible desde la PC que realizará la impresión**.

En una instalación LAN sencilla, lo habitual es colocarla en la misma subred.

Ejemplo:

```text
Red:        192.168.88.0/24
Router:     192.168.88.1
PC:         192.168.88.65
Comandera:  192.168.88.210
Puerto:     9100
```

Se recomienda utilizar una **IP estática** que no esté siendo utilizada por otro dispositivo.

Ejemplo:

```text
IP:          192.168.88.210
Máscara:     255.255.255.0
Gateway:     192.168.88.1
Puerto RAW:  9100
```

> El puerto 9100 es habitual en impresoras térmicas que utilizan impresión RAW, pero debe confirmarse según el modelo.

---

# 2. Comprobar conectividad

Antes de configurar MaxiRest, comprobar primero la red.

### 2.1 Comprobar IP mediante ping

Desde PowerShell:

```powershell
ping 192.168.88.210
```

Si responde, tenemos conectividad ICMP.

### 2.2 Comprobar el puerto de impresión

El ping no garantiza que el puerto TCP de impresión esté disponible.

Comprobar también:

```powershell
Test-NetConnection 192.168.88.210 -Port 9100
```

El resultado esperado es:

```text
TcpTestSucceeded : True
```

La diferencia es importante:

```text
ping               → comprueba ICMP
Test-NetConnection → comprueba TCP/9100
```

Si estas pruebas fallan, **todavía no hay que tocar MaxiRest**. Primero solucionar la conectividad.

---

# 3. Crear la impresora en Windows

En la **PC principal**:

```text
Configuración
→ Bluetooth y dispositivos
→ Impresoras y escáneres
→ Agregar dispositivo
```

Si Windows no encuentra automáticamente la impresora:

```text
Agregar manualmente
→ Agregar una impresora local o de red con configuración manual
```

## 3.1 Crear el puerto TCP/IP

Seleccionar:

```text
Crear un puerto nuevo
→ Standard TCP/IP Port
```

Ingresar la IP de la comandera:

```text
192.168.88.210
```

Para impresión RAW:

```text
Protocol: RAW
Port:     9100
```

## 3.2 Driver

En esta instalación se utilizó:

```text
Generic / Text Only
```

Este driver puede ser apropiado para comandas ESC/POS cuando la aplicación y la impresora son compatibles.

> El driver no tiene que ser necesariamente "Generic / Text Only". Si el fabricante proporciona un driver específico compatible con MaxiRest, también puede utilizarse.

## 3.3 Nombre de la impresora

Utilizar un nombre descriptivo.

Ejemplo:

```text
cocinasonido
```

---

# 4. Compartir la impresora

En las propiedades de la impresora:

```text
Propiedades de impresora
→ Compartir
```

Activar:

```text
Compartir esta impresora
```

Nombre del recurso:

```text
cocinasonido
```

Desde otra PC debería ser accesible mediante:

```text
\\PC-PRINCIPAL\cocinasonido
```

---

# 5. Probar Windows antes de MaxiRest

Este paso es fundamental.

Primero realizar una prueba de impresión desde Windows en la PC principal.

Si funciona, ya sabemos que:

```text
Windows
   ↓
Puerto TCP/IP
   ↓
192.168.88.210:9100
   ↓
Comandera
```

está funcionando.

No conviene diagnosticar MaxiRest mientras Windows todavía no puede imprimir.

---

# 6. Configurar MaxiRest

En la PC principal, abrir la configuración correspondiente a las comandas de cocina.

En la instalación utilizada:

```text
Comprobantes del sistema
→ Comanda Cocina
```

Seleccionar la impresora:

```text
COCINASONIDO
```

Realizar una comanda real de prueba.

El flujo esperado es:

```text
MaxiRest
   ↓
COCINASONIDO
   ↓
Windows
   ↓
TCP/IP
   ↓
SOL801
```

---

# 7. Configurar los puestos de mozos

Cuando hay varias PCs utilizando MaxiRest, cada puesto puede requerir su propia configuración de Windows.

Primero comprobar que el puesto pueda acceder al recurso:

```text
\\PC-PRINCIPAL\cocinasonido
```

También puede comprobarse desde:

```text
Red
→ PC-PRINCIPAL
```

La impresora compartida debería aparecer allí.

## Probar antes de MaxiRest

Desde el puesto de mozos, realizar una prueba de impresión de Windows.

Si funciona:

- La red funciona.
- La PC principal es accesible.
- El recurso compartido funciona.
- Windows puede utilizar la impresora.

En ese punto, si MaxiRest imprime en otra impresora, hay que revisar la asociación entre **Windows y MaxiRest**.

---

# 8. Solución: Local Port

## El problema real

En la instalación documentada ocurrió lo siguiente:

- La PC de mozos veía `\\PC-PRINCIPAL\cocinasonido`.
- Windows podía imprimir correctamente.
- La comandera funcionaba.
- Pero MaxiRest seguía enviando las comandas a **BARRA**.

La solución fue agregar un **Local Port** apuntando al recurso compartido de la PC principal.

## Crear el puerto

En el puesto de mozos:

```text
Panel de control
→ Dispositivos e impresoras
→ Propiedades de la impresora
→ Puertos
→ Agregar puerto...
```

Seleccionar:

```text
Local Port
```

Como nombre del puerto:

```text
\\PC-PRINCIPAL\cocinasonido
```

Aceptar y aplicar los cambios.

### Flujo resultante

```text
MaxiRest
   ↓
Impresora de Windows
   ↓
Local Port
   ↓
\\PC-PRINCIPAL\cocinasonido
   ↓
PC principal
   ↓
Impresora cocinasonido
   ↓
192.168.88.210:9100
   ↓
Comandera
```

> No todas las instalaciones necesitan exactamente esta solución. Es especialmente útil cuando Windows puede imprimir desde el puesto, pero MaxiRest no está utilizando correctamente la impresora compartida.

---

# 9. Diagnóstico

## ❌ No responde al ping

Revisar:

- IP de la comandera.
- Cable de red.
- Puerto del switch/router.
- Subred.
- VLAN.
- Firewall.
- Configuración de red de la impresora.

Probar:

```powershell
ping 192.168.88.210
```

---

## ❌ Ping funciona, pero TCP 9100 falla

Probar:

```powershell
Test-NetConnection 192.168.88.210 -Port 9100
```

Si devuelve:

```text
TcpTestSucceeded : False
```

revisar:

- Puerto configurado en la impresora.
- Modo de impresión RAW.
- Firewall.
- Configuración TCP/IP de la impresora.

---

## ❌ Windows no imprime desde la PC principal

Revisar:

- IP.
- Standard TCP/IP Port.
- RAW.
- Puerto 9100.
- Driver.
- Estado de la impresora.

---

## ❌ Windows imprime, pero MaxiRest no

Revisar la configuración de impresoras de MaxiRest.

Confirmar que **Comanda Cocina** esté asociada a la impresora correcta.

---

## ❌ MaxiRest imprime en BARRA

Este fue el problema encontrado durante la instalación real.

En el puesto afectado revisar:

```text
Propiedades de impresora
→ Puertos
```

Si corresponde, crear:

```text
Local Port
\\PC-PRINCIPAL\cocinasonido
```

Después probar una comanda real.

---

## ❌ Windows ve la impresora, pero no imprime

Probar primero:

```text
\\PC-PRINCIPAL\cocinasonido
```

y realizar una impresión de prueba.

Si Windows tampoco imprime, el problema todavía no está en MaxiRest.

---

# 10. Ejemplo real

Los siguientes valores corresponden a la instalación que dio origen a esta guía.

## Comandera

```text
Modelo:       Elitronic SOL801
IP:           192.168.88.210
Máscara:      255.255.255.0
Gateway:      192.168.88.1
Puerto:       9100
MAC:          00-2E-8F-AA-7F-2F
Protocolo:    ESC/POS
```

## PC principal

```text
Hostname:     PC-PRINCIPAL
IP:           192.168.88.65
```

## Impresora de Windows

```text
Nombre:       cocinasonido
Driver:       Generic / Text Only
Puerto:       Standard TCP/IP
IP:           192.168.88.210
RAW:          9100
Compartida:   Sí
```

## Recurso compartido

```text
\\PC-PRINCIPAL\cocinasonido
```

## Puerto local de los puestos

```text
\\PC-PRINCIPAL\cocinasonido
```

---

## 🧪 Orden recomendado para diagnosticar

Cuando algo no funciona, seguir esta secuencia:

```text
1. ¿La comandera está encendida?
        ↓
2. ¿Tiene la IP correcta?
        ↓
3. ¿Responde al ping?
        ↓
4. ¿TCP/9100 responde?
        ↓
5. ¿Windows imprime desde la PC principal?
        ↓
6. ¿La impresora está compartida?
        ↓
7. ¿El puesto accede al recurso compartido?
        ↓
8. ¿Windows imprime desde el puesto?
        ↓
9. ¿MaxiRest tiene asociada la impresora correcta?
        ↓
10. ¿El puerto de Windows es correcto?
        ↓
11. ¿La comanda real llega a la cocina?
```

La regla general es:

> **No diagnosticar MaxiRest mientras la capa inferior todavía no funciona.**

---

## 💡 Resumen

Una comandera de red no es simplemente:

```text
IMPRESORA → IP
```

En una instalación con MaxiRest y varios puestos intervienen varias capas:

```text
COMANDERA FÍSICA
      ↓
RED TCP/IP
      ↓
IMPRESORA DE WINDOWS
      ↓
RECURSO COMPARTIDO
      ↓
PUERTO LOCAL
      ↓
MAXIREST
      ↓
COMANDA
```

En la instalación real, la red, la impresora y Windows funcionaban correctamente. El problema estaba en la asociación del puesto de mozos con la impresora compartida.

La solución fue crear:

```text
\\PC-PRINCIPAL\cocinasonido
```

como **Local Port** en el puesto afectado.

---

## Licencia

Esta documentación puede utilizarse, modificarse y adaptarse libremente para instalaciones propias.
