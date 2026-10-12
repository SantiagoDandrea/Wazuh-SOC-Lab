# Arquitectura del Entorno de Laboratorio

Este documento describe la topología de red, la distribución de componentes y los roles asignados a cada equipo dentro de **Wazuh-SOC-Lab**.

---

## Diagrama de la Red

```text
+-----------------------------------------------------------------------------------------------+
|                                      HOST FÍSICO: WINDOWS                                     |
|  IP Host-Only: 192.168.56.1 | Hostname del Agente: LAPTOP-6MIJ8MV4 (ID: 005)                  |
|                                                                                               |
|  - Consola de Analista SOC: Acceso web al Wazuh Dashboard vía navegador                       |
|  - Origen de simulaciones de ataque hacia las VMs (Hydra, pruebas de red)                     |
|  - Monitoreado con Agente Wazuh (Windows) [Canal Security: Event IDs 4625, 4720, 4728/4732]   |
|                                                                                               |
|  +-----------------------------------------------------------------------------------------+  |
|  |                    ORACLE VIRTUALBOX (Segmento Host-Only: 192.168.56.0/24)              |  |
|  |                                                                                         |  |
|  |  +-----------------------------------+       +---------------------------------------+  |  |
|  |  |   VM 1: Servidor Wazuh            |       |   VM 2: Servidor Objetivo             |  |  |
|  |  |   IP: 192.168.56.101 / .103       |       |   Hostname: 'atacante-server'         |  |  |
|  |  |                                   |       |   IP: 192.168.56.102 / .104 / 10.0.2.15   |  |
|  |  |  - Wazuh Manager (v4.7.5)         |       |                                       |  |  |
|  |  |  - Wazuh Indexer (OpenSearch)     |       |  - Rol: Víctima / Objetivo monitoreado|  |  |
|  |  |  - Wazuh Dashboard                |       |  - Agente Wazuh (Linux)               |  |  |
|  |  |  - Puertos de ingesta: 1514/1515  |       |  - Servicios: OpenSSH, Apache2        |  |  |
|  |  |  - Consola Web: 443/TCP           |       |  - Módulos activos: FIM, SCA          |  |  |
|  |  |                                   |       |  - Defensa: Fail2ban (iptables)       |  |  |
|  |  +-----------------------------------+       +---------------------------------------+  |  |
|  +-----------------------------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------------------------+
```

---

## Nota Técnica sobre Hostnames e IPs en las Evidencias

> [!NOTE]
> **Aclaración sobre el hostname `atacante-server` y direccionamiento IP:**
> 1. **Rol de la VM 2:** Durante el aprovisionamiento de las máquinas en VirtualBox, a la VM 2 se le asignó el nombre de máquina `atacante-server`. Al instalar el agente de Wazuh, tomó ese hostname automáticamente y así quedó catalogado en el Dashboard. **Su rol operativo es estrictamente el servidor objetivo/víctima** donde residen los servicios expuestos (Apache, SSH), las defensas (Fail2ban) y los módulos FIM/SCA. Los ataques fueron lanzados desde el host Windows hacia esta máquina.
> 2. **Direccionamiento IP en las capturas:** Durante el ciclo de vida del laboratorio (reinstalaciones, pruebas con adaptador NAT para salida a internet y reinicios de interfaces en VirtualBox), los equipos tomaron direcciones dentro del segmento Host-only (`192.168.56.101` a `.104`) y en la interfaz NAT (`10.0.2.15`). Por ello, en algunas evidencias del Dashboard la URL figura en `https://192.168.56.103/` y la IP del agente como `.104` o `10.0.2.15`, con IDs de agente `004` y `007`. Funcionalmente, la topología mantiene la separación estricta entre el servidor central y los endpoints monitoreados.

---

## Distribución de Componentes y Recursos

Para asegurar la estabilidad del stack completo sobre un equipo físico de recursos moderados, se distribuyeron las capacidades de la siguiente forma:

### 1. VM 1: Servidor Wazuh All-in-One
- **Sistema Operativo**: Ubuntu Server 22.04 LTS.
- **Recursos VirtualBox**: 2 vCPU, 4.5 GB RAM, 30 GB almacenamiento.
- **Servicios Principales**:
  - **Wazuh Manager**: Motor central de recolección, decodificación por reglas y correlación de alertas. Escucha en los puertos `1514/TCP` (agentes) y `1515/TCP` (enrolamiento automático).
  - **Wazuh Indexer**: Motor de búsqueda y analítica basado en OpenSearch. Requiere configuración específica de Heap Size (`-Xms2g -Xmx2g`) para evitar que el kernel termine el proceso por falta de memoria (OOM).
  - **Wazuh Dashboard**: Consola web HTTPS accesible desde el navegador del host.

### 2. VM 2: Endpoint Linux Monitoreado (`atacante-server`)
- **Sistema Operativo**: Ubuntu Server 22.04 LTS.
- **Recursos VirtualBox**: 1 vCPU, 2 GB RAM, 20 GB almacenamiento.
- **Servicios y Módulos Activos**:
  - **Wazuh Agent (Linux)**: Transmite telemetría de `/var/log/auth.log`, `syslog`, y eventos de integridad de `/etc`.
  - **OpenSSH Server**: Servicio protegido mediante políticas locales y Fail2ban.
  - **Servidor Web Apache**: Utilizado para simulación de reconocimiento y análisis de `access.log`.
  - **Fail2ban**: Demonio de prevención que analiza intentos fallidos en `auth.log` y aplica bloqueos dinámicos en `iptables`.

### 3. Host Físico Windows (`LAPTOP-6MIJ8MV4` - `192.168.56.1`)
- **Hipervisor**: Oracle VirtualBox gestionando la red Host-only aislada.
- **Wazuh Agent (Windows)**: Registrado como agente activo (ID: 005), transmitiendo registros del visor de eventos (*Security*): inicios de sesión fallidos (Event ID 4625), creación de usuarios locales (Event ID 4720) y modificaciones del grupo Administradores (Event ID 4728/4732).
- **Consola de Analista**: Navegador web para interacción directa con el Wazuh Dashboard.

---

## Aislamiento y Seguridad de la Red

La elección del adaptador **Host-only** (`192.168.56.0/24`) garantiza un entorno confinado:
- Todo el tráfico generado durante las simulaciones (fuerza bruta SSH, escaneos web de rutas) queda estrictamente contenido entre el host físico y las VMs.
- No existe exposición de los servicios del laboratorio hacia redes externas ni afectación de la red local física del hogar.

---

## Documentos Relacionados

- Proceso de despliegue y resolución de problemas: [Configuración y Despliegue](../configuracion/README.md).
- Criterios de análisis y triage: [Metodología SOC](../metodologia/README.md).
