# Source Installation

<cite>
**Referenced Files in This Document**
- [README.md](file://README.md)
- [conf/service_conf.yaml](file://conf/service_conf.yaml)
- [api/ragflow_server.py](file://api/ragflow_server.py)
- [web/package.json](file://web/package.json)
- [pyproject.toml](file://pyproject.toml)
- [docker/launch_backend_service.sh](file://docker/launch_backend_service.sh)
- [docker/docker-compose-base.yml](file://docker/docker-compose-base.yml)
- [download_deps.py](file://download_deps.py)
- [api/db/init_data.py](file://api/db/init_data.py)
- [common/settings.py](file://common/settings.py)
- [docs/develop/launch_ragflow_from_source.md](file://docs/develop/launch_ragflow_from_source.md)
- [docker/.env](file://docker/.env)
- [docker/init.sql](file://docker/init.sql)
- [api/db/db_models.py](file://api/db/db_models.py)
- [api/db/runtime_config.py](file://api/db/runtime_config.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [System Requirements](#system-requirements)
4. [Installation Steps](#installation-steps)
5. [Configuration](#configuration)
6. [Database Setup](#database-setup)
7. [Launching Services](#launching-services)
8. [Frontend Development](#frontend-development)
9. [Production Deployment](#production-deployment)
10. [Troubleshooting](#troubleshooting)
11. [Development Workflow](#development-workflow)
12. [Best Practices](#best-practices)

## Introduction

RAGFlow is an open-source Retrieval-Augmented Generation (RAG) engine that provides a comprehensive solution for building AI-powered knowledge systems. This guide covers the complete process of installing and running RAGFlow directly from source code, enabling developers to contribute to the project or customize it for specific use cases.

The source installation method allows for:
- Full customization of the application logic
- Debugging and development of new features
- Integration with custom infrastructure
- Access to the complete codebase for learning and modification

## Prerequisites

Before installing RAGFlow from source, ensure your system meets all prerequisites:

### Software Requirements

- **Python 3.10 or higher**: Required for running the backend services
- **Node.js 16 or higher**: Required for the frontend development server
- **Docker 24.0.0 or higher**: Required for managing dependent services
- **Docker Compose v2.26.1 or higher**: Required for orchestrating multi-container deployments
- **Git**: For cloning the repository and version control
- **uv**: Package manager for Python dependencies (recommended)
- **jemalloc**: Memory allocation library for improved performance

### Operating System Support

RAGFlow supports the following operating systems:
- **Linux**: Ubuntu 18.04+, CentOS 7+, Red Hat Enterprise Linux 7+
- **macOS**: macOS 10.15+ (Intel and Apple Silicon)
- **Windows**: Windows 10/11 with WSL2 or native support

**Section sources**
- [README.md](file://README.md#L144-L150)
- [pyproject.toml](file://pyproject.toml#L8)

## System Requirements

### Hardware Specifications

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU Cores | 4 cores | 8+ cores |
| RAM | 16 GB | 32 GB+ |
| Storage | 50 GB free | 100 GB+ SSD |
| Network | Stable internet connection | Gigabit Ethernet |

### Environment Variables

Set these environment variables for optimal performance:

```bash
# Memory optimization
export PYTHONPATH=$(pwd)
export HF_ENDPOINT=https://hf-mirror.com  # For HuggingFace access in China

# Performance tuning
export OMP_NUM_THREADS=4
export MKL_NUM_THREADS=4
```

**Section sources**
- [README.md](file://README.md#L144-L150)

## Installation Steps

### Step 1: Clone the Repository

```bash
git clone https://github.com/infiniflow/ragflow.git
cd ragflow/
```

### Step 2: Install uv Package Manager

```bash
# Install uv globally
pipx install uv

# Verify installation
uv --version
```

### Step 3: Install Python Dependencies

```bash
# Install Python dependencies with uv
uv sync --python 3.10

# Download additional dependencies
uv run download_deps.py
```

### Step 4: Install Pre-commit Hooks

```bash
# Install pre-commit hooks for code quality
pre-commit install
```

**Section sources**
- [README.md](file://README.md#L307-L320)
- [docs/develop/launch_ragflow_from_source.md](file://docs/develop/launch_ragflow_from_source.md#L36-L49)

## Configuration

### Environment Configuration

Create a `.env` file in the root directory with the following content:

```bash
# Database Configuration
DB_TYPE=mysql
MYSQL_PASSWORD=infini_rag_flow
MYSQL_PORT=5455

# Service Ports
SVR_HTTP_PORT=9380
ADMIN_SVR_HTTP_PORT=9381
TEI_PORT=6380

# Storage Configuration
STORAGE_IMPL=MINIO
MINIO_USER=rag_flow
MINIO_PASSWORD=infini_rag_flow
MINIO_PORT=9000

# Authentication
REGISTER_ENABLED=1
SECRET_KEY=your-secret-key-here

# Logging Configuration
LOG_LEVEL=INFO
```

### Service Configuration

Configure the main service settings in `conf/service_conf.yaml`:

```yaml
ragflow:
  host: 0.0.0.0
  http_port: 9380

admin:
  host: 0.0.0.0
  http_port: 9381

mysql:
  name: 'rag_flow'
  user: 'root'
  password: 'infini_rag_flow'
  host: 'localhost'
  port: 5455
  max_connections: 900

minio:
  user: 'rag_flow'
  password: 'infini_rag_flow'
  host: 'localhost:9000'

redis:
  db: 1
  password: 'infini_rag_flow'
  host: 'localhost:6379'
```

### Host Configuration

Update your `/etc/hosts` file to resolve service names:

```bash
127.0.0.1       es01 infinity mysql minio redis sandbox-executor-manager
```

**Section sources**
- [conf/service_conf.yaml](file://conf/service_conf.yaml#L1-L151)
- [docker/.env](file://docker/.env#L1-L245)
- [docs/develop/launch_ragflow_from_source.md](file://docs/develop/launch_ragflow_from_source.md#L59-L68)

## Database Setup

### Launch Database Services

```bash
# Start all required services
docker compose -f docker/docker-compose-base.yml up -d
```

### Initialize Database Schema

The database initialization is handled automatically when starting the backend services, but you can manually initialize it:

```bash
# Activate virtual environment
source .venv/bin/activate
export PYTHONPATH=$(pwd)

# Initialize database schema
python -c "
from api.db.db_models import init_database_tables
init_database_tables()
"

# Initialize default data
python -c "
from api.db.init_data import init_web_data, init_superuser
init_web_data()
init_superuser()
"
```

### Database Migration

RAGFlow includes automatic database migration capabilities. The system will automatically detect and apply migrations when starting the backend services.

**Section sources**
- [api/db/db_models.py](file://api/db/db_models.py#L565-L587)
- [api/db/init_data.py](file://api/db/init_data.py#L164-L178)

## Launching Services

### Backend Services

#### Step 1: Configure Environment

```bash
# Set environment variables
export PYTHONPATH=$(pwd)
export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu/

# Check jemalloc availability
JEMALLOC_PATH=$(pkg-config --variable=libdir jemalloc)/libjemalloc.so
echo "Using jemalloc: $JEMALLOC_PATH"
```

#### Step 2: Start Task Executors

```bash
# Start task executor processes
JEMALLOC_PATH=$JEMALLOC_PATH \
LD_PRELOAD=$JEMALLOC_PATH \
python rag/svr/task_executor.py 1 &
```

#### Step 3: Start Main Server

```bash
# Start the main RAGFlow server
python api/ragflow_server.py
```

### Alternative: Using the Launch Script

For easier management, use the provided launch script:

```bash
# Execute the launch script
bash docker/launch_backend_service.sh
```

### Service Verification

Monitor the backend services:

```bash
# Check service logs
tail -f .logs/ragflow_server.log

# Verify service is running
curl http://localhost:9380/health
```

**Section sources**
- [api/ragflow_server.py](file://api/ragflow_server.py#L74-L167)
- [docker/launch_backend_service.sh](file://docker/launch_backend_service.sh#L1-L130)

## Frontend Development

### Step 1: Install Dependencies

```bash
# Navigate to frontend directory
cd web/

# Install Node.js dependencies
npm install
```

### Step 2: Configure Proxy Settings

Update the proxy configuration in `.umirc.ts`:

```typescript
// Proxy configuration for backend API
proxy: {
  '/api': {
    target: 'http://127.0.0.1:9380',
    changeOrigin: true,
  },
},
```

### Step 3: Start Development Server

```bash
# Start the React development server
npm run dev
```

### Step 4: Access the Application

Open your browser and navigate to:
- **Frontend**: `http://localhost:8000`
- **Backend API**: `http://localhost:9380`
- **Admin Panel**: `http://localhost:9381`

**Section sources**
- [web/package.json](file://web/package.json#L1-L180)
- [docs/develop/launch_ragflow_from_source.md](file://docs/develop/launch_ragflow_from_source.md#L102-L126)

## Production Deployment

### Building Production Assets

```bash
# Navigate to web directory
cd web/

# Build production frontend
npm run build

# Serve production build
npx serve -s dist -p 8000
```

### Production Configuration

Create a production configuration file:

```bash
# .env.production
NODE_ENV=production
SVR_HTTP_PORT=80
ADMIN_SVR_HTTP_PORT=443
SECRET_KEY=your-production-secret-key
REGISTER_ENABLED=0
```

### Production Service Management

Use systemd for production service management:

```ini
# /etc/systemd/system/ragflow.service
[Unit]
Description=RAGFlow Production Service
After=network.target

[Service]
Type=simple
User=ragflow
WorkingDirectory=/opt/ragflow
Environment=PYTHONPATH=/opt/ragflow
ExecStart=/opt/ragflow/.venv/bin/python /opt/ragflow/api/ragflow_server.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Nginx Configuration

Configure Nginx for production deployment:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /api/ {
        proxy_pass http://127.0.0.1:9380;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Dependency Conflicts

**Problem**: Python dependency conflicts during installation

**Solution**:
```bash
# Clear uv cache
uv cache clean

# Reinstall dependencies
uv sync --force-reinstall
```

#### 2. Memory Allocation Issues

**Problem**: Out of memory errors during processing

**Solution**:
```bash
# Install jemalloc for better memory management
# Ubuntu/Debian
sudo apt-get install libjemalloc-dev

# CentOS/RHEL
sudo yum install jemalloc

# macOS
brew install jemalloc
```

#### 3. Database Connection Issues

**Problem**: Cannot connect to MySQL database

**Solution**:
```bash
# Check MySQL service status
docker ps | grep mysql

# Verify connection
mysql -h localhost -P 5455 -u root -pinfinit_rag_flow rag_flow

# Reset permissions if needed
mysql -h localhost -P 5455 -u root -pinfinit_rag_flow -e "GRANT ALL PRIVILEGES ON rag_flow.* TO 'root'@'%' IDENTIFIED BY 'infini_rag_flow'; FLUSH PRIVILEGES;"
```

#### 4. Port Conflicts

**Problem**: Port 9380 or 8000 already in use

**Solution**:
```bash
# Find processes using the port
lsof -i :9380
# Kill the process
kill -9 <PID>

# Or change the port in service_conf.yaml
```

#### 5. Frontend Build Issues

**Problem**: Frontend build fails with Node.js errors

**Solution**:
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Debug Mode

Enable debug mode for detailed logging:

```bash
# Start server with debug mode
python api/ragflow_server.py --debug

# Set debug environment variable
export RAGFLOW_DEBUG=1
```

### Log Analysis

Monitor logs for troubleshooting:

```bash
# Backend logs
tail -f .logs/ragflow_server.log

# Frontend logs
cd web/
npm run dev  # Check terminal output

# Database logs
docker logs -f docker-mysql-1
```

**Section sources**
- [api/ragflow_server.py](file://api/ragflow_server.py#L110-L125)

## Development Workflow

### Setting Up Development Environment

1. **Clone and Install**:
```bash
git clone https://github.com/infiniflow/ragflow.git
cd ragflow/
uv sync --python 3.10
uv run download_deps.py
pre-commit install
```

2. **Start Services**:
```bash
# Start backend services
bash docker/launch_backend_service.sh

# Start frontend development server
cd web/
npm run dev
```

3. **Database Initialization**:
```bash
# Initialize database and create superuser
python -c "
from api.db.init_data import init_superuser
init_superuser()
"
```

### Code Quality Standards

RAGFlow uses automated code quality tools:

```bash
# Run pre-commit hooks
pre-commit run --all-files

# Run linting
npm run lint

# Run tests
pytest
```

### Testing

Run the test suite:

```bash
# Backend tests
pytest test/

# Frontend tests
cd web/
npm test
```

### Contributing

Follow the contribution guidelines:

1. Fork the repository
2. Create a feature branch
3. Make changes and test thoroughly
4. Submit a pull request

**Section sources**
- [pyproject.toml](file://pyproject.toml#L160-L203)

## Best Practices

### Development Environment

1. **Virtual Environment**: Always use a virtual environment for Python dependencies
2. **Version Control**: Commit changes frequently with meaningful messages
3. **Testing**: Test changes thoroughly before committing
4. **Documentation**: Update documentation for new features

### Performance Optimization

1. **Memory Management**: Use jemalloc for better memory allocation
2. **Database Connections**: Configure appropriate connection pools
3. **Caching**: Implement caching for frequently accessed data
4. **Monitoring**: Set up monitoring for production deployments

### Security

1. **Secrets Management**: Never commit sensitive information
2. **Access Control**: Use proper authentication and authorization
3. **Network Security**: Secure network communications
4. **Regular Updates**: Keep dependencies updated

### Maintenance

1. **Regular Backups**: Backup databases regularly
2. **Log Rotation**: Implement log rotation for long-term operation
3. **Monitoring**: Set up monitoring and alerting
4. **Documentation**: Keep documentation up to date

### Scaling

1. **Load Balancing**: Use load balancers for high availability
2. **Horizontal Scaling**: Scale horizontally as needed
3. **Database Optimization**: Optimize database queries and indexing
4. **Caching Strategy**: Implement appropriate caching strategies

This comprehensive guide provides everything needed to successfully install and run RAGFlow from source code, covering everything from basic setup to production deployment and maintenance best practices.