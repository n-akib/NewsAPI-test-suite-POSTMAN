# 📰 NewsAPI Automated API Test Suite

[![Postman](https://img.shields.io/badge/Postman-v12+-orange?logo=postman&logoColor=white)](https://www.postman.com/)
[![Newman](https://img.shields.io/badge/CLI-Newman-orange?logo=postman)](https://github.com/postmanlabs/newman)
[![CI](https://img.shields.io/badge/CI%2FCD-GitHub_Actions-blue?logo=github-actions&logoColor=white)](.github/workflows/newman-tests.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

An end-to-end automated REST API testing suite built for [NewsAPI v2](https://newsapi.org/v2) using **Postman**, **JavaScript**, and **Newman CLI**. This project demonstrates industry best practices for API testing, including schema validation, parameterized query testing, negative boundary testing, token security, and headless CI/CD integration.

---

## 🎯 Key Highlights & Features

- **Comprehensive Endpoint Coverage:** Validates `/v2/everything`, `/v2/top-headlines`, and `/v2/top-headlines/sources`.
- **Dynamic & Non-Brittle Assertions:** Avoids brittle assertions on live changing news feeds by dynamically validating response envelope structures, data types (`string`, `number`, `array`), URL regex patterns (`^https?://`), and ISO 8601 timestamps.
- **Robust Negative & Security Testing:** Verifies authentication enforcement (missing vs. invalid API key) and mandatory parameter validation.
- **Secure Secret Handling:** API keys are injected at the collection level via environment variables (`{{newsApiKey}}`) without hardcoding credentials into source control.
- **CI/CD & CLI Ready:** Configured with Newman CLI scripts and GitHub Actions for automated regression test runs.

---

## 📊 Test Case Catalog & Matrix

| Case ID | Endpoint | Method | Scenario Type | HTTP Status | Assertion Highlights |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **`NA-01`** | `/v2/everything?q=technology&pageSize=5` | `GET` | Positive | `200 OK` | `status: "ok"`, `totalResults >= 0`, `articles` array shape, `title`/`url`/`publishedAt` data types and regex. |
| **`NA-02`** | `/v2/top-headlines?country=us&pageSize=5` | `GET` | Positive | `200 OK` | Top headlines filtering by country; validates envelope and payload formatting. |
| **`NA-03`** | `/v2/everything?q=technology` | `GET` | Negative | `401 Unauthorized` | Missing authentication header; asserts `status: "error"` and `code: "apiKeyMissing"`. |
| **`NA-04`** | `/v2/everything?q=technology` | `GET` | Negative | `401 Unauthorized` | Invalid/dummy API key; asserts `status: "error"` and `code: "apiKeyInvalid"`. |
| **`NA-05`** | `/v2/everything` | `GET` | Negative | `400 Bad Request` | Missing required search query (`q`/`sources`/`domains`); validates `code: "parametersMissing"`. |
| **`NA-06`** | `/v2/everything?q=technology&sortBy=invalid` | `GET` | Negative | `400 Bad Request` | Invalid sorting parameter enum validation. |
| **`NA-07`** | `/v2/top-headlines/sources?language=en` | `GET` | Positive | `200 OK` | Validates sources metadata directory and verifies `sources` array element fields (`id`, `name`). |

---

## 🛠️ Tech Stack & Tools

- **API Platform:** [NewsAPI.org](https://newsapi.org/) (RESTful endpoints)
- **Test Authoring:** Postman Desktop App & Postman Sandbox (JavaScript/Chai Assertion Library)
- **Test Runner:** Newman CLI / Newman HTML Extra Reporter
- **CI/CD Automation:** GitHub Actions
- **Documentation:** Markdown & Structured Test Spreadsheet (`.xlsx`)

---

## 📁 Repository Structure

```plaintext
├── .github/
│   └── workflows/
│       └── newman-tests.yml          # GitHub Actions CI workflow
├── News API.postman_collection.json  # Postman test collection with JavaScript assertions
├── News API.postman_environment.json # Environment variables template
├── News-API-Test-Cases-and-Results.xlsx # Test execution matrix & design spreadsheet
├── reports.md                        # Detailed QA test report & contract observations
├── package.json                      # NPM scripts for running Newman tests & reports
└── README.md                         # Project documentation
```

---

## 🚀 Getting Started

### Prerequisites
- [Node.js](https://nodejs.org/) (v16 or higher)
- [Postman](https://www.postman.com/downloads/) (for GUI execution)
- A free API key from [NewsAPI.org](https://newsapi.org/register)

### Option 1: Running in Postman (GUI)
1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/newsapi-test-automation.git
   cd newsapi-test-automation
   ```
2. Open Postman and import:
   - `News API.postman_collection.json`
   - `News API.postman_environment.json`
3. Select the **News API Environment** in the top-right environment selector.
4. Set your `newsApiKey` under the environment's **Current Value** (keep Initial Value blank to prevent accidental commits).
5. Open the Collection Runner and click **Run News API Automation Test Suite**.

---

### Option 2: Running with Newman CLI

1. Install project dependencies:
   ```bash
   npm install
   ```

2. Run the test suite via Newman in your terminal:
   ```bash
   npm test -- --env-var "newsApiKey=YOUR_API_KEY_HERE"
   ```

3. Generate a rich HTML report:
   ```bash
   npm run test:report -- --env-var "newsApiKey=YOUR_API_KEY_HERE"
   ```
   *The interactive HTML report will be generated in the `newman/` directory.*

---

## ⚙️ CI/CD Integration (GitHub Actions)

This repository includes a GitHub Actions workflow (`.github/workflows/newman-tests.yml`) that automatically runs the test suite on every push or pull request.

To enable automated testing in GitHub:
1. Go to your repository settings on GitHub: **Settings $\rightarrow$ Secrets and variables $\rightarrow$ Actions**.
2. Click **New repository secret**.
3. Name: `NEWS_API_KEY`, Value: `<Your NewsAPI Key>`.
4. Trigger the workflow manually or by opening a pull request.

---

## 📝 Sample Assertion Code

Here is an example of the dynamic schema and type validation script used in `NA-01`:

```javascript
// Validate status code & headers
pm.test("Status code is 200", () => {
    pm.response.to.have.status(200);
});
pm.test("Content-Type is JSON", () => {
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
});

// Validate JSON Envelope & Dynamic Array Types
const res = pm.response.json();
pm.test("Envelope status is ok", () => {
    pm.expect(res.status).to.eql("ok");
    pm.expect(res.totalResults).to.be.a("number").and.at.least(0);
    pm.expect(res.articles).to.be.an("array");
});

// Assert data contracts without hardcoding brittle live headlines
pm.test("Articles adhere to expected schema", () => {
    res.articles.forEach(article => {
        pm.expect(article.title).to.be.a("string");
        pm.expect(article.url).to.be.a("string").and.match(/^https?:\/\//);
        pm.expect(article.publishedAt).to.be.a("string");
    });
});
```

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
