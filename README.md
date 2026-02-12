# WordPress Monitoring Stack

Pełny monitoring stack dla aplikacji WordPress używając Prometheus i Grafany.

## 🏗️ Architektura
```
WordPress + MySQL (aplikacja)
    ↓
Exporters (zbieranie metryk)
├─ Node Exporter (OS: CPU, RAM, disk)
├─ cAdvisor (Docker containers)
├─ Blackbox Exporter (HTTP availability)
└─ MySQL Exporter (database metrics)
    ↓
Prometheus (przechowywanie metryk)
    ↓
Grafana (wizualizacja)
```

## 🚀 Uruchomienie
```bash
# Uruchom cały stack
docker compose up -d

# Sprawdź status
docker compose ps

# Zobacz logi
docker compose logs -f
```

## 🌐 Adresy

- **WordPress**: http://localhost:8080
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000 (admin/admin)
- **cAdvisor**: http://localhost:8081

## 📊 Monitorowane warstwy

### 1. Infrastruktura (Node Exporter)
- CPU usage
- Memory usage
- Disk I/O
- Network traffic

### 2. Aplikacja (Blackbox Exporter)
- HTTP availability (probe_success)
- Response time (probe_duration_seconds)
- HTTP status codes

### 3. Kontenery (cAdvisor)
- Container CPU/Memory
- Network per container
- Disk I/O per container

### 4. Baza danych (MySQL Exporter)
- Connections
- Queries per second
- Slow queries

## 📁 Struktura projektu
```
monitoring/
├── docker-compose.yml       # Główna konfiguracja
├── prometheus/
│   └── prometheus.yml       # Scrape configs
├── blackbox/
│   └── blackbox.yml         # HTTP probe modules
└── grafana/
    └── dashboards/          # Custom dashboards
```

## 🛠️ Technologie

- Docker Compose
- Prometheus
- Grafana
- Node Exporter
- cAdvisor
- Blackbox Exporter
- MySQL Exporter
- WordPress
- MySQL 8.0

## 📝 Notatki

- Healthchecks zapewniają poprawną kolejność startowania
- Volumes przechowują dane między restartami
- Wszystkie serwisy w jednej sieci Docker (monitoring)