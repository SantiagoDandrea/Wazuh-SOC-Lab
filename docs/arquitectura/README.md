# Arquitectura del Entorno

Este documento describe la arquitectura técnica y de red del laboratorio **Wazuh-SOC-Lab**.

---

## Esquema General

El despliegue está implementado sobre una máquina física **Windows**, la cual cumple un doble rol: actúa como host de virtualización (Oracle VirtualBox) y como endpoint monitoreado con el agente de Wazuh instalado. Dentro de VirtualBox se ejecutan dos máquinas virtuales con **Ubuntu Server**: una dedicada exclusivamente al servidor Wazuh y otra configurada como servidor/objetivo monitoreado (`atacante-server`).

```text
+-----------------------------------------------------------------------------------+
|                            Host Físico: Windows                                   |
|                                                                                   |
|  - Agente Wazuh (Windows) [Monitoreo de eventos locales: logon, usuarios, grupos] |
|  - Navegador Web [Acceso al Wazuh Dashboard: https://<IP_Wazuh_Server>:443]       |
|                                                                                   |
|  +-----------------------------------------------------------------------------+  |
|  |                    Oracle VirtualBox (Red Host-only)                        |  |
|  |                                                                             |  |
|  |  +-----------------------------------+   +-------------------------------+  |  |
|  |  |      VM 1: Ubuntu Server          |   |      VM 2: Ubuntu Server      |  |  |
|  |  |         (Wazuh Server)            |   |     ('atacante-server')       |  |  |
|  |  |                                   |   |                               |  |  |
|  |  |  - Wazuh Manager (v4.7.5)         |   |  - Agente Wazuh (Linux)       |  |  |
|  |  |  - Wazuh Indexer                  |   |  - Servicio SSH / Apache      |  |  |
|  |  |  - Wazuh Dashboard                |   |  - Módulo FIM (/etc)          |  |  |
|  |  |  - Ingesta de puertos 1514/1515   |   |  - Módulo SCA (Hardening)     |  |  |
|  |  |                                   |   |  - Fail2ban (Respuesta local) |  |  |
|  |  +-----------------------------------+   +-------------------------------+  |  |
|  +-----------------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------------+
```

---

## Componentes del Sistema

### 1. VM 1: Servidor Wazuh (Ubuntu Server - Wazuh v4.7.5)
Máquina virtual dedicada al procesamiento, indexación y visualización centralizada de la telemetría de seguridad:
- **Wazuh Manager**: Recibe los eventos enviados por los agentes en los puertos `1514/TCP` y `1515/TCP`, ejecuta la decodificación, correlación por reglas y dispara las alertas.
- **Wazuh Indexer**: Motor de búsqueda y analítica distribuida que almacena e indexa los registros en tiempo real.
- **Wazuh Dashboard**: Interfaz web analítica servida por la VM, accesible desde el navegador del host Windows para la gestión de agentes, auditorías FIM/SCA y análisis de eventos.

### 2. VM 2: Agente Linux Monitoreado (`atacante-server` - Ubuntu Server)
Máquina virtual utilizada como entorno de pruebas y servidor objetivo:
- **Wazuh Agent**: Transmite logs de autenticación (`/var/log/auth.log`), logs del sistema (`syslog`) y telemetría de módulos hacia el servidor Wazuh.
- **Servicios evaluados**: Servicio OpenSSH (`sshd`) y servidor web Apache.
- **Módulos activos de Wazuh**:
  - **FIM (`syscheck`)**: Monitoreo de integridad de archivos críticos en `/etc`.
  - **SCA**: Evaluación continua de configuración de seguridad y cumplimiento de buenas prácticas.
- **Defensa perimetral local**: Fail2ban configurado para correlacionar eventos de autenticación fallida y aplicar bloqueos automáticos en el firewall.

### 3. Host Windows: Host de Virtualización y Agente Wazuh
- **Rol de Host**: Ejecuta Oracle VirtualBox y conecta los endpoints mediante un adaptador de red exclusivo tipo *Host-only*.
- **Rol de Agente Wazuh**: Monitorea eventos nativos del sistema operativo mediante el canal de seguridad de Windows (Event IDs 4625 de inicios fallidos, 4720 de creación de cuentas y 4728/4732 de cambios en grupos de administradores).
- **Consola de Operación SOC**: Visualización e investigación de alertas desde el navegador web de Windows conectado a la interfaz web del Wazuh Dashboard.

### 4. Conectividad y Red
- **Tipo de red**: Red local aislada tipo **Host-only** gestionada en VirtualBox.
- Permite la comunicación segura y controlada entre el host Windows, la VM del Wazuh Server y la VM `atacante-server` sin exponer los servicios del laboratorio hacia redes externas no controladas.

---

## Referencias
- Para los pasos detallados de instalación y registro de endpoints, consultar [Configuración y Despliegue](../configuracion/README.md).
- Para conocer la metodología de análisis SOC aplicada al entorno, consultar [Metodología](../metodologia/README.md).
