# Wazuh-SOC-Lab: Monitoreo Defensivo y Análisis de Seguridad

Repositorio centralizado que documenta la implementación, configuración, evaluación y análisis de eventos de seguridad utilizando la plataforma SIEM **Wazuh** en un entorno heterogéneo (Linux y Windows), junto con el análisis e investigación de incidentes desde una perspectiva **Blue Team / SOC Tier 1**.

---

## Estructura del Proyecto

```text
Wazuh-SOC-Lab/
├── docs/
│   ├── arquitectura/                     # Arquitectura de virtualización, Wazuh Server y agentes
│   ├── configuracion/                    # Proceso de despliegue y enrolamiento de agentes
│   └── metodologia/                      # Enfoque analítico SOC y ciclo de incidentes
│
├── labs/
│   ├── 01-fuerza-bruta-ssh/              # Análisis de fuerza bruta SSH y respuesta con Fail2ban
│   ├── 02-monitoreo-integridad-archivos/ # File Integrity Monitoring (FIM) con Wazuh
│   ├── 03-evaluacion-configuracion-seguridad/ # Hardening y remediación con Wazuh SCA
│   └── 04-monitoreo-linux-windows/       # Monitoreo de eventos en sistemas Linux y Windows
│
├── casos/
│   ├── caso-01-actividad-sospechosa/     # Ticket SOC: Escaneo web, intentos SSH y contención
│   └── caso-02-incidente-seguridad/      # Ticket SOC: Acceso no autorizado, sudo y respuesta
│
└── evidencias/                           # Capturas y evidencias recolectadas en el entorno
    └── labs/
        ├── fim/
        ├── sca/
        └── monitoreo/
```

---

## Entorno y Tecnologías Utilizadas

- **Host Físico (Windows)**:
  - Host de virtualización para Oracle VirtualBox.
  - Endpoint monitoreado con **Wazuh Agent** (Windows).
  - Consola de visualización de eventos y acceso al Wazuh Dashboard vía navegador web.
- **Máquinas Virtuales (Ubuntu Server en VirtualBox)**:
  - **VM 1 (Wazuh Server)**: Despliegue de Wazuh Manager, Indexer y Dashboard (v4.7.5).
  - **VM 2 (`atacante-server`)**: Servidor objetivo y endpoint monitoreado con **Wazuh Agent** (Linux), servicios SSH/Apache, módulos FIM, SCA y Fail2ban.
- **Red**:
  - Red local aislada tipo _Host-only_ en VirtualBox.

---

## Documentación General

1. [Arquitectura del Entorno](docs/arquitectura/README.md): Esquema de virtualización en Windows, distribución de las VMs Ubuntu Server (servidor Wazuh y `atacante-server`), roles de agentes y red Host-only.
2. [Configuración y Despliegue](docs/configuracion/README.md): Despliegue de la VM del servidor Wazuh, registro del agente en la VM Linux y del agente en el host Windows, y verificación de conectividad.
3. [Metodología de Monitoreo y Análisis SOC](docs/metodologia/README.md): Criterios de triage, evaluación de impacto, ciclo de vida del ticket y enfoque defensivo.

---

## Laboratorios Prácticos (Labs)

Los laboratorios abarcan pruebas controladas y validación de capacidades defensivas del entorno:

- **[Lab 01: Análisis de Fuerza Bruta SSH y Fail2ban](labs/01-fuerza-bruta-ssh/README.md)**  
  Simulación de intentos fallidos por SSH en la VM Linux, análisis directo de registros en `/var/log/auth.log`, respuesta automática perimetral en `/var/log/fail2ban.log` y documentación de limitaciones técnicas.

- **[Lab 02: Monitoreo de Integridad de Archivos (FIM)](labs/02-monitoreo-integridad-archivos/README.md)**  
  Supervisión en tiempo real de directorios críticos del sistema (`/etc`) en el agente Linux, detección de eventos de modificación (`Integrity checksum changed`) y creación (`File added to the system`), con verificación visual en el Dashboard.

- **[Lab 03: Evaluación de Configuración de Seguridad (SCA)](labs/03-evaluacion-configuracion-seguridad/README.md)**  
  Auditoría de cumplimiento basada en benchmarks en el agente Linux, detección de configuraciones deficientes y remediación técnica verificada (estados iniciales `FAILED` a finales `PASSED`) para el servicio NTP y el banner de acceso remoto (`/etc/issue.net`).

- **[Lab 04: Monitoreo de Eventos en Linux y Windows](labs/04-monitoreo-linux-windows/README.md)**  
  Recolección y análisis de telemetría heterogénea: eventos de primer uso de sudo, modificación de contraseñas y creación de grupos en la VM Linux; junto con inicios de sesión fallidos, creación de usuarios locales y alteración del grupo Administradores en el host Windows.

---

## Casos de Seguridad (Blue Team / SOC Cases)

Escenarios de investigación estructurados bajo el formato de tickets de seguridad:

- **[Caso 01: Actividad Sospechosa Contenida (Ticket SOC-2026-001)](casos/caso-01-actividad-sospechosa/README.md)**  
  Detección de escaneo web automatizado en Apache (`access.log`) seguido de intentos de fuerza bruta SSH contra `root` en `auth.log`. Análisis de correlación, contención automática con Fail2ban y cierre sin impacto.

- **[Caso 02: Incidente de Seguridad con Impacto y Contención (Ticket SOC-2026-002)](casos/caso-02-incidente-seguridad/README.md)**  
  Investigación de inicio de sesión SSH exitoso con credenciales legítimas desde una IP no habitual, seguido de elevación de privilegios mediante `sudo`. Evaluación de impacto, acciones manuales de contención, revocación de accesos y recomendaciones defensivas.

---

## Evidencias

Todas las capturas de pantalla que respaldan las alertas, estados y verificaciones se encuentran preservadas y enlazadas dentro de cada laboratorio respectivo bajo el directorio [evidencias/](evidencias/).
