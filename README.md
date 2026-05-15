# 🛡️ Home SOC Lab: Wazuh + Suricata + Fail2Ban

Guía técnica para el despliegue de un centro de operaciones de seguridad (SOC) híbrido.

---

## 🏗️ Arquitectura del Laboratorio

El sistema utiliza un modelo de gestión centralizada y sensores distribuidos:

* **Manager (Fedora Server):** Virtualizado en VirtualBox. Corre el stack de Wazuh en Docker.
* **Sensor (Debian 13):** Hardware real (Intel Pentium B940). Corre Suricata, Fail2Ban y el Agente de Wazuh.

---

## 🚀 1. Configuración del Manager (Fedora)

Desplegamos Wazuh usando contenedores. El acceso al Dashboard se configuró en el puerto **8443**.

### Docker Compose (Mapeo de Puertos)

```yaml
services:
  wazuh.dashboard:
    image: wazuh/wazuh-dashboard:4.7.2
    ports:
      - "8443:5601"

Para iniciar el servicio:
Bash

docker-compose up -d

👁️ 2. Configuración del Sensor (Debian)

En el nodo sensor instalamos las herramientas de detección de intrusos (IDS) y prevención.
Instalación de Herramientas
Bash

sudo apt update
sudo apt install suricata fail2ban -y
sudo suricata-update

Habilitar Fail2Ban
Bash

sudo systemctl enable --now fail2ban

🔗 3. Integración Agente -> Manager

Para que el Dashboard en Fedora visualice la actividad del Debian, configuramos el agente local del sensor.
Edición del archivo ossec.conf
Bash

sudo nano /var/ossec/etc/ossec.conf

Añadimos estos bloques para monitorear los logs de Suricata y Fail2Ban:
XML

<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/fail2ban.log</location>
</localfile>

🛠️ Tuning: Reglas Personalizadas

Para evitar el ruido visual de las alertas de la API de Telegram, aplicamos el siguiente filtro en el Manager:
XML

<group name="local,syslog,sshd,">
  <rule id="100001" level="0">
    <if_sid>86601</if_sid>
    <description>Silenciar alertas de Telegram (OpenClaw)</description>
  </rule>
</group>

📝 Notas de Optimización

    Ahorro de recursos: Se desactivó el servicio de Bluetooth (systemctl disable --now bluetooth) en el Debian para maximizar el rendimiento del Core 2 Duo.

    Seguridad: El acceso al Dashboard es vía HTTPS en https://[IP-FEDORA]:8443
