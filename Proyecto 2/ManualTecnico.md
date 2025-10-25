# Manual Tecnico

## Proyecto 2 - Redes de Computadoras 1

#### JOHAN MOISES CARDONA ROSALES

----------------

Este proyecto tiene como objetivo central el diseño, configuración y simulación de una infraestructura de red interedificios para el campus central, enfocada en segmentación, seguridad, redundancia y enrutamiento avanzado. 

----

## 1. Descripción General

El proyecto NetUSAC tiene como propósito diseñar, configurar y simular una red interedificios para el campus central de la USAC, aplicando segmentación lógica, seguridad, redundancia y enrutamiento avanzado. Se interconectan los edificios:

* Biblioteca Central
* Edificio T4
* Edificio S11
* Edificio S12
* DIGA

Cada edificio implementa VLANs para áreas funcionales: Estudiantes, Docentes, Administración, Videovigilancia y Biblioteca.

## 2. Arquitectura General

### Segmentación y VLANs

Cada VLAN está numerada con base en los últimos dígitos del carnet:

Y = 0 + 5 = 5

Ejemplo:
- `Estudiantes` → VLAN 15
- `Docentes` → VLAN 25
- `Vigilancia` → VLAN 35
- `Administración` → VLAN 45
- `Biblioteca` → VLAN 55

Cada VLAN tendrá su subred configurada con VLSM o FLSM, optimizando el uso de direcciones IP.

## 3. Topología General

La red conecta los cinco edificios mediante un core/backbone, empleando protocolos de enrutamiento mixtos: RIP, OSPF, EIGRP y rutas estáticas. Los routers principales realizan redistribución de rutas en puntos clave para garantizar conectividad entre todos los segmentos.

## 4. Configuración por Edificio

### 4.1 DIGA

**Red base:** `192.168.05.0/24`

| VLAN | Nombre | Equipos | ID |
|------|--------|---------|-----|
| Estudiantes | VLAN 15 | 5 | 15 |
| Docentes | VLAN 25 | 5 | 25 |
| Vigilancia | VLAN 35 | 20 | 35 |
| Administración | VLAN 45 | 120 | 45 |

**Configuraciones:**
- Redundancia HSRP entre MS1 y MS2
- Router-on-a-Stick configurado en subinterfaces
- VTP
  - dominio: 202201405
  - password: usac2025
  - modo: server (SW5), cliente (resto)
- ACLs (FW1): permitir solo tráfico entre dispositivos de la misma VLAN

### 4.2 Biblioteca Central

**Red base:** `192.158.05.0/24`

| VLAN | Nombre | Equipos | ID |
|------|--------|---------|-----|
| Estudiantes | VLAN 15 | 75 | 15 |
| Vigilancia | VLAN 35 | 30 | 35 |
| Biblioteca | VLAN 55 | 25 | 55 |

**Configuraciones:**
- Redundancia VRRP entre routers R0 y R1
- Router-on-a-Stick con subinterfaces
- VTP
  - dominio: 202201405
  - password: usac2025
  - modo: server (MS0), cliente (resto)
- ACLs (FW2): solo comunicación entre equipos de la misma VLAN

### 4.3 Edificio T4

**Red base:** `172.16.05.0/24`

| VLAN | Nombre | Equipos | ID |
|------|--------|---------|-----|
| Estudiantes | VLAN 15 | 60 | 15 |
| Docentes | VLAN 25 | 10 | 25 |
| Vigilancia | VLAN 35 | 15 | 35 |
| Administración | VLAN 45 | 75 | 45 |
| Biblioteca | VLAN 55 | 12 | 55 |

**Configuraciones:**
- Rutas estáticas entre router R6 y MS5
- VTP
  - dominio: 202201405
  - password: usac2025
  - modo: server (SW8), cliente (resto)
- ACLs (FW3): solo intra-VLAN
- Puertas de enlace virtuales configuradas en MS5

### 4.4 Edificio S11

**Red base:** `172.148.05.0/24`

| VLAN | Nombre | Equipos | ID |
|------|--------|---------|-----|
| Estudiantes | VLAN 15 | 100 | 15 |
| Docentes | VLAN 25 | 15 | 25 |
| Vigilancia | VLAN 35 | 10 | 35 |
| Administración | VLAN 45 | 55 | 45 |

**Configuraciones:**
- Redundancia VRRP entre R10 y R11
- OSPF como protocolo de enrutamiento
- VTP
  - dominio: 202201405
  - password: usac2025
  - modo: server (SW11), cliente (resto)
- ACLs (FW4): tráfico solo dentro de la misma VLAN

### 4.5 Edificio S12

**Red base:** `192.128.05.0/24`

| VLAN | Nombre | Equipos | ID |
|------|--------|---------|-----|
| Estudiantes | VLAN 15 | 125 | 15 |
| Docentes | VLAN 25 | 35 | 25 |
| Vigilancia | VLAN 35 | 20 | 35 |
| Administración | VLAN 45 | 25 | 45 |

**Configuraciones:**
- Redundancia HSRP entre MS7 y MS8
- OSPF
- VTP
  - dominio: 202201405
  - password: usac2025
  - modo: server (MS9), cliente (resto)
- ACLs (FW5): comunicación solo entre miembros de la misma VLAN

## 5. CORE / BACKBONE

**Red base:** `10.0.0.0/24`

| Segmento | Protocolo | Dispositivos principales |
|----------|-----------|--------------------------|
| DIGA ↔ Biblioteca | RIP | R0, R1, R2, R3, R4, FW1, FW2 |
| S11 ↔ S12 | OSPF | R7, R8, R9, R10, R11, FW4, FW5 |
| Enlace general | EIGRP | R4, R5, R6, R7, MS3, MS4, MS6 |
| T4 ↔ Backbone | Rutas estáticas | R6 ↔ MS5 |

**Configuraciones adicionales:**
- Redistribución de rutas entre RIP, EIGRP y OSPF en R4, R6, R7
- EtherChannel entre MS3, MS4 y MS6

## 6. Seguridad

- ACLs extendidas en todos los firewalls
- Solo se permite tráfico intra-VLAN entre edificios
- Comunicación inter-VLAN bloqueada
- Las ACLs se aplican en el tráfico de salida (outbound) en las inter