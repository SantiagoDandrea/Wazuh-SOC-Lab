# Wazuh-SOC-Lab: Monitoreo Defensivo, Detección y Análisis de Incidentes

[![SIEM](https://img.shields.io/badge/SIEM-Wazuh%204.7.5-blue.svg)](https://wazuh.com/)
[![OS](https://img.shields.io/badge/Endpoints-Ubuntu%20%7C%20Windows-brightgreen.svg)]()
[![Defense](https://img.shields.io/badge/Defense-Fail2ban%20%7C%20FIM%20%7C%20SCA-orange.svg)]()
[![Role](https://img.shields.io/badge/Focus-SOC%20Tier%201%20%2F%20Blue%20Team-navy.svg)]()

Repositorio que documenta la implementación práctica, configuración, auditoría y análisis de telemetría de seguridad utilizando la plataforma SIEM **Wazuh** en un entorno heterogéneo (**Linux y Windows**), junto con el análisis e investigación de incidentes simulados desde la perspectiva de un analista **Blue Team / SOC Tier 1**.

El objetivo de este proyecto fue salir del plano puramente teórico para experimentar directamente cómo se reciben e interpretan los registros del sistema operativo, por qué en ocasiones las herramientas de prevención local (Fail2ban) pueden generar brechas de visibilidad en el SIEM, cómo resolver problemas reales de estabilidad y consumo de recursos, y cómo estructurar formalmente la investigación y contención de incidentes de seguridad.

---

## Estructura del Proyecto

```text
Wazuh-SOC-Lab/
├── docs/
│   ├── arquitectura/                     # Topología de red en VirtualBox, recursos y roles
│   ├── configuracion/                    # Despliegue All-in-One y resolución de problemas reales
│   └── metodologia/                      # Criterios de triage, matriz de severidad y flujo de trabajo
│
├── labs/
│   ├── 01-fuerza-bruta-ssh/              # Fuerza bruta SSH, Fail2ban y análisis del visibility gap
│   │   ├── local_rules.xml               # Regla custom adaptada para correlación por IP
│   │   └── jail.local                    # Configuración de Fail2ban para protección de SSH
│   ├── 02-monitoreo-integridad-archivos/ # Monitoreo de integridad de archivos en /etc (FIM)
│   │   └── ossec-fim-snippet.conf        # Snippet seguro con realtime inotify y directiva nodiff
│   ├── 03-evaluacion-configuracion-seguridad/ # Hardening con SCA (CIS Benchmarks)
│   │   └── issue.net                     # Banner remediado que cumple con la política CIS
│   └── 04-monitoreo-linux-windows/       # Telemetría heterogénea (sudo, claves, grupos, eventos de Windows)
│
├── casos/
│   ├── caso-01-actividad-sospechosa/     # Ticket SOC 01: Reconocimiento web y fuerza bruta mitigada
│   └── caso-02-incidente-seguridad/      # Ticket SOC 02: Acceso no autorizado, sudo y contención manual
│
└── evidencias/                           # Capturas del Dashboard que respaldan las alertas
    └── labs/
        ├── fim/
        ├── sca/
        └── monitoreo/
```

---

## Arquitectura del Entorno

- **Host Físico (Windows - `192.168.56.1` | Hostname: `LAPTOP-6MIJ8MV4`)**:
  - Hipervisor (Oracle VirtualBox) y consola de analista (acceso web al Wazuh Dashboard).
  - Endpoint monitoreado con agente Wazuh para Windows (canales de eventos de seguridad).
  - Origen de las simulaciones de ataque hacia las VMs a través de la red Host-only.
- **Máquinas Virtuales (Ubuntu Server en red Host-only `192.168.56.0/24`)**:
  - **VM 1: Servidor Wazuh (`192.168.56.101` / `.103`)**: Despliegue All-in-One de Wazuh Manager, Indexer y Dashboard (v4.7.5).
  - **VM 2: Servidor objetivo (`atacante-server` - `192.168.56.102` / `.104`)**:
    - Servidor Linux monitoreado con Apache, OpenSSH, Fail2ban y agente de Wazuh.
    > *Nota sobre el hostname y direccionamiento:* Durante el aprovisionamiento de la máquina virtual se le asignó el nombre `atacante-server` y con ese identificador quedó registrada en el Dashboard. Opera estrictamente como el servidor objetivo/víctima del laboratorio. Para más detalles sobre las IPs en las evidencias, consultar [Arquitectura](docs/arquitectura/README.md).

---

## Documentación Técnica

- [Arquitectura del Entorno](docs/arquitectura/README.md): Topología de red Host-only, roles de los equipos, recursos de memoria asignados y aclaraciones sobre hostnames e IPs en las capturas.
- [Configuración y Troubleshooting](docs/configuracion/README.md): Instalación del servidor, conexión de agentes y problemas resueltos durante el despliegue (OOM Killer en el Indexer, latencia en FIM y firewall de Windows).
- [Metodología SOC Tier 1](docs/metodologia/README.md): Criterios de triage analítico, matriz de severidad técnica y procedimiento formal de escalamiento e informes.

---

## Laboratorios Prácticos (Labs)

- **[Lab 01: Fuerza Bruta SSH, Fail2ban y Análisis de Detección](labs/01-fuerza-bruta-ssh/README.md)**  
  Simulación de ataques contra SSH, contención perimetral con Fail2ban y análisis del *gap* de detección en el SIEM: por qué la regla nativa no alertaba ante el corte temprano de Fail2ban y diseño de la regla custom `100001` en `local_rules.xml` para correlacionar antes del bloqueo.

- **[Lab 02: Monitoreo de Integridad de Archivos (FIM / syscheck)](labs/02-monitoreo-integridad-archivos/README.md)**  
  Supervisión de cambios en `/etc` en modo programado y en tiempo real con `inotify`, detección de creación (`fim_test2.conf`) y modificación (`/etc/hosts`), junto con la aplicación de la directiva `<nodiff>/etc/shadow` para proteger hashes de contraseñas.

- **[Lab 03: Auditoría de Seguridad (SCA) y Hardening](labs/03-evaluacion-configuracion-seguridad/README.md)**  
  Auditoría de cumplimiento con CIS Benchmarks en Linux, remediación técnica de controles en estado `FAILED` (sincronización horaria con NTP y banner de advertencia en `/etc/issue.net`) y verificación del cambio a `PASSED`.

- **[Lab 04: Monitoreo de Eventos en Linux y Windows](labs/04-monitoreo-linux-windows/README.md)**  
  Correlación de eventos críticos en ambos sistemas: primer uso de `sudo` (`Rule 5403`), modificación de contraseñas (`Rule 5555`) y creación de grupos (`Rule 5901`) en Linux; junto con inicios de sesión fallidos (Event ID 4625), creación de cuentas (Event ID 4720) y cambios en el grupo Administradores (Event ID 4728/4732) en Windows.

---

## Casos de Investigación (Tickets SOC)

- **[Caso 01: Reconocimiento Web y Fuerza Bruta SSH Contenida](casos/caso-01-actividad-sospechosa/README.md)**  
  *Severidad: Baja*. Triage de escaneo de rutas administrativas en Apache (`access.log`) seguido de intentos SSH contra `root` en `auth.log`, contenidos automáticamente por Fail2ban sin impacto operativo.

- **[Caso 02: Acceso No Autorizado y Elevación de Privilegios](casos/caso-02-incidente-seguridad/README.md)**  
  *Severidad: Alta*. Investigación de inicio de sesión SSH con credenciales legítimas desde una IP no habitual, ejecución de `sudo cat /etc/shadow`, línea de tiempo del evento, contención manual inmediata (`pkill`, `passwd -l`) y reporte técnico de remediación para Sysadmin.

---

## Evidencias Verificadas

Todas las capturas de pantalla obtenidas en el Wazuh Dashboard que respaldan las alertas, estados de agentes y evaluaciones de cumplimiento se encuentran catalogadas en el directorio [evidencias/](evidencias/).
