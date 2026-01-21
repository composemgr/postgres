## 👋 Welcome to postgres 🚀

PostgreSQL - Powerful open-source relational database

## 📋 Description

PostgreSQL is a powerful, open-source object-relational database system with over 35 years of active development. Known for reliability, feature robustness, and performance.

## 🚀 Services

- **db**: PostgreSQL 16 Alpine (`postgres:16-alpine`)

## 📦 Installation

### Using curl
```shell
curl -q -LSsf "https://raw.githubusercontent.com/composemgr/postgres/main/docker-compose.yaml" | docker compose -f - up -d
```

### Using git
```shell
git clone "https://github.com/composemgr/postgres" ~/.local/srv/docker/postgres
cd ~/.local/srv/docker/postgres
docker compose up -d
```

### Using composemgr
```shell
composemgr install postgres
```

## 🔧 Configuration

### Environment Variables

```shell
TZ=America/New_York

# Database credentials
POSTGRES_USER=postgres                     # Superuser username
POSTGRES_PASSWORD=changeme_admin_password  # Superuser password
POSTGRES_DB=postgres                       # Default database name

# Optional: Create additional user/database
POSTGRES_INITDB_ARGS=--encoding=UTF-8 --locale=en_US.UTF-8
```

### Performance Tuning (Advanced)

Add to docker-compose.yaml under command:
```yaml
command:
  - "postgres"
  - "-c"
  - "max_connections=200"
  - "-c"
  - "shared_buffers=256MB"
  - "-c"
  - "effective_cache_size=1GB"
  - "-c"
  - "maintenance_work_mem=64MB"
  - "-c"
  - "checkpoint_completion_target=0.9"
  - "-c"
  - "wal_buffers=16MB"
  - "-c"
  - "default_statistics_target=100"
  - "-c"
  - "random_page_cost=1.1"
  - "-c"
  - "effective_io_concurrency=200"
```

## 🌐 Access

- **PostgreSQL**: localhost:50225 (from Docker host)
- **Connection String**: `postgresql://postgres:password@localhost:50225/postgres`

### Connect from another container
```yaml
services:
  myapp:
    environment:
      DATABASE_URL: postgresql://postgres:password@postgres-db:5432/postgres
    depends_on:
      - db
    networks:
      - postgres
```

## 📂 Volumes

- `./rootfs/db/postgres/postgres` - Database files and WAL logs

### Data Directory Structure
```
/var/lib/postgresql/data/
├── base/           # Database files
├── global/         # Cluster-wide tables
├── pg_wal/         # Write-Ahead Logs
├── pg_stat/        # Statistics
└── postgresql.conf # Main config
```

## 🔐 Security

- **Change default password** immediately
- **Create application users**: Don't use superuser in apps
- **Use pg_hba.conf**: Configure connection authentication
- **Enable SSL**: For production deployments
- **Regular updates**: Keep PostgreSQL version current
- **Principle of least privilege**: Grant minimal permissions

### Create Application User
```sql
-- Connect as postgres superuser
CREATE USER myapp WITH PASSWORD 'secure_password';
CREATE DATABASE myapp_db OWNER myapp;
GRANT ALL PRIVILEGES ON DATABASE myapp_db TO myapp;
```

## 🔍 Logging

```shell
# View logs
docker compose logs -f db

# View last 100 lines
docker compose logs --tail=100 db
```

## 🛠️ Management

### Start services
```shell
docker compose up -d
```

### Stop services
```shell
docker compose down
```

### Update PostgreSQL
```shell
# Major version upgrades require pg_dump/pg_restore
docker compose exec db pg_dumpall -U postgres > backup.sql
docker compose pull && docker compose up -d
docker compose exec db psql -U postgres < backup.sql
```

### Connect to PostgreSQL shell
```shell
docker compose exec db psql -U postgres
```

### Run SQL commands
```shell
# Execute SQL file
docker compose exec -T db psql -U postgres -d mydb < schema.sql

# Run single command
docker compose exec db psql -U postgres -c "SELECT version();"
```

## 🔄 Backup & Restore

### Backup

```shell
# Backup single database
docker compose exec db pg_dump -U postgres mydb > mydb-backup.sql

# Backup all databases
docker compose exec db pg_dumpall -U postgres > all-databases-backup.sql

# Compressed backup
docker compose exec db pg_dump -U postgres -Fc mydb > mydb-backup.dump
```

### Restore

```shell
# Restore SQL backup
cat mydb-backup.sql | docker compose exec -T db psql -U postgres mydb

# Restore compressed backup
docker compose exec -T db pg_restore -U postgres -d mydb < mydb-backup.dump

# Restore all databases
cat all-databases-backup.sql | docker compose exec -T db psql -U postgres
```

### Automated Backups

Add to crontab:
```cron
0 2 * * * cd /path/to/postgres && docker compose exec -T db pg_dumpall -U postgres | gzip > backups/backup-$(date +\%Y\%m\%d).sql.gz
```

## 📋 Requirements

- Docker Engine 20.10+
- Docker Compose V2+
- Sufficient disk space for data
- 256MB+ RAM minimum (1GB+ recommended)

## 🆘 Troubleshooting

### Connection refused
```shell
# Check if container is running
docker compose ps

# Check logs for errors
docker compose logs db

# Verify port mapping
docker compose port db 5432
```

### Permission denied on data directory
```shell
# Fix ownership
sudo chown -R 999:999 rootfs/db/postgres/
```

### Out of disk space
```shell
# Check database size
docker compose exec db psql -U postgres -c "SELECT pg_database.datname, pg_size_pretty(pg_database_size(pg_database.datname)) FROM pg_database;"

# Vacuum and analyze
docker compose exec db psql -U postgres -c "VACUUM FULL ANALYZE;"
```

### Slow queries
```shell
# Enable query logging temporarily
docker compose exec db psql -U postgres -c "ALTER SYSTEM SET log_min_duration_statement = 1000;"
docker compose restart db

# View slow queries in logs
docker compose logs db | grep "duration:"
```

## 🤝 Author

🤖 casjay: [Github](https://github.com/casjay) 🤖  
🦄 composemgr: [Github](https://github.com/composemgr) 🦄

## 📚 Resources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Wiki](https://wiki.postgresql.org/)
- [PgTune](https://pgtune.leopard.in.ua/) - Configuration calculator
