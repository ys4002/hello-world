# hello-world Backend: Test Report

- Tester: qa (Team Lead - QA)
- Date: 2026-10-13
- Code under test: `master` @ 92746e7 (merge of `feature/backend-hello-endpoint`), branch `feature/qa-test-report`
- Environment: OpenJDK 21.0.12.1, Maven 3.9.11, Spring Boot 3.5.16, Tomcat 10.1.55, Linux
- Application code under `backend/` was not modified.

## Summary

All 14 test-plan cases: **14 PASS, 0 FAIL**. Defects found: **none**.

| ID | Description | Result |
|---|---|---|
| U1 | Context loads (`@SpringBootTest`) | PASS |
| U2 | MockMvc GET /api/hello: 200, JSON, `$.message` == "Hello, World!" | PASS |
| U3 | MockMvc GET /api/unknown: 404 | PASS |
| U4 | MockMvc POST /api/hello: 405 | PASS |
| U5 | `mvn test` exits 0, 0 failures, 0 errors | PASS |
| A1 | GET /api/hello: 200, application/json, exact body | PASS |
| A2 | GET /api/hello?name=x with Accept JSON: same as A1 | PASS |
| A3 | GET /api/unknown: 404 JSON with status 404 | PASS |
| A4 | POST /api/hello: 405, Allow contains GET | PASS |
| A5 | PUT and DELETE /api/hello: 405 both | PASS |
| A6 | GET /api/hello/ (trailing slash): 404 expected, record actual | PASS (informational) |
| N1 | pom.xml dependencies | PASS |
| N2 | App listens on 8080 | PASS |
| N3 | No errors or stack traces in startup log | PASS |

Raw output: `reports/mvn-test.log` (Q1), `reports/api-curl.log` (Q2), `reports/app-startup.log` (N3).
Note: the app `.gitignore` ignores `*.log`; these logs were added with `git add -f`.

## Q1: Unit / slice level

Command: `cd backend && mvn test` (full output in `reports/mvn-test.log`).

| ID | Test method (class) | Result | Evidence |
|---|---|---|---|
| U1 | `contextLoads` (HelloWorldApplicationTests) | PASS | `Tests run: 1, Failures: 0, Errors: 0` for HelloWorldApplicationTests; context started in 0.482 s |
| U2 | `getHelloReturnsGreetingJson` (HelloControllerTest) | PASS | Asserts status 200, content type compatible with application/json, `$.message` == "Hello, World!"; passed |
| U3 | `getUnknownPathReturnsNotFound` (HelloControllerTest) | PASS | Asserts 404; passed |
| U4 | `postHelloReturnsMethodNotAllowed` (HelloControllerTest) | PASS | Asserts 405 and `Allow` header contains GET; passed |
| U5 | Whole build | PASS | `Tests run: 4, Failures: 0, Errors: 0, Skipped: 0`, `BUILD SUCCESS`, `EXIT CODE: 0`. HelloControllerTest: 3 tests, 0 failures. |

## Q2: API level (running app)

App started with `cd backend && mvn spring-boot:run`; started in 1.3 s on port 8080. Requests via `curl -sS -i --noproxy '*'`. Full headers and bodies in `reports/api-curl.log`.

| ID | Request | Result | Evidence |
|---|---|---|---|
| A1 | `curl -i http://localhost:8080/api/hello` | PASS | `HTTP/1.1 200`, `Content-Type: application/json`, body `{"message":"Hello, World!"}` (exact) |
| A2 | `curl -i -H "Accept: application/json" "http://localhost:8080/api/hello?name=x"` | PASS | `HTTP/1.1 200`, `Content-Type: application/json`, body `{"message":"Hello, World!"}`. Query param ignored. |
| A3 | `curl -i -H "Accept: application/json" http://localhost:8080/api/unknown` | PASS | `HTTP/1.1 404`, `Content-Type: application/json`, body `{"timestamp":"2026-10-13T20:13:34.184+00:00","status":404,"error":"Not Found","path":"/api/unknown"}` |
| A4 | `curl -i -X POST http://localhost:8080/api/hello` | PASS | `HTTP/1.1 405`, `Allow: GET`, JSON body with `"status":405,"error":"Method Not Allowed"` |
| A5 | `curl -i -X PUT ...` and `curl -i -X DELETE http://localhost:8080/api/hello` | PASS | Both `HTTP/1.1 405`, `Allow: GET`, JSON error body |
| A6 | `curl -i http://localhost:8080/api/hello/` | PASS (informational) | Actual: `HTTP/1.1 404`, JSON body `"status":404,"error":"Not Found","path":"/api/hello/"`. Matches the expected 404 (no trailing-slash match in Spring Boot 3); no 5xx. Also 404 with `Accept: application/json`. |

## Non-functional checks

| ID | Check | Result | Evidence |
|---|---|---|---|
| N1 | pom.xml dependencies | PASS | `backend/pom.xml` declares exactly two dependencies: `spring-boot-starter-web` and `spring-boot-starter-test` (scope test). Parent is `spring-boot-starter-parent:3.5.16`, `java.version` 21. |
| N2 | App listens on 8080 | PASS | Startup log: `Tomcat started on port 8080 (http) with context path '/'`; `lsof -i :8080 -sTCP:LISTEN` showed the java process (PID 1454) listening; all curl cases answered on 8080. After shutdown, port 8080 was free. |
| N3 | No errors or stack traces at startup | PASS | `reports/app-startup.log`: lines up to `Started HelloWorldApplication in 1.316 seconds` contain only INFO entries. No ERROR, no stack traces. The only WARN lines come later from the deliberate 405 requests (A4/A5: `HttpRequestMethodNotSupportedException ... Resolved`), which is Spring's normal one-line log for an unsupported method, not a startup problem. Graceful shutdown completed cleanly. |

## Observations (not defects)

- 405 and 404 responses are JSON in all cases tested, including A4/A5/A6 without an explicit Accept header.
- 404 responses carry `Vary: Origin` headers (Spring default); 405 does not. Harmless.
- The 405 `Allow` header is exactly `GET` (HEAD/OPTIONS not listed), which still satisfies the contract ("contains GET").

## Defects

None.

## Cleanup

App stopped (graceful shutdown logged); port 8080 confirmed free; no java/mvn processes left running.
