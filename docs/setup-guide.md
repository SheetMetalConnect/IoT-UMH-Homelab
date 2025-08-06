# Complete Setup Guide

Follow this guide to deploy the IoT Homelab with UMH Core integration.

## Prerequisites

- Raspberry Pi with Docker and Docker Compose
- Separate system for UMH Core deployment
- Tasmota devices or other MQTT-capable IoT devices
- Network access between all systems

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

### 3. Configure UMH Core

Deploy UMH Core on a separate system, then configure it using the examples in `configs/umh/umh-config-example.yaml`.

Replace placeholders:
- `YOUR_PI_IP`: Your Raspberry Pi's fixed IP
- `YOUR_ORG`: Organization name for InfluxDB
- `YOUR_SITE`: Site/location identifier
- `YOUR_INFLUX_TOKEN`: InfluxDB access token

### 4. Device Configuration

Configure your IoT devices to publish to `tcp://YOUR_PI_IP:1883` using consistent topic naming.

Example Tasmota configuration:
```
SetOption19 1
MqttHost YOUR_PI_IP
MqttPort 1883
MqttTopic tele/YOUR_ORG/YOUR_SITE/tasmota/%topic%
```

## Verification

Test the complete data flow:
1. Devices publish MQTT data
2. UMH Core processes data through Kafka
3. Processed data appears in InfluxDB
4. Grafana displays visualizations

## Troubleshooting

See [Network Setup Guide](./network-setup.md) for common issues and solutions.