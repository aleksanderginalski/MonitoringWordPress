# WordPress Monitoring Stack

A comprehensive monitoring solution for WordPress applications using Prometheus and Grafana, running on Docker Compose.

## 🏗️ Architecture
```
WordPress + MySQL (Application Layer)
    ↓
Exporters (Metrics Collection)
├─ Node Exporter (OS: CPU, RAM, Disk)
├─ cAdvisor (Docker Containers)
├─ Blackbox Exporter (HTTP Availability)
└─ MySQL Exporter (Database Metrics)
    ↓
Prometheus (Time-Series Database)
    ↓
Grafana (Visualization & Dashboards)
```

## 📋 Prerequisites

### Software Requirements
- **Docker Desktop** (with WSL2 integration) - min. v20.10
- **Docker Compose** - min. v2.0
- **WSL2** (Ubuntu 24.04 or newer) - for Linux environment on Windows
- **Git** - for cloning the repository

### System Resources
- **RAM:** minimum 4GB free (8GB recommended)
- **Disk:** ~2GB for Docker images + volumes
- **Ports:** 3000, 8080, 8081, 9090, 9100, 9104, 9115 must be available

## 🚀 Quick Start

### 1. Clone the repository
```bash
git clone https://https://github.com/aleksanderginalski/MonitoringWordPress.git
cd wordpress-monitoring
```

### 2. Start the stack
```bash
docker compose up -d
```

### 3. Wait for health checks (~60 seconds)
```bash
# Check status
docker compose ps

# All containers should show "Up" or "Up (healthy)"
```

### 4. Access the services

| Service | URL | Credentials |
|---------|-----|-------------|
| **WordPress** | http://localhost:8080 | Setup on first visit |
| **Prometheus** | http://localhost:9090 | No auth |
| **Grafana** | http://localhost:3000 | admin / admin |
| **cAdvisor** | http://localhost:8081 | No auth |

## 📊 Monitoring Layers

### 1. Infrastructure Monitoring (Node Exporter)
- CPU usage and load average
- Memory usage (RAM)
- Disk I/O and space
- Network traffic

### 2. Application Monitoring (Blackbox Exporter)
- HTTP availability (`probe_success`)
- Response time (`probe_duration_seconds`)
- HTTP status codes
- SSL certificate expiry (if applicable)

### 3. Container Monitoring (cAdvisor)
- Per-container CPU usage
- Per-container memory usage
- Container network traffic
- Container disk I/O

### 4. Database Monitoring (MySQL Exporter)
- Active connections (`mysql_global_status_threads_connected`)
- Queries per second (`mysql_global_status_queries`)
- Slow queries (`mysql_global_status_slow_queries`)
- Buffer pool usage

## 📁 Project Structure
```
monitoring/
├── docker-compose.yml          # Main orchestration file
├── prometheus/
│   └── prometheus.yml          # Prometheus scrape configuration
├── blackbox/
│   └── blackbox.yml            # HTTP probe modules
└── grafana/
    └── dashboards/             # Custom Grafana dashboards
```

## ⚙️ Configuration Details

### Prometheus (`prometheus/prometheus.yml`)

**Scrape jobs:**
- `prometheus` - Self-monitoring (meta-monitoring)
- `node_exporter` - OS metrics (port 9100)
- `cadvisor` - Container metrics (port 8080)
- `mysql` - Database metrics via mysqld_exporter (port 9104)
- `blackbox_wordpress` - HTTP probing of WordPress (port 9115)

**Scrape interval:** 15 seconds (global default)

**Important:** The `blackbox_wordpress` job uses `relabel_configs` to redirect Prometheus:
1. Target to check: `http://wordpress:80`
2. Actual endpoint: `http://blackbox_exporter:9115/probe?target=...`
3. Relabel configs handle this transformation automatically

### Blackbox Exporter (`blackbox/blackbox.yml`)

**Available modules:**
- `http_2xx` - Checks for HTTP 200 OK responses
- `http_post_2xx` - Tests form submissions (POST)
- `tcp_connect` - Verifies TCP port availability
- `icmp` - ICMP ping test

### Docker Compose Health Checks

**Services with health checks:**
- **mysql** - Uses `mysqladmin ping` to verify database readiness
- **wordpress** - Uses `curl` to verify HTTP response
- **prometheus** - Uses `wget` to verify API availability

**Dependency chain:**
```
1. mysql (foundation)
2. wordpress + mysqld_exporter + node_exporter + cadvisor (parallel)
3. blackbox_exporter (depends on wordpress healthy)
4. prometheus (depends on all exporters)
5. grafana (depends on prometheus healthy)
```

## 🔧 Troubleshooting

### Container keeps restarting
```bash
# View logs for specific container
docker logs mysqld_exporter

# View logs for all containers
docker compose logs

# Follow logs in real-time
docker compose logs -f
```

### Port already in use
```
Error: bind: address already in use
```

**Solution 1:** Find and stop the process using the port
```bash
# Linux/WSL
sudo lsof -i :9090

# Windows (PowerShell)
netstat -ano | findstr :9090
```

**Solution 2:** Change the port mapping in `docker-compose.yml`
```yaml
ports:
  - "9091:9090"  # Use 9091 instead of 9090
```

### mysqld_exporter shows "no user specified" error

**Cause:** Old `DATA_SOURCE_NAME` syntax doesn't work in MySQL Exporter v0.18+

**Solution:** Ensure `docker-compose.yml` uses `command:` instead of `environment:`
```yaml
mysqld_exporter:
  command:
    - "--mysqld.username=root:rootpassword"
    - "--mysqld.address=mysql:3306"
```

### No metrics visible in Grafana

1. **Check Prometheus targets:**
```
   http://localhost:9090/targets
```
   All targets should show "UP" status

2. **Verify Grafana datasource:**
   - Settings → Data sources → Prometheus
   - URL must be: `http://prometheus:9090` (NOT `localhost`!)
   - Click "Save & Test" - should show success

3. **Test query directly in Prometheus:**
```
   http://localhost:9090/graph
```
   Try query: `up`

### cAdvisor metrics use `container=` not `name=`

**Important:** cAdvisor v0.45+ changed label names:
- **Old:** `{name="wordpress"}`
- **New:** `{container="wordpress"}`

**Correct query format:**
```promql
container_cpu_usage_seconds_total{job="cadvisor", container="wordpress"}
```

## 🛠️ Technology Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| Docker Compose | v2.0+ | Container orchestration |
| Prometheus | latest | Time-series metrics database |
| Grafana | latest | Visualization and dashboards |
| Node Exporter | latest | OS metrics collector |
| cAdvisor | latest | Container metrics collector |
| Blackbox Exporter | latest | HTTP/TCP probe |
| MySQL Exporter | latest | MySQL metrics collector |
| WordPress | latest | Demo application |
| MySQL | 8.0 | Database |

## 📊 Default Dashboards

### WordPress Monitoring Dashboard (Custom)
- **Availability Panel:** WordPress UP/DOWN status
- **Response Time:** HTTP request latency
- **CPU Usage:** WordPress container CPU utilization
- **Memory Usage:** WordPress container RAM consumption
- **MySQL Connections:** Active database connections
- **MySQL Queries/sec:** Database query rate
- **Network Traffic:** Container ingress/egress traffic

### Node Exporter Full (Dashboard ID: 1860)
- System CPU usage
- Memory usage and swap
- Disk I/O operations
- Network traffic by interface
- Filesystem usage

## 🌐 Networking

All containers run in a single Docker network: `monitoring_monitoring`

**DNS resolution within containers:**
- `wordpress` → WordPress container IP
- `mysql` → MySQL container IP
- `prometheus` → Prometheus container IP
- `grafana` → Grafana container IP

**Important:** Within containers, **DO NOT use `localhost`**! Use service names for DNS resolution.

## 💾 Data Persistence

Docker volumes ensure data persists across container restarts:
```yaml
volumes:
  prometheus_data    # Prometheus time-series database
  grafana_data       # Grafana dashboards and settings
  mysql_data         # WordPress database
  wordpress_data     # WordPress files (themes, plugins, uploads)
```

To completely remove all data:
```bash
docker compose down -v  # ⚠️ WARNING: Deletes all volumes!
```

## 🔒 Security Considerations

**⚠️ This setup is for DEVELOPMENT/TESTING only!**

**For production, you MUST:**
- Change default passwords (Grafana, MySQL)
- Use secrets management (Docker secrets, Vault)
- Enable TLS/SSL for all web interfaces
- Implement authentication for Prometheus
- Use read-only MySQL user for mysqld_exporter
- Set up firewall rules (expose only necessary ports)
- Regular security updates for all images

## 📝 License

MIT

## 🤝 Contributing

Contributions welcome! Please open an issue or pull request.


## 🙏 Acknowledgments

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [cAdvisor](https://github.com/google/cadvisor)
- [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter)