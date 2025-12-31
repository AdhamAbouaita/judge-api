# Judge API

**The Open-Source, Easy-to-Use Custom Judge API for Competitive Programming.**

Judge API is a high-performance, dockerized remote code execution engine designed specifically for competitive programming platforms, coding interview tools, and educational grading systems. It provides a robust, isolated environment to compile and execute untrusted code against multiple test cases concurrently, delivering precise results with minimal latency.

Built with **Node.js (Express)** and **Docker**, it offers a production-ready solution that is trivially easy to deploy and scale.

---

## 📚 Table of Contents

-   [Features](#-features)
-   [Architecture & Internals](#-architecture--internals)
-   [Getting Started](#-getting-started)
    -   [Prerequisites](#prerequisites)
    -   [Running Locally](#running-locally)
    -   [Running with Docker](#running-with-docker)
-   [Language Support](#-language-support)
-   [API Reference](#-api-reference)
    -   [Execute Code (`/submit`)](#1-execute-code-submit)
    -   [Generate Tests (`/generate-tests`)](#2-generate-tests-generate-tests)
    -   [Health Check (`/health`)](#3-health-check-health)
-   [Configuration](#-configuration)

---

## 🚀 Features

*   **⚡ Concurrent Execution**: Optimized to run multiple test cases in parallel for a single submission, significantly reducing feedback time for users.
*   **🛡️ Isolated Environments**: Every submission runs in a temporary, dedicated directory to ensure file system cleanliness and prevent crosstalk.
*   **🌍 Multi-Language Native Support**: First-class support for **Python 3**, **C++17**, and **Java 17**.
*   **🏗️ Test Generation API**: Unique support for running C++ "Generator" logic to programmatically create test cases—ideal for problem setters.
*   **🐳 Docker Native**: Packaged with a `bullseye` base image containing all necessary compilers and runtimes, ensuring identical behavior across dev and prod.
*   **📊 Detailed Feedback**: Returns granular results for every test case, including stdout, stderr, exit codes, and diff status.

---

## 🧠 Architecture & Internals

The Judge API operates as a stateless microservice. Here is the lifecycle of a request:

1.  **Request Handling**: The API receives a JSON payload containing source code, language, input arrays, and expected output arrays.
2.  **Workspace Creation**: A unique, ephemeral temporary directory (`/tmp/judge-xxxx`) is created for the request.
3.  **Compilation (if applicable)**:
    *   **Python**: Checked for syntax/runtime availability.
    *   **C++**: Compiled with `g++ -O2 -std=c++17`.
    *   **Java**: Compiled with `javac`.
4.  **Execution Strategy**:
    *   The compiled binary (or script) is executed against each input string.
    *   **Concurrency**: The Node.js event loop spawns multiple child processes in parallel batches (controlled by `MAX_CONCURRENT_TESTS`) to utilize multi-core systems efficiently.
    *   **Normalization**: Inputs and outputs are normalized (whitespace trimming, JSON parsing) to ensure fair comparisons.
5.  **Result Aggregation**: Results are gathered, compared against expected outputs, and summarized (Pass/Fail).
6.  **Cleanup**: The temporary directory is forcefully removed to free up resources.

---

## 🏁 Getting Started

### Prerequisites

*   **Docker** (Recommended for all deployments)
*   *Or for manual local dev*: Node.js v14+, Python 3, G++, and JDK 17 installed on your host machine.

### Running with Docker (Recommended)

This is the easiest way to ensure all compilers are present and configured correctly.

1.  **Build the Image**
    ```bash
    docker build -t judge-api .
    ```

2.  **Run the Container**
    ```bash
    docker run -p 4001:4001 judge-api
    ```

The API is now active at `http://localhost:4001`.

### Running Locally

If you want to develop on the API logic itself:

1.  **Install Dependencies**
    ```bash
    npm install
    ```

2.  **Start Development Server**
    ```bash
    npm run dev
    ```
    *Note: You must have `python3`, `g++`, and `javac` in your system PATH for this to work.*

---

## 💻 Language Support

The Docker image comes pre-installed with the following runtimes:

| Language | Compiler / Runtime | Flags / Version |
| :--- | :--- | :--- |
| **Python** | Python 3.9+ | `python3` |
| **C++** | GCC (g++) | `-O2 -std=c++17` |
| **Java** | OpenJDK 17 | `javac` / `java` |

---

## 🔌 API Reference

### 1. Execute Code (`/submit`)

This is the core endpoint used to grade user submissions.

*   **Method**: `POST`
*   **URL**: `/submit`
*   **Content-Type**: `application/json`

**Request Body Schema**:

```json
{
  "language": "python" | "cpp" | "java",
  "code": "string",       // The full source code
  "input": ["string"],    // Array of stdin inputs (one per test case)
  "output": ["string"],   // Array of expected stdout outputs
  "timeLimit": number,    // Optional: Max execution time in ms (default: 5000)
  "memoryLimit": number   // Optional: Max memory in MB (not currently enforced)
}
```

**Example Request**:

```json
{
  "language": "python",
  "code": "n = int(input())\nprint(n * 2)",
  "input": ["5", "10", "100"],
  "output": ["10", "20", "200"]
}
```

**Response**:

```json
{
  "summary": {
    "total": 3,
    "passed": 3,
    "failed": 0
  },
  "results": [
    {
      "index": 0,
      "exitCode": 0,
      "passed": true,
      "expected": "10",
      "received": "10",
      "stderr": "",
      "timedOut": false
    },
    // ... results for cases 1 and 2
  ]
}
```

### 2. Generate Tests (`/generate-tests`)

Useful for problem setters. Runs a C++ generator to produce input/output pairs. The generator should print the **Input JSON array** to `stdout` and the **Output JSON array** to `stderr`.

*   **Method**: `POST`
*   **URL**: `/generate-tests`

**Request Body**:

```json
{
  "language": "cpp",
  "code": "/* C++ code printing JSON to stdout/stderr */"
}
```

**Response**:

```json
{
  "input": ["test_case_1", "test_case_2"],
  "output": ["expected_1", "expected_2"],
  "inputJson": "[...]",
  "outputJson": "[...]"
}
```

### 3. Health Check (`/health`)

Simple endpoint to verify the service is up.

*   **Method**: `GET`
*   **URL**: `/health`
*   **Response**: `{ "status": "ok" }`

---

## ⚙️ Configuration

The application is configured via environment variables.

| Variable | Description | Default |
| :--- | :--- | :--- |
| `PORT` | The primary port the server listens on. | `4001` |
| `JUDGE_PORT` | Fallback port variable. | - |
| `MAX_CONCURRENT_TESTS` | (Internal const) Number of tests running in parallel per request. | `3` |