# Arquitectura del Entorno

Este documento describe la arquitectura técnica del entorno utilizado a lo largo de los laboratorios y casos de análisis de seguridad en **Wazuh-SOC-Lab**.

---

## Esquema General

El entorno está diseñado como un laboratorio de monitoreo de seguridad y respuesta ante incidentes basado en una arquitectura cliente-servidor centralizada.

```text
               +----------------------------------------+
               |              Wazuh Server              |
               |                                        |
               |  +----------------+ +---------------+  |
               |  | Wazuh Manager  | | Wazuh Indexer |  |
               |  +----------------+ +---------------+  |
               |             +-----------------+        |
               |             | Wazuh Dashboard |        |
               |             +-----------------+        |
               +-------------------+--------------------+
                                   |
                     Red Host-only (VirtualBox)
                                   |
         +-------------------------+-------------------------+
         |                                                   |
+-------------------+                               +-------------------+
|   Agente Linux    |                               |  Agente Windows   |
|  (Ubuntu Server)  |                               |   (Host Windows)  |
|                   |                               |                   |
| - Recolección log |                               | - Logs de eventos |
| - Fail2ban        |                               |   de seguridad    |
| - Módulo FIM      |                               | - Gestión de      |
| - Módulo SCA      |                               |   cuentas         |
+-------------------+                               +-------------------+
```

---

## Componentes del Sistema

### 1. Wazuh Server (Versión 4.7.5)
Concentra las funciones de procesamiento, indexación y presentación de eventos de seguridad:
- **Wazuh Manager**: Motor central encargado de recibir datos desde los agentes, procesar y normalizar logs, ejecutar reglas de decodificación y correlación, y disparar alertas.
- **Wazuh Indexer**: Motor de búsqueda y almacenamiento distribuido que indexa los eventos y alertas procesados en tiempo real.
- **Wazuh Dashboard**: Interfaz web analítica que permite consultar eventos, visualizar métricas de seguridad, auditar módulos (FIM, SCA) y revisar el estado operativo de los agentes.

### 2. Agentes de Monitoreo
Endpoints configurados con el agente de Wazuh que recolectan eventos y envían datos hacia el servidor mediante canales cifrados:
- **Agente Linux**: Máquina virtual con **Ubuntu Server**, utilizada para pruebas de servicios de red (SSH, Apache), monitoreo de integridad de archivos (`/etc`), auditorías de hardening y respuesta con herramientas del sistema (Fail2ban).
- **Agente Windows**: Endpoint **Windows**, utilizado para monitoreo de eventos de autenticación, creación de cuentas locales y modificación de grupos privilegiados del sistema.

### 3. Red y Virtualización
- **Virtualización**: Oracle VirtualBox.
- **Topología de red**: Red local aislada tipo **Host-only**, permitiendo la comunicación directa y controlada entre los agentes y el servidor Wazuh sin exponer servicios hacia redes no confiables.

---

## Referencias
- Para los pasos detallados de instalación y registro de endpoints, consultar [Configuración y Despliegue](../configuracion/README.md).
- Para conocer la metodología de análisis SOC aplicada al entorno, consultar [Metodología](../metodologia/README.md).
