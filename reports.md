# News API Test Execution & QA Analysis Report

## 1. Executive Summary

This report provides a comprehensive summary of the automated API testing suite designed and executed against the **NewsAPI v2 REST endpoints** (`https://newsapi.org/v2`). The primary objective of this project is to validate functional correctness, response schema compliance, authentication enforcement, parameter handling, and resilience against invalid input scenarios.

### Deliverables & Artifacts:
* **Postman Collection:** `News API.postman_collection.json`
* **Postman Environment:** `News API.postman_environment.json`
* **Test Matrix & Results Spreadsheet:** `News-API-Test-Cases-and-Results.xlsx`

---

## 2. Test Scope & Execution Strategy

### Environment Details:
* **Base URL:** `https://newsapi.org/v2`
* **Execution Tools:** Postman (Interactive) / Newman CLI (Automated/CI-ready)
* **Authentication:** Collection-level `X-Api-Key` header mapped to `{{newsApiKey}}` variable.
* **Credential Security:** API keys are managed exclusively via environment variables and never committed directly into version control.

### Test Architecture & Assertions:
1. **Status Code Verification:** Validates accurate HTTP responses (`200 OK`, `400 Bad Request`, `401 Unauthorized`).
2. **Schema & Header Validation:** Ensures standard `application/json` Content-Type and verifies response envelopes (`status`, `totalResults`, `articles`, `sources`).
3. **Data Type & Format Integrity:** Dynamically iterates over returned arrays to assert field types (`string`, ISO 8601 date format for `publishedAt`, valid URL regex for `url`) without relying on brittle, hardcoded live news content.
4. **Error Envelope Verification:** Confirms negative scenarios produce standard error responses with `status: "error"`, structured `code`, and descriptive `message`.

---

## 3. Test Execution Summary

| Total Test Cases | Passed | Failed / Observations | Blocked | Execution Mode |
| :---: | :---: | :---: | :---: | :---: |
| **7** | **6** | **1** | **0** | Automated (Postman / Newman) |

### Detailed Test Case Breakdown:

| Test ID | Endpoint / Scenario | Method | HTTP Status | Result | Key Validations |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **`NA-01`** | `/v2/everything?q=technology&pageSize=5` | `GET` | `200 OK` | **PASS** | Status `ok`, `totalResults >= 0`, array shape, string types for `title`, `publishedAt`, and valid `url` regex. |
| **`NA-02`** | `/v2/top-headlines?country=us&pageSize=5` | `GET` | `200 OK` | **PASS** | Top headlines by country; verifies valid payload structure without combining conflicting query params. |
| **`NA-03`** | `/v2/everything?q=technology` (No Auth) | `GET` | `401 Unauthorized` | **PASS** | Authentication omission; validates `code: "apiKeyMissing"` and descriptive error message. |
| **`NA-04`** | `/v2/everything?q=technology` (Invalid Key) | `GET` | `401 Unauthorized` | **PASS** | Fake API key rejection; confirms `401 Unauthorized` and `code: "apiKeyInvalid"`. |
| **`NA-05`** | `/v2/everything` (Missing `q` param) | `GET` | `400 Bad Request` | **PASS** | Mandatory parameter enforcement; validates `code: "parametersMissing"`. |
| **`NA-06`** | `/v2/everything?q=technology&sortBy=invalid` | `GET` | `400 Bad Request` | **OBSERVATION** | Evaluates query rejection on illegal sort enum. (See Observation 1 below). |
| **`NA-07`** | `/v2/top-headlines/sources?language=en` | `GET` | `200 OK` | **PASS** | Validates publisher metadata directory; asserts `sources` array schema and `id`/`name` fields. |

---

## 4. Observations & Contract Findings

### Observation 1: Error Code Envelope for Invalid Sorting Parameter (`NA-06`)
* **Endpoint:** `GET /v2/everything?q=technology&sortBy=invalidSort`
* **Observed Response:** HTTP `400 Bad Request` with payload:
  ```json
  {
    "status": "error",
    "code": "parameterInvalid",
    "message": "You have requested too many results or invalid parameters."
  }
  ```
* **Analysis:** While the API correctly blocks the invalid sorting value with a `400 Bad Request`, the returned error message bundles multiple potential root causes ("too many results or invalid parameters") rather than explicitly stating that the `sortBy` parameter value is invalid and enumerating valid values (`relevancy`, `popularity`, `publishedAt`).
* **QA Recommendation:** Enhance API contract error details with granular parameter-level validation messages for improved developer experience.

---

## 5. Risks & Considerations for Continuous Testing

1. **Developer Tier Rate Limiting:**
   * Free developer accounts are limited to **100 requests per 24-hour window**.
   * Automated CI/CD pipelines should run on a controlled trigger (e.g., Pull Requests or scheduled nightly runs) to avoid hitting `429 Too Many Requests`.
2. **Date Range Boundaries (`from` & `to`):**
   * Free tier endpoints restrict queries to articles published within the last 30 days. Test suites querying explicit date ranges should compute dates dynamically relative to `Date.now()`.
3. **CORS Restrictions:**
   * NewsAPI restricts requests originating from browser frontends on the free tier. All automated suites and backend clients should execute from server/CLI environments.
