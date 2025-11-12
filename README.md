# Home Assistant Docker Stack

A complete Docker-based Home Assistant setup with MariaDB and Mosquitto MQTT broker.

## Overview

This repository contains a production-ready Home Assistant stack running on Docker with:
- **Home Assistant Core**: Open-source home automation platform
- **MariaDB**: Reliable database for storing Home Assistant data
- **Mosquitto**: Lightweight MQTT broker for IoT device communication

## Features

- Docker Compose orchestration for easy deployment
- MariaDB database integration for improved performance
- MQTT broker for IoT device communication
- Cloudflare proxy configuration included
- Environment variable based configuration
- Persistent data storage
- Auto-restart on failure

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- Git

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
└── mosquitto/
    ├── config/             # Mosquitto configuration
    │   └── mosquitto.conf
    ├── data/               # Mosquitto data
    └── log/                # Mosquitto logs
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
