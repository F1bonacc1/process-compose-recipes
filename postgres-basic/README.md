# PostgreSQL Basic Recipe 🐘

A PostgreSQL database setup with automatic initialization and built-in monitoring using native PostgreSQL binaries.

## Overview

This recipe creates a PostgreSQL database instance with:
- **Automatic initialization** of data directory and database cluster
- **Unix domain socket** connections for optimal performance
- **Built-in logging** with real-time log monitoring
- **Health checks** with `pg_isready` probes
- **Graceful shutdown** with proper PostgreSQL stop procedures

## Architecture

```
┌─────────────────────────────────┐
│        PostgreSQL Server        │
│                                 │
│  Port: 5432 (default)           │
│  Socket: ./postgres_data        │
│  Data: ./postgres_data/data     │
│  User: postgres                 │
│  Database: postgres             │
└─────────────────────────────────┘
                │
                ▼
    ┌─────────────────────────────┐
    │       Log Monitoring        │
    │                             │
    │  • PostgreSQL logs          │
    │  • Process-Compose logs     │
    └─────────────────────────────┘
```

## Prerequisites

### Required Software
- **PostgreSQL**: Install from [PostgreSQL downloads](https://www.postgresql.org/download/)
- **Process-Compose**: v1.75.0 or later

## Usage

### Start the Database
```bash
process-compose -f <recipe-path>/process-compose.yaml
```

### Verify Installation
The database will automatically:
1. **Sanity check** - Verify PostgreSQL is installed
2. **Initialize data directory** - Create and initialize `PGDATA` if needed
3. **Start PostgreSQL server** - Launch database with Unix socket
4. **Health monitoring** - Continuous readiness checks
5. **Log monitoring** - Real-time PostgreSQL and Process-Compose logs

### Connection Details

| Parameter | Value |
|-----------|-------|
| **Host** | Unix socket: `./postgres_data` |
| **Port** | 5432 (default) |
| **User** | postgres |
| **Database** | postgres |
| **Data Directory** | `./postgres_data/data` |

## Testing the Database

### Basic Connection
```bash
# Connect using Unix socket (fastest)
psql -h ./postgres_data -U postgres -d postgres

# Connect using TCP (if configured)
psql -h localhost -U postgres -d postgres -p 5432
```

### Database Operations
```bash
# Create a new database
createdb -h ./postgres_data -U postgres myapp

# Connect to your database
psql -h ./postgres_data -U postgres -d myapp

# Run some SQL
psql -h ./postgres_data -U postgres -d myapp -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name TEXT);"
psql -h ./postgres_data -U postgres -d myapp -c "INSERT INTO users (name) VALUES ('Alice'), ('Bob');"
psql -h ./postgres_data -U postgres -d myapp -c "SELECT * FROM users;"
```

### Database Status
```bash
# Check if PostgreSQL is ready
pg_isready -h ./postgres_data -U postgres

# View active connections
psql -U postgres -d postgres -c "SELECT * FROM pg_stat_activity;"

# Check database size
psql -U postgres -d postgres -c "SELECT pg_size_pretty(pg_database_size('postgres'));"
```

## Monitoring

### Real-time Log Monitoring
The recipe includes dual log monitoring:
- **PostgreSQL logs**: `./postgres_data/data/pg.log`
- **Process-Compose logs**: `/tmp/process-compose-${USER}.log`

### Health Checks
The database includes readiness probes:
- **Command**: `pg_isready`
- **Automatic**: Runs continuously to ensure database availability
- **Integration**: Works with Process-Compose health system

### Log Files Location
```
postgres_data/
└── data/
    ├── pg.log           # PostgreSQL server logs
    ├── postgresql.conf  # Main configuration
    ├── pg_hba.conf      # Authentication settings
    └── base/            # Database files
```

## Configuration

### Environment Variables
The recipe uses these environment variables:
- **PGDATA**: `./postgres_data/data` - Data directory location
- **PGUSER**: `postgres` - Default database user
- **PGDATABASE**: `postgres` - Default database name
- **PGPORT**: `5432` - PostgreSQL port
- **PGHOST**: `./postgres_data` - Unix socket directory

### Connection Configuration
- **Unix Socket**: Primary connection method via `./postgres_data`
- **TCP**: Available on localhost:5432 (if network configured)
- **Authentication**: Trust mode for local connections

## Customization

### Change Data Directory
```bash
# Set custom data location before starting
export PGDATA=/custom/path/to/data
process-compose -f process-compose.yaml
```

### Enable Network Connections
To accept TCP connections, modify the startup command:
```yaml
postgresql:
  command: "pg_ctl start -D $$PGDATA -o '-k $$PGHOST -c listen_addresses=localhost' -w -t 30 -l $$PGDATA/pg.log"
```

### Set Custom Port
```yaml
environment:
  - "PGPORT=5433"  # Custom port

postgresql:
  command: "pg_ctl start -D $$PGDATA -o '-k $$PGHOST -p $$PGPORT' -w -t 30 -l $$PGDATA/pg.log"
```

### Configure Authentication
Edit `pg_hba.conf` after initialization:
```bash
# After first run, edit authentication
echo "local   all   postgres   md5" >> ./postgres_data/data/pg_hba.conf

# Set password for postgres user
psql -U postgres -c "ALTER USER postgres PASSWORD 'mypassword';"
```

### Performance Tuning
```yaml
postgresql:
  command: >
    pg_ctl start -D $$PGDATA 
    -o '-k $$PGHOST -c shared_buffers=256MB -c max_connections=200' 
    -w -t 30 -l $$PGDATA/pg.log
```

## Troubleshooting

### Common Issues

#### 1. "pg_ctl: command not found"
**Solution**: Install PostgreSQL binaries

#### 2. "Permission denied" on data directory
**Error**: Cannot initialize data directory
**Solution**: Check directory permissions
```bash
# Fix permissions
chmod 700 ./postgres_data/data

# Or remove and let recipe recreate
rm -rf ./postgres_data/data
```

#### 3. "Database system is starting up"
**Symptoms**: Long startup time or connection refused
**Debug**:
```bash
# Check PostgreSQL logs
tail -f ./postgres_data/data/pg.log

# Check if process is running
ps aux | grep postgres
```

#### 4. "Connection refused"
**Error**: Can't connect to database
**Solution**: Verify socket path and permissions
```bash
# Check socket file exists
ls -la ./postgres_data/.s.PGSQL.5432

# Test connection
pg_isready -h ./postgres_data -U postgres
```

#### 5. Port already in use
**Error**: "Address already in use"
**Solution**: Check for existing PostgreSQL instances
```bash
# Find processes using port 5432
lsof -i :5432

# Kill existing PostgreSQL
pkill -f postgres
```

### Debug Commands
```bash
# Check PostgreSQL version
pg_ctl --version

# Verify data directory initialization
ls -la ./postgres_data/data/

# Test database connectivity
pg_isready -h ./postgres_data -U postgres -d postgres

# View server status
pg_ctl status -D ./postgres_data/data

# Check server configuration
psql -h ./postgres_data -U postgres -c "SHOW ALL;"
```

## Performance Tuning

### Memory Settings
```yaml
postgresql:
  command: >
    pg_ctl start -D $$PGDATA 
    -o '-k $$PGHOST -c shared_buffers=512MB -c work_mem=64MB -c maintenance_work_mem=256MB'
    -w -t 30 -l $$PGDATA/pg.log
```

### Connection Limits
```yaml
postgresql:
  command: >
    pg_ctl start -D $$PGDATA 
    -o '-k $$PGHOST -c max_connections=300 -c max_prepared_transactions=100'
    -w -t 30 -l $$PGDATA/pg.log
```

### Logging Configuration
```yaml
postgresql:
  command: >
    pg_ctl start -D $$PGDATA 
    -o '-k $$PGHOST -c log_statement=all -c log_duration=on -c log_min_duration_statement=1000'
    -w -t 30 -l $$PGDATA/pg.log
```

## Backup and Maintenance

### Database Backup
```bash
# Full database backup
pg_dump -h ./postgres_data -U postgres postgres > backup.sql

# Compressed backup
pg_dump -h ./postgres_data -U postgres postgres | gzip > backup.sql.gz

# All databases backup
pg_dumpall -h ./postgres_data -U postgres > all_databases.sql
```

### Restore Database
```bash
# Restore from backup
psql -h ./postgres_data -U postgres postgres < backup.sql

# Restore compressed backup
gunzip -c backup.sql.gz | psql -h ./postgres_data -U postgres postgres
```

### Maintenance Tasks
```bash
# Vacuum database
psql -h ./postgres_data -U postgres -d postgres -c "VACUUM ANALYZE;"

# Reindex database
psql -h ./postgres_data -U postgres -d postgres -c "REINDEX DATABASE postgres;"

# Check database integrity
psql -h ./postgres_data -U postgres -d postgres -c "SELECT * FROM pg_stat_database;"
```

## Production Considerations

### Security
- **Change default passwords** immediately
- **Configure proper authentication** in `pg_hba.conf`
- **Enable SSL/TLS** for network connections
- **Regular security updates** for PostgreSQL
- **Limit network exposure** using firewalls

### High Availability
- **Regular backups** with point-in-time recovery
- **Streaming replication** for standby servers
- **Connection pooling** with pgBouncer
- **Monitor disk space** and performance metrics

### Monitoring & Observability
- **pg_stat_activity** for active connections
- **pg_stat_database** for database statistics
- **Log analysis** for performance issues
- **Disk usage monitoring** for data growth

## Resources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Administration](https://www.postgresql.org/docs/current/admin.html)
- [PostgreSQL Performance Tuning](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [Process-Compose Documentation](https://f1bonacc1.github.io/process-compose/launcher/)

## License

This recipe is part of the Process-Compose Recipes collection, licensed under MIT License.