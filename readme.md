# IoT Homelab with United Manufacturing Hub

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)
[![Docker](https://img.shields.io/badge/Docker-Required-blue.svg)](https://www.docker.com/)

> Educational project: Learn industrial IoT using United Manufacturing Hub architecture with real devices

## What This Is

My actual working IoT homelab setup using United Manufacturing Hub (UMH) for industrial-grade data processing. This demonstrates how to bridge consumer IoT devices (like Tasmota power sockets) into a proper industrial data architecture.

<img width="1472" height="819" alt="image" src="https://github.com/user-attachments/assets/d3c94faf-fa7b-4ac7-9fbc-71031b4ec951" />

### My Current Setup

- **3x Tasmota Smart Sockets** - Publishing power metrics via MQTT
- **Raspberry Pi 5** - Running the IoT stack
- **UMH Core** - Separate instance for data processing
- **Data Flow**: Devices → MQTT → UMH → InfluxDB → Grafana

## Architecture & Data Flow

```
[Tasmota Devices] --MQTT--> [Mosquitto:1883] <---> [UMH Core]
                                |                       |
                                v                       v
                         [Stock Docker Stack]    [Kafka Topics]
                           - InfluxDB                    |
                           - Grafana                     v
                           - Node-RED (unused)    [Data Processing]
                                                        |
                                                        v
                                                 [InfluxDB Writer]
```

### Key Insight
The magic happens in UMH Core - it subscribes to your MQTT topics and creates a **Unified Namespace** using Kafka, then processes and forwards data to your time-series database. Your IoT stack just provides the basic services, UMH adds the industrial intelligence.

## Complete Setup Guide

### Step 1: Set Fixed IP for Raspberry Pi

**CRITICAL**: Your Raspberry Pi needs a fixed IP so devices can reliably connect.

#### Option A: Router DHCP Reservation (Recommended)
1. Log into your router admin panel
2. Find "DHCP Reservations" or "Static DHCP" 
3. Add your Pi's MAC address with desired IP (e.g., `YOUR_PI_IP`)
4. Reboot your Pi

#### Option B: Static IP on Pi
```bash
sudo nano /etc/dhcpcd.conf

# Add these lines (adjust for your network):
interface eth0
static ip_address=YOUR_PI_IP/24
static routers=YOUR_ROUTER_IP
static domain_name_servers=YOUR_ROUTER_IP
```

Reboot: `sudo reboot`

### Step 2: Deploy the IoT Stack

```bash
git clone https://github.com/yourusername/iot-homelab-umh
cd iot-homelab-umh
docker compose up -d
```

Services will be available on your network:
- **Grafana**: http://YOUR_PI_IP:3000 
- **InfluxDB**: http://YOUR_PI_IP:8086
- **Node-RED**: http://YOUR_PI_IP:1880 (available but not used in this setup)
- **Mosquitto MQTT**: tcp://YOUR_PI_IP:1883

### Step 3: Set Up UMH Core (Separate System)

Deploy UMH Core on a separate system or VM following [UMH Installation Docs](https://www.umh.app/docs/getting-started/installation/).

**Important**: UMH Core and your IoT stack should be on the same network but different systems for proper separation.

### Step 4: Configure MQTT Data Producers (Your Devices)

Set up your Tasmota devices (or Shelly, ESPHome, etc.) to publish to your MQTT broker:

- **MQTT Server**: `YOUR_PI_IP:1883`
- **Topics**: Use consistent naming like `tele/YOUR_ORG/YOUR_SITE/tasmota/DEVICE_NAME/SENSOR`

Example Tasmota config:
```
SetOption19 1
MqttHost YOUR_PI_IP
MqttPort 1883
MqttTopic tele/YOUR_ORG/YOUR_SITE/tasmota/%topic%
```

### Step 5: Configure UMH Data Bridge - MQTT Input

In UMH Core, configure it to subscribe to your device topics:

```yaml
input:
  mqtt:
    client_id: YOUR_UNIQUE_CLIENT_ID
    topics:
      - tele/YOUR_ORG/YOUR_SITE/tasmota/desk/SENSOR
      - tele/YOUR_ORG/YOUR_SITE/tasmota/siderack/SENSOR
      - tele/YOUR_ORG/YOUR_SITE/tasmota/cabinet/SENSOR
    urls:
      - tcp://YOUR_PI_IP:1883
```

This creates your **Unified Namespace (UNS)** - you can now explore your topics. Read UMH docs for specifications or ask a good LLM for help.

### Step 6: Set Up InfluxDB

1. Access InfluxDB at `http://YOUR_PI_IP:8086`
2. Complete initial setup:
   - Create organization: `YOUR_ORG`
   - Create bucket: `YOUR_BUCKET`
   - Generate access token: `YOUR_INFLUX_TOKEN`

### Step 7: Configure UMH Data Bridge - InfluxDB Output

Configure UMH Core to write processed data to InfluxDB:

```yaml
input:
  kafka:
    addresses:
      - localhost:9092
    client_id: tasmota_influx_benthos
    consumer_group: influxdb_consumer
    topics:
      - umh.messages

pipeline:
  processors:
    - bloblang: |-
        root = if metadata("kafka_key").contains("umh.v1.YOUR_ORG.YOUR_SITE") && metadata("kafka_key").contains("_historian") {
          "tasmota_sensors,topic=" + metadata("kafka_key") + " value=" + this.value.string() + " " + this.timestamp_ms.string()
        } else {
          deleted()
        }

output:
  http_client:
    batching:
      count: 1
    headers:
      Authorization: Token YOUR_INFLUX_TOKEN
      Content-Type: text/plain
    max_in_flight: 1
    retry_period: 1s
    timeout: 5s
    url: http://YOUR_PI_IP:8086/api/v2/write?org=YOUR_ORG&bucket=YOUR_BUCKET&precision=ms
    verb: POST
```

Data will now flow: **Devices → MQTT → UMH → Kafka → Processing → InfluxDB**

### Step 8: Set Up Grafana

1. Access Grafana at `http://YOUR_PI_IP:3000`
2. Default login: `admin/admin` (change immediately)
3. Add InfluxDB as data source:
   - URL: `http://influxdb:8086`
   - Organization: `YOUR_ORG`
   - Token: `YOUR_INFLUX_TOKEN`
   - Default Bucket: `YOUR_BUCKET`
4. **VERY IMPORTANT**: Use latest Grafana version for optimal Flux query experience (even though in beta)
5. Create dashboards for your sensor data

### Step 9: Bring It All Together

Now you have a complete industrial IoT system:

1. **✅ Stack deployed** - Core services running
2. **✅ UMH Core running** - Industrial data processing layer
3. **✅ Network configured** - Fixed IP, all services accessible
4. **✅ Devices publishing** - MQTT data flowing
5. **✅ UNS active** - Data bridge to UMH creating unified namespace
6. **✅ Data processing** - UMH transforming and routing data
7. **✅ Storage configured** - InfluxDB receiving processed data
8. **✅ Visualization ready** - Grafana connected with Flux queries

## What You Learn

- **Unified Namespace Architecture** - How industrial IoT organizes data
- **MQTT Pub/Sub Patterns** - Device communication protocols  
- **Industrial Data Processing** - UMH's Kafka-based data transformation
- **Time-Series Analytics** - Store and query IoT data efficiently with InfluxDB
- **Modern Visualization** - Flux queries in Grafana for advanced analytics
- **System Integration** - Bridging consumer IoT into industrial architecture

## Network Architecture

```
Router (YOUR_ROUTER_IP)
│
├── Raspberry Pi (YOUR_PI_IP) - FIXED IP
│   ├── Mosquitto MQTT :1883
│   ├── InfluxDB :8086
│   ├── Grafana :3000
│   └── Node-RED :1880 (unused)
│
├── UMH Core System (YOUR_UMH_IP) - Separate computer/VM
│   ├── Subscribes to: tcp://YOUR_PI_IP:1883
│   ├── Processes via Kafka
│   └── Writes to: http://YOUR_PI_IP:8086
│
└── IoT Devices (Dynamic IPs OK)
    ├── Tasmota Socket 1 → tcp://YOUR_PI_IP:1883
    ├── Tasmota Socket 2 → tcp://YOUR_PI_IP:1883
    └── Tasmota Socket 3 → tcp://YOUR_PI_IP:1883
```

## Key Benefits of This Architecture

- **Industrial Grade**: Based on manufacturing standards (UMH)
- **Scalable**: Add devices without infrastructure changes
- **Unified Namespace**: All data follows consistent patterns
- **Real-time Processing**: Kafka-based data transformation
- **Future-Proof**: Built on industry-standard protocols
- **Educational**: Learn actual industrial IoT concepts

## Project Structure

```
iot-homelab-umh/
├── docker-compose.yml          # Your exact working stack
├── configs/
│   └── umh/                   # UMH configuration examples
├── docs/
│   ├── setup-guide.md         # This complete guide
│   └── networking.md          # Network configuration details
├── examples/
│   ├── tasmota/               # Device configuration examples
│   └── grafana/               # Dashboard examples
└── README.md                  # This file
```

## Important Notes

- **Stock Configuration**: This uses stock Docker images with no custom configs - simple and reliable
- **UMH Separation**: UMH Core runs separately - this provides proper industrial architecture
- **Network Design**: Fixed IP is critical - everything connects via network addresses
- **Latest Software**: Use latest Grafana for best Flux query support
- **Industrial Concepts**: This isn't just dashboards - it's learning real manufacturing data architecture

## Contributing

This is an educational project showing real industrial IoT concepts. Contributions welcome:
- Additional device integrations
- Better UMH configuration examples
- Real-world use case documentation
- Dashboard templates

## License

MIT License - Use it, learn from it, improve it.

---

**The Big Picture**: This setup transforms simple consumer IoT devices (Tasmota sockets) into an industrial-grade data system using United Manufacturing Hub's proven architecture. You're not just monitoring power consumption - you're learning the data patterns used in actual smart factories.
