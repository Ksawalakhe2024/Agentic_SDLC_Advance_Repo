# Local Setup Guide: Minimal MCP Server for AI‑Powered Task Data Injection (with Podman)

This guide provides a step-by-step approach for running the MCP Server locally on your computer using **Podman** for PostgreSQL containerization and Spring Boot for the backend application.

---

## 1. Prerequisites

**You’ll need:**
- Java 17+ installed (`java -version`)
- Podman (for PostgreSQL container)
- Maven or Gradle ([Install Maven](https://maven.apache.org/install.html))
- Git

### 1.1 Install Java 17+

#### Windows/Mac/Linux:
- Visit [Adoptium Downloads](https://adoptium.net/temurin/releases/?version=17) and install Java 17 LTS for your platform.
- Verify installation:
  ```bash
  java -version
  ```

### 1.2 Install Podman

#### Windows:
- Download and install from the [Podman Windows Installer](https://podman.io/getting-started/installation).
- After installation, open a new terminal and run:
  ```bash
  podman --version
  ```

#### Mac:
- If Homebrew is installed:
  ```bash
  brew install podman
  ```
- Initialize Podman machine:
  ```bash
  podman machine init
  podman machine start
  ```
- Verify:
  ```bash
  podman --version
  ```

#### Linux (Ubuntu/Debian):
  ```bash
  sudo apt-get -y update
  sudo apt-get -y install podman
  podman --version
  ```

#### Troubleshooting Podman
- If you see errors about the Podman machine, try `podman machine start` as needed.
- See [Podman Getting Started](https://podman.io/getting-started/installation) for more.

### 1.3 Install Maven (if needed)
- Download from [Maven Downloads](https://maven.apache.org/download.cgi) and follow instructions for your OS.
- Verify:
  ```bash
  mvn -version
  ```

---

## 2. Clone the Repository

```bash
git clone https://github.com/Ksawalakhe2024/Agentic_SDLC_Advance_Repo.git
cd Agentic_SDLC_Advance_Repo
```

---

## 3. Start PostgreSQL with Podman

**Run this command to start a local PostgreSQL instance:**

```bash
podman run --name mcp-postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=tasks_db \
  -p 5432:5432 \
  -d postgres:15
```

**Check that the container is running:**
```bash
podman ps
```

**(Optional) Stop and remove the container:**
```bash
podman stop mcp-postgres
podman rm mcp-postgres
```

---

## 4. Configure Application Properties

**Edit `src/main/resources/application.yml` (create if missing):**

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/tasks_db
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: update
    open-in-view: false
    properties:
      hibernate.jdbc.time_zone: UTC
      hibernate.jdbc.batch_size: 100
      hibernate.order_inserts: true
      hibernate.order_updates: true
server:
  port: 8081
logging:
  level:
    root: INFO
```

---

## 5. Build the Project

If using Maven (with wrapper):
```bash
./mvnw clean install
```
Or, if Maven is installed globally:
```bash
mvn clean install
```

If using Gradle:
```bash
./gradlew build
```

---

## 6. Run the Spring Boot Application

Start with Maven:
```bash
./mvnw spring-boot:run
```
Or (if Maven installed globally):
```bash
mvn spring-boot:run
```
Or with Gradle:
```bash
./gradlew bootRun
```

**The server should start on** [http://localhost:8081](http://localhost:8081)

---

## 7. Test Endpoints Manually

**Check the API is running:**
```bash
curl http://localhost:8081/mcp/help
```

**Expected JSON output:**
```json
{
  "description": "MCP Task Data Gateway",
  "endpoints": [
    "/mcp/help",
    "/mcp/schema/tasks",
    "/mcp/tasks (POST)",
    "/mcp/tasks/summary"
  ]
}
```

---

## 8. Insert Sample Tasks

**Create a test file (`sample_tasks.json`):**
```json
[
  {
    "title": "Test Task 1",
    "description": "First test task",
    "status": "NEW",
    "priority": 2,
    "dueDate": "2025-10-15T12:00:00Z"
  },
  {
    "title": "Test Task 2",
    "description": "Second test task",
    "status": "IN_PROGRESS",
    "priority": 1,
    "dueDate": "2025-10-20T09:00:00Z"
  }
]
```

**Insert tasks using curl:**
```bash
curl -X POST http://localhost:8081/mcp/tasks \
  -H "Content-Type: application/json" \
  --data-binary @sample_tasks.json
```

---

## 9. Get Summary

**Check summary:**
```bash
curl http://localhost:8081/mcp/tasks/summary
```

**Expected output:**
```json
{
  "total": 2,
  "byStatus": {
    "NEW": 1,
    "IN_PROGRESS": 1,
    "DONE": 0,
    "BLOCKED": 0
  },
  "latestInsertedAt": "2025-09-18T09:30:11Z"
}
```

---

## 10. Stop Services

**To stop PostgreSQL:**
```bash
podman stop mcp-postgres
```

**To remove the container (optional):**
```bash
podman rm mcp-postgres
```

---

## 11. Troubleshooting

- If the app can’t connect to the DB, ensure the Podman container is running and port 5432 is open.
- If you get validation errors, check your JSON matches the schema (`/mcp/schema/tasks`).
- For logs, see the output in your terminal.

---

## 12. Evidence and Next Steps

- Insert 1,000 tasks in batches (split into files of ≤500 tasks each).
- Use `/mcp/tasks/summary` for confirmation.
- Document your results using the template in the main README.

---

**You’re now ready to use the MCP server locally with Podman!**