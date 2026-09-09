# Configuración y Despliegue del Laboratorio

Este documento detalla el procedimiento técnico de instalación, configuración y registro de los componentes del entorno **Wazuh-SOC-Lab**.

---

## Requisitos y Entorno Base

- **Wazuh Server**: Versión 4.7.5 desplegada con sus tres componentes (Manager, Indexer, Dashboard).
- **Virtualización**: Oracle VirtualBox configurado con una red interna/Host-only.
- **Sistemas Operativos**:
  - Ubuntu Server (Agente Linux).
  - Windows (Agente Windows).

Para detalles sobre la topología e interacción entre componentes, consultar [Arquitectura](../arquitectura/README.md).

---

## Procedimiento de Implementación

### 1. Despliegue del Servidor Wazuh
1. Instalación y levantamiento de los servicios del Wazuh Manager, Indexer y Dashboard en el host servidor.
2. Verificación de conectividad de los puertos de comunicación:
   - `1514/TCP`: Registro y comunicación de agentes.
   - `1515/TCP`: Servicio de enrolamiento automático.
   - `443/TCP` / `5601/TCP`: Acceso al Dashboard de Wazuh.

### 2. Instalación y Registro del Agente Linux (Ubuntu Server)
1. Instalación del paquete `wazuh-agent` en la máquina virtual Ubuntu Server.
2. Configuración de la dirección IP del Wazuh Server en `/var/ossec/etc/ossec.conf`.
3. Inicio y habilitación del servicio del agente:
   ```bash
   systemctl daemon-reload
   systemctl enable wazuh-agent
   systemctl start wazuh-agent
   ```
4. Verificación de recolección de logs del sistema (`/var/log/auth.log`, `/var/log/syslog`).

### 3. Instalación y Registro del Agente Windows
1. Descarga y ejecución del instalador del agente Wazuh (MSI) en el sistema Windows.
2. Parámetros de enrolamiento indicando la IP del Wazuh Server y autenticación de registro.
3. Inicio del servicio `Wazuh` en Windows (`services.msc` o mediante PowerShell).
4. Verificación del monitoreo del visor de eventos de Windows (Canal *Security*, eventos 4625, 4720, 4728/4732).

### 4. Verificación del Estado de los Agentes
Desde el **Wazuh Dashboard**:
1. Acceso al módulo **Agents**.
2. Validación de que ambos agentes (Ubuntu Server y Windows) se encuentren en estado **Active**.

> [!NOTE]
> La evidencia de ambos agentes activos y conectados simultáneamente en el Dashboard puede verse en [Evidencias de Monitoreo](../../evidencias/labs/monitoreo/Lab-agents.png).

---

## Próximos Pasos
- Para revisar los lineamientos analíticos de investigación, ver [Metodología](../metodologia/README.md).
- Para comenzar las pruebas prácticas, consultar los laboratorios en [Laboratorios](../../labs/01-fuerza-bruta-ssh/README.md).
