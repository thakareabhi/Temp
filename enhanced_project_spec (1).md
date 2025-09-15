# Simple Environment Automation & Monitoring System

## Project Overview
Create a basic modular system that automates setup and validation tasks on non-production environments. The system provides a simple dashboard to manage servers, execute commands, and view real-time logs.

**Key Goals:**
- Simple server management and command execution
- Real-time log viewing during command execution
- Easy-to-use web interface for operations
- Minimal setup and maintenance overhead

---

## System Architecture

### High-Level Components
```
┌─────────────────┐    ┌──────────────────────┐    ┌─────────────────────┐
│   React UI      │◄──►│  Central Backend     │◄──►│  Agent Services     │
│   Dashboard     │    │  (FastAPI)           │    │  (Target Servers)   │
└─────────────────┘    └──────────────────────┘    └─────────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   MongoDB       │
                       │   Database      │
                       └─────────────────┘
```

### Technology Stack
- **Backend & Agent:** Python 3.11+ with FastAPI
- **Frontend:** React 18+
- **Database:** MongoDB
- **Real-time Communication:** WebSockets

---

## Component Requirements

### 1. Agent Service (Deployed on each target server)

**Core Features:**
- Execute shell commands and scripts
- Check file/folder existence and permissions
- Validate service status
- Check database connectivity
- Stream command output in real-time
- Simple heartbeat mechanism

**Supported Operations:**
```python
# Shell command execution
{
    "type": "shell_command",
    "command": "systemctl status nginx",
    "timeout": 30
}

# File validation
{
    "type": "file_check",
    "path": "/data/exports/latest.sql",
    "check_type": "exists"  # exists, readable, size
}

# Service check
{
    "type": "service_check",
    "service_name": "postgresql"
}
```

**Simple API Endpoints:**
- `POST /execute` - Execute a command
- `GET /health` - Agent health status
- `POST /register` - Register with central backend

### 2. Central Backend Service

**Core Features:**
- Simple REST API for CRUD operations
- WebSocket for real-time log streaming
- Basic server registration and management
- Command execution orchestration

**API Endpoints:**
```
# Server Management
GET    /api/servers                       # List all servers
POST   /api/servers                       # Add new server
DELETE /api/servers/{id}                  # Remove server
GET    /api/servers/{id}/status           # Server status

# Action Management
GET    /api/actions                       # List all actions
POST   /api/actions                       # Create new action
PUT    /api/actions/{id}                  # Update action
DELETE /api/actions/{id}                  # Delete action

# Execution
POST   /api/execute                       # Execute action on servers
GET    /api/executions/{id}/logs          # Get execution logs

# Real-time
WebSocket /ws/logs/{execution_id}         # Live log streaming
```

### 3. Frontend Dashboard

**Key Pages:**
- **Servers Page:** List servers with status (online/offline)
- **Actions Page:** Manage action templates
- **Execute Page:** Select servers and actions, view live logs

**Simple UI Components:**
- Server list with add/remove functionality
- Action builder form
- Server selection checkboxes
- Live log viewer with WebSocket connection

---

## Simplified Database Schema

### MongoDB Collections

```javascript
// servers collection
{
  "_id": "ObjectId(...)",
  "name": "nonprod-web-01",
  "hostname": "nonprod-web-01.local",
  "ip_address": "10.0.1.10",
  "port": 8080,
  "status": "online",        // online, offline
  "last_heartbeat": "ISODate(...)",
  "created_at": "ISODate(...)"
}

// actions collection
{
  "_id": "ObjectId(...)",
  "name": "Check NGINX Status",
  "description": "Verify NGINX service is running",
  "command": "systemctl status nginx",
  "timeout": 30,
  "created_at": "ISODate(...)"
}

// execution_logs collection
{
  "_id": "ObjectId(...)",
  "execution_id": "exec_123",
  "server_id": "ObjectId(...)",
  "action_id": "ObjectId(...)",
  "status": "completed",     // running, completed, failed
  "output": "● nginx.service - nginx...",
  "error": "",
  "started_at": "ISODate(...)",
  "completed_at": "ISODate(...)"
}
```

---

## Step-by-Step Implementation Tasks for Devin

### Task 1: Project Structure Setup
**Objective:** Create the complete project directory structure and configuration files

**Deliverables:**
- Create folder structure as defined in "File Structure" section
- Generate `requirements.txt` for both agent and backend with exact dependencies:
  - FastAPI, uvicorn, motor (MongoDB async driver), websockets, requests
- Create `package.json` for frontend with React, axios, and websocket dependencies
- Create basic `docker-compose.yml` file for MongoDB
- Create `.gitignore` files for Python and Node.js
- Create placeholder `README.md` with setup instructions

### Task 2: Database Models and Connection
**Objective:** Set up MongoDB connection and data models

**Deliverables:**
- Create `backend/database.py` with MongoDB connection using motor
- Create `backend/models.py` with Pydantic models for:
  - Server model (name, hostname, ip_address, port, status, last_heartbeat)
  - Action model (name, description, command, timeout)
  - ExecutionLog model (execution_id, server_id, action_id, status, output, error, timestamps)
- Test database connection and model validation

### Task 3: Agent Service Implementation
**Objective:** Build the agent service that runs on target servers

**Deliverables:**
- Create `agent/main.py` with FastAPI application
- Implement `agent/executor.py` with command execution logic:
  - Execute shell commands with timeout
  - Capture stdout/stderr output
  - Handle command failures gracefully
- Implement three API endpoints:
  - `POST /execute` - Execute command and return result
  - `GET /health` - Return agent status and system info
  - `POST /register` - Register with central backend
- Add basic logging configuration
- Test agent independently with sample commands

### Task 4: Backend Server Management API
**Objective:** Create backend API for server management

**Deliverables:**
- Create `backend/main.py` with FastAPI application
- Create `backend/routes/servers.py` with endpoints:
  - `GET /api/servers` - List all servers
  - `POST /api/servers` - Add new server (with validation)
  - `DELETE /api/servers/{id}` - Remove server
  - `GET /api/servers/{id}/status` - Get server status
- Implement server registration logic
- Add basic input validation and error handling
- Test all server management endpoints with sample data

### Task 5: Backend Action Management API
**Objective:** Create backend API for action templates

**Deliverables:**
- Create `backend/routes/actions.py` with endpoints:
  - `GET /api/actions` - List all actions
  - `POST /api/actions` - Create new action
  - `PUT /api/actions/{id}` - Update action
  - `DELETE /api/actions/{id}` - Delete action
- Add validation for command templates
- Test all action management endpoints
- Create sample action templates for common operations

### Task 6: Command Execution Engine
**Objective:** Implement the core execution logic that orchestrates commands across servers

**Deliverables:**
- Create `backend/routes/executions.py` with endpoints:
  - `POST /api/execute` - Execute action on selected servers
  - `GET /api/executions/{id}/logs` - Get execution logs
- Implement execution orchestration logic:
  - Send commands to multiple agents simultaneously
  - Collect and store results in execution_logs collection
  - Handle agent communication failures
- Generate unique execution IDs
- Test execution with multiple servers and actions

### Task 7: WebSocket Real-time Logging
**Objective:** Add real-time log streaming capability

**Deliverables:**
- Add WebSocket endpoint to backend: `/ws/logs/{execution_id}`
- Modify execution logic to stream logs in real-time during command execution
- Implement WebSocket connection management (connect/disconnect handling)
- Test real-time log streaming with sample executions
- Ensure proper cleanup of WebSocket connections

### Task 8: Frontend Project Setup
**Objective:** Create React application foundation

**Deliverables:**
- Initialize React application in `frontend/` directory
- Set up basic routing with React Router for three main pages
- Create `src/api/client.js` for backend API communication using axios
- Create `src/components/` directory with placeholder components
- Set up basic CSS styling (can use a simple CSS framework like Bootstrap)
- Create `src/App.js` with navigation layout
- Test basic routing and API connection

### Task 9: Server Management UI
**Objective:** Build the server management interface

**Deliverables:**
- Create `src/pages/Servers.js` page component
- Create `src/components/ServerList.js` component to display servers table
- Create `src/components/AddServerForm.js` component with form validation
- Implement functionality:
  - Display servers with status indicators (online/offline)
  - Add new servers with hostname, IP, port validation
  - Delete servers with confirmation dialog
  - Show last heartbeat time
- Test all server management operations

### Task 10: Action Management UI
**Objective:** Build the action template management interface

**Deliverables:**
- Create `src/pages/Actions.js` page component
- Create `src/components/ActionList.js` component to display actions
- Create `src/components/ActionForm.js` for creating/editing actions
- Implement functionality:
  - List all action templates
  - Create new actions with command and timeout
  - Edit existing actions
  - Delete actions with confirmation
- Add form validation for required fields
- Test all action management operations

### Task 11: Execution Interface
**Objective:** Build the main execution interface with real-time logs

**Deliverables:**
- Create `src/pages/Execute.js` page component
- Create `src/components/ServerSelector.js` with checkboxes for server selection
- Create `src/components/ActionSelector.js` dropdown for action selection
- Create `src/components/LogViewer.js` for real-time log display
- Implement WebSocket connection for live log streaming
- Add execution controls (start execution, view progress)
- Test complete execution workflow with real-time log updates

### Task 12: Integration Testing and Polish
**Objective:** Complete end-to-end integration and basic error handling

**Deliverables:**
- Test complete workflow: add servers → create actions → execute → view logs
- Add error handling for common scenarios:
  - Agent unreachable
  - Command execution failures
  - WebSocket connection issues
- Add loading states and user feedback messages
- Create sample data and test scenarios
- Update README.md with complete setup and usage instructions
- Fix any bugs found during integration testing

### Task 13: Deployment Configuration
**Objective:** Make the application ready for deployment

**Deliverables:**
- Update `docker-compose.yml` to include all services (MongoDB, Backend, Agent, Frontend)
- Create environment configuration files
- Add health check endpoints for all services
- Create startup scripts for each component
- Test deployment using Docker Compose
- Document deployment process and troubleshooting steps

---

## Validation Criteria for Each Task

Each task should be considered complete when:
1. **Code Quality:** All code runs without errors and follows basic Python/React conventions
2. **Functionality:** All specified endpoints/components work as described
3. **Testing:** Manual testing confirms the feature works end-to-end
4. **Documentation:** Basic inline comments and docstrings are added
5. **Integration:** The component integrates properly with existing code

## Dependencies Between Tasks
- Tasks 1-2 must be completed first (foundation)
- Tasks 3-7 can be done in parallel (backend services)
- Task 8 depends on Tasks 1-2 (frontend foundation)
- Tasks 9-11 depend on Task 8 and respective backend tasks
- Tasks 12-13 require all previous tasks to be completed

---

## File Structure

```
project/
├── agent/
│   ├── main.py                 # FastAPI agent app
│   ├── executor.py             # Command execution logic
│   └── requirements.txt
├── backend/
│   ├── main.py                 # FastAPI backend app
│   ├── database.py             # MongoDB connection
│   ├── models.py               # Data models
│   ├── routes/
│   │   ├── servers.py
│   │   ├── actions.py
│   │   └── executions.py
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── ServerList.js
│   │   │   ├── ActionForm.js
│   │   │   └── ExecutionView.js
│   │   ├── pages/
│   │   │   ├── Servers.js
│   │   │   ├── Actions.js
│   │   │   └── Execute.js
│   │   └── App.js
│   └── package.json
├── docker-compose.yml          # For local development
└── README.md
```

---

## Basic Configuration

### Agent Configuration (agent/config.py)
```python
BACKEND_URL = "http://localhost:8000"
AGENT_PORT = 8080
HEARTBEAT_INTERVAL = 60  # seconds
COMMAND_TIMEOUT = 300    # seconds
```

### Backend Configuration (backend/config.py)
```python
MONGODB_URL = "mongodb://localhost:27017"
DATABASE_NAME = "automation"
WEBSOCKET_TIMEOUT = 300
```

---

## Example Usage Flow

1. **Add Server:** Use the dashboard to add a new server (hostname, IP, port)
2. **Create Action:** Define a command template (e.g., "systemctl status nginx")
3. **Execute:** Select servers and action, click execute
4. **Monitor:** Watch real-time logs in the dashboard
5. **Review:** Check completion status and output

---

## Development Guidelines

### Keep It Simple
- No authentication/authorization
- No complex locking mechanisms
- No execution history/audit trails
- No advanced scheduling
- No user management
- Basic error handling only
- Simple logging to console/files

### Focus Areas
- Reliable command execution
- Real-time log streaming
- Simple, clean UI
- Easy setup and deployment
- Basic error recovery

This simplified version removes all the enterprise features while maintaining the core functionality you need for environment automation and monitoring.