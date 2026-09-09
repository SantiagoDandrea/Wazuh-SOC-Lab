# Laboratorio 01: Análisis de Fuerza Bruta SSH y Respuesta con Fail2ban

## Objetivo

Analizar el comportamiento de intentos reiterados de autenticación fallida contra el servicio SSH en un sistema Linux, comprender el flujo de detección a nivel de registros del sistema y evaluar los mecanismos de bloqueo y respuesta automática mediante Fail2ban.

---

## Entorno

- **Endpoint víctima**: Máquina virtual con Ubuntu Server (Agente Linux).
- **Servicio analizado**: OpenSSH Server (`sshd`).
- **Mecanismo de respuesta**: Fail2ban configurado para monitorear `sshd` y bloquear direcciones IP atacantes tras exceder el umbral de intentos fallidos.
- **Plataforma SIEM**: Wazuh 4.7.5.
- Para consultar la infraestructura general de red y virtualización, ver [Arquitectura](../../docs/arquitectura/README.md).

---

## Procedimiento y Pruebas Realizadas

1. **Simulación de ataque**: Se ejecutaron múltiples intentos continuos de inicio de sesión con credenciales erróneas dirigidos contra el servicio SSH del sistema Linux.
2. **Inspección de logs del sistema**:
   - Monitoreo en tiempo real del archivo `/var/log/auth.log` observando los eventos `Failed password for invalid user` y `Failed password for root`.
3. **Validación de la respuesta con Fail2ban**:
   - Supervisión de `/var/log/fail2ban.log`.
   - Tras superarse el número máximo de reintentos configurados (`maxretry`), Fail2ban ejecutó una regla de iptables bloqueando la dirección IP de origen.
   - Confirmación del cese del tráfico y bloqueo efectivo de la conexión.

---

## Resultados y Eventos Analizados

- Los registros en `/var/log/auth.log` reflejaron de forma inmediata los intentos de autenticación fallidos con marcas temporales e IP de origen.
- Fail2ban identificó la recurrencia de los fallos y procedió al ban automático de la IP atacante sin requerir intervención manual.
- El servicio SSH permaneció operativo y accesible para IPs legítimas.

---

## Alcance y Limitaciones

- **Inspección a nivel de host**: La detección y el análisis de la fuerza bruta fueron validados directamente a nivel de los registros locales del sistema operativo y de Fail2ban.
- **Limitación en el SIEM**: Debido a limitaciones de arquitectura y recursos en el entorno de pruebas, no se visualizaron alertas de fuerza bruta SSH en el dashboard de Wazuh, por lo que no se recolectaron capturas de evidencias dentro de la interfaz del SIEM para este escenario específico.

---

## Referencias
- Para ver la aplicación de este escenario dentro de un ticket formal de investigación SOC, consultar el [Caso 01: Actividad Sospechosa Contenida](../../casos/caso-01-actividad-sospechosa/README.md).
- Siguiente laboratorio: [Laboratorio 02 - Monitoreo de Integridad de Archivos (FIM)](../02-monitoreo-integridad-archivos/README.md).
