# Complete Setup Guide

Follow this guide to deploy the IoT Homelab with UMH Core integration.

## Prerequisites

- Raspberry Pi with Docker and Docker Compose
- Separate system for UMH Core deployment
- **Any MQTT devices**: Tasmota, ESP32 sensors, custom builds, existing MQTT producers, etc.
- Network access between all systems

**Note**: This is an example setup - adapt it to whatever IoT devices you have in your home.

## Quick Start

1. **Set Fixed IP**: Follow [Network Setup Guide](./network-setup.md) to configure fixed IP for your Raspberry Pi
2. **Deploy Stack**: Run `docker compose up -d` in this repository
3. **Install UMH**: Deploy UMH Core on separate system following [UMH documentation](https://www.umh.app/docs/getting-started/installation/)
4. **Configure Bridge**: Use configuration examples in `configs/umh/` to connect UMH to your stack

## Detailed Steps

### 1. Network Configuration

Your Raspberry Pi **MUST** have a fixed IP address. See [Network Setup Guide](./network-setup.md) for complete instructions.

### 2. Deploy IoT Stack

```bash
git clone https://github.com/vanenkhuizen/IoT-UMH-Homelab.git
cd IoT-UMH-Homelab
docker compose up -d
```

Services available at:
- Grafana: `http://YOUR_PI_IP:3000`
- InfluxDB: `http://YOUR_PI_IP:8086`
- MQTT: `tcp://YOUR_PI_IP:1883`
- Node-RED: `http://YOUR_PI_IP:1880`

**Management**: Use Docker CLI or install CasaOS for a beautiful web interface to manage containers and quick access to your services.

### 3. Configure UMH Core

Deploy UMH Core on a separate system, then configure it using the examples in `configs/umh/umh-config-example.yaml`.

**Networking Options:**
- **Option A (Recommended)**: Keep UMH Core on separate system using shared Docker network
- **Option B**: Merge UMH Core into the same Docker Compose stack for easier management

**For Option A (Separate Systems)**: Create a shared Docker network between both systems:
```bash
# On both systems, create the same network
docker network create umh-homelab-network

# Add this network to your docker-compose.yaml
networks:
  umh-homelab-network:
    external: true

# UMH Core containers connect to the same network
# Use container names instead of IP addresses:
# - MQTT: tcp://mosquitto:1883
# - InfluxDB: http://influxdb:8086
```

**For Option B (Same Stack)**: Add UMH Core services to your `docker-compose.yaml` and use internal Docker networking with container names.

Replace placeholders:
- `YOUR_ORG`: Organization name for InfluxDB
- `YOUR_SITE`: Site/location identifier
- `YOUR_INFLUX_TOKEN`: InfluxDB access token
- **For Option A**: Use container names (`mosquitto`, `influxdb`) instead of IP addresses
- **For Option B**: Use internal Docker networking with container names

### 4. Device Configuration

Configure your IoT devices to publish to `tcp://YOUR_PI_IP:1883` using consistent topic naming.

**Examples**:
- **Tasmota**: See `examples/tasmota/` for configuration
- **ESP32/Custom devices**: Use any MQTT client library to publish to the broker
- **Existing MQTT producers**: If you already have devices publishing MQTT, just point them to this broker

### 5. Node-RED (Optional)

Use Node-RED at `http://YOUR_PI_IP:1880` for:
- Additional data processing
- Home automation flows
- Custom device integrations
- Data transformation before UMH

## Verification

Test the complete data flow:
1. Devices publish MQTT data
2. UMH Core processes data through Kafka
3. Processed data appears in InfluxDB
4. Grafana displays visualizations

## Troubleshooting

See [Network Setup Guide](./network-setup.md) for common issues and solutions.