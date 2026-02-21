# Lab de Redes – VLANs, Router-on-a-Stick y Seguridad SSH/Telnet

Este laboratorio fue diseñado para practicar conceptos de **switching, routing y seguridad en redes LAN** utilizando Cisco Packet Tracer.

---

## Objetivos del Lab

- Configurar **VLANs** en switches y segmentar la red.
- Establecer **trunks** entre switches y un router (Router-on-a-Stick).
- Configurar **subinterfaces** y **SVIs** para comunicación inter-VLAN.
- Habilitar acceso remoto seguro con **SSH** y **Telnet**.
- Aplicar **hardening básico** y **ACLs** para restringir acceso solo a usuarios autorizados.

---

## Topología de Red

- **SW1** – VLANs 2 (Management) y 3 (Staff)
- **SW2** – VLANs 10 (Finance) y 20 (HR/Guest)
- **R1** – Router con subinterfaces para cada VLAN, actuando como Router-on-a-Stick.
- Conexión trunk entre switches y router.
- PCs conectados a cada VLAN para pruebas de conectividad y acceso remoto.

---

## Contenido del Lab

- `Red_corporativo_Telnet_SSH_ConsoleSecurity_Vlans_Acl_basic.pkt` – Archivo de Cisco Packet Tracer con la topología completa.
- Configuración de switches y router lista para usar.
- Comentarios dentro del Packet Tracer para guiar la práctica.

---

## Instrucciones

1. Abrir el archivo `.pkt` en **Cisco Packet Tracer**.
2. Encender todos los dispositivos.
3. Revisar las **configuraciones de VLANs, SVIs y trunks**.
4. Probar conectividad entre PCs de distintas VLANs (ping).
5. Probar acceso remoto vía SSH y Telnet desde la red de administración.
6. Revisar y experimentar con ACLs para controlar el acceso.

---

## Requisitos

- [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (versión 8.x o superior recomendada)
- Conocimientos básicos de redes LAN, VLANs y ACLs.

---

## Notas

- Asegúrate de que las interfaces VLAN estén activas (`no shutdown`) en los switches.
- El router debe tener **encapsulation dot1Q** configurado en todas las subinterfaces.
- Solo la red de administración tiene permitido acceso SSH/Telnet a los switches según ACLs implementadas.
