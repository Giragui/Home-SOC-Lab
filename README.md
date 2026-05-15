🛡️ Laboratorio SOC Híbrido: Wazuh + Suricata + Docker

Este proyecto documenta la implementación de un sistema de monitoreo de seguridad centralizado utilizando una arquitectura de Manager y Agentes distribuidos en una red local.
🏗️ Arquitectura del Laboratorio

El sistema se divide en dos nodos principales:

    Nodo de Gestión (Fedora Server en VirtualBox):

        Rol: Manager / SIEM.

        Despliegue: Wazuh centralizado mediante Docker Containers (Wazuh-Manager, Wazuh-Indexer, Wazuh-Dashboard).

        Función: Recibir, procesar y visualizar alertas de toda la red.

    Nodo Sensor (Debian 13 - Hardware Real):

        Rol: IDS / Agente de Seguridad.

        Servicios: Suricata (Detección de intrusiones en red), Wazuh Agent (HIDS), Fail2Ban (Prevención).

        Hardware: Intel Pentium B940 @ 2.00GHz con 8GB RAM.

🚀 Guía de Instalación
1. El Cerebro: Wazuh Manager (Fedora)

Se utilizó el despliegue de un solo nodo mediante contenedores para facilitar la portabilidad:
Bash

# Dentro de Fedora Server
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.2
cd wazuh-docker/single-node
docker-compose up -d

2. El Vigilante: Suricata (Debian)

Instalación y configuración del IDS para inspeccionar el tráfico de la interfaz de red:
Bash

sudo apt install suricata -y
# Configuración de la red en /etc/suricata/suricata.yaml
# Actualización de reglas
sudo suricata-update

3. El Vínculo: Integración Suricata -> Wazuh

Para que el "cerebro" vea lo que detecta el "vigilante", configuramos el agente de Wazuh en el Debian para que lea los logs JSON de Suricata:

Archivo: /var/ossec/etc/ossec.conf (en el Agente Debian)
XML

<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>

🛠️ Tuning y Personalización

Para reducir el ruido en el Dashboard, se implementaron reglas personalizadas en el Manager para silenciar falsos positivos o servicios conocidos (ej: APIs de mensajería como Telegram).
📈 Beneficios de esta Configuración

    Visibilidad Completa: Monitoreo de procesos, puertos abiertos e intentos de login (SSH/FTP).

    Detección de Red: Alertas en tiempo real sobre escaneos de puertos (Nmap) o tráfico malicioso gracias a Suricata.

    Bajo Consumo: Optimización de servicios en hardware antiguo (Core 2 Duo) desactivando servicios innecesarios (como bluetoothd).
