# Caso 02: Incidente de Seguridad con Impacto y Contención

## Identificación del Ticket

| Campo              | Detalle                                       |
| ------------------ | --------------------------------------------- |
| **ID de Ticket**   | SOC-2026-002                                  |
| **Tipo de Evento** | Incidente de seguridad (Acceso no autorizado) |
| **Severidad**      | Media                                         |
| **Estado**         | Cerrado (Contenido)                           |
| **Escalamiento**   | Requerido                                     |

---

## Resumen del Evento

Se detectó un inicio de sesión exitoso mediante SSH utilizando credenciales válidas de un usuario del sistema, originado desde una dirección IP no habitual. Posteriormente se identificó la ejecución inmediata de comandos con privilegios elevados utilizando `sudo`. El incidente requirió intervención manual defensiva para terminar la sesión, revocar credenciales, analizar el alcance de las acciones y asegurar el sistema.

---

## Detalle Técnico de la Investigación

### 1. Acceso Exitoso por SSH

- **Fuente de telemetría**: `/var/log/auth.log` (Servicio SSH).
- **Eventos observados**:
  - `Accepted password for <usuario> from <IP_externa> port <puerto> ssh2`.
  - Inicio de sesión interactivo exitoso.
- **Análisis**: El acceso se originó desde una dirección IP no catalogada dentro del segmento operativo del laboratorio. No se registraron intentos fallidos previos desde esa dirección, lo cual indica conocimiento previo de credenciales válidas o compromiso de las mismas fuera del host.

### 2. Uso de Privilegios Elevados

- **Fuente de telemetría**: `/var/log/auth.log` (sudo).
- **Eventos observados**:
  - Ejecución de comandos mediante `sudo` a los pocos segundos del inicio de sesión.
- **Análisis**: La cuenta vulnerada formaba parte del grupo `sudoers`. La invocación inmediata de privilegios administrativos representó un cambio anómalo de comportamiento y un riesgo crítico para el entorno.

### 3. Actividad Posterior al Acceso y Alcance

- **Fuente de telemetría**: `/var/log/auth.log` y `/var/log/syslog`.
- **Eventos observados**: Comandos de inspección administrativa y accesos a archivos de configuración del sistema.
- **Análisis y verificación**:
  - No se detectaron modificaciones persistentes en `/etc` (validado mediante el módulo FIM de Wazuh).
  - No se crearon nuevas cuentas de usuario ni se modificaron grupos del sistema.
  - No se identificó la instalación de paquetes o binarios sospechosos.

---

## Evaluación de Impacto

- **Confidencialidad**: En riesgo potencial por inspección de archivos del sistema por un actor no autorizado.
- **Integridad**: Preservada; no se confirmaron alteraciones en archivos ni cambios persistentes en la configuración.
- **Disponibilidad**: Los servicios continuaron en ejecución sin caída del sistema.
- **Evaluación global**: **Medio**.

---

## Clasificación del Evento

| Criterio         | Evaluación                           |
| ---------------- | ------------------------------------ |
| **Tipo**         | Acceso no autorizado                 |
| **Vector**       | Credenciales legítimas comprometidas |
| **Origen**       | IP no habitual / externa             |
| **Severidad**    | Media                                |
| **Estado**       | Contenido mediante respuesta manual  |
| **Escalamiento** | Requerido                            |

---

## Acciones de Contención y Respuesta Aplicadas

1. **Terminación de la sesión activa**: Finalización forzada inmediata de los procesos asociados a la sesión SSH del usuario comprometido.
2. **Revocación y rotación de credenciales**: Invalidación de la contraseña actual del usuario y bloqueo preventivo del acceso por contraseña.
3. **Auditoría de integridad**: Inspección de registros y validación de integridad de archivos críticos con Wazuh FIM para descartar puertas traseras.
4. **Escalamiento y reporte**: Notificación al administrador del entorno documentando el vector de entrada y el alcance mitigado.

---

## Recomendaciones Defensivas

- **Autenticación por clave pública**: Deshabilitar la autenticación SSH mediante contraseñas (`PasswordAuthentication no`) en favor de llaves criptográficas robustas.
- **Principio de menor privilegio**: Limitar la membresía en `sudoers` y auditar el uso de comandos administrativos (similar a lo documentado en [Laboratorio 04](../../labs/04-monitoreo-linux-windows/README.md)).
- **Monitoreo proactivo**: Configurar reglas en el SIEM para alertar sobre inicios de sesión exitosos desde direcciones IP fuera de los rangos autorizados.

---

## Conclusión

El incidente fue detectado oportunamente durante la fase inicial de reconocimiento. La contención manual rápida impidió la persistencia o modificación del sistema.

**Resolución:** Ticket cerrado tras la remediación de credenciales y auditoría de integridad.

---

## Referencias

- Para consultar el marco metodológico del análisis, ver [Metodología SOC](../../docs/metodologia/README.md).
- Caso anterior: [Caso 01 - Actividad Sospechosa Contenida](../caso-01-actividad-sospechosa/README.md).
