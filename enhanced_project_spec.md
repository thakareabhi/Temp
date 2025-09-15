# Environment Automation & Monitoring System

## Project Overview
Create a comprehensive modular system that automates setup, validation, and monitoring of non-production environments. The system provides centralized orchestration with distributed execution across multiple servers, real-time monitoring, and a user-friendly dashboard for operations teams.

**Key Value Propositions:**
- Zero-downtime environment provisioning and validation
- Centralized visibility into distributed operations
- Dynamic configuration without code deployment
- Comprehensive audit trail and compliance reporting
- Scalable architecture supporting hundreds of servers

---

## System Architecture

### High-Level Components
```
┌─────────────────┐    ┌──────────────────────┐    ┌─────────────────────┐
│   React UI      │◄──►│  Central Backend     │◄──►│  Agent Services     │
│   Dashboard     │    │  (Orchestrator)      │    │  (Target Servers)   │
└─────────────────┘    └──────────────────────┘    └─────────────────────┘
                                │                           │
                                ▼                           │
                       ┌─────────────────┐                  │
                       │   MongoDB       │                  │
                       │   Database      │                  │
                       └─────────────────┘                  │
                                                           │
                       ┌─────────────────┐                  │
                       │   File System   │◄─────────────────┘
                       │   (Logs/Assets) │
                       └─────────────────┘
```

### Technology Stack
- **Backend Services:** Python 3.11+ with FastAPI
- **Frontend:** React 18+ with TypeScript
- **Database:** MongoDB 6.0+
- **Message Queue:** Redis (for job queuing and pub/sub)
- **Authentication:** JWT tokens with refresh mechanism
- **Logging:** Structured JSON logging with ELK stack integration
- **Containerization:** Docker with Docker Compose for development

---

## Detailed Requirements

### 1. Agent Service (Distributed on Target Servers)

**Core Responsibilities:**
- Execute diverse automation tasks with proper error handling
- Maintain persistent WebSocket connection with Central Backend
- Implement graceful degradation and retry mechanisms
- Provide comprehensive system introspection capabilities

**Supported Operations:**
```python
# Command execution with timeout and resource limits
{
    "type": "shell_command",
    "command": "systemctl status nginx",
    "timeout": 30,
    "working_directory": "/opt/app",
    "environment_vars": {"ENV": "staging"}
}

# File system operations
{
    "type": "file_validation",
    "path": "/data/exports/latest.sql",
    "checks": ["exists", "readable", "size_gt_mb:100", "modified_within:24h"]
}

# Service health checks
{
    "type": "service_health",
    "service_name": "postgresql",
    "health_endpoint": "http://localhost:5432/health",
    "expected_response": {"status": "healthy"}
}

# Database operations
{
    "type": "database_query",
    "connection_string": "postgresql://...",
    "query": "SELECT COUNT(*) FROM users WHERE created_at > NOW() - INTERVAL '1 day'",
    "expected_result_type": "scalar",
    "expected_range": {"min": 0, "max": 10000}
}
```

**Technical Specifications:**
- HTTP API with comprehensive OpenAPI documentation
- Automatic service registration with heartbeat mechanism
- Resource usage monitoring (CPU, memory, disk I/O)
- Configurable logging levels with log rotation
- Graceful shutdown handling with cleanup procedures
- Health check endpoint with detailed system metrics

### 2. Central Backend Service (Orchestration Hub)

**Core Responsibilities:**
- RESTful API with comprehensive CRUD operations
- Real-time WebSocket communication for live updates
- Distributed locking with automatic cleanup
- Job queuing and execution orchestration
- Comprehensive audit logging and compliance reporting

**API Endpoints Structure:**
```
# Server Management
GET    /api/v1/servers                    # List all servers
POST   /api/v1/servers                    # Register new server
PUT    /api/v1/servers/{id}               # Update server configuration
DELETE /api/v1/servers/{id}               # Deregister server
GET    /api/v1/servers/{id}/health        # Server health status
POST   /api/v1/servers/{id}/actions       # Execute action on server

# Action Management
GET    /api/v1/actions                    # List all action templates
POST   /api/v1/actions                    # Create action template
PUT    /api/v1/actions/{id}               # Update action template
DELETE /api/v1/actions/{id}               # Delete action template
GET    /api/v1/actions/{id}/validate      # Validate action configuration

# Execution Management
POST   /api/v1/executions                 # Start bulk execution
GET    /api/v1/executions/{id}            # Get execution status
DELETE /api/v1/executions/{id}            # Cancel execution
GET    /api/v1/executions/{id}/logs       # Get execution logs
GET    /api/v1/executions/history         # Execution history with filtering

# Real-time Communication
WebSocket /ws/executions/{id}             # Live execution updates
WebSocket /ws/servers/status              # Server status updates
```

**Advanced Features:**
- Distributed locking with TTL and automatic cleanup
- Job scheduling and queuing with priority support
- Rate limiting and circuit breaker patterns
- Comprehensive metrics collection (Prometheus format)
- Multi-tenant architecture support
- Advanced filtering and pagination for all list endpoints

### 3. Frontend Dashboard (Operations Interface)

**Core Features:**
- Responsive design supporting mobile and desktop
- Real-time updates without manual refresh
- Advanced filtering and search capabilities
- Bulk operations with confirmation dialogs
- Comprehensive error handling and user feedback

**Key Components:**

**Server Management:**
```typescript
interface ServerListProps {
  servers: Server[];
  onServerSelect: (servers: Server[]) => void;
  filters: {
    status: ServerStatus[];
    environment: string[];
    lastSeen: DateRange;
  };
}
```

**Action Builder:**
```typescript
interface ActionBuilder {
  template: ActionTemplate;
  parameters: Record<string, any>;
  validation: ValidationResult;
  preview: string;
}
```

**Execution Monitor:**
```typescript
interface ExecutionMonitor {
  execution: ExecutionStatus;
  realTimeLogs: LogEntry[];
  progress: ProgressInfo;
  metrics: ExecutionMetrics;
}
```

**UI/UX Requirements:**
- Dark/light theme support
- Keyboard shortcuts for power users
- Export capabilities (PDF, CSV, JSON)
- Advanced notification system
- Accessibility compliance (WCAG 2.1 AA)
- Progressive Web App (PWA) capabilities

---

## Enhanced Database Schema

### MongoDB Collections

```javascript
// servers collection - Enhanced with metadata and configuration
{
  "_id": "ObjectId(...)",
  "server_id": "nonprod-web-01",
  "hostname": "nonprod-web-01.company.local",
  "ip_address": "10.0.1.10",
  "environment": "staging", // dev, staging, uat, preprod
  "server_group": "web-tier",
  "tags": ["nginx", "php", "redis"],
  "capabilities": ["shell_execution", "file_operations", "service_management"],
  "configuration": {
    "agent_version": "1.2.0",
    "python_version": "3.11.2",
    "os_info": "Ubuntu 22.04.3 LTS",
    "max_concurrent_jobs": 5,
    "timeout_default": 300
  },
  "credentials": {
    "auth_token_hash": "sha256_hash",
    "certificate_fingerprint": "ssl_cert_fingerprint",
    "last_rotation": "ISODate(...)"
  },
  "status": {
    "current": "idle", // idle, busy, maintenance, unreachable, error
    "last_heartbeat": "ISODate(...)",
    "uptime_seconds": 86400,
    "load_average": [0.5, 0.7, 0.8],
    "memory_usage_percent": 65.2,
    "disk_usage_percent": 78.1
  },
  "metadata": {
    "created_at": "ISODate(...)",
    "created_by": "admin@company.com",
    "updated_at": "ISODate(...)",
    "last_successful_execution": "ISODate(...)"
  }
}

// action_templates collection - Comprehensive action definitions
{
  "_id": "ObjectId(...)",
  "name": "Database Refresh Validation",
  "description": "Validates database refresh completion and data integrity",
  "category": "database", // system, database, application, network, security
  "action_type": "validation", // validation, execution, maintenance
  "version": "1.0.0",
  "parameters": {
    "database_name": {
      "type": "string",
      "required": true,
      "description": "Target database name",
      "validation": "^[a-zA-Z][a-zA-Z0-9_]*$"
    },
    "expected_table_count": {
      "type": "integer",
      "required": false,
      "default": null,
      "description": "Expected number of tables"
    }
  },
  "execution_config": {
    "timeout_seconds": 300,
    "retry_count": 3,
    "retry_delay_seconds": 10,
    "requires_lock": true,
    "parallel_execution": false
  },
  "command_template": "python3 /opt/scripts/db_validate.py --db={database_name} --tables={expected_table_count}",
  "success_criteria": {
    "exit_code": 0,
    "output_contains": ["VALIDATION_PASSED"],
    "output_not_contains": ["ERROR", "FAILED"],
    "execution_time_max": 180
  },
  "applicable_server_tags": ["database", "postgresql"],
  "metadata": {
    "created_at": "ISODate(...)",
    "created_by": "admin@company.com",
    "updated_at": "ISODate(...)",
    "usage_count": 145
  }
}

// executions collection - Comprehensive execution tracking
{
  "_id": "ObjectId(...)",
  "execution_id": "exec_20241215_001",
  "batch_id": "batch_20241215_staging_refresh",
  "initiated_by": "john.doe@company.com",
  "execution_type": "bulk", // single, bulk, scheduled
  "target_servers": ["ObjectId(...)", "ObjectId(...)"],
  "action_template_id": "ObjectId(...)",
  "parameters": {
    "database_name": "staging_db",
    "expected_table_count": 50
  },
  "status": {
    "overall": "running", // pending, running, completed, failed, cancelled
    "per_server": {
      "server_01": {"status": "completed", "progress": 100},
      "server_02": {"status": "running", "progress": 75}
    }
  },
  "timing": {
    "created_at": "ISODate(...)",
    "started_at": "ISODate(...)",
    "estimated_completion": "ISODate(...)",
    "completed_at": null,
    "duration_seconds": null
  },
  "results": {
    "success_count": 1,
    "failure_count": 0,
    "total_count": 2,
    "details": [
      {
        "server_id": "server_01",
        "status": "success",
        "exit_code": 0,
        "output": "VALIDATION_PASSED: 50 tables verified",
        "error_output": "",
        "execution_time": 45.2,
        "resource_usage": {"cpu_percent": 25.5, "memory_mb": 128}
      }
    ]
  },
  "logs": {
    "storage_path": "/logs/executions/exec_20241215_001.jsonl",
    "log_level": "INFO",
    "entries_count": 156,
    "size_bytes": 45678
  }
}

// server_locks collection - Advanced locking mechanism
{
  "_id": "ObjectId(...)",
  "server_id": "ObjectId(...)",
  "lock_type": "execution", // execution, maintenance, emergency
  "locked_by": {
    "process_id": "backend_worker_1",
    "execution_id": "exec_20241215_001",
    "user_id": "admin@company.com"
  },
  "lock_metadata": {
    "reason": "Database refresh validation",
    "priority": "high",
    "estimated_duration_minutes": 15
  },
  "timing": {
    "acquired_at": "ISODate(...)",
    "expires_at": "ISODate(...)",
    "max_extension_count": 3,
    "current_extensions": 0
  },
  "status": "active" // active, released, expired, force_released
}

// audit_logs collection - Comprehensive audit trail
{
  "_id": "ObjectId(...)",
  "timestamp": "ISODate(...)",
  "event_type": "execution_started", // server_registered, action_created, execution_started, etc.
  "actor": {
    "type": "user", // user, system, agent
    "id": "admin@company.com",
    "ip_address": "192.168.1.100",
    "user_agent": "Mozilla/5.0..."
  },
  "resource": {
    "type": "execution",
    "id": "exec_20241215_001",
    "name": "Database Refresh Validation"
  },
  "details": {
    "affected_servers": ["server_01", "server_02"],
    "parameters": {"database_name": "staging_db"},
    "metadata": {"batch_id": "batch_20241215_staging_refresh"}
  },
  "result": {
    "success": true,
    "error_message": null,
    "affected_count": 2
  }
}
```

---

## Implementation Roadmap

### Phase 1: Core Infrastructure (Weeks 1-2)
1. **Database Setup & Schema**
   - MongoDB installation and configuration
   - Index creation for performance optimization
   - Data validation rules and constraints

2. **Agent Service Foundation**
   - Basic FastAPI application structure
   - Command execution framework with security sandboxing
   - Registration and heartbeat mechanisms
   - Comprehensive logging setup

3. **Central Backend Core**
   - FastAPI application with dependency injection
   - Database connection and ORM setup
   - Basic CRUD operations for servers and actions
   - Authentication and authorization framework

### Phase 2: Core Features (Weeks 3-4)
1. **Agent Enhancement**
   - Command execution with timeout and resource limits
   - File system operations with permission checks
   - Service management capabilities
   - WebSocket connection for real-time updates

2. **Backend Enhancement**
   - Execution orchestration engine
   - Distributed locking mechanism with Redis
   - WebSocket endpoints for real-time communication
   - Job queuing and processing

3. **Frontend Foundation**
   - React application setup with TypeScript
   - Component library and design system
   - Server list and status display
   - Basic action management interface

### Phase 3: Advanced Features (Weeks 5-6)
1. **Advanced Execution Features**
   - Bulk operations with progress tracking
   - Execution templates and parameterization
   - Scheduling and recurring executions
   - Advanced error handling and retry logic

2. **Dashboard Enhancement**
   - Real-time log streaming interface
   - Advanced filtering and search capabilities
   - Execution history and reporting
   - Export and notification features

3. **Monitoring and Observability**
   - Metrics collection and dashboards
   - Health check endpoints
   - Performance monitoring
   - Alerting integration

### Phase 4: Production Readiness (Weeks 7-8)
1. **Security Hardening**
   - Comprehensive input validation
   - Rate limiting and DDoS protection
   - Security headers and CORS configuration
   - Vulnerability scanning and remediation

2. **Performance Optimization**
   - Database query optimization
   - Caching strategy implementation
   - Connection pooling and resource management
   - Load testing and performance tuning

3. **Deployment and Operations**
   - Docker containerization
   - CI/CD pipeline setup
   - Environment configuration management
   - Backup and disaster recovery procedures

---

## Configuration Examples

### Agent Configuration
```yaml
# agent_config.yaml
agent:
  server_id: "nonprod-web-01"
  backend_url: "https://automation.company.local"
  auth_token: "${AGENT_AUTH_TOKEN}"
  
execution:
  max_concurrent_jobs: 3
  default_timeout: 300
  temp_directory: "/tmp/automation"
  log_level: "INFO"
  
security:
  allowed_commands: ["/usr/bin/*", "/opt/scripts/*"]
  forbidden_paths: ["/etc/passwd", "/root/*"]
  resource_limits:
    cpu_percent: 80
    memory_mb: 512
    execution_time: 600

monitoring:
  heartbeat_interval: 30
  metrics_collection: true
  health_check_port: 8081
```

### Backend Configuration
```yaml
# backend_config.yaml
database:
  mongodb_url: "${MONGODB_URL}"
  database_name: "automation_system"
  connection_pool_size: 50

redis:
  url: "${REDIS_URL}"
  job_queue: "automation_jobs"
  pubsub_channel: "automation_events"

security:
  jwt_secret: "${JWT_SECRET}"
  token_expiry_hours: 8
  refresh_token_days: 30
  
execution:
  max_concurrent_executions: 100
  default_timeout: 300
  lock_ttl_minutes: 30
  
logging:
  level: "INFO"
  format: "json"
  file_path: "/var/log/automation/backend.log"
  max_file_size: "100MB"
  retention_days: 90
```

---

## Development Guidelines

### Code Quality Standards
- **Test Coverage:** Minimum 80% code coverage
- **Type Safety:** Full TypeScript for frontend, type hints for Python
- **Documentation:** Comprehensive API documentation with examples
- **Error Handling:** Graceful error handling with user-friendly messages
- **Security:** Security-first approach with regular vulnerability scans

### Development Workflow
1. Feature branches with descriptive names
2. Pull request reviews with automated testing
3. Integration testing in staging environment
4. Production deployment with rollback capabilities
5. Post-deployment monitoring and validation

### Performance Requirements
- **API Response Time:** < 200ms for 95th percentile
- **Concurrent Users:** Support 50+ simultaneous users
- **Server Capacity:** Support 500+ managed servers
- **Real-time Updates:** < 1 second latency for live updates
- **Uptime:** 99.9% availability target

---

## Deliverables Checklist

### Code Deliverables
- [ ] Complete Agent Service implementation
- [ ] Complete Central Backend implementation  
- [ ] Complete Frontend Dashboard implementation
- [ ] Database migration scripts and seed data
- [ ] Docker configurations and docker-compose setup
- [ ] Comprehensive test suites for all components

### Documentation Deliverables
- [ ] Complete README with setup instructions
- [ ] API documentation with Swagger/OpenAPI
- [ ] User manual for dashboard operations
- [ ] Administrator guide for deployment and maintenance
- [ ] Security documentation and best practices
- [ ] Troubleshooting guide with common issues

### Configuration Deliverables
- [ ] Environment-specific configuration templates
- [ ] Sample action templates for common operations
- [ ] Monitoring and alerting configurations
- [ ] Backup and recovery procedures
- [ ] Performance tuning recommendations

This enhanced specification provides a comprehensive blueprint for building a production-ready environment automation system. The detailed requirements, technical specifications, and implementation roadmap should enable Devin to create a robust, scalable solution that meets all operational requirements.