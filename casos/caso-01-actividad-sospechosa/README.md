# Caso 01: Análisis de Actividad Sospechosa Contenida

> **Nota Operativa**: Este ticket documenta la investigación de una simulación controlada lanzada desde el host físico Windows (`192.168.56.1`) hacia el servidor objetivo (`atacante-server`, `192.168.56.102`), analizando la correlación entre solicitudes al servidor web Apache, intentos fallidos en SSH y la contención automática con Fail2ban.

---

## Ficha del Ticket SOC

| Campo | Detalle |
|---|---|
| **Ticket ID** | SOC-2026-001 |
| **Tipo de Evento** | Reconocimiento web y fuerza bruta SSH |
| **Severidad** | Baja |
| **Estado** | Cerrado (Contenido automáticamente) |
| **Escalamiento** | No requerido |
| **Analista Asignado** | Santiago D'Andrea |

---

## Resumen del Evento

Durante la revisión de registros en la VM objetivo (`atacante-server`), se identificó actividad sospechosa secuencial proveniente de la dirección IP interna `192.168.56.1`:
1. Múltiples solicitudes HTTP en ráfaga contra rutas sensibles del servidor web (`/admin`, `/privado`), recibiendo códigos de estado de error (`404`, `401`, `403`).
2. Transición inmediata hacia el puerto 22 (SSH), ejecutando intentos de inicio de sesión con contraseña errónea contra la cuenta `root`.
3. Al acumular 3 intentos fallidos consecutivos, el demonio **Fail2ban** activó una regla de bloqueo en `iptables`, interrumpiendo el ataque sin impacto sobre la operatividad del sistema.

---

## Análisis Técnico del Incidente

### 1. Actividad de Reconocimiento Web
- **Fuente de telemetría**: `/var/log/apache2/access.log`.
- **Registros crudos observados**:
  ```text
  192.168.56.1 - - [12/Oct/2026:01:12:05 -0300] "GET /admin HTTP/1.1" 404 488 "-" "Mozilla/5.0 (Recon-Probe)"
  192.168.56.1 - - [12/Oct/2026:01:12:08 -0300] "GET /privado HTTP/1.1" 401 458 "-" "Mozilla/5.0 (Recon-Probe)"
  192.168.56.1 - - [12/Oct/2026:01:12:12 -0300] "GET /privado/index.html HTTP/1.1" 403 490 "-" "Mozilla/5.0 (Recon-Probe)"
  ```
- **Evaluación**: Peticiones consecutivas en un lapso de 7 segundos dirigidas a rutas administrativas típicas. Las respuestas confirman que ningún recurso restringido fue accedido exitosamente (código 200).

### 2. Intentos de Acceso por Fuerza Bruta SSH
- **Fuente de telemetría**: `/var/log/auth.log`.
- **Registros crudos observados**:
  ```text
  Oct 12 01:14:20 atacante-server sshd[2841]: Failed password for root from 192.168.56.1 port 54820 ssh2
  Oct 12 01:14:22 atacante-server sshd[2843]: Failed password for root from 192.168.56.1 port 54822 ssh2
  Oct 12 01:14:24 atacante-server sshd[2845]: Failed password for root from 192.168.56.1 port 54824 ssh2
  ```
- **Evaluación**: La misma dirección IP pasó a ejecutar fuerza bruta directa contra la cuenta `root` en SSH.

### 3. Contención Perimetral Automática
- **Fuente de telemetría**: `/var/log/fail2ban.log`.
- **Registro de acción**:
  ```text
  Oct 12 01:14:25 atacante-server fail2ban.actions[2850]: NOTICE [sshd] Ban 192.168.56.1
  ```
- **Evaluación**: Fail2ban identificó que la IP superó el límite de `maxretry = 3` y la agregó de inmediato a la cadena de descarte de `iptables`. El tráfico fue interrumpido a nivel de red sin requerir intervención manual.

---

## Evaluación de Impacto (Tríada CIA)

- **Confidencialidad**: **No comprometida**. No existió autenticación exitosa ni lectura de archivos o rutas protegidas.
- **Integridad**: **Preservada**. El módulo FIM de Wazuh confirmó que no se crearon ni modificaron archivos en el sistema.
- **Disponibilidad**: **Preservada**. Los servicios continuaron operando normalmente para el resto de la red.
- **Severidad Asignada**: **Baja** (intento hostil neutralizado automáticamente por los mecanismos de prevención del host).

---

## Acciones de Validación y Cierre

1. Se verificó con `sudo fail2ban-client status sshd` que la dirección IP atacante estuviese efectivamente registrada en la lista de baneo activa.
2. Se consultó el módulo FIM de Wazuh para ratificar la ausencia de alteraciones en el sistema de archivos (`/etc`).
3. Se corroboró que el acceso directo de `root` por SSH permanezca bloqueado en `/etc/ssh/sshd_config` (`PermitRootLogin no`).

**Resolución:** Ticket cerrado formalmente a nivel Tier 1 como actividad sospechosa mitigada por defensas locales. No requiere escalamiento.

---

## Documentos Relacionados

- Configuración de la defensa perimetral: [Laboratorio 01 - Fuerza Bruta SSH y Fail2ban](../../labs/01-fuerza-bruta-ssh/README.md).
- Siguiente caso: [Caso 02 - Acceso No Autorizado y Elevación de Privilegios](../caso-02-incidente-seguridad/README.md).
