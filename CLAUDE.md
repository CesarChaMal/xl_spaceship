# CLAUDE.md - XL Spaceship Project Guide for AI Assistants

> **Last Updated:** 2025-11-13
> **Project:** XL Spaceship - Distributed Battleship Game System
> **Architecture:** Microservices with Spring Boot

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Repository Structure](#repository-structure)
4. [Technology Stack](#technology-stack)
5. [Development Setup](#development-setup)
6. [Key Components](#key-components)
7. [Conventions and Patterns](#conventions-and-patterns)
8. [Common Tasks](#common-tasks)
9. [Important Notes](#important-notes)

---

## Project Overview

**XL Spaceship** is a distributed multiplayer battleship-style game built with a microservices architecture. The system allows users to create games, place spaceships on 16x16 game boards, and engage in turn-based combat by firing salvos at opponents.

### Core Features
- 16x16 game boards for ship placement
- Multiple spaceship shapes and configurations
- Salvo-based combat system
- Auto-pilot gameplay mode
- Multi-instance spaceship services
- Web-based user interface
- RESTful API with Swagger documentation

---

## Architecture

### Microservices Design

The project follows a **microservices architecture** with three independent backend services:

```
┌─────────────────────────────────────────────────────────────┐
│                    Web Client (8087)                         │
│              Thymeleaf + Spring Security                     │
└───────────────────┬─────────────────────────────────────────┘
                    │ (uses all SDKs)
        ┌───────────┼───────────┐
        │           │           │
        ▼           ▼           ▼
┌─────────────┐ ┌─────────────┐ ┌──────────────────┐
│   User      │ │  Protocol   │ │  Spaceship       │
│  Service    │ │  Service    │ │  Service         │
│  (8084)     │ │  (8083)     │ │  (8080-8082)     │
└──────┬──────┘ └──────┬──────┘ └────────┬─────────┘
       │               │                  │
       ▼               ▼                  ▼
┌─────────────┐ ┌─────────────┐ ┌──────────────────┐
│   MySQL     │ │   MySQL     │ │     MySQL        │
│ user schema │ │protocol sch.│ │ spaceship schema │
└─────────────┘ └─────────────┘ └──────────────────┘
```

### Service Responsibilities

| Service | Port | Database | Purpose |
|---------|------|----------|---------|
| **XL-spaceship** | 8080-8082 | spaceship | Game board management, ship placement, salvo handling |
| **XL-spaceship-protocal** | 8083 | protocol | Game orchestration, turn management, auto-pilot |
| **XL-spaceship-user** | 8084 | user | User management, authentication, user challenges |
| **spaceship_client** | 8087 | N/A | Web UI for game interaction |

### Communication Pattern
- Services communicate via **REST APIs**
- Each service has a **dedicated SDK** (Java client library)
- Client application uses **all three SDKs** for service integration
- **Database-per-service** pattern for data isolation

---

## Repository Structure

```
xl_spaceship/
│
├── XL-spaceship/              # Spaceship service (game boards)
│   ├── src/main/
│   │   ├── java/com/spaceship/
│   │   │   ├── resources/     # REST controllers
│   │   │   ├── services/      # Business logic
│   │   │   ├── repositories/  # Data access layer
│   │   │   ├── model/         # JPA entities, VOs, constants
│   │   │   ├── validator/     # Request validators
│   │   │   └── util/          # Utility classes
│   │   ├── resources/
│   │   │   ├── application.yml
│   │   │   └── log4j.properties
│   │   └── docker/            # Docker configuration
│   ├── docker-compose.yml     # Multi-container setup
│   ├── .env                   # Environment variables
│   └── pom.xml                # Maven configuration
│
├── XL-spaceship-protocal/     # Protocol service (game logic)
│   └── [same structure as XL-spaceship]
│
├── XL-spaceship-user/         # User service
│   └── [same structure as XL-spaceship]
│
├── spaceship_client/          # Web UI client
│   ├── src/main/
│   │   ├── java/com/spaceship/client/
│   │   │   ├── controller/    # MVC controllers
│   │   │   ├── vo/            # View objects
│   │   │   └── util/          # Utilities
│   │   └── resources/
│   │       ├── templates/     # Thymeleaf templates
│   │       ├── static/        # CSS, JS, images
│   │       └── application.properties
│   └── pom.xml
│
├── spaceship-sdk/             # Spaceship API client SDK
│   ├── src/main/java/io/swagger/client/
│   ├── docs/                  # API documentation
│   └── pom.xml
│
├── protocal_sdk/              # Protocol API client SDK
│   └── [same structure as spaceship-sdk]
│
├── user-sdk/                  # User API client SDK
│   └── [same structure as spaceship-sdk]
│
├── XLSpaceship Game Guide.pdf # Game rules and documentation
└── README.md                  # Basic project info
```

---

## Technology Stack

### Backend Services
- **Java 8** - Programming language
- **Spring Boot 1.5.3** - Application framework
- **Spring Data JPA** - Object-relational mapping
- **Spring Data REST** - RESTful web services
- **MySQL 5.6+** - Relational database
- **Hibernate** - JPA implementation
- **Log4j** - Logging framework

### API & Documentation
- **Swagger 2.0** - API specification
- **Springfox 2.2.2** - Swagger integration
- **Swagger UI** - Interactive API documentation (available at `/swagger-ui.html`)

### Frontend (Client)
- **Thymeleaf** - Server-side templating engine
- **Spring Security** - Authentication & authorization
- **Bootstrap CSS** - UI framework
- **JavaScript** - Client-side interactivity
- **HTML5/CSS3** - Markup and styling

### SDK Libraries
- **Swagger Codegen** - Auto-generated API clients
- **OkHttp 2.7.5** - HTTP client
- **Gson 2.6.2** - JSON serialization
- **Joda-Time 2.9.3** - Date/time handling

### Build & Deployment
- **Maven** - Build automation and dependency management
- **Maven Wrapper** - Version-consistent builds
- **Docker** - Containerization (Spotify maven plugin 0.4.11)
- **Docker Compose** - Multi-container orchestration

### Testing
- **JUnit 4.12** - Unit testing framework
- **Spring Boot Test** - Integration testing

---

## Development Setup

### Prerequisites
```bash
# Required
- Java 8 JDK
- Maven 3.x
- MySQL 5.6+ (or Docker)
- Docker & Docker Compose (optional, for containerized deployment)

# Recommended
- IDE: IntelliJ IDEA, Eclipse, or VS Code with Java extensions
- Postman or similar tool for API testing
```

### Local Development Setup

#### 1. Database Setup

**Option A: Local MySQL**
```bash
# Install MySQL and create databases
mysql -u root -p

CREATE DATABASE spaceship;
CREATE DATABASE protocol;
CREATE DATABASE user;

# Create user and grant permissions
CREATE USER 'dbuser'@'localhost' IDENTIFIED BY 'dbp4ss';
GRANT ALL PRIVILEGES ON spaceship.* TO 'dbuser'@'localhost';
GRANT ALL PRIVILEGES ON protocol.* TO 'dbuser'@'localhost';
GRANT ALL PRIVILEGES ON user.* TO 'dbuser'@'localhost';
FLUSH PRIVILEGES;
```

**Option B: Docker MySQL**
```bash
cd XL-spaceship
docker-compose -f docker-compose-database.yml up -d
# Database will be available on localhost:3307
```

#### 2. Build SDKs First

SDKs must be built and installed to local Maven repository before building services:

```bash
# Build and install spaceship-sdk
cd spaceship-sdk
mvn clean install

# Build and install protocal-sdk
cd ../protocal_sdk
mvn clean install

# Build and install user-sdk
cd ../user-sdk
mvn clean install
```

#### 3. Build and Run Services

**Spaceship Service** (run 1-3 instances on ports 8080-8082)
```bash
cd XL-spaceship
mvn clean package
java -jar target/spaceship-0.0.1-SNAPSHOT.jar --server.port=8080
# For additional instances, use ports 8081, 8082
```

**Protocol Service**
```bash
cd XL-spaceship-protocal
mvn clean package
java -jar target/XL-spaceship-protocal-0.0.1-SNAPSHOT.jar
# Runs on port 8083
```

**User Service**
```bash
cd XL-spaceship-user
mvn clean package
java -jar target/XL-spaceship-user-0.0.1-SNAPSHOT.jar
# Runs on port 8084
```

**Web Client**
```bash
cd spaceship_client
mvn clean package
java -jar target/spaceship_client-0.0.1-SNAPSHOT.jar
# Runs on port 8087
# Access at http://localhost:8087
```

#### 4. Docker Deployment

**Build Docker Images**
```bash
cd XL-spaceship
mvn clean package docker:build
```

**Run with Docker Compose**
```bash
cd XL-spaceship
# Edit .env file if needed for custom database credentials
docker-compose up -d

# Services will start:
# - MySQL on port 3307
# - spaceship1 on port 8080
# - spaceship2 on port 8081
# - spaceship3 on port 8082
```

### Environment Configuration

**Database Configuration (.env)**
```bash
MYSQL_ROOT_PASSWORD=p4SSW0rd
MYSQL_DATABASE=spaceship
MYSQL_USER=dbuser
MYSQL_PASSWORD=dbp4ss
MYSQL_HOST=mysql-server  # Use 'localhost' for local dev
MYSQL_PORT=3306
```

**Application Profiles**
- `application.yml` - Default configuration
- `application-container.yml` - Docker container configuration
- Activate profile: `--spring.profiles.active=container`

---

## Key Components

### 1. XL-spaceship Service

**Base Path:** `/xl-spaceship/spaceships`

**Key Endpoints:**
```
POST   /placespaceship           - Place spaceship on board
GET    /getspaceshipsshapes      - Get available spaceship shapes
PUT    /                         - Receive incoming salvo shots
PUT    /lockboard                - Lock game board (prevent modifications)
PUT    /unlockboard              - Unlock game board
POST   /initSpaceship            - Initialize spaceship instance
GET    /getByUserId              - Get spaceship by user ID
GET    /                         - Get all spaceships
```

**Domain Model:**
```java
@Entity
@Table(name = "spaceship")
public class Spaceship {
    private String spaceshipId;
    private String userId;
    private String spaceshipStatus;
    private String spaceshipAddress;
    // 16x16 game board stored as serialized data
}
```

**Important Files:**
- `SpaceshipResource.java` - Main REST controller (XL-spaceship/src/main/java/com/spaceship/resources/)
- `SpaceshipService.java` - Business logic (XL-spaceship/src/main/java/com/spaceship/services/)
- `SpaceshipRepository.java` - Data access (XL-spaceship/src/main/java/com/spaceship/repositories/)

### 2. XL-spaceship-protocal Service

**Base Path:** `/xl-spaceship/protocol`

**Key Endpoints:**
```
POST   /game/new                              - Create new game
GET    /game/{id}                             - Get game status
GET    /game/getallgames/{id}                 - Get all games for user
PUT    /game/enableautopilot/{userId}/{gameId} - Enable auto-pilot
PUT    /game/disableautopilot/{userId}/{gameId} - Disable auto-pilot
GET    /game/getAutoPilotStatus/{userId}/{gameId} - Check auto-pilot status
PUT    /fire                                  - Fire salvo at opponent
```

**Domain Model:**
```java
@Entity
public class Game {
    private String gameId;
    private String gameName;
    private String playerTurn;
    private String winner;
    private String gameStatus;

    @OneToMany(mappedBy = "game")
    private List<Player> players;
}

@Entity
public class Player {
    private String userId;
    private String fullName;
    private String spaceshipProtocol;

    @ManyToOne
    private Game game;
}
```

**Important Files:**
- `GameResource.java` - Game management controller (XL-spaceship-protocal/src/main/java/com/spaceship/protocol/resources/)
- `GameService.java` - Game orchestration logic (XL-spaceship-protocal/src/main/java/com/spaceship/protocol/services/)

### 3. XL-spaceship-user Service

**Base Path:** `/xl-spaceship/user`

**Key Endpoints:**
```
GET    /getUserByUserName        - Get user by username
GET    /getAllUsers              - Get all registered users
GET    /getAllOpponentUsers      - Get potential opponents
POST   /createUser               - Register new user
POST   /challengeUser            - Challenge another user to a game
POST   /assignSpaceship          - Assign spaceship to user
GET    /getAllSpaceships         - Get all spaceships
```

**Domain Model:**
```java
@Entity
public class User {
    private String userId;
    private String userName;
    private String email;
    // Additional user fields
}
```

**Important Files:**
- `UserResource.java` - User management controller (XL-spaceship-user/src/main/java/com/spaceship/user/resources/)
- `UserService.java` - User business logic (XL-spaceship-user/src/main/java/com/spaceship/user/services/)

### 4. spaceship_client (Web UI)

**Port:** 8087

**Key Pages:**
- `/` - Welcome page
- `/user/login` - User login
- `/user/create` - User registration
- `/user/list` - List all users
- `/spaceship/board` - View game board
- `/spaceship/list` - List spaceships
- `/game/list` - List games
- `/game/fire` - Fire salvo interface
- `/game/status` - View game status

**Technology:**
- Thymeleaf templates in `src/main/resources/templates/`
- Static resources in `src/main/resources/static/`
- Spring Security for authentication
- Bootstrap CSS for styling

---

## Conventions and Patterns

### Code Organization

**Package Structure (All Services):**
```
com.spaceship.[service]/
├── [Service]Application.java    # Main Spring Boot application class
├── resources/                    # REST controllers (@RestController)
├── services/                     # Business logic (@Service)
├── repositories/                 # Data access (@Repository, extends JpaRepository)
├── model/
│   ├── entities/                # JPA entities (@Entity)
│   ├── vo/                      # Value Objects / DTOs
│   └── constants/               # Enums, constants
├── validator/                    # Request validators (@InitBinder)
├── util/                        # Utility classes
└── exception/                    # Custom exception classes
```

### Naming Conventions

1. **Classes:**
   - Resources (Controllers): `{Entity}Resource.java` (e.g., `SpaceshipResource.java`)
   - Services: `{Entity}Service.java` (e.g., `GameService.java`)
   - Repositories: `{Entity}Repository.java` (e.g., `UserRepository.java`)
   - Entities: `{Entity}.java` (e.g., `Spaceship.java`)
   - Value Objects: `{Entity}Vo.java` or `{Purpose}Request/Response.java`

2. **Endpoints:**
   - Base path includes service name: `/xl-spaceship/{service-name}/`
   - Use lowercase with hyphens or camelCase
   - RESTful conventions: GET for reads, POST for creates, PUT for updates

3. **Database:**
   - Table names: lowercase, same as entity name
   - Column names: snake_case (e.g., `spaceship_id`, `user_id`)
   - Primary keys: `{entity}_id`

### Design Patterns

**1. Layered Architecture:**
```
Controller Layer (Resources)
    ↓ calls
Service Layer (Business Logic)
    ↓ uses
Repository Layer (Data Access)
    ↓ queries
Database
```

**2. Repository Pattern:**
```java
public interface SpaceshipRepository extends JpaRepository<Spaceship, String> {
    Spaceship findByUserId(String userId);
    List<Spaceship> findBySpaceshipStatus(String status);
}
```

**3. DTO/VO Pattern:**
- Request/Response objects separate from entities
- Validation in controller or validator classes
- Transformation in service layer

**4. Validator Pattern:**
```java
@InitBinder
protected void initBinder(WebDataBinder binder) {
    binder.setValidator(new SpaceshipRequestValidator());
}
```

### Error Handling

**Standard HTTP Status Codes:**
- `200` - Success
- `302` - Redirect/Found
- `403` - Forbidden
- `409` - Conflict (e.g., duplicate entry)
- `500` - Internal Server Error

**Pattern:**
```java
try {
    // Business logic
    return new ResponseEntity<>(result, HttpStatus.OK);
} catch (Exception e) {
    logger.error("Error message", e);
    return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
}
```

### API Documentation

**Swagger Configuration:**
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
            .groupName("{service-name}")
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.spaceship"))
            .paths(PathSelectors.any())
            .build();
    }
}
```

**Access Swagger UI:**
- Spaceship: http://localhost:8080/swagger-ui.html
- Protocol: http://localhost:8083/swagger-ui.html
- User: http://localhost:8084/swagger-ui.html

---

## Common Tasks

### Adding a New Endpoint

1. **Define the request/response VOs** in `model/vo/`
2. **Create validator** (optional) in `validator/`
3. **Add business logic** in service class
4. **Add REST endpoint** in resource class with Swagger annotations
5. **Test with Swagger UI** or Postman

Example:
```java
// In SpaceshipResource.java
@ApiOperation(value = "New endpoint description")
@RequestMapping(value = "/newEndpoint", method = RequestMethod.POST)
public ResponseEntity<ResponseVo> newEndpoint(@RequestBody RequestVo request) {
    try {
        ResponseVo response = spaceshipService.processRequest(request);
        return new ResponseEntity<>(response, HttpStatus.OK);
    } catch (Exception e) {
        logger.error("Error in newEndpoint", e);
        return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### Adding a New Service

1. **Create new Spring Boot project** using similar structure
2. **Define domain model** (entities, VOs)
3. **Implement repositories** extending JpaRepository
4. **Create service layer** with business logic
5. **Add REST controllers** with Swagger
6. **Configure database** in application.yml
7. **Generate SDK** using Swagger Codegen
8. **Build and deploy** with Docker support

### Running Tests

```bash
# Run tests for a service
cd XL-spaceship
mvn test

# Run with coverage
mvn clean test jacoco:report

# Integration tests
mvn verify
```

### Debugging

**Enable Debug Logging:**
```properties
# In application.properties or application.yml
logging.level.com.spaceship=DEBUG
logging.level.org.springframework=INFO
```

**Remote Debugging:**
```bash
java -jar -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005 \
    target/spaceship-0.0.1-SNAPSHOT.jar
```

### Database Migrations

- Database schema is auto-created by Hibernate
- For production, use SQL scripts in `docker-entrypoint-initdb.d/`
- Manual migrations: place `.sql` files in initialization directory

### Building Docker Images

```bash
# Build specific service
cd XL-spaceship
mvn clean package docker:build

# Tag and push (if using registry)
docker tag spaceships/spaceship:latest myregistry/spaceship:v1.0
docker push myregistry/spaceship:v1.0
```

---

## Important Notes

### Known Issues & Quirks

1. **Typo in naming:** "protocal" instead of "protocol" in directory and package names
   - Keep consistent with existing naming when adding code
   - Don't rename without updating all references

2. **Multiple Spaceship Instances:**
   - Support 3 instances by default (ports 8080-8082)
   - All share the same database
   - Each instance can handle different games simultaneously

3. **Database Initialization:**
   - Scripts in `docker-entrypoint-initdb.d/` run only on first container creation
   - Delete volume to re-initialize: `docker-compose down -v`

4. **Spring Boot Version:**
   - Using Spring Boot 1.5.3 (older version)
   - Be cautious with dependency updates
   - Some newer Spring features not available

5. **SDK Dependencies:**
   - Must build and install SDKs to local Maven repo before building services
   - Order: spaceship-sdk → protocal_sdk → user-sdk → services → client

### Security Considerations

**Development credentials in .env file:**
```bash
# ⚠️ DO NOT use these in production
MYSQL_ROOT_PASSWORD=p4SSW0rd
MYSQL_PASSWORD=dbp4ss
```

**For production:**
- Use environment variables or secret management
- Enable HTTPS/TLS
- Configure Spring Security properly
- Use strong database passwords
- Implement proper authentication/authorization

### Performance Tips

1. **Database Connection Pooling:**
   - Configure HikariCP in application.yml
   - Monitor connection pool usage

2. **Enable Caching:**
   - Use Spring Cache for frequent queries
   - Consider Redis for distributed caching

3. **Async Processing:**
   - Protocol service uses @EnableAsync
   - Use @Async for long-running operations

4. **API Response Optimization:**
   - Use pagination for list endpoints
   - Implement DTOs to avoid over-fetching

### API Testing

**Using Swagger UI:**
1. Navigate to http://localhost:{port}/swagger-ui.html
2. Expand endpoint section
3. Click "Try it out"
4. Enter parameters
5. Execute and view response

**Using cURL:**
```bash
# Create a game
curl -X POST "http://localhost:8083/xl-spaceship/protocol/game/new" \
  -H "Content-Type: application/json" \
  -d '{
    "gameName": "Test Game",
    "players": [...]
  }'

# Get game status
curl -X GET "http://localhost:8083/xl-spaceship/protocol/game/{gameId}"
```

### File Locations Reference

**Configuration Files:**
- Service config: `*/src/main/resources/application.yml`
- Logging config: `*/src/main/resources/log4j.properties`
- Docker config: `*/docker-compose.yml`, `*/.env`
- Maven config: `*/pom.xml`

**Source Code:**
- Controllers: `*/src/main/java/com/spaceship/*/resources/`
- Services: `*/src/main/java/com/spaceship/*/services/`
- Entities: `*/src/main/java/com/spaceship/*/model/entities/`
- Repositories: `*/src/main/java/com/spaceship/*/repositories/`

**Frontend (Client):**
- Templates: `spaceship_client/src/main/resources/templates/`
- Static files: `spaceship_client/src/main/resources/static/`
- Controllers: `spaceship_client/src/main/java/com/spaceship/client/controller/`

---

## Quick Reference Commands

```bash
# Build all SDKs
for dir in spaceship-sdk protocal_sdk user-sdk; do
    cd $dir && mvn clean install && cd ..
done

# Build all services
for dir in XL-spaceship XL-spaceship-protocal XL-spaceship-user spaceship_client; do
    cd $dir && mvn clean package && cd ..
done

# Start Docker environment
cd XL-spaceship
docker-compose up -d

# View logs
docker-compose logs -f spaceship1

# Stop all services
docker-compose down

# Rebuild Docker images
cd XL-spaceship
mvn clean package docker:build
docker-compose up -d --build

# Check service health
curl http://localhost:8080/swagger-ui.html
curl http://localhost:8083/swagger-ui.html
curl http://localhost:8084/swagger-ui.html
curl http://localhost:8087/
```

---

## Additional Resources

- **Game Rules:** See `XLSpaceship Game Guide.pdf`
- **API Documentation:** Access Swagger UI at `/swagger-ui.html` for each service
- **SDK Documentation:** See `*/docs/*.md` files in SDK directories
- **Spring Boot Docs:** https://docs.spring.io/spring-boot/docs/1.5.3.RELEASE/reference/html/

---

**For AI Assistants:**
- Always check Swagger documentation before implementing new endpoints
- Follow existing package and naming conventions strictly
- Build and install SDKs before modifying dependent services
- Test changes locally before suggesting Docker deployment
- Consider the microservices architecture when making changes
- Remember the typo: "protocal" not "protocol" in existing code
- Use the layered architecture pattern consistently
- Always include proper error handling and logging
- Document new endpoints with Swagger annotations

---

*This document is maintained to help AI assistants understand and work with the XL Spaceship codebase effectively.*
