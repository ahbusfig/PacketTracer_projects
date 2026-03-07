# LAB 5 — Red Empresarial | Packet Tracer

## 📋 Descripción

Laboratorio completo de red empresarial con dos sedes conectadas por enlace WAN serial. Cubre los conceptos fundamentales del **CCNA 200-301**:

- VLANs y Trunking
- Inter-VLAN Routing (Router-on-a-Stick)
- OSPF
- DHCP centralizado con `helper-address`
- NAT/PAT

---

## 🗺️ Topología

<img width="2585" height="979" alt="Captura de pantalla 2026-03-07 122550" src="https://github.com/user-attachments/assets/728c2c1f-a5b8-44f2-ba9c-62ad501396ce" />


---

## 🖥️ Dispositivos necesarios

| Dispositivo | Modelo | Cantidad |
|---|---|---|
| Router HQ | Cisco 2911 | 1 |
| Router Branch | Cisco 2911 | 1 |
| Router ISP | Cisco 2911 | 1 |
| Switch Central | Cisco 2960-24TT | 1 |
| Switch Remota | Cisco 2960-24TT | 1 |
| PC | PC-PT | 6 |
| Servidor | Server-PT | 1 |

---

## 📊 Tabla de direccionamiento

| Dispositivo | Interfaz | IP | Máscara |
|---|---|---|---|
| ISP | Se0/3/0 | 200.0.0.2 | /30 |
| ISP | Gi0/1 | 8.8.8.1 | /24 |
| R1-HQ | Se0/3/1 | 200.0.0.1 | /30 |
| R1-HQ | Se0/3/0 | 10.0.0.1 | /30 |
| R1-HQ | Gi0/1.10 | 192.168.10.1 | /24 |
| R1-HQ | Gi0/1.20 | 192.168.20.1 | /24 |
| R1-HQ | Gi0/1.30 | 192.168.30.1 | /24 |
| R1-HQ | Gi0/1.99 | 192.168.99.1 | /24 |
| R2-BR | Se0/3/0 | 10.0.0.2 | /30 |
| R2-BR | Gi0/1.10 | 172.16.10.1 | /24 |
| R2-BR | Gi0/1.20 | 172.16.20.1 | /24 |
| R2-BR | Gi0/1.30 | 172.16.30.1 | /24 |
| R2-BR | Gi0/1.99 | 172.16.99.1 | /24 |
| SW1-Central | VLAN99 | 192.168.99.100 | /24 |
| SW2-Remota | VLAN99 | 172.16.99.100 | /24 |
| WebServer | Fa0 | 8.8.8.8 | /24 |
| PCs HQ | Fa0 | DHCP | /24 |
| PCs BR | Fa0 | DHCP | /24 |

---

## 🗂️ Tabla de VLANs

| VLAN | Nombre | Puertos | Red HQ | Red BR |
|---|---|---|---|---|
| 10 | VENTAS | Fa0/1–Fa0/9 | 192.168.10.0/24 | 172.16.10.0/24 |
| 20 | RRHH | Fa0/10–Fa0/18 | 192.168.20.0/24 | 172.16.20.0/24 |
| 30 | TI | Fa0/19–Fa0/23 | 192.168.30.0/24 | 172.16.30.0/24 |
| 99 | MNGM nativa | Gi0/1 trunk | 192.168.99.0/24 | 172.16.99.0/24 |

---

## 🔗 Tabla de conexiones

| Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|
| WebServer | Fa0 | ISP | Gi0/1 | Directo |
| ISP | Se0/3/0 | R1-HQ | Se0/3/1 | Serial |
| R1-HQ | Se0/3/0 | R2-BR | Se0/3/0 | Serial R1=DCE |
| R1-HQ | Gi0/1 | SW1-Central | Gi0/1 | Cruzado |
| R2-BR | Gi0/1 | SW2-Remota | Gi0/1 | Cruzado |
| SW1-Central | Fa0/1 | PC-HQ-Ventas1 | Fa0 | Directo |
| SW1-Central | Fa0/10 | PC-HQ-RRHH1 | Fa0 | Directo |
| SW1-Central | Fa0/20 | PC-HQ-IT1 | Fa0 | Directo |
| SW2-Remota | Fa0/1 | PC-BR-Ventas1 | Fa0 | Directo |
| SW2-Remota | Fa0/10 | PC-BR-RRHH1 | Fa0 | Directo |
| SW2-Remota | Fa0/20 | PC-BR-IT1 | Fa0 | Directo |

---

## 📚 Conceptos practicados

| # | Concepto | Descripción |
|---|---|---|
| 1 | Configuración básica | Hostname, contraseñas, banner |
| 2 | Switching | VLANs, Trunk, Router-on-a-Stick |
| 3 | WAN Serial | DCE/DTE, clock rate |
| 4 | ISP | Interfaces, rutas estáticas |
| 5 | DHCP | Servidor centralizado + helper-address |
| 6 | OSPF | Routing dinámico entre sedes |
| 7 | NAT/PAT | Salida a Internet |

---

## ⚙️ Configuraciones

---

### 🔧 1 — Configuración Básica

<details>
<summary>R1-HQ</summary>

```
enable
configure terminal
hostname R1-HQ
no ip domain-lookup
enable secret Cisco123
line console 0
 password Cisco123
 login
 logging synchronous
line vty 0 4
 password Cisco123
 login
service password-encryption
banner motd $=== R1-HQ | SOLO ACCESO AUTORIZADO ===$
copy running-config startup-config
```
</details>

<details>
<summary>R2-BR</summary>

```
enable
configure terminal
hostname R2-BR
no ip domain-lookup
enable secret Cisco123
line console 0
 password Cisco123
 login
 logging synchronous
line vty 0 4
 password Cisco123
 login
service password-encryption
banner motd $=== R2-BR | SOLO ACCESO AUTORIZADO ===$
copy running-config startup-config
```
</details>

<details>
<summary>ISP</summary>

```
enable
configure terminal
hostname ISP
no ip domain-lookup
enable secret Cisco123
banner motd $=== ISP | SOLO ACCESO AUTORIZADO ===$
copy running-config startup-config
```
</details>

<details>
<summary>SW1-Central</summary>

```
enable
configure terminal
hostname SW1-Central
no ip domain-lookup
enable secret Cisco123
line console 0
 password Cisco123
 login
 logging synchronous
line vty 0 4
 password Cisco123
 login
service password-encryption
banner motd $=== SW1-Central | SOLO ACCESO AUTORIZADO ===$
copy running-config startup-config
```
</details>

<details>
<summary>SW2-Remota</summary>

```
enable
configure terminal
hostname SW2-Remota
no ip domain-lookup
enable secret Cisco123
line console 0
 password Cisco123
 login
 logging synchronous
line vty 0 4
 password Cisco123
 login
service password-encryption
banner motd $=== SW2-Remota | SOLO ACCESO AUTORIZADO ===$
copy running-config startup-config
```
</details>

---

### 🔧 2 — Switching: VLANs + Trunk + Router-on-a-Stick

#### PASO A — VLANs y puertos de acceso

<details>
<summary>SW1-Central</summary>

```
configure terminal

vlan 10
 name VENTAS
vlan 20
 name RRHH
vlan 30
 name TI
vlan 99
 name MNGM

interface range fa0/1 - 9
 description VLAN10-VENTAS
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown

interface range fa0/10 - 18
 description VLAN20-RRHH
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown

interface range fa0/19 - 23
 description VLAN30-TI
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown

copy running-config startup-config
```
</details>

<details>
<summary>SW2-Remota</summary>

```
configure terminal

vlan 10
 name VENTAS
vlan 20
 name RRHH
vlan 30
 name TI
vlan 99
 name MNGM

interface range fa0/1 - 9
 description VLAN10-VENTAS
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown

interface range fa0/10 - 18
 description VLAN20-RRHH
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown

interface range fa0/19 - 23
 description VLAN30-TI
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown

copy running-config startup-config
```
</details>

```
! Verificación
show vlan brief
```

#### PASO B — Trunk

<details>
<summary>SW1-Central</summary>

```
configure terminal

interface gi0/1
 description Trunk-hacia-R1-HQ
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 no shutdown

interface vlan 99
 description MNGM-SW1-Central
 ip address 192.168.99.100 255.255.255.0
 no shutdown

ip default-gateway 192.168.99.1
copy running-config startup-config
```
</details>

<details>
<summary>SW2-Remota</summary>

```
configure terminal

interface gi0/1
 description Trunk-hacia-R2-BR
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 no shutdown

interface vlan 99
 description MNGM-SW2-Remota
 ip address 172.16.99.100 255.255.255.0
 no shutdown

ip default-gateway 172.16.99.1
copy running-config startup-config
```
</details>

```
! Verificación
show interfaces trunk
```

#### PASO C — Router-on-a-Stick

<details>
<summary>R1-HQ</summary>

```
configure terminal

interface gi0/1
 description Trunk-hacia-SW1-Central
 no shutdown

interface gi0/1.10
 description GW-VLAN10-VENTAS
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface gi0/1.20
 description GW-VLAN20-RRHH
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface gi0/1.30
 description GW-VLAN30-TI
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0

interface gi0/1.99
 description GW-VLAN99-MNGM
 encapsulation dot1Q 99 native
 ip address 192.168.99.1 255.255.255.0

copy running-config startup-config
```
</details>

<details>
<summary>R2-BR</summary>

```
configure terminal

interface gi0/1
 description Trunk-hacia-SW2-Remota
 no shutdown

interface gi0/1.10
 description GW-VLAN10-VENTAS-Remota
 encapsulation dot1Q 10
 ip address 172.16.10.1 255.255.255.0

interface gi0/1.20
 description GW-VLAN20-RRHH-Remota
 encapsulation dot1Q 20
 ip address 172.16.20.1 255.255.255.0

interface gi0/1.30
 description GW-VLAN30-TI-Remota
 encapsulation dot1Q 30
 ip address 172.16.30.1 255.255.255.0

interface gi0/1.99
 description GW-VLAN99-MNGM-Remota
 encapsulation dot1Q 99 native
 ip address 172.16.99.1 255.255.255.0

copy running-config startup-config
```
</details>

```
! Verificación
show ip interface brief
```

---

### 🔧 3 — WAN Serial

<details>
<summary>R1-HQ — DCE</summary>

```
configure terminal

interface se0/3/0
 description WAN-hacia-R2-BR
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown

interface se0/3/1
 description WAN-hacia-ISP
 ip address 200.0.0.1 255.255.255.252
 no shutdown

copy running-config startup-config
```
</details>

<details>
<summary>R2-BR — DTE</summary>

```
configure terminal

interface se0/3/0
 description WAN-hacia-R1-HQ
 ip address 10.0.0.2 255.255.255.252
 no shutdown

copy running-config startup-config
```
</details>

```
! Verificación
show ip interface brief
ping 10.0.0.2     ! R1-HQ → R2-BR
ping 200.0.0.2    ! R1-HQ → ISP
```

---

### 🔧 4 — ISP y WebServer

<details>
<summary>ISP</summary>

```
configure terminal

interface se0/3/0
 description WAN-hacia-R1-HQ
 ip address 200.0.0.2 255.255.255.252
 no shutdown

interface gi0/1
 description Hacia-WebServer
 ip address 8.8.8.1 255.255.255.0
 no shutdown

ip route 192.168.10.0 255.255.255.0 200.0.0.1
ip route 192.168.20.0 255.255.255.0 200.0.0.1
ip route 192.168.30.0 255.255.255.0 200.0.0.1
ip route 192.168.99.0 255.255.255.0 200.0.0.1
ip route 172.16.10.0 255.255.255.0 200.0.0.1
ip route 172.16.20.0 255.255.255.0 200.0.0.1
ip route 172.16.30.0 255.255.255.0 200.0.0.1
ip route 172.16.99.0 255.255.255.0 200.0.0.1

copy running-config startup-config
```
</details>

> WebServer → Config → FastEthernet0:
> `IP: 8.8.8.8` | `Mask: 255.255.255.0` | `GW: 8.8.8.1`

```
! Verificación
ping 200.0.0.1    ! ISP → R1-HQ
ping 8.8.8.8      ! ISP → WebServer
```

---

### 🔧 5 — DHCP + Helper-Address

<details>
<summary>R1-HQ — Pools DHCP centralizados</summary>

```
configure terminal

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp excluded-address 192.168.99.1 192.168.99.10
ip dhcp excluded-address 172.16.10.1 172.16.10.10
ip dhcp excluded-address 172.16.20.1 172.16.20.10
ip dhcp excluded-address 172.16.30.1 172.16.30.10
ip dhcp excluded-address 172.16.99.1 172.16.99.10

ip dhcp pool VLAN10-VENTAS-HQ
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 lease 1

ip dhcp pool VLAN20-RRHH-HQ
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
 lease 1

ip dhcp pool VLAN30-TI-HQ
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
 lease 1

ip dhcp pool VLAN10-VENTAS-BR
 network 172.16.10.0 255.255.255.0
 default-router 172.16.10.1
 dns-server 8.8.8.8
 lease 1

ip dhcp pool VLAN20-RRHH-BR
 network 172.16.20.0 255.255.255.0
 default-router 172.16.20.1
 dns-server 8.8.8.8
 lease 1

ip dhcp pool VLAN30-TI-BR
 network 172.16.30.0 255.255.255.0
 default-router 172.16.30.1
 dns-server 8.8.8.8
 lease 1

copy running-config startup-config
```
</details>

<details>
<summary>R2-BR — Helper-address</summary>

```
configure terminal

interface gi0/1.10
 ip helper-address 10.0.0.1

interface gi0/1.20
 ip helper-address 10.0.0.1

interface gi0/1.30
 ip helper-address 10.0.0.1

copy running-config startup-config
```
</details>

```
! Verificación
show ip dhcp binding
show ip dhcp pool
show ip interface gi0/1.10   ! R2-BR → Helper address is 10.0.0.1
```

---

### 🔧 6 — OSPF

<details>
<summary>R1-HQ</summary>

```
configure terminal

router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.99.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 passive-interface gi0/1.10
 passive-interface gi0/1.20
 passive-interface gi0/1.30
 passive-interface gi0/1.99
 passive-interface se0/3/1
 default-information originate

copy running-config startup-config
```
</details>

<details>
<summary>R2-BR</summary>

```
configure terminal

router ospf 1
 router-id 2.2.2.2
 network 172.16.10.0 0.0.0.255 area 0
 network 172.16.20.0 0.0.0.255 area 0
 network 172.16.30.0 0.0.0.255 area 0
 network 172.16.99.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 passive-interface gi0/1.10
 passive-interface gi0/1.20
 passive-interface gi0/1.30
 passive-interface gi0/1.99

copy running-config startup-config
```
</details>

```
! Verificación
show ip ospf neighbor          ! Estado FULL entre R1-HQ y R2-BR
show ip route ospf             ! Rutas O en ambos routers
```

---

### 🔧 7 — NAT/PAT

<details>
<summary>R1-HQ</summary>

```
configure terminal

interface gi0/1.10
 ip nat inside
interface gi0/1.20
 ip nat inside
interface gi0/1.30
 ip nat inside
interface gi0/1.99
 ip nat inside
interface se0/3/0
 ip nat inside
interface se0/3/1
 ip nat outside

access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 permit 192.168.20.0 0.0.0.255
access-list 10 permit 192.168.30.0 0.0.0.255
access-list 10 permit 192.168.99.0 0.0.0.255
access-list 10 permit 172.16.10.0 0.0.0.255
access-list 10 permit 172.16.20.0 0.0.0.255
access-list 10 permit 172.16.30.0 0.0.0.255
access-list 10 permit 172.16.99.0 0.0.0.255

ip nat inside source list 10 interface se0/3/1 overload

ip route 0.0.0.0 0.0.0.0 200.0.0.2

copy running-config startup-config
```
</details>

```
! Verificación
show ip nat translations
show ip nat statistics
ping 8.8.8.8    ! desde cualquier PC
```

---

## ✅ Verificación final

| Prueba | Origen | Destino | Resultado |
|---|---|---|---|
| DHCP HQ | PC-HQ-Ventas1 | — | ✅ 192.168.10.x |
| DHCP HQ | PC-HQ-RRHH1 | — | ✅ 192.168.20.x |
| DHCP HQ | PC-HQ-IT1 | — | ✅ 192.168.30.x |
| DHCP BR | PC-BR-Ventas1 | — | ✅ 172.16.10.x |
| DHCP BR | PC-BR-RRHH1 | — | ✅ 172.16.20.x |
| DHCP BR | PC-BR-IT1 | — | ✅ 172.16.30.x |
| Inter-VLAN HQ | PC-HQ-Ventas1 | PC-HQ-RRHH1 | ✅ Ping OK |
| Inter-VLAN HQ | PC-HQ-Ventas1 | PC-HQ-IT1 | ✅ Ping OK |
| Inter-VLAN BR | PC-BR-Ventas1 | PC-BR-RRHH1 | ✅ Ping OK |
| Inter-VLAN BR | PC-BR-Ventas1 | PC-BR-IT1 | ✅ Ping OK |
| Entre sedes | PC-HQ-Ventas1 | PC-BR-Ventas1 | ✅ Ping OK OSPF |
| Entre sedes | PC-HQ-IT1 | PC-BR-IT1 | ✅ Ping OK OSPF |
| Internet HQ | PC-HQ-Ventas1 | 8.8.8.8 | ✅ Ping OK NAT |
| Internet BR | PC-BR-Ventas1 | 8.8.8.8 | ✅ Ping OK NAT |
| Gestión SW1 | PC-HQ-IT1 | 192.168.99.100 | ✅ Ping OK |
| Gestión SW2 | PC-BR-IT1 | 172.16.99.100 | ✅ Ping OK |

---

## 📝 Notas importantes

> **DHCP centralizado:** Todos los pools están en R1-HQ. R2-BR usa `ip helper-address 10.0.0.1` en sus subinterfaces para reenviar las solicitudes DHCP de la Branch al servidor centralizado.

> **NAT/PAT:** Solo se configura en R1-HQ. Las redes de la Branch también salen a Internet a través de R1-HQ via OSPF.

> **Rutas ISP:** Las rutas estáticas en el ISP son únicamente para facilitar la verificación en el laboratorio. En un entorno real no existirían ya que el NAT oculta las IPs privadas.

> **VLAN 99 nativa:** Se usa como VLAN de gestión de los switches y como VLAN nativa en los trunks para evitar el uso de VLAN 1 por seguridad.

> **DCE/DTE:** R1-HQ es DCE en ambos enlaces seriales → lleva `clock rate 64000`. R2-BR es DTE → sin clock rate.


---

## 🛠️ Herramientas
- Cisco Packet Tracer 8.x o superior

---
