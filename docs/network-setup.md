# Network Setup Guide

## Critical: Fixed IP for Raspberry Pi

Your Raspberry Pi **MUST** have a fixed IP address so that:
- Tasmota devices can reliably connect to MQTT
- UMH Core can consistently reach the services
- You don't lose connection when DHCP leases renew

### Method 1: Router DHCP Reservation (Recommended)

1. Log into your router admin panel
2. Find "DHCP Reservations" or "Static DHCP"
3. Add your Pi's MAC address with desired IP
4. Reboot your Pi

### Method 2: Static IP on Pi

Edit the network configuration:

```bash
sudo nano /etc/dhcpcd.conf
```

Add these lines (replace placeholders with your values):

```bash
# Static IP configuration
interface eth0
static ip_address=YOUR_PI_IP/24
static routers=YOUR_ROUTER_IP
static domain_name_servers=YOUR_ROUTER_IP 8.8.8.8

# If using WiFi instead:
interface wlan0
static ip_address=YOUR_PI_IP/24
static routers=YOUR_ROUTER_IP
static domain_name_servers=YOUR_ROUTER_IP 8.8.8.8
```

Reboot: `sudo reboot`

### Verify Fixed IP

```bash
ip addr show eth0  # or wlan0 for WiFi
ping google.com    # Test internet connectivity
```

## Complete Network Architecture

```
Router (YOUR_ROUTER_IP)
│
├── Raspberry Pi (YOUR_PI_IP) - FIXED IP REQUIRED
│   ├── Docker Stack:
│   │   ├── Mosquitto MQTT :1883
│   │   ├── InfluxDB :8086  
│   │   ├── Grafana :3000
│   │   └── Node-RED :1880 (available but unused)
│   └── Services accessible at YOUR_PI_IP
│
├── UMH Core System (YOUR_UMH_IP) - Separate computer/VM
│   ├── Subscribes to: tcp://YOUR_PI_IP:1883
│   ├── Internal Kafka processing
│   └── Writes to: http://YOUR_PI_IP:8086/api/v2/write
│
└── IoT Devices (Dynamic IPs OK)
    ├── Tasmota Socket (desk) → tcp://YOUR_PI_IP:1883
    ├── Tasmota Socket (siderack) → tcp://YOUR_PI_IP:1883
    └── Tasmota Socket (cabinet) → tcp://YOUR_PI_IP:1883
```

## Data Flow Network Path

1. **Tasmota Devices** → Publish to `tcp://YOUR_PI_IP:1883` 
2. **UMH Core** → Subscribes to `tcp://YOUR_PI_IP:1883`
3. **UMH Core** → Processes data internally via Kafka
4. **UMH Core** → Writes to `http://YOUR_PI_IP:8086/api/v2/write`
5. **Grafana** → Reads from `http://influxdb:8086` (internal Docker network)

## Port Access Verification

After deployment, test all services are accessible on your network:

```bash
# From another computer on your network (replace YOUR_PI_IP):
telnet YOUR_PI_IP 1883           # MQTT should connect
curl http://YOUR_PI_IP:3000      # Grafana should respond
curl http://YOUR_PI_IP:8086/ping # InfluxDB should respond
curl http://YOUR_PI_IP:1880      # Node-RED should respond
```

## Firewall Configuration

If you have connectivity issues, open the required ports on your Pi:

```bash
sudo ufw allow 1883  # MQTT
sudo ufw allow 3000  # Grafana  
sudo ufw allow 8086  # InfluxDB
sudo ufw allow 1880  # Node-RED
```

## Common Network Issues & Solutions

**Problem**: Services only work from Pi itself (localhost)
**Solution**: Docker containers binding incorrectly. Verify docker-compose.yml uses `"port:port"` format which binds to all network interfaces.

**Problem**: Tasmota devices can't connect to MQTT
**Solution**: 
- Verify Pi has fixed IP
- Check firewall allows port 1883
- Test MQTT connectivity: `mosquitto_pub -h YOUR_PI_IP -p 1883 -t test -m "hello"`

**Problem**: UMH Core can't reach Pi services  
**Solution**:
- Ensure both systems on same network subnet
- Verify Pi's fixed IP configuration
- Test connectivity from UMH system: `telnet YOUR_PI_IP 1883`

**Problem**: Data not flowing to InfluxDB
**Solution**:
- Check UMH Core configuration uses correct YOUR_PI_IP
- Verify InfluxDB token and bucket names match
- Check UMH Core logs for connection errors

## Placeholder Values for Your Setup

Replace these in all configuration files:

- **YOUR_PI_IP**: Your Raspberry Pi's fixed IP (e.g., 192.168.1.100)
- **YOUR_ROUTER_IP**: Your router's IP (usually 192.168.1.1)
- **YOUR_UMH_IP**: IP of system running UMH Core
- **YOUR_ORG**: Your organization name for InfluxDB
- **YOUR_SITE**: Your site/location name
- **YOUR_BUCKET**: Your InfluxDB bucket name
- **YOUR_INFLUX_TOKEN**: InfluxDB access token
- **YOUR_UNIQUE_CLIENT_ID**: Unique identifier for MQTT client

---

**Critical Success Factor**: Everything depends on the Raspberry Pi having a **reliable, fixed IP address**. Without this, devices lose connection and the whole system becomes unreliable. Set this up first, before anything else.