# Home Assistant Integration

Example configuration for integrating Home Assistant with the IoT Homelab stack.

## MQTT Configuration

Configure Home Assistant to publish to your IoT stack:

```yaml
# configuration.yaml
mqtt:
  broker: YOUR_PI_IP
  port: 1883
  discovery: true
  discovery_prefix: homeassistant
  
  # Optional: Use consistent topic structure
  birth_message:
    topic: 'homeassistant/status'
    payload: 'online'
  will_message:
    topic: 'homeassistant/status'  
    payload: 'offline'
```

## Device Topics

Home Assistant devices will publish to topics like:
```
homeassistant/sensor/YOUR_DEVICE/state
homeassistant/switch/YOUR_DEVICE/state
homeassistant/binary_sensor/YOUR_DEVICE/state
```

## UMH Integration

Configure UMH Core to subscribe to Home Assistant topics by adding to your `configs/umh/umh-config-example.yaml`:

```yaml
topics:
  - homeassistant/+/+/state
  - homeassistant/sensor/+/state
  # Add specific device patterns as needed
```

## Mix and Match

You can run both Home Assistant and this IoT stack simultaneously:
- Home Assistant for automation and UI
- This stack for industrial data processing patterns
- Both feeding the same MQTT broker for unified data flow

Replace `YOUR_PI_IP` with your actual Raspberry Pi IP address.