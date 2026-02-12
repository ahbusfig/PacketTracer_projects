# 🚀 Configuración de DHCP Relay con Servidor Centralizado

En este laboratorio he implementado un escenario donde un **Servidor DHCP central** asigna direcciones IP a departamentos en redes distintas. Es una solución profesional para evitar tener un servidor en cada subred.

---

## ⚙️ Guía de Configuración Paso a Paso

### 🟢 PASO 1: Configuración de R1 (Router de Acceso)
Configuramos las puertas de enlace y el **DHCP Relay** mediante el comando `ip helper-address` para que los broadcasts de los PCs lleguen al servidor.

```cisco
enable
configure terminal

# Interfaz Ventas
interface X1
 description RED-VENTAS
 ip address 192.168.10.254 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown
exit

# Interfaz IT
interface X2
 description RED-IT
 ip address 192.168.20.254 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown
exit

# Interfaz hacia R2 (Enlace WAN)
interface X3
 description ENLACE-A-CORE
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit

# RUTA ESTÁTICA: Para llegar al Data Center (donde está el DHCP)
# El siguiente salto para alcanzar la red 192.168.100.0/24 es R2 (10.0.0.2)
ip route 192.168.100.0 255.255.255.0 10.0.0.2
```
🔵 PASO 2: 
Configuración de R2 (Router Core/Data Center)Este router conecta con el servidor y debe tener las rutas de regreso para que los paquetes DHCP lleguen a los clientes.Cisco CLIenable
```cisco
enable
configure terminal
# Interfaz hacia R1 (Enlace WAN)
interface X1
 description ENLACE-A-ACCESO
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

# Interfaz hacia el Data Center (Switch-DC)
interface X2
 description LAN-DATA-CENTER
 ip address 192.168.100.1 255.255.255.0
 no shutdown
exit

# RUTAS DE REGRESO: R2 debe saber cómo volver a Ventas e IT vía R1 (10.0.0.1)
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
```
🔹 PASO 3: Configurar DHCP Server
En Packet Tracer, configuramos el servidor con los siguientes parámetros:

1. Configuración IP Estática
IP: 192.168.100.10
Mask: 255.255.255.0
Gateway: 192.168.100.1

2. Pools de DHCP
Pool Name,Network,Gateway,DNS Server,Start IP,Max Users
Ventas,192.168.10.0,192.168.10.254,8.8.8.8,192.168.10.50,50
IT,192.168.20.0,192.168.20.254,8.8.8.8,192.168.20.50,50

--- 

🧠 ¿Qué he aprendido con este laboratorio?

DHCP Relay: El comando ip helper-address es vital. Transforma un mensaje de broadcast (que no pasa routers) en un mensaje de unicast dirigido al servidor.

Routing Bidireccional: El servidor puede recibir la petición, pero si el router del Data Center no sabe volver a la red de origen (Rutas Estáticas), el PC nunca recibirá su configuración IP.

Segmentación: Mantener departamentos en redes distintas mejora la seguridad y el orden sin perder la gestión centralizada de servicios.
