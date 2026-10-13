# hello-world: Plan

## Overview

A minimal Spring Boot backend that exposes one endpoint, `GET /api/hello`, returning `{"message": "Hello, World!"}` as JSON with HTTP 200.

- Backend only. No frontend, no database, no auth.
- Stack: Java 21, Spring Boot **3.5.16** (latest stable 3.x on Maven Central at planning time), Maven 3.9.x.
- Dependencies: `spring-boot-starter-web` and `spring-boot-starter-test` (test scope) only. No other libraries.
- Runs on `localhost:8080`.
- Backend branch: `feature/backend-hello-endpoint`.

Out of scope: frontend, persistence, configuration profiles, Docker, Actuator, security.

## Architecture

Single-module Maven project in `backend/`. With one constant response there is no business logic or data, so a service and repository layer would be empty. The layered structure is cut down to:

- **Application class** (`HelloWorldApplication`): `@SpringBootApplication` entry point.
- **Controller** (`HelloController`): `@RestController` under `/api`, with one `@GetMapping("/hello")`.
- **Response DTO** (`HelloResponse`): a Java `record HelloResponse(String message)`. Jackson (included with starter-web) serializes it to `{"message": "..."}`. A record gives a typed response with no boilerplate and no extra library. It is a nested record in `HelloController` or its own file, at the developer's choice. If it gets its own file, use `HelloResponse.java`.
- **Errors**: use Spring Boot's default error handling and its default JSON error body. No custom `@ControllerAdvice` is needed.

Package: `com.example.helloworld`.

Maven coordinates: groupId `com.example`, artifactId `hello-world`, version `0.0.1-SNAPSHOT`, parent `org.springframework.boot:spring-boot-starter-parent:3.5.16`, `<java.version>21</java.version>`, `spring-boot-maven-plugin` in the build.

File layout (all under `backend/`):

```
backend/
  pom.xml
  .gitignore                  (optional; app-level .gitignore already ignores target/)
  src/main/java/com/example/helloworld/HelloWorldApplication.java
  src/main/java/com/example/helloworld/HelloController.java
  src/main/resources/application.properties   (server.port=8080, spring.application.name=hello-world)
  src/test/java/com/example/helloworld/HelloControllerTest.java
  src/test/java/com/example/helloworld/HelloWorldApplicationTests.java
```

A Maven wrapper (`mvnw`) is optional. Maven 3.9.11 is installed, so plain `mvn` is the documented way to build and run.

## API Contract

### GET /api/hello

| Item | Value |
|---|---|
| Method | `GET` |
| Path | `/api/hello` |
| Request body | none |
| Query params | none (any supplied are ignored) |
| Success status | `200 OK` |
| Content-Type | `application/json` |

Response body (exact):

```json
{"message": "Hello, World!"}
```

- The body has exactly one field, `message`, a string with the exact value `Hello, World!` (capital H, comma, space, capital W, exclamation mark).

### Error responses (Spring Boot defaults, no custom handling)

| Case | Status | Notes |
|---|---|---|
| Unknown path, e.g. `GET /api/unknown` | `404 Not Found` | Default Spring Boot error JSON |
| Wrong method, e.g. `POST /api/hello` | `405 Method Not Allowed` | Response includes an `Allow` header that contains `GET` |

Default error body shape (for clients that send `Accept: application/json`):

```json
{
  "timestamp": "2026-10-13T12:00:00.000+00:00",
  "status": 404,
  "error": "Not Found",
  "path": "/api/unknown"
}
```

Only the status codes and the `status`/`error` fields are part of the contract. The `timestamp` value varies. `path` may be present.

## Tasks by Role

### dev-backend (branch `feature/backend-hello-endpoint`, all files under `backend/`)

| ID | Task | Deliverable | Depends on |
|---|---|---|---|
| B1 | Scaffold the Maven project | `pom.xml` (parent 3.5.16, Java 21, starter-web, starter-test, spring-boot-maven-plugin), `HelloWorldApplication.java`, `application.properties` (port 8080, app name). `mvn -q -DskipTests package` succeeds. | none |
| B2 | Implement the endpoint | `HelloController.java` (+ `HelloResponse` record) serving `GET /api/hello` exactly as in the API Contract | B1 |
| B3 | Tests | `HelloControllerTest.java` using `@WebMvcTest(HelloController.class)` + `MockMvc`: 200 + JSON body for GET, 404 for unknown path, 405 for POST. `HelloWorldApplicationTests.java` with `@SpringBootTest` context-load test. `mvn test` green. | B2 |
| B4 | Run check and hand-off | Start with `mvn spring-boot:run`. Confirm `curl -i http://localhost:8080/api/hello` returns 200 and the exact JSON. Commit on the feature branch and notify the Team Lead for review. Add a short build/run note in the hand-off message. | B3 |

### dev-frontend

None (backend only).

### qa (after Team Lead merges B1-B4 to `master`)

| ID | Task | Deliverable | Depends on |
|---|---|---|---|
| Q1 | Run unit/MockMvc tests | `mvn test` result recorded | B4 merged |
| Q2 | Run API tests against the running app | Results for cases A1-A6 below | Q1 |
| Q3 | Write report | `reports/test-report.md` with pass/fail per case ID, commands used, and any defects (sent to the Team Lead) | Q2 |

## Test Plan

Setup: `cd backend && mvn spring-boot:run` (port 8080 must be free). Use `curl -i` for API cases.

### Unit / slice level (`mvn test`)

| ID | Test | Expected |
|---|---|---|
| U1 | Context loads (`@SpringBootTest`) | Passes |
| U2 | MockMvc `GET /api/hello` | Status 200, content type `application/json`, `$.message` == `Hello, World!` |
| U3 | MockMvc `GET /api/unknown` | Status 404 |
| U4 | MockMvc `POST /api/hello` | Status 405 |
| U5 | Build | `mvn test` exits 0, 0 failures, 0 errors |

### API level (running app, curl)

| ID | Request | Expected |
|---|---|---|
| A1 | `curl -i http://localhost:8080/api/hello` | `HTTP/1.1 200`, `Content-Type: application/json`, body exactly `{"message":"Hello, World!"}` |
| A2 | `curl -i -H "Accept: application/json" http://localhost:8080/api/hello?name=x` | Same as A1 (query params ignored) |
| A3 | `curl -i -H "Accept: application/json" http://localhost:8080/api/unknown` | `404`, JSON error body with `"status":404` |
| A4 | `curl -i -X POST http://localhost:8080/api/hello` | `405`, `Allow` header contains `GET` |
| A5 | `curl -i -X PUT http://localhost:8080/api/hello` and `-X DELETE` | `405` for both |
| A6 | `curl -i http://localhost:8080/api/hello/` (trailing slash) | `404` (Spring Boot 3 does not match trailing slashes by default). Record the actual result. This is informational, not a defect, unless it returns a 5xx. |

### Non-functional checks

| ID | Check | Expected |
|---|---|---|
| N1 | `pom.xml` dependencies | Only `spring-boot-starter-web` and `spring-boot-starter-test` (test scope) |
| N2 | Port | App listens on 8080 |
| N3 | Startup | No errors or stack traces in the startup log |
