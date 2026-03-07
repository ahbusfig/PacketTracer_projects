# 🧪 Lab 5 — Cisco WLC: Redes Users & Guest con VLANs

> **Objetivo:** Configurar un entorno inalámbrico con Wireless LAN Controller (WLC-3504), dos SSIDs diferenciadas (corporativa y guest), segmentación por VLANs y DHCP centralizado en el router.

---

## 📋 Tabla de Contenidos

1. [Topología](#1-topología)
2. [Plan de direccionamiento](#2-plan-de-direccionamiento)
3. [Paso 1 — Configurar el Router R1](#3-paso-1--configurar-el-router-r1)
4. [Paso 2 — Configurar el Switch SW1](#4-paso-2--configurar-el-switch-sw1)
5. [Paso 3 — Setup inicial del WLC](#5-paso-3--setup-inicial-del-wlc)
6. [Paso 4 — Activar los LAPs](#6-paso-4--activar-los-laps)
7. [Paso 5 — Crear interfaces dinámicas en el WLC](#7-paso-5--crear-interfaces-dinámicas-en-el-wlc)
8. [Paso 6 — Crear las WLANs](#8-paso-6--crear-las-wlans)
9. [Paso 7 — Pruebas de conectividad](#9-paso-7--pruebas-de-conectividad)

---

## 1. Topología
<img width="2518" height="1030" alt="image" src="https://github.com/user-attachments/assets/889469e4-5ad2-46d1-8e64-379d0aa155c9" />

### Dispositivos del laboratorio

| Dispositivo | Modelo | IP / Rol |
|------------|--------|----------|
| R1 | Cisco 1941 | Router-on-a-Stick + DHCP Server |
| SW1 | Cisco 2960 | Switch L2 con trunks |
| WLC | Cisco 3504 | Wireless LAN Controller |
| LAP1, LAP2 | Cisco AP | Lightweight APs (CAPWAP) |
| PC1 | PC | Admin (VLAN 99, 10.4.99.x) |
| Smartphone0/1 | - | Clientes WiFi Guest / Users |

---

## 2. Plan de Direccionamiento

| VLAN | Nombre | Red | Gateway | DHCP Pool | Uso |
|------|--------|-----|---------|-----------|-----|
| 5 | Guest | 10.4.5.0/24 | 10.4.5.1 | 10.4.5.10–254 | Invitados (sin contraseña) |
| 13 | Users | 10.4.13.0/24 | 10.4.13.1 | 10.4.13.10–254 | Corporativa (WPA2-AES) |
| 99 | Management | 10.4.99.0/24 | 10.4.99.1 | 10.4.99.10–254 | WLC, APs, administración |

> ⚠️ **VLAN 99 es la VLAN nativa** en todos los trunks. Por eso el WLC se configura con `Management VLAN ID: 0`.

---

## 3. Paso 1 — Configurar el Router R1

El router usa la técnica **Router-on-a-Stick**: una sola interfaz física (`G0/0`) con subinterfaces para cada VLAN.

### 3.1 Entrar al router y configurar subinterfaces

```bash
enable
configure terminal
hostname R1

# Excluir IPs de infraestructura de los pools DHCP
ip dhcp excluded-address 10.4.5.1 10.4.5.9
ip dhcp excluded-address 10.4.13.1 10.4.13.9
ip dhcp excluded-address 10.4.99.1 10.4.99.9

# Pool DHCP para VLAN Guest
ip dhcp pool Guest
 network 10.4.5.0 255.255.255.0
 default-router 10.4.5.1
 dns-server 1.1.1.1

# Pool DHCP para VLAN Users
ip dhcp pool Users
 network 10.4.13.0 255.255.255.0
 default-router 10.4.13.1
 dns-server 1.1.1.1

# Pool DHCP para VLAN Management
ip dhcp pool Management
 network 10.4.99.0 255.255.255.0
 default-router 10.4.99.1
 dns-server 1.1.1.1

# Subinterfaz VLAN 5 (Guest)
interface GigabitEthernet0/0.5
 encapsulation dot1Q 5
 ip address 10.4.5.1 255.255.255.0

# Subinterfaz VLAN 13 (Users)
interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.4.13.1 255.255.255.0

# Subinterfaz VLAN 99 (Management - NATIVA)
interface GigabitEthernet0/0.99
 encapsulation dot1Q 99 native
 ip address 10.4.99.1 255.255.255.0

# Activar la interfaz física
interface GigabitEthernet0/0
 no shutdown

end
write memory
```

---

## 4. Paso 2 — Configurar el Switch SW1

Todos los puertos que conectan a dispositivos de red (Router, WLC, APs) van en modo **trunk** con VLAN 99 como nativa. El PC de administración va en modo **access** en VLAN 99.

```bash
enable
configure terminal
hostname SW1

# Uplink al Router R1
interface FastEthernet0/1
 switchport trunk native vlan 99
 switchport mode trunk

# Uplink al WLC
interface FastEthernet0/2
 switchport trunk native vlan 99
 switchport mode trunk

# Uplink a LAP1
interface FastEthernet0/3
 switchport trunk native vlan 99
 switchport mode trunk

# Uplink a LAP2
interface FastEthernet0/4
 switchport trunk native vlan 99
 switchport mode trunk

# PC1 - Administración (access VLAN 99)
interface FastEthernet0/5
 switchport access vlan 99
 switchport mode access
 spanning-tree portfast

end
write memory
```

> 💡 **¿Por qué trunk en los APs?** Aunque los APs son Lightweight, el switch necesita transportar las VLANs hasta el WLC. El tráfico viaja encapsulado en el túnel CAPWAP.

---

## 5. Paso 3 — Setup Inicial del WLC

> **Prerequisito:** PC1 debe tener IP en la red 10.4.99.x (la obtiene por DHCP) y acceso al WLC.

### 5.1 Acceder al WLC por primera vez

Conectarse desde **PC1** al WLC por HTTP:

```
http://10.4.99.5
```

> En Packet Tracer: abre PC1 → Desktop → Web Browser → escribe la URL

### 5.2 Pantalla 1 — Login

| Campo | Valor |
|-------|-------|
| Username | `admin` |
| Password | `Cisco123` |

### 5.3 Pantalla 2 — System Info (Configuración básica)

| Campo | Valor |
|-------|-------|
| System Name | `WLC1` |
| Management IP Address | `10.4.99.5` |
| Mask | `255.255.255.0` (/24) |
| Gateway | `10.4.99.1` |
| **Management VLAN ID** | **`0`** ← porque usamos la VLAN nativa (99) |

### 5.4 Pantalla 3 — WLAN inicial

| Campo | Valor |
|-------|-------|
| Network Name (SSID) | `WLC_Lab` |
| Security | WPA2 Personal |
| Passphrase | `Cisco321` |

### 5.5 Pantalla 4 — RF / Advanced

| Campo | Valor |
|-------|-------|
| RF Parameter Optimization | Default |
| Virtual IP Address | `192.0.2.1` |
| Local Mobility Group | `Default` |

> 📝 La IP virtual `192.0.2.1` es una dirección reservada (RFC 5737) que usa el WLC internamente para autenticación web.

Finalizar el wizard y esperar a que el WLC se reinicie.

---

## 6. Paso 4 — Activar los LAPs

Una vez el WLC esté operativo, los LAPs deben recibir energía y conectividad.

1. En Packet Tracer, hacer clic en **LAP1** → pestaña **Physical** → conectar el cable de alimentación (o asegurarse de que esté encendido).
2. El LAP obtiene IP por **DHCP** desde el pool de Management (VLAN 99).
3. El LAP descubre el WLC automáticamente mediante el proceso **CAPWAP** y se registra.

> ✅ Si el proceso es correcto, en el WLC verás los APs en: **Wireless → Access Points → All APs**

---

## 7. Paso 5 — Crear Interfaces Dinámicas en el WLC

Acceder al WLC via HTTPS:

```
https://10.4.99.5
```

Ir a **Controller → Interfaces → New**

### 7.1 Interface Guest (VLAN 5)

| Campo | Valor |
|-------|-------|
| Interface Name | `Guest` |
| VLAN Id | `5` |

Hacer clic en **Apply**, luego en **Edit**:

| Campo | Valor |
|-------|-------|
| Port Number | `1` |
| VLAN Identifier | `5` |
| IP Address | `10.4.5.5` |
| Netmask | `255.255.255.0` |
| Gateway | `10.4.5.1` |
| Primary DHCP Server | `10.4.5.1` |

### 7.2 Interface Users (VLAN 13)

| Campo | Valor |
|-------|-------|
| Interface Name | `User` |
| VLAN Id | `13` |

Hacer clic en **Apply**, luego en **Edit**:

| Campo | Valor |
|-------|-------|
| Port Number | `1` |
| VLAN Identifier | `13` |
| IP Address | `10.4.13.5` |
| Netmask | `255.255.255.0` |
| Gateway | `10.4.13.1` |
| Primary DHCP Server | `10.4.13.1` |

---

## 8. Paso 6 — Crear las WLANs

Ir a **WLANs → Create New → Go**

### 8.1 WLAN Guest (sin contraseña)

**WLANs → New:**

| Campo | Valor |
|-------|-------|
| Type | WLAN |
| Profile Name | `Guest` |
| SSID | `Guest` |

Hacer clic en **Apply**, luego configurar en **Edit 'Guest'**:

**Pestaña General:**

| Campo | Valor |
|-------|-------|
| Status | ✅ Enabled |
| Interface/Interface Group | `Guest` |

**Pestaña Advanced:**

| Campo | Valor |
|-------|-------|
| FlexConnect Local Switching | ✅ Enabled |
| FlexConnect Local Auth | ✅ Enabled |

**Pestaña Security → Layer 2:**

| Campo | Valor |
|-------|-------|
| Layer 2 Security | `None` (abierta) |

---

### 8.2 WLAN Users (con contraseña WPA2)

**WLANs → New:**

| Campo | Valor |
|-------|-------|
| Type | WLAN |
| Profile Name | `Users` |
| SSID | `Users` |

Hacer clic en **Apply**, luego configurar en **Edit 'Users'**:

**Pestaña General:**

| Campo | Valor |
|-------|-------|
| Status | ✅ Enabled |
| Interface/Interface Group | `User` |

**Pestaña Advanced:**

| Campo | Valor |
|-------|-------|
| FlexConnect Local Switching | ✅ Enabled |
| FlexConnect Local Auth | ✅ Enabled |

**Pestaña Security → Layer 2:**

| Campo | Valor |
|-------|-------|
| Layer 2 Security | `WPA+WPA2` |
| WPA Policy | ❌ No seleccionar |
| WPA2 Policy | ✅ Seleccionar |
| WPA2 Encryption | `AES` |
| Auth Key Management | `PSK` → ✅ Enable |
| PSK Format | `ASCII` |
| PSK (contraseña) | `Prueba123` |

---

## 9. Paso 7 — Pruebas de Conectividad

### 9.1 Test red Guest

1. En **Smartphone0** (u otro dispositivo) → Config → Wireless0
2. Buscar SSID: `Guest`
3. Conectar → **no pide contraseña** ✅
4. Verificar que obtiene IP en el rango `10.4.5.x`
5. Hacer ping a `10.4.5.1` (gateway) → debe responder ✅

### 9.2 Test red Users

1. En **Laptop1** → Config → Wireless0
2. Buscar SSID: `Users`
3. Conectar → **pide contraseña**: `Prueba123` ✅
4. Verificar que obtiene IP en el rango `10.4.13.x`
5. Hacer ping a `10.4.13.1` (gateway) → debe responder ✅

### 9.3 Verificar aislamiento entre redes

```bash
# Desde un cliente Guest (10.4.5.x):
ping 10.4.13.1   # Debería llegar (routing habilitado en R1)
ping 10.4.13.x   # Debería llegar al otro cliente si no hay ACL
```

> 🔒 Para aislar completamente la red Guest de la red Users, se deberían configurar **ACLs en R1** bloqueando tráfico entre VLAN 5 y VLAN 13.

---

## 🔍 Resumen del flujo de tráfico

```
Cliente WiFi
    │
    ▼ (SSID: Guest o Users)
[LAP] ── CAPWAP tunnel ──► [WLC 10.4.99.5]
                                  │
                         FlexConnect: tráfico
                         sale localmente en el AP
                                  │
                             [SW1 trunk]
                                  │
                             [R1 subinterfaces]
                                  │
                          DHCP Response + Routing
```

---

## ⚠️ Troubleshooting frecuente

| Problema | Causa probable | Solución |
|----------|---------------|----------|
| LAP no se registra en el WLC | DHCP de management no responde | Verificar pool en R1 y trunk en SW1 |
| Clientes no obtienen IP | DHCP server incorrecto en la interfaz del WLC | Revisar Primary DHCP Server en cada interfaz dinámica |
| No se puede acceder al WLC | PC1 no tiene IP en VLAN 99 | Verificar Fa0/5 en SW1 como access VLAN 99 |
| SSID no aparece | WLAN deshabilitada | Verificar Status = Enabled en el WLC |
| No hay FlexConnect | Opción no activada | Habilitar FlexConnect Local Switching en Advanced |

---

## 📚 Conceptos clave

- **Router-on-a-Stick:** Una sola interfaz física del router gestiona múltiples VLANs usando subinterfaces con `encapsulation dot1Q`.
- **WLC (Wireless LAN Controller):** Dispositivo centralizado que gestiona todos los APs de forma unificada.
- **LAP (Lightweight AP):** AP sin configuración local; depende del WLC para todo. Usa el protocolo **CAPWAP** para comunicarse con él.
- **FlexConnect:** Modo que permite a los APs seguir funcionando y switchear tráfico localmente aunque pierdan conexión con el WLC.
- **VLAN nativa:** VLAN que viaja sin etiquetar en un trunk. Aquí la VLAN 99 es nativa, por eso el `Management VLAN ID` del WLC se pone en `0`.

---

*Lab realizado con Cisco Packet Tracer — Topología: WLC 3504 + Cisco 1941 + Cisco 2960*
