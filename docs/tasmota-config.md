# Tasmota Device Configuration

Example configuration for Tasmota devices (smart sockets, sensors, etc.) to work with the IoT Homelab setup.

This is one example - adapt the concepts to whatever MQTT devices you have in your home.

## Basic MQTT Configuration

Configure your Tasmota device to connect to your IoT stack:

```
SetOption19 1
MqttHost YOUR_PI_IP
MqttPort 1883
MqttTopic tele/YOUR_ORG/YOUR_SITE/tasmota/%topic%
MqttFullTopic tele/YOUR_ORG/YOUR_SITE/tasmota/%prefix%/%topic%
```

## Device Naming

Use consistent device names for organization:
- `desk` - Desk power socket
- `siderack` - Side rack equipment
- `cabinet` - Cabinet power monitoring

## Topic Structure

Tasmota will publish to topics like:
```
tele/YOUR_ORG/YOUR_SITE/tasmota/desk/SENSOR
tele/YOUR_ORG/YOUR_SITE/tasmota/siderack/SENSOR
tele/YOUR_ORG/YOUR_SITE/tasmota/cabinet/SENSOR
```

## Sample Payload

Power monitoring data structure:
```json
{
  "Time": "2024-01-15T10:30:00",
  "ENERGY": {
    "TotalStartTime": "2024-01-01T00:00:00",
    "Total": 12.345,
    "Yesterday": 1.234,
    "Today": 0.567,
    "Power": 89,
    "ApparentPower": 92,
    "ReactivePower": 12,
    "Factor": 0.97,
    "Voltage": 230,
    "Current": 0.387
  }
}
```

Replace placeholders with your actual values before applying configuration.