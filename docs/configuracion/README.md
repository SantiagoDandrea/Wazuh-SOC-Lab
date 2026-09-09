# Configuración y Despliegue del Laboratorio

Este documento detalla el procedimiento técnico de instalación, configuración y registro de los componentes del entorno **Wazuh-SOC-Lab**.

---

## Requisitos y Entorno Base

- **Host Físico**: Sistema Windows con Oracle VirtualBox instalado y adaptador de red Host-only configurado.
- **Máquinas Virtuales (VirtualBox)**:
  - **VM 1**: Ubuntu Server dedicada al **Wazuh Server** (Manager, Indexer y Dashboard v4.7.5).
  - **VM 2**: Ubuntu Server (`atacante-server`) configurada como agente de monitoreo Linux y servidor objetivo.
- **Agente Windows**: Instalado directamente sobre el host Windows.

Para detalles sobre el diagrama y la interacción entre componentes, consultar [Arquitectura](../arquitectura/README.md).

---

## Procedimiento de Implementación

### 1. Despliegue de la VM del Servidor Wazuh (Ubuntu Server)
1. Creación e instalación de la VM Ubuntu Server en VirtualBox conectada a la red Host-only.
2. Despliegue de los servicios del Wazuh Server (Manager, Indexer y Dashboard).
3. Verificación de los puertos de servicio y comunicación:
   - `1514/TCP`: Canal cifrado de recolección y comunicación de agentes.
   - `1515/TCP`: Servicio de registro y enrolamiento automático (`authd`).
   - `443/TCP` / `5601/TCP`: Acceso al Wazuh Dashboard desde el navegador del host Windows.
4. Verificación de acceso al Dashboard mediante el navegador web en Windows: `https://<IP_VM_Wazuh_Server>`.

### 2. Instalación y Registro del Agente Linux en la VM 2 (`atacante-server`)
1. Instalación del paquete `wazuh-agent` en la VM Ubuntu Server `atacante-server`.
2. Configuración de la IP de la VM Wazuh Server en `/var/ossec/etc/ossec.conf`.
3. Inicio y habilitación del servicio:
   ```bash
   systemctl daemon-reload
   systemctl enable wazuh-agent
   systemctl start wazuh-agent
   ```
4. Verificación de recolección de logs locales (`/var/log/auth.log`, `/var/log/syslog`) y módulos FIM y SCA.

### 3. Instalación y Registro del Agente Windows (Host)
1. Descarga y ejecución del instalador del agente Wazuh (MSI) en el sistema operativo host Windows.
2. Configuración de la dirección IP de la VM Wazuh Server y autenticación de registro hacia el puerto `1515/TCP`.
3. Inicio del servicio del agente Wazuh en Windows (`services.msc` o mediante PowerShell).
4. Verificación del monitoreo del visor de eventos de Windows (canal *Security*, eventos 4625, 4720, 4728/4732).

### 4. Verificación del Estado de los Agentes
Desde el **Wazuh Dashboard** en el navegador del host Windows:
1. Acceso a la sección **Agents**.
2. Validación de que ambos agentes (`atacante-server` en Ubuntu y el host Windows) se encuentren en estado **Active**.

> [!NOTE]
> La evidencia de ambos agentes activos y reportando simultáneamente en el Dashboard puede verse en [Evidencias de Monitoreo](../../evidencias/labs/monitoreo/Lab-agents.png).

---

## Próximos Pasos
- Para revisar los lineamientos analíticos de investigación, ver [Metodología](../metodologia/README.md).
- Para comenzar las pruebas prácticas, consultar los laboratorios en [Laboratorios](../../labs/01-fuerza-bruta-ssh/README.md).
