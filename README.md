# Home Assistant Docker Stack

A complete Docker-based Home Assistant setup with MariaDB and Mosquitto MQTT broker.

## Overview

This repository contains a production-ready Home Assistant stack running on Docker with:
- **Home Assistant Core**: Open-source home automation platform
- **MariaDB**: Reliable database for storing Home Assistant data
- **Mosquitto**: Lightweight MQTT broker for IoT device communication
- **Frigate**: GPU-accelerated NVR with AI object detection
- **Pi-hole**: Network-wide ad blocking and DNS management
- **Node-RED**: Visual automation and workflow tool

## Features

- Docker Compose orchestration for easy deployment
- MariaDB database integration for improved performance
- MQTT broker for IoT device communication
- GPU-accelerated video surveillance with Frigate
- Network-wide ad blocking with Pi-hole
- Visual automation workflows with Node-RED
- Cloudflare proxy configuration included
- Environment variable based configuration
- Persistent data storage
- Auto-restart on failure

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- Git
- NVIDIA GPU (optional, for Frigate GPU acceleration)
- NVIDIA Container Toolkit (required if using Frigate with GPU)

## Quick Start

### 1. Clone the Repository

```bash
git clone git@github.com:sudobigboss/home-assistant-stack.git
cd home-assistant-stack
```

### 2. Configure Environment Variables

Copy the example environment file and configure your settings:

```bash
cp .env.example .env
```

Edit `.env` and set secure passwords:

```env
TIMEZONE=America/New_York
MYSQL_ROOT_PASSWORD=your_secure_root_password_here
MYSQL_PASSWORD=your_secure_password_here
MYSQL_DATABASE=homeassistant
MYSQL_USER=homeassistant
```

### 3. Update Home Assistant Configuration

Edit `homeassistant/config/configuration.yaml` and replace `YOUR_PASSWORD` with your actual database password:

```yaml
recorder:
  db_url: mysql://homeassistant:YOUR_PASSWORD@homeassistant-mariadb:3306/homeassistant?charset=utf8mb4
```

### 4. Start the Stack

```bash
docker-compose up -d
```

### 5. Access Home Assistant

Open your browser and navigate to:
```
http://localhost:8123
```

Follow the onboarding process to create your first user account.

## Service Details

### Home Assistant
- **Container Name**: homeassistant
- **Port**: 8123 (host network mode)
- **Volume**: `./homeassistant/config`

### MariaDB
- **Container Name**: homeassistant-mariadb
- **Port**: 3306
- **Volume**: `./mariadb/data`
- **Database**: homeassistant

### Mosquitto MQTT
- **Container Name**: homeassistant-mosquitto
- **Ports**: 1883 (MQTT), 9001 (WebSocket)
- **Volumes**: `./mosquitto/config`, `./mosquitto/data`, `./mosquitto/log`

### Frigate NVR
- **Container Name**: frigate
- **Port**: 5000 (Web UI), 8554 (RTSP), 8555 (WebRTC)
- **Volumes**: `./frigate/config`, `./frigate/storage`
- **GPU**: NVIDIA GPU acceleration enabled
- **Features**: AI object detection, person/vehicle tracking, 24/7 recording

## Directory Structure

```
home-assistant-stack/
├── docker-compose.yml       # Docker Compose configuration
├── .env                     # Environment variables (not in git)
├── .env.example            # Example environment variables
├── homeassistant/
│   └── config/             # Home Assistant configuration
│       ├── configuration.yaml
│       ├── automations.yaml
│       ├── scripts.yaml
│       └── scenes.yaml
├── mariadb/
│   └── data/               # MariaDB data directory
├── mosquitto/
│   ├── config/             # Mosquitto configuration
│   │   └── mosquitto.conf
│   ├── data/               # Mosquitto data
│   └── log/                # Mosquitto logs
├── frigate/
│   ├── config/             # Frigate configuration
│   │   └── config.yml
│   └── storage/            # Recordings and snapshots
├── pihole/
│   ├── etc-pihole/         # Pi-hole configuration
│   └── etc-dnsmasq.d/      # DNS configuration
└── nodered/
    └── data/               # Node-RED flows and data
```

## Configuration

### Timezone

Update the `TIMEZONE` variable in your `.env` file to match your location:

```env
TIMEZONE=America/New_York
```

### MQTT Configuration

The default Mosquitto configuration allows anonymous connections. To add authentication:

1. Create a password file:
```bash
docker exec -it homeassistant-mosquitto mosquitto_passwd -c /mosquitto/config/passwd username
```

2. Update `mosquitto/config/mosquitto.conf`:
```conf
allow_anonymous false
password_file /mosquitto/config/passwd
```

3. Restart the Mosquitto container:
```bash
docker-compose restart mosquitto
```

### Database Backup

To backup your MariaDB database:

```bash
docker exec homeassistant-mariadb mysqldump -u root -p homeassistant > backup.sql
```

To restore:

```bash
docker exec -i homeassistant-mariadb mysql -u root -p homeassistant < backup.sql
```

## Management Commands

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f homeassistant
docker-compose logs -f mariadb
docker-compose logs -f mosquitto
```

### Restart Services

```bash
# All services
docker-compose restart

# Specific service
docker-compose restart homeassistant
```

### Stop Services

```bash
docker-compose down
```

### Update Containers

```bash
docker-compose pull
docker-compose up -d
```

## Security Considerations

- Never commit `.env` file to version control
- Use strong, unique passwords for database credentials
- Keep your containers updated regularly
- Consider using secrets management for production deployments
- Enable MQTT authentication in production
- Use HTTPS with proper SSL certificates for external access

## Troubleshooting

### Home Assistant won't start
- Check logs: `docker-compose logs homeassistant`
- Verify configuration.yaml syntax
- Ensure database password is correct in both `.env` and `configuration.yaml`

### Database connection errors
- Verify MariaDB is running: `docker-compose ps`
- Check database credentials in `.env`
- Ensure the database password matches in `configuration.yaml`

### MQTT connection issues
- Verify Mosquitto is running: `docker-compose ps`
- Check Mosquitto logs: `docker-compose logs mosquitto`
- Test connection: `mosquitto_pub -h localhost -t test -m "hello"`

## Frigate Setup Guide

Frigate is configured to use your NVIDIA GPU for AI object detection. Follow these steps to set it up:

### 1. Configure Cameras

Edit `frigate/config/config.yml` and configure your camera streams:

```yaml
cameras:
  front_door:
    enabled: true  # Set to true when ready
    ffmpeg:
      inputs:
        - path: rtsp://username:password@camera_ip:554/stream_path
          roles:
            - detect
            - record
```

**Finding your camera RTSP URL:**
- Check your camera manufacturer's documentation
- Common paths: `/stream1`, `/cam/realmonitor`, `/h264`, `/live/ch00_0`
- Tools like VLC or ONVIF Device Manager can help discover streams

### 2. Set RTSP Password

Update your `.env` file:

```env
FRIGATE_RTSP_PASSWORD=your_secure_password
```

This password is used for re-streaming cameras from Frigate.

### 3. GPU Configuration

Frigate is pre-configured to use TensorRT with your NVIDIA RTX 3080. On first run, Frigate will:
- Download the YOLOv7 model
- Compile it for TensorRT (this takes 5-10 minutes)
- Cache the compiled model for future use

**Monitor GPU usage:**
```bash
nvidia-smi -l 1
```

### 4. Adjust Detection Settings

Key settings to tune in `config.yml`:

```yaml
detect:
  fps: 5  # Lower = less CPU/GPU usage
  width: 1280
  height: 720

objects:
  track:
    - person  # Add objects you want to detect
    - car
    - dog
```

### 5. Configure Zones

Define detection zones to reduce false positives:

```yaml
zones:
  front_yard:
    coordinates: x1,y1,x2,y2,x3,y3,x4,y4  # Four corner points
    objects:
      - person
```

**Getting zone coordinates:**
- Start Frigate: `docker-compose up -d frigate`
- Access UI: `http://localhost:5000`
- Use the built-in zone editor in Settings > Camera > Zones

### 6. Storage Management

Recordings are stored in `./frigate/storage/`. Configure retention:

```yaml
record:
  retain:
    days: 7  # Keep all recordings for 7 days

snapshots:
  retain:
    default: 30  # Keep snapshots for 30 days
```

**Disk usage:**
- Estimate: ~1-2GB per camera per day (depends on motion)
- Monitor: `du -sh frigate/storage/`

### 7. Home Assistant Integration

Once Frigate is running, add the integration in Home Assistant:

**Option A: UI Integration (Recommended)**
1. Open Home Assistant at `http://localhost:8123`
2. Go to **Settings → Devices & Services**
3. Click **+ Add Integration** (bottom right)
4. Search for **"Frigate"**
5. Enter URL: `http://frigate:5000`
6. Click Submit

**Option B: HACS (for latest features)**
If you have HACS installed:
1. Go to HACS → Integrations
2. Search for "Frigate"
3. Install the Frigate integration
4. Restart Home Assistant
5. Add integration via Settings → Devices & Services
6. Enter URL: `http://frigate:5000`

**What you'll get:**
- Live camera feeds
- Motion/person/object detection sensors
- Snapshots and clips from events
- Notification automations
- Camera entity cards
- Frigate event viewer

### 8. Start Frigate

```bash
docker-compose up -d frigate
```

Access the web UI at `http://localhost:5000`

### Troubleshooting Frigate

**GPU not detected:**
```bash
# Verify NVIDIA runtime
docker run --rm --gpus all nvidia/cuda:11.8.0-base-ubuntu22.04 nvidia-smi
```

**TensorRT compilation fails:**
- Check logs: `docker-compose logs frigate`
- Ensure 4GB+ free disk space
- May need to increase shm_size in docker-compose.yml

**Camera won't connect:**
- Test RTSP URL with VLC: Media > Open Network Stream
- Check camera is accessible from Docker host
- Verify credentials and IP address
- Some cameras require substream for detection

**High CPU usage:**
- Lower detection fps in config.yml
- Use substream (lower resolution) for detection
- Reduce number of tracked objects
- Ensure GPU is actually being used (check nvidia-smi)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is open source and available under the MIT License.

## Support

For issues and questions:
- Home Assistant: https://community.home-assistant.io/
- This repository: Open an issue on GitHub

## Acknowledgments

- Home Assistant Community
- Eclipse Mosquitto Project
- MariaDB Foundation
