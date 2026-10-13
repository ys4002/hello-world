# hello-world

A tiny Spring Boot backend with a single JSON endpoint. No UI.

## Stack
Java 21, Spring Boot 3.5.16, Maven. Dependencies: `spring-boot-starter-web`, `spring-boot-starter-test` (test scope).

## Prerequisites
JDK 21 and Maven 3.9+ (set `JAVA_HOME` to a JDK 21 if your default is newer).

## Run
```bash
cd backend
mvn spring-boot:run
curl -i http://localhost:8080/api/hello
```
Response: `200 OK` with `{"message":"Hello, World!"}`.

Other behaviour (Spring Boot defaults): unknown path returns `404`, non-GET on `/api/hello` returns `405` with `Allow: GET`.

## Configuration
- `server.port=8080` in `backend/src/main/resources/application.properties`. Override with `SERVER_PORT` or `--server.port`.
- No credentials, no database, no other environment variables.

## Tests
```bash
cd backend && mvn test
```
4 tests (MockMvc controller tests and a context-load test). QA results: `reports/test-report.md`.

## Layout
```
backend/   Spring Boot app (com.example.helloworld)
docs/      plan.md, build-summary.md
reports/   QA report and raw logs
```

## Known limitations
- Single static endpoint; no frontend, persistence, auth or CORS.
- A trailing slash (`/api/hello/`) returns 404 (Spring Boot 3 default).
