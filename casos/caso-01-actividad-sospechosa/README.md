# Caso 01: Análisis de Actividad Sospechosa Contenida

## Identificación del Ticket

| Campo | Detalle |
|---|---|
| **ID de Ticket** | SOC-2026-001 |
| **Tipo de Evento** | Actividad sospechosa (Escaneo y fuerza bruta) |
| **Severidad** | Baja |
| **Estado** | Cerrado (Contenido) |
| **Escalamiento** | No requerido |

---

## Resumen del Evento

Se detectó actividad sospechosa proveniente de una dirección IP interna del laboratorio, consistente en solicitudes HTTP repetidas hacia rutas comunes del servidor web seguidas de múltiples intentos fallidos de autenticación SSH dirigidos contra la cuenta `root`.  
La actividad fue mitigada de forma automática mediante mecanismos locales de defensa en el host (Fail2ban), sin generar impacto operativo sobre la disponibilidad ni la integridad del sistema.

---

## Detalle Técnico de la Investigación

### 1. Actividad Web Sospechosa
- **Fuente de telemetría**: Servidor Apache (`access.log`).
- **Eventos observados**:
  ```text
  GET /admin HTTP/1.1 404
  GET /privado HTTP/1.1 401
  GET /privado/index.html HTTP/1.1 403
  ```
- **Análisis**: Solicitudes secuenciales dirigidas a endpoints sensibles con respuestas 401, 403 y 404. El patrón es característico de reconocimiento y escaneo automatizado en búsqueda de paneles administrativos. No se produjeron accesos exitosos a recursos protegidos.

### 2. Intentos de Acceso por SSH
- **Fuente de telemetría**: Registro de autenticación del sistema (`/var/log/auth.log`).
- **Eventos observados**: Múltiples intentos fallidos de autenticación con contraseña incorrecta contra el usuario `root` desde la misma IP que originó el escaneo web.
- **Análisis**: Uso del usuario `root` como vector de acceso directo, típico de herramientas automatizadas de fuerza bruta. Las políticas locales de hardening bloquearon el acceso directo de root y rechazaron las credenciales erróneas.

### 3. Respuesta Automática del Sistema
- **Fuente de telemetría**: `/var/log/fail2ban.log`.
- **Eventos observados**: La IP de origen superó el umbral de fallos permitido y fue inmediatamente agregada a la lista de bloqueo del firewall.
- **Análisis**: Fail2ban correlacionó los eventos repetitivos en `auth.log` y aplicó una regla de contención perimetral en el host de manera automática. No se requirió intervención manual para frenar la agresión.

---

## Evaluación de Impacto

- **Acceso exitoso**: No detectado.
- **Alteración de archivos**: No detectada (verificado con [Laboratorio 02 - FIM](../../labs/02-monitoreo-integridad-archivos/README.md)).
- **Escalada de privilegios**: No ocurrida.
- **Disponibilidad**: Sin interrupciones en los servicios expuestos.
- **Evaluación global**: **Bajo**.

---

## Clasificación del Evento

| Criterio | Evaluación |
|---|---|
| **Tipo** | Escaneo web y fuerza bruta SSH |
| **Origen** | IP interna (entorno de laboratorio) |
| **Severidad** | Baja |
| **Estado** | Contenido automáticamente |
| **Escalamiento** | No requerido |

---

## Acciones Tomadas y Contención

1. Bloqueo automático de la dirección IP atacante mediante Fail2ban en el host Linux.
2. Revisión analítica de los registros en `access.log`, `auth.log` y `fail2ban.log`.
3. Verificación de la operatividad del firewall local y de los servicios expuestos.

---

## Conclusión

Las capas de prevención, registro y contención automática del host funcionaron acorde a lo esperado. No existió compromiso de confidencialidad ni de integridad en el servidor.

**Resolución:** Ticket cerrado sin necesidad de escalamiento a niveles superiores.

---

## Referencias
- Para consultar el funcionamiento técnico del bloqueo de fuerza bruta, ver [Laboratorio 01 - Fuerza Bruta SSH](../../labs/01-fuerza-bruta-ssh/README.md).
- Siguiente caso: [Caso 02 - Incidente de Seguridad con Impacto](../caso-02-incidente-seguridad/README.md).
