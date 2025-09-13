# Process-Compose Recipes 🍳

A curated collection of ready-to-use [Process-Compose](https://github.com/F1bonacc1/process-compose) recipes for common development and infrastructure scenarios.

## What are Process-Compose Recipes?

Process-Compose Recipes are pre-configured `process-compose.yaml` files for popular services and applications. Instead of writing complex configurations from scratch, you can pull tested recipes and customize them for your needs.

## 🚀 Quick Start

### Prerequisites

- [Process-Compose](https://github.com/F1bonacc1/process-compose) v1.75 or later

### Using Recipes

```bash
# Search for available recipes
process-compose recipe search postgres

# Pull a recipe to your local machine
process-compose recipe pull postgres-basic

# List your installed recipes
process-compose recipe list

# Run a recipe
process-compose -f $XDG_CONFIG_HOME/recipes/postgres-basic/process-compose.yaml
```

## 📋 Available Recipes

| Recipe | Description | Tags | Complexity |
|--------|-------------|------|------------|
| [postgres-basic](./postgres-basic) | PostgreSQL database with initialization | `database`, `sql`, `postgres` | ⭐ |
| [nats-cluster](./nats-cluster) | NATS messaging cluster (3 nodes) | `messaging`, `cluster`, `pubsub` | ⭐⭐⭐ |


*Complexity: ⭐ = Simple, ⭐⭐⭐⭐ = Advanced*

## 🔍 Recipe Categories

### Databases
- [x] **postgres-basic**: Single PostgreSQL instance with persistence
- [ ] **mysql-replication**: MySQL master-slave setup
- [ ] **mongodb-replica**: MongoDB replica set configuration

### Message Queues & Streaming
- [x] **nats-cluster**: NATS messaging cluster
- [ ] **kafka-cluster**: Apache Kafka cluster setup
- [ ] **rabbitmq-cluster**: RabbitMQ cluster setup

### Caching & Storage
- [ ] **redis-sentinel**: Redis with high availability
- [ ] **memcached-cluster**: Distributed Memcached setup
- [ ] **minio-cluster**: MinIO object storage cluster

### Observability
- [ ] **prometheus-grafana**: Monitoring stack
- [ ] **elk-stack**: Elasticsearch, Logstash, Kibana
- [ ] **jaeger-tracing**: Distributed tracing setup

## 📖 Recipe Structure

Each recipe directory contains:

```
recipe-name/
├── recipe.yaml           # Recipe metadata and configuration
├── process-compose.yaml  # Main process configuration
├── README.md             # Recipe-specific documentation
└── configs/              # Additional configuration files (optional)
    ├── app.conf
    └── init.sql
```

### Recipe Metadata (`recipe.yaml`)

```yaml
name: postgres-basic
description: Basic PostgreSQL database server with initialization script support
version: 1.2.0
author: process-compose-community
tags:
  - database
  - postgres
  - sql
last_updated: 2025-01-15T10:30:00Z
min_version: "0.50.0"
repository: https://github.com/f1bonacc1/process-compose-recipes
```

## 🛠 Customizing Recipes

### Override Files

Create override files to modify recipes without changing the original:

```yaml
# my-postgres-override.yaml
version: "0.5"

processes:
  postgres:
    environment:
      - "POSTGRES_DB=myproject"
      - "POSTGRES_PASSWORD=mysecret"
```

```bash
process-compose -f $XDG_CONFIG_HOME/recipes/postgres-basic/process-compose.yaml -f my-postgres-override.yaml
```

## 📝 Recipe Commands Reference

### Search Recipes
```bash
# Search all recipes
process-compose recipe search

# Search by name
process-compose recipe search postgres

# Search by tags
process-compose recipe search --tags database,sql

# Search by author
process-compose recipe search --author community
```

### Manage Recipes
```bash
# Pull a recipe
process-compose recipe pull <recipe-name>

# Force pull (overwrite existing)
process-compose recipe pull <recipe-name> --force

# List local recipes
process-compose recipe list

# Remove a recipe
process-compose recipe remove <recipe-name>

# Show a recipe
process-compose recipe show <recipe-name>
```

### Using Recipes
```bash
# Run a recipe directly
process-compose -f $XDG_CONFIG_HOME/recipes/<recipe-name>/process-compose.yaml

# Run with custom environment
POSTGRES_PASSWORD=secret process-compose -f ~/.process-compose/recipes/postgres-basic/process-compose.yaml

# Run with override file
process-compose -f ~/.process-compose/recipes/postgres-basic/process-compose.yaml -f my-overrides.yaml
```

## 🤝 Contributing

We welcome contributions! Here's how you can help:

### Adding New Recipes

1. **Fork this repository**
2. **Create a new recipe directory** following the naming convention: `service-variant`
3. **Add required files**:
   - `recipe.yaml` - Recipe metadata
   - `process-compose.yaml` - Process configuration
   - `README.md` - Recipe documentation
4. **Test your recipe** thoroughly
5. **Submit a Pull Request**

### Recipe Guidelines

- **Use semantic versioning** for recipe versions
- **Include comprehensive health checks** for all services
- **Support customization** through environment variables
- **Provide sensible defaults** for all configuration options
- **Document dependencies** and requirements clearly
- **Test on multiple platforms** when possible

### Recipe Naming Convention

- Use lowercase with hyphens: `service-variant`
- Be descriptive but concise: `postgres-basic`, `redis-sentinel`
- Include complexity indicators: `kafka-simple`, `kafka-cluster`

### Metadata Requirements

```yaml
name: recipe-name              # Must match directory name
description: Clear description # One-line summary
version: X.Y.Z                 # Semantic version
author: your-name              # Author/maintainer
tags: [tag1, tag2]             # Searchable tags
last_updated: 2025-01-15T10:30:00Z
min_version: "0.50.0"          # Min process-compose version
```

## 🐛 Issues and Support

- **Recipe Issues**: Open an issue in this repository
- **Process-Compose Issues**: Open an issue in the [main repository](https://github.com/F1bonacc1/process-compose)
- **Discussions**: Use GitHub Discussions for questions and ideas

### Issue Templates

When reporting issues, please include:
- Recipe name and version
- Process-Compose version
- Operating system and architecture
- Complete error logs
- Steps to reproduce

## 📚 Examples

### Basic Database Setup
```bash
# Pull and run PostgreSQL
process-compose recipe pull postgres-basic
process-compose -f $XDG_CONFIG_HOME/recipes/postgres-basic/process-compose.yaml

# Connect to database
psql -h localhost -U postgres -d myapp
```

### Microservices with Messaging
```bash
# Set up NATS cluster
process-compose recipe pull nats-cluster
process-compose -f $XDG_CONFIG_HOME/recipes/nats-cluster/process-compose.yaml

# Test cluster connectivity
nats pub test "Hello NATS!"
nats sub test
```

---

**Happy Composing!** 🎵

For more information about Process-Compose, visit the [main repository](https://github.com/F1bonacc1/process-compose).