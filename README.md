# 🛡️ Home SOC Lab: Wazuh + Suricata + Fail2Ban

Guía de despliegue de un sistema de detección y respuesta ante intrusiones en un entorno híbrido.

## 🏗️ Arquitectura del Proyecto
* **Manager (Fedora Server):** Nodo centralizado en Docker que corre el stack de Wazuh.
* **Sensor (Debian 13):** Equipo físico (Intel Pentium B940) que actúa como IDS de red y host.



---

## 🚀 1. Configuración del Manager (Fedora)

Utilizamos Docker para el despliegue del Manager. El Dashboard está mapeado al puerto **8443** para el acceso web.

### Docker Compose (Resumen de puertos)
```yaml
# Fragmento del docker-compose.yml
services:
  wazuh.dashboard:
    image: wazuh/wazuh-dashboard:4.7.2
    ports:
      - "8443:5601"
    environment:
      - WAZUH_MANAGER_URL=[https://wazuh.manager](https://wazuh.manager)

Para levantar el stack:
Bash

docker-compose up -d

👁️ 2. Configuración del Sensor (Debian)

En el nodo Debian instalamos las herramientas de detección y prevención.
Instalación de Suricata y Fail2Ban
Bash

sudo apt update
sudo apt install suricata fail2ban -y
sudo suricata-update

Configuración de Fail2Ban

Para asegurar que Fail2Ban esté activo en servicios críticos como SSH:
Bash

sudo systemctl enable fail2ban
sudo systemctl start fail2ban

🔗 3. Integración con el Dashboard de Wazuh

Para que el nodo Fedora reciba los eventos del Debian, configuramos el agente de Wazuh localmente.
Monitoreo de Suricata y Fail2Ban

Editamos el archivo de configuración del agente:
Bash

sudo nano /var/ossec/etc/ossec.conf

Agregamos estas líneas dentro de <ossec_config> para que los logs lleguen al dashboard:
XML

<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/fail2ban.log</location>
</localfile>

🛠️ Reglas de Tuning Personalizadas

Para evitar el ruido de las alertas de la API de Telegram en el Dashboard, aplicamos esta regla en el Manager:
XML

<group name="local,syslog,sshd,">
  <rule id="100001" level="0">
    <if_sid>86601</if_sid>
    <description>Silenciar alertas de Telegram (OpenClaw)</description>
  </rule>
</group>

📝 Notas Técnicas

    Acceso: El dashboard es accesible en https://[IP-FEDORA]:8443.

    Rendimiento: Se optimizó el Debian desactivando servicios como bluetooth.service para priorizar el análisis de tráfico.
