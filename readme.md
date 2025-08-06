# 🏭 IoT Homelab with United Manufacturing Hub

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED.svg)](#)
[![UMH](https://img.shields.io/badge/UMH-Core-orange.svg)](#)
[![MQTT](https://img.shields.io/badge/MQTT-Mosquitto-purple.svg)](#)
[![InfluxDB](https://img.shields.io/badge/InfluxDB-v2.7-22ADF6.svg)](#)
[![Grafana](https://img.shields.io/badge/Grafana-Latest-F46800.svg)](#)

> Educational IoT project demonstrating industrial-grade data processing with United Manufacturing Hub

<img width="1472" height="819" alt="image" src="https://github.com/user-attachments/assets/d3c94faf-fa7b-4ac7-9fbc-71031b4ec951" />

---

## 🧠 Problem

Most IoT homelabs use basic dashboards. This project applies United Manufacturing Hub patterns to home devices - the same industrial data architecture used in factories.

**My homelab example** showing MQTT device integration with industrial-grade data processing.

---

## 💡 Solution

Docker Compose stack with:
- **UMH Core** - Kafka processing and Unified Namespace
- **MQTT Broker** - Device communication (Mosquitto)
- **Time-Series DB** - InfluxDB for metrics storage
- **Visualization** - Grafana dashboards
- **Processing** - Node-RED for automation

**vs UMH Classic**: Simpler Docker Compose + InfluxDB instead of Kubernetes + TimescaleDB

---

## 🛠 How to Use / Deploy

**Requirements:**
- Raspberry Pi with Docker Compose
- Separate system for UMH Core
- MQTT devices (Tasmota, ESP32, etc.)
- Network connectivity

### 📦 Installation

```bash
git clone https://github.com/SheetMetalConnect/IoT-UMH-Homelab.git
cd IoT-UMH-Homelab
docker compose up -d
```

**Advanced Users**: If you already have Docker and understand the stack, just run `docker compose up -d` and configure UMH Core separately. Don't forget to network the stack to UMH-core.

**Setup Steps:**
1. **Fixed IP**: Configure Pi networking - [docs/network-setup.md](docs/network-setup.md)
2. **UMH Core**: Deploy on separate system - [UMH docs](https://www.umh.app/docs/getting-started/installation/)
3. **Bridge Config**: Use examples in `configs/umh/umh-config-example.yaml`
4. **Devices**: Connect MQTT devices - see [docs/](docs/) 

**Services:**
- Grafana: `http://YOUR_PI_IP:3000`
- InfluxDB: `http://YOUR_PI_IP:8086`
- MQTT: `tcp://YOUR_PI_IP:1883`
- Node-RED: `http://YOUR_PI_IP:1880`

**Management**: Docker CLI or CasaOS for web UI

See [docs/setup-guide.md](docs/setup-guide.md) for complete instructions.

---

## 📁 Project Structure

```
IoT-UMH-Homelab/
├── docker-compose.yaml    # Main stack
├── docs/                  # Documentation
├── configs/umh/           # UMH configuration  
├── examples/grafana/      # Dashboard templates
└── README.md
```

---

## 🏗️ Architecture

```
[Your MQTT Devices] --MQTT--> [Mosquitto:1883] <---> [UMH Core]
(Tasmota, ESP32 sensors,          |                       |
 custom MQTT producers)           v                       v
                           [Docker Services]       [Kafka Processing]
                           - InfluxDB                    |
                           - Grafana                     |
                           - Node-RED                    v
                                                 [InfluxDB Writer]
```

**Flow**: Devices → MQTT → UMH → Kafka → InfluxDB → Grafana

UMH Core creates Unified Namespace and processes data through Kafka. Simple home devices get industrial-grade analytics.

---

## ⚠️ Important Notes

- **Educational**: My homelab example - adapt to your setup
- **No Guarantees**: Experimental, use at own risk  
- **Any MQTT Devices**: Tasmota, ESP32, custom builds
- **Fixed IP Required**: For reliable operation

---

## Contact

Questions? Open an issue or reach me at luke@vanenkhuizen.com

## License

MIT License - see LICENSE file.