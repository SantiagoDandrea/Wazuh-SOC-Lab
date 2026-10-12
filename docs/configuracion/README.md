# Configuración, Despliegue y Resolución de Problemas

Este documento detalla el procedimiento técnico de instalación del servidor Wazuh, el enrolamiento de los agentes en ambos sistemas operativos y los desafíos técnicos reales que surgieron durante el despliegue del entorno.

---

## 1. Instalación del Servidor Wazuh (VM 1 - Ubuntu Server)

Para optimizar recursos en un entorno de laboratorio sobre VirtualBox, se utilizó el instalador *All-in-One* de Wazuh v4.7.5:

```bash
# Descarga del script y configuración oficial de Wazuh
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.7/config.yml

# Generación de certificados e inicio del cluster
bash wazuh-install.sh --generate-config-files
bash wazuh-install.sh --start-cluster
```

### Verificación de servicios y puertos
Finalizado el script, se validó el estado de los tres componentes del stack:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard

# Puertos esenciales de comunicación
sudo ss -tulpn | grep -E "1514|1515|443"
```
- `1514/TCP`: Canal cifrado de recolección de eventos desde los agentes.
- `1515/TCP`: Servicio de registro y enrolamiento automático (`authd`).
- `443/TCP`: Consola web HTTPS del Wazuh Dashboard.

---

## 2. Enrolamiento de Agentes

### Agente Linux (VM 2: `atacante-server`)
En la máquina virtual objetivo se instaló el paquete `wazuh-agent` apuntando a la IP del servidor Wazuh:

```bash
# Instalación con variables de entorno para registro automático
sudo WAZUH_MANAGER='192.168.56.101' WAZUH_AGENT_NAME='atacante-server' apt-get install wazuh-agent

# Habilitación y arranque del servicio
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent

# Verificación de conexión en los logs
sudo tail -f /var/ossec/logs/ossec.log | grep -i "connected"
```

### Agente Windows (Host Físico)
1. Se descargó el instalador oficial `wazuh-agent-4.7.5-1.msi`.
2. Se ejecutó la instalación desatendida desde una consola de PowerShell como Administrador:
   ```powershell
   msiexec.exe /i wazuh-agent-4.7.5-1.msi /q WAZUH_MANAGER="192.168.56.101" WAZUH_REGISTRATION_SERVER="192.168.56.101"
   NET START WazuhSvc
   ```
3. Se verificó en el Dashboard que el agente comenzó a recolectar el canal de eventos *Security* de Windows.

---

## 3. Desafíos Técnicos Reales y Solución de Problemas

Durante la puesta a punto del laboratorio se presentaron tres problemas técnicos que requirieron diagnóstico y resolución:

### 1. Caída del Wazuh Indexer por agotamiento de memoria (OOM Killer)
- **Síntoma**: Al cabo de unos minutos de actividad, el Indexer dejaba de responder y la interfaz web arrojaba error de conexión (502 / conexión rechazada).
- **Causa raíz**: Wazuh Indexer corre sobre OpenSearch (JVM). En una máquina virtual con 4.5 GB de RAM, el consumo por defecto excedía la capacidad física y el kernel de Linux ejecutaba el mecanismo de *Out of Memory Killer* finalizando el proceso.
- **Solución técnica aplicada**:
  1. Se incrementó el límite de memoria virtual del kernel en `/etc/sysctl.conf`:
     ```bash
     echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
     sudo sysctl -p
     ```
  2. Se configuró un límite estricto de Heap Size en `/etc/wazuh-indexer/jvm.options` asignando 2 GB:
     ```text
     -Xms2g
     -Xmx2g
     ```
  3. Se reinició el servicio (`sudo systemctl restart wazuh-indexer`), estabilizando el consumo sin nuevas caídas.

### 2. Retardo en la detección de FIM (Barrido Programado vs. Tiempo Real)
- **Síntoma**: Al modificar archivos en `/etc/hosts` o crear archivos de prueba, las alertas no aparecían en el Dashboard de forma inmediata.
- **Causa raíz**: El motor `syscheck` opera por defecto con una frecuencia de barrido periódico cada 12 horas (`frequency="43200"`).
- **Solución técnica aplicada**:
  Se editó `/var/ossec/etc/ossec.conf` en el agente Linux agregando la directiva `realtime="yes"` sobre `/etc`, habilitando la monitorización continua basada en llamadas `inotify` del kernel.

### 3. Agente Windows en estado "Never Connected" por Perfil de Red
- **Síntoma**: El agente de Windows no lograba registrarse ni aparecía activo en la consola de Wazuh.
- **Causa raíz**: El Firewall de Windows clasificó el adaptador virtual "VirtualBox Host-Only Network" bajo el perfil de **Red Pública**, bloqueando el tráfico saliente hacia los puertos `1514` y `1515`.
- **Solución técnica aplicada**:
  Se reasignó el perfil de la interfaz a **Privado** mediante PowerShell:
  ```powershell
  Get-NetConnectionProfile -InterfaceAlias "VirtualBox Host-Only Network" | Set-NetConnectionProfile -NetworkCategory Private
  ```
  Inmediatamente después el agente estableció el túnel cifrado y cambió a estado `Active`.

---

## Documentos Relacionados

- Topología y roles de red: [Arquitectura](../arquitectura/README.md).
- Metodología de análisis y severidades: [Metodología SOC](../metodologia/README.md).
