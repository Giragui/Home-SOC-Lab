# 🛡️ Home SOC Lab: Wazuh + Suricata

Este repositorio documenta la implementación de un entorno de monitoreo de seguridad híbrido.

## 🏗️ Arquitectura
* **Manager (Fedora Server):** Corre en una VM con Docker. Centraliza las alertas.
* **Sensor (Debian 13):** Hardware real (Core 2 Duo). Corre Suricata y el Agente de Wazuh.



---

## 🚀 Instalación del Manager (Fedora)

Para levantar el stack de Wazuh con Docker en el Fedora:

```bash

git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.2
cd wazuh-docker/single-node
docker-compose up -d

👁️ Instalación del Sensor (Debian)
1. Instalar Suricata
Bash

sudo apt update && sudo apt install suricata -y
sudo suricata-update

2. Vincular con Wazuh

Para que el agente mande los logs de Suricata al Fedora, editamos el ossec.conf en el Debian:
Bash

sudo nano /var/ossec/etc/ossec.conf

Y agregamos este bloque dentro de la sección <ossec_config>:
XML

<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>

🛠️ Reglas Personalizadas (Tuning)

Para evitar el ruido de las alertas de Telegram en el Dashboard, editamos las reglas locales en el Manager:
XML

<group name="local,syslog,sshd,">
  <rule id="100001" level="0">
    <if_sid>86601</if_sid>
    <description>Silenciar alertas de Telegram (OpenClaw)</description>
  </rule>
</group>

📝 Notas de Mantenimiento

    Permisos en Docker: Si editás archivos dentro del contenedor, recordá aplicar chown wazuh:wazuh para evitar errores de acceso.

    Optimización: Se desactivó el servicio de Bluetooth en el Debian para ahorrar recursos en el Core 2 Duo.


---

### ¿Por qué ahora se va a ver bien?
Fijate que cada comando está envuelto entre:
* **Arriba:** \` \` \` bash (o xml)
* **Abajo:** \` \` \`

Eso le dice a GitHub: "Che, esto es código, ponelo en un cuadrito y pintame los comandos de colores".

**Probá pegando este texto tal cual y dale a "Commit changes".** Vas a ver que la prolijidad cambia un 100%. ¿Te animás a probarlo?
