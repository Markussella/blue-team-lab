# blue-team-lab
[EN] SOC Automation Lab — Real-time threat detection with Suricata IDS, SIEM with Elasticsearch/Kibana, and automated incident response using n8n SOAR. Includes IP blocking, email &amp; Telegram alerting. [ES] Laboratorio SOC — Detección de amenazas con Suricata IDS, SIEM con Elasticsearch/Kibana y respuesta automática a incidentes con n8n SOAR.
## 📋 Table of Contents / Índice
- [Overview / Descripción](#-overview--descripción)
- [Architecture / Arquitectura](#-architecture--arquitectura)
- [Components / Componentes](#-components--componentes)
- [Demo Flow / Flujo Demo](#-demo-flow--flujo-demo)
- [Setup / Instalación](#-setup--instalación)
- [Results / Resultados](#-results--resultados)
- [Skills Demonstrated / Habilidades](#-skills-demonstrated--habilidades-demostradas)

---

## 🔍 Overview / Descripción

**[EN]**  
A fully functional SOC automation lab built on a Kali Linux VM. The system monitors network traffic in real-time using Suricata IDS (49,000+ rules), indexes events into Elasticsearch via Filebeat, and triggers automated incident response workflows through n8n — including IP blocking, incident logging, email alerts, and Telegram notifications.

**[ES]**  
Laboratorio SOC funcional desplegado en una VM Kali Linux. El sistema monitoriza el tráfico de red en tiempo real con Suricata IDS (+49,000 reglas), indexa eventos en Elasticsearch mediante Filebeat, y dispara flujos de respuesta automática a incidentes a través de n8n — incluyendo bloqueo de IPs, registro de incidentes, alertas por email y notificaciones por Telegram.

---

## 🏗️ Architecture / Arquitectura

```
┌─────────────────────────────────────────────────────────────────┐
│                        KALI LINUX VM                            │
│                                                                 │
│  Network Traffic                                                │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────┐    eve.json    ┌──────────┐    ┌───────────────┐  │
│  │Suricata │ ─────────────► │ Filebeat │───►│Elasticsearch  │  │
│  │  IDS    │                │          │    │  + Kibana     │  │
│  │49K rules│                └──────────┘    └───────┬───────┘  │
│  └─────────┘                                        │          │
│                                          ┌──────────▼────────┐ │
│                                          │  Suricata Watcher │ │
│                                          │   (Python script) │ │
│                                          └──────────┬────────┘ │
│                                                     │          │
│  ┌──────────────────┐    ngrok tunnel               │          │
│  │  Firewall Mock   │◄──────────────────────────────┤          │
│  │  (Flask :5050)   │                               │          │
│  └──────────────────┘                               │          │
└─────────────────────────────────────────────────────┼──────────┘
                                                      │
                                                      ▼
                                          ┌───────────────────┐
                                          │       n8n         │
                                          │  (Render Cloud)   │
                                          └─────────┬─────────┘
                                                    │
                          ┌─────────────────────────┼──────────────────┐
                          │                         │                  │
                          ▼                         ▼                  ▼
                   ┌─────────────┐        ┌──────────────┐    ┌──────────────┐
                   │ Block IP on │        │  Log to      │    │  Notify via  │
                   │  Firewall   │        │Elasticsearch │    │Email+Telegram│
                   └─────────────┘        └──────────────┘    └──────────────┘
```

---

## ⚙️ Components / Componentes

| Component | Role / Función | Technology |
|-----------|----------------|------------|
| **Suricata IDS** | Network threat detection / Detección de amenazas | Suricata 8.0.3 + ET Open Rules |
| **Filebeat** | Log shipping / Envío de logs | Filebeat 8.19 |
| **Elasticsearch** | Event storage / Almacenamiento de eventos | Elasticsearch 8.x |
| **Kibana** | Visualization / Visualización | Kibana 8.x |
| **Suricata Watcher** | Alert polling / Consulta de alertas | Python 3 |
| **n8n** | SOAR orchestration / Orquestación SOAR | n8n (Render) |
| **Firewall Mock** | IP blocking simulation / Simulación de firewall | Flask (Python) |
| **ngrok** | Tunnel / Túnel | ngrok |
| **Mailtrap** | Email alerting / Alertas email | SMTP |
| **Telegram Bot** | Instant notifications / Notificaciones | Telegram API |

---

## 🔄 Demo Flow / Flujo Demo

### Automated Response Pipeline / Pipeline de Respuesta Automática

```
1. Suricata detects threat on eth1
   └─► Writes alert to /var/log/suricata/eve.json

2. Filebeat indexes alert
   └─► Pushes to Elasticsearch (.ds-filebeat-*)

3. Suricata Watcher (polls every 60s)
   └─► Queries Elasticsearch for new alerts
   └─► POSTs to n8n webhook

4. n8n Workflow — "SOAR Suricata"
   ├─► Normalize Alert (extract fields)
   ├─► [IF severity = high/critical]
   │   ├─► Block IP on Firewall Mock
   │   ├─► Log incident to soar-incidents index
   │   ├─► Send email notification (Mailtrap)
   │   └─► Send Telegram alert
   └─► [ELSE] Log to soar-low-severity index
```

### n8n Workflow / Flujo n8n

![n8n Workflow](#) <!-- Add screenshot -->

### Telegram Alert Example / Ejemplo Alerta Telegram

```
🚨 Alerta SOAR
Regla: GPL ATTACK_RESPONSE id check returned root
Severidad: high
IP Origen: 52.222.132.64
IP Destino: 192.168.1.134
Timestamp: 2026-03-12T12:05:37Z
```

---

## 🚀 Setup / Instalación

### Prerequisites / Requisitos Previos
- Kali Linux VM (VirtualBox)
- Python 3.x
- Suricata 8.x
- Elasticsearch + Kibana 8.x
- Filebeat 8.x
- ngrok account (free tier)
- n8n account (Render free tier)
- Telegram Bot (via @BotFather)

### Quick Start / Inicio Rápido

```bash
# 1. Start Firewall Mock
cd ~/soar-lab && python3 firewall_api_mock.py

# 2. Start ngrok tunnel
ngrok http 5050

# 3. Verify core services
sudo systemctl status elasticsearch kibana filebeat suricata

# 4. Start Suricata Watcher
python3 ~/soar-lab/suricata_watcher.py
```

### Suricata Rules Update / Actualización de Reglas

```bash
sudo suricata-update
sudo systemctl restart suricata
sudo suricatasc -c "ruleset-stats"
# Expected: rules_loaded: 49023
```

---

## 📊 Results / Resultados

| Metric | Value |
|--------|-------|
| Suricata rules loaded | 49,023 |
| Alerts indexed in Elasticsearch | 622+ |
| Incidents auto-logged (high severity) | ✅ |
| IPs auto-blocked | ✅ |
| Email notifications | ✅ |
| Telegram notifications | ✅ |
| End-to-end response time | < 60s |

### Elasticsearch Indices / Índices Elasticsearch

```bash
soar-incidents        # High severity incidents
soar-low-severity     # Low severity events  
.ds-filebeat-*        # Raw Suricata events (84,000+ docs)
```

---

## 🎯 Skills Demonstrated / Habilidades Demostradas

**[EN]**
- Network intrusion detection (IDS) configuration and tuning
- SIEM implementation with Elasticsearch + Kibana
- SOAR workflow design and automation with n8n
- Log ingestion pipeline (Filebeat → Elasticsearch)
- REST API integration (Firewall, Telegram, SMTP)
- Python scripting for security automation
- Incident response process design
- Network traffic analysis

**[ES]**
- Configuración y ajuste de IDS (Detección de Intrusiones)
- Implementación de SIEM con Elasticsearch + Kibana
- Diseño y automatización de flujos SOAR con n8n
- Pipeline de ingesta de logs (Filebeat → Elasticsearch)
- Integración de APIs REST (Firewall, Telegram, SMTP)
- Scripting Python para automatización de seguridad
- Diseño de procesos de respuesta a incidentes
- Análisis de tráfico de red

---

## 📁 Repository Structure / Estructura del Repositorio

```
soar-lab/
├── firewall_api_mock.py    # Flask mock firewall API
├── suricata_watcher.py     # Elasticsearch alert poller
├── screenshots/            # Portfolio screenshots
│   ├── n8n-workflow.png
│   ├── kibana-alerts.png
│   ├── telegram-alert.png
│   └── elastic-incidents.png
└── README.md
```

---

## 👤 Author / Autor

**Marcos** — [@Markussella](https://github.com/Markussella)

> 🔗 Connect on LinkedIn | ⭐ Star this repo if you found it useful!

---

*Built as a hands-on cybersecurity portfolio project — Blue Team / SOC Analyst skills demonstration.*
