🛡️ Hybrid SOC Laboratory: Wazuh + Suricata

Este repositorio contiene la configuración y documentación de un laboratorio de Seguridad (SOC) híbrido, integrado por un nodo de gestión en Fedora y un nodo de sensorización en Debian.
🏗️ Arquitectura del Sistema

El laboratorio utiliza un modelo de detección distribuida donde el tráfico es analizado en el borde y centralizado en un gestor para su visualización y respuesta.
Componentes:

    Manager (Cerebro): * OS: Fedora Linux (VirtualBox VM).

        Software: Wazuh Manager, Indexer y Dashboard corriendo en contenedores Docker.

        Función: Centralización de logs, motor de reglas y panel de control.

    Sensor (Vigilante):

        OS: Debian GNU/Linux 13 (Trixie) - Intel Pentium B940 / 8GB RAM.

        Software: Suricata (IDS), Wazuh Agent, Fail2Ban, Caddy.

        Función: Inspección profunda de paquetes (NIDS) y reporte de integridad de archivos.

🛠️ Configuraciones Destacadas
1. Integración de Logs

Para que Wazuh procese lo que ve Suricata, configuramos el agente en Debian para leer el archivo eve.json:
XML

<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>

2. Tuning de Reglas (Reducción de Ruido)

Un paso crítico fue silenciar las alertas repetitivas de la API de Telegram (utilizada por OpenClaw) que generaban cientos de eventos innecesarios. Se creó una regla personalizada en el Manager:
XML

<group name="local,syslog,sshd,">
  <rule id="100001" level="0">
    <if_sid>86601</if_sid>
    <description>Silenciar alertas de Telegram (OpenClaw)</description>
  </rule>
</group>

💡 Lecciones Aprendidas (Troubleshooting)
Gestión de Archivos en Contenedores Docker

Al editar reglas directamente en el contenedor del Manager, nos encontramos con errores de Permission Denied. La solución profesional fue:

    Extraer el archivo: docker cp manager:/path/file.xml .

    Editar localmente y devolverlo.

    Corregir permisos: Es vital devolver la propiedad al usuario interno de Wazuh:
    Bash

    chown wazuh:wazuh /var/ossec/etc/rules/local_rules.xml
    chmod 660 /var/ossec/etc/rules/local_rules.xml

🚀 Próximos Pasos

    [ ] Configurar Active Response para bloquear IPs automáticamente tras ataques de fuerza bruta en SSH/FTP.

    [ ] Implementar escaneos de vulnerabilidades (SCA) programados.

    [ ] Integrar alertas críticas hacia un bot de Telegram dedicado.
