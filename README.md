# Java-template

Spring Boot structure is much more opinionated than Go. The good news: less existential folder confusion. The bad news: Java developers compensate by inventing 19 layers for a login endpoint.

For backend interviews and sane projects, I’d recommend **feature-first + layered architecture**.

Avoid giant god folders like:

```txt
controller/
service/
repository/
entity/
dto/
```

with 200 files dumped into each. It becomes archaeological work.

---

# Good Spring Boot Structure (for you)

For an Artistify-like app:

```txt
artistify/

├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── artistify/
│   │   │
│   │   │           ├── ArtistifyApplication.java
│   │   │
│   │   │           ├── config/
│   │   │           │   ├── SecurityConfig.java
│   │   │           │   ├── RedisConfig.java
│   │   │           │   └── SwaggerConfig.java
│   │   │
│   │   │           ├── common/
│   │   │           │   ├── exception/
│   │   │           │   ├── response/
│   │   │           │   └── util/
│   │   │
│   │   │           ├── auth/
│   │   │           │   ├── controller/
│   │   │           │   ├── service/
│   │   │           │   ├── repository/
│   │   │           │   ├── entity/
│   │   │           │   ├── dto/
│   │   │           │   └── security/
│   │   │
│   │   │           ├── album/
│   │   │           │   ├── controller/
│   │   │           │   ├── service/
│   │   │           │   ├── repository/
│   │   │           │   ├── entity/
│   │   │           │   └── dto/
│   │   │
│   │   │           ├── grpc/
│   │   │           │   ├── client/
│   │   │           │   └── server/
│   │   │
│   │   │           └── metrics/
│   │
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── db/
│   │           └── migration/
│   │
│   └── test/
│
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── .github/
│   └── workflows/
│
├── pom.xml
└── README.md
```

---

## Why this structure?

Instead of:

```txt
controller/
service/
repository/
entity/
```

globally,

Do:

```txt
auth/
album/
user/
payment/
```

inside each feature.

Why?

Because features stay together.

Without feature-first:

You end up with:

```txt
controller/
├── AuthController.java
├── AlbumController.java
├── PaymentController.java
├── UserController.java
```

and:

```txt
service/
├── AuthService.java
├── AlbumService.java
├── PaymentService.java
```

At 100 files:

Pure suffering.

With feature-first:

Everything auth-related lives here:

```txt
auth/
```

Cleaner.

---

## What goes where?

### `controller/`

Only request handling.

Thin layer.

```java
@RestController
@RequestMapping("/albums")
public class AlbumController {

    private final AlbumService service;

    @GetMapping
    public ResponseEntity<?> getAlbums() {
        return ResponseEntity.ok(
            service.getAlbums()
        );
    }
}
```

Controller should NOT contain business logic.

No:

```java
if(user.hasPermission() && dbCall())
```

inside controller.

Humans keep doing this.

---

### `service/`

Business logic.

Example:

```txt
create album
validate rules
check permission
call repository
```

```java
@Service
public class AlbumService {

    public Album createAlbum(
            CreateAlbumRequest req) {

        validate(req);

        return repository.save(
                mapper.toEntity(req));
    }
}
```

---

### `repository/`

Database access only.

Usually:

```java
public interface AlbumRepository
       extends JpaRepository<
           Album, Long> {
}
```

No business logic here.

---

### `entity/`

Database models.

```java
@Entity
@Table(name="albums")
public class Album {

    @Id
    private Long id;

    private String name;
}
```

---

### `dto/`

Request/response models.

Example:

```java
CreateAlbumRequest
AlbumResponse
LoginRequest
```

Important because:

**Never expose entities directly to API**

Interviewers like this question.

Why?

Because entity changes break API.

---

## Where does `application.yml` go?

Always:

```txt
resources/
```

Example:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/musicdb
    username: postgres
    password: password

server:
  port: 8080
```

Profiles:

```txt
application-dev.yml
application-prod.yml
```

Very useful.

---

## Where do migrations go?

Use:

Flyway

or

Liquibase

Folder:

```txt
resources/db/migration
```

Example:

```txt
V1__create_user_table.sql
V2__create_album_table.sql
```

Spring auto-runs them.

Tiny piece of engineering magic.

---

## Where does gRPC go?

If using microservices:

```txt
grpc/
├── client/
└── server/
```

Proto:

```txt
src/main/proto/
```

Generated classes:

```txt
target/generated-sources/
```

Usually auto-generated by Maven/Gradle plugin.

---

## Where Docker & CI/CD?

Docker:

```txt
docker/
```

CI/CD:

```txt
.github/workflows/
```

Example:

```txt
ci.yml
deploy.yml
```

---

## The biggest Spring Boot mistake juniors make

They write:

```txt
Controller
↓
Repository
```

Skipping service layer.

Works for CRUD.

Becomes nightmare later.

Use:

```txt
Controller
↓
Service
↓
Repository
```

Always.

---

For interview + practical backend, this is the sweet spot:

**Feature-first + layered**

Not overengineered hexagonal architecture with:

```txt
adapter/
port/
domain/
usecase/
facade/
assembler/
factory/
strategy/
```

for a student CRUD app. That ecosystem occasionally mistakes complexity for maturity.
