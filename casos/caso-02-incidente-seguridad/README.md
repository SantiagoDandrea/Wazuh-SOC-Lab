# Caso 02: Acceso No Autorizado y Elevación de Privilegios

> **Nota Operativa**: Este escenario reproduce la investigación de un incidente de seguridad simulado donde un actor no autorizado utiliza credenciales legítimas comprometidas del usuario `analyst` para ingresar por SSH a la VM objetivo (`atacante-server`), ejecutando inmediatamente comandos privilegiados mediante `sudo`.

---

## Ficha del Ticket SOC

| Campo | Detalle |
|---|---|
| **Ticket ID** | SOC-2026-002 |
| **Tipo de Evento** | Acceso no autorizado con credenciales válidas y uso de `sudo` |
| **Severidad** | Alta |
| **Estado** | Cerrado a nivel SOC (Contenido manualmente y reportado) |
| **Escalamiento** | Requerido (Informe técnico derivado a Sysadmin) |
| **Analista Asignado** | Santiago D'Andrea |

---

## Resumen del Incidente

Se detectó un inicio de sesión interactivo por SSH hacia la cuenta `analyst` desde la dirección IP `192.168.56.1`. La autenticación fue exitosa en el primer intento y sin fallos previos, lo que evidencia el uso de credenciales legítimas comprometidas.

A los pocos segundos de ingresar, la sesión ejecutó `sudo cat /etc/shadow` (intento de extracción de hashes criptográficos de contraseñas) y posteriormente invocó una shell interactiva de root (`sudo /bin/bash`). Como la actividad no contaba con orden de cambio autorizada, se procedió a la contención manual inmediata, finalización forzada de la sesión y bloqueo de la cuenta.

---

## Análisis Técnico de Telemetría

### 1. Acceso Inicial por SSH
- **Fuente de telemetría**: `/var/log/auth.log`.
- **Registros crudos**:
  ```text
  Oct 12 02:15:10 atacante-server sshd[3120]: Accepted password for analyst from 192.168.56.1 port 55102 ssh2
  Oct 12 02:15:10 atacante-server systemd-logind[780]: New session 4 of user analyst.
  ```
- **Evaluación**: Autenticación sin intentos erróneos precedentes. El vector de entrada fue el uso indebido de contraseñas previamente filtradas o conocidas fuera del host.

### 2. Invocación de Privilegios Elevados (`sudo`)
- **Fuente de telemetría**: `/var/log/auth.log`.
- **Registros crudos**:
  ```text
  Oct 12 02:15:28 atacante-server sudo[3155]: analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/usr/bin/cat /etc/shadow
  Oct 12 02:15:45 atacante-server sudo[3168]: analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/bin/bash
  ```
- **Evaluación**: El comando `/usr/bin/cat /etc/shadow` apunta a la recolección de credenciales para ataque fuera de línea (*offline cracking*), mientras que `/bin/bash` otorga control interactivo total como superusuario.

---

## Cronología del Incidente (Timeline)

```text
[02:15:10] - Inicio de sesión SSH exitoso del usuario 'analyst' desde 192.168.56.1.
[02:15:28] - Ejecución de 'sudo cat /etc/shadow' (lectura de hashes de contraseñas).
[02:15:45] - Ejecución de 'sudo /bin/bash' (apertura de shell interactiva de root).
[02:16:05] - Detección analítica de los eventos en logs y Dashboard. Inicio de contención.
[02:16:22] - Terminación forzada de la sesión y procesos activos: 'pkill -9 -u analyst'.
[02:16:45] - Bloqueo de la cuenta a nivel de sistema operativo: 'passwd -l analyst'.
[02:17:10] - Auditoría de integridad con Wazuh FIM en /etc (sin alteraciones detectadas).
[02:18:00] - Cierre del ticket SOC Tier 1 y elaboración de informe técnico para Sysadmin.
```

---

## Evaluación de Impacto (Tríada CIA)

- **Confidencialidad**: **Comprometida parcialmente**. Al ejecutarse la lectura directa de `/etc/shadow`, se asume que los hashes de contraseñas del sistema quedaron expuestos al atacante.
- **Integridad**: **Preservada**. Mediante el módulo FIM de Wazuh se auditó `/etc` y se constató que no se añadieron puertas traseras, scripts ni modificaciones en `/etc/passwd`.
- **Disponibilidad**: **Preservada**. El servidor y los servicios en ejecución mantuvieron su disponibilidad sin interrupciones.
- **Severidad Asignada**: **Alta** (acceso privilegiado y exposición de secretos del sistema operativo).

---

## Acciones de Contención Manual Aplicadas

Para neutralizar la amenaza de forma inmediata en el servidor se ejecutaron los siguientes comandos:

```bash
# 1. Terminación inmediata de todos los procesos asociados a la cuenta comprometida
sudo pkill -9 -u analyst

# 2. Bloqueo de la cuenta de usuario para impedir nuevos inicios de sesión
sudo passwd -l analyst
sudo usermod -s /usr/sbin/nologin analyst

# 3. Verificación de conexiones de red activas y ausencia de procesos residuales
ps aux | grep analyst
sudo ss -tulpn | grep :22
```

---

## Informe Técnico y Recomendaciones para Sysadmin

Al operar en un laboratorio personal donde el rol de SOC y administración recae en el mismo analista, se documentan las medidas de remediación estructural que en producción se derivarían al equipo de Administración de Sistemas:

1. **Rotación General de Contraseñas**: Ante la lectura confirmada de `/etc/shadow`, se requiere el cambio inmediato de contraseñas de todas las cuentas locales del sistema.
2. **Hardening de SSH**: Modificar `/etc/ssh/sshd_config` estableciendo `PasswordAuthentication no`, forzando el uso exclusivo de autenticación criptográfica por llaves públicas robustas (Ed25519 o RSA 4096).
3. **Principio de Menor Privilegio en Sudoers**: Restringir las directivas en `/etc/sudoers` para que los usuarios operativos no cuenten con permisos de ejecución sobre binarios de shell (`/bin/bash`, `/bin/sh`) ni herramientas de lectura masiva de archivos de sistema.

**Resolución:** Ticket cerrado a nivel Tier 1 con contención aplicada e informe de remediación emitido.

---

## Documentos Relacionados

- Monitoreo de comandos sudo y cuentas locales: [Laboratorio 04 - Monitoreo de Linux y Windows](../../labs/04-monitoreo-linux-windows/README.md).
- Caso anterior: [Caso 01 - Actividad Sospechosa Contenida](../caso-01-actividad-sospechosa/README.md).
