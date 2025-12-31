# Judge API

A robust, dockerized remote code execution engine built with Node.js and Express. The Judge API compiles and executes code in a sandboxed environment, supporting multiple programming languages and concurrent test case execution.

## 🚀 Features

*   **Multi-Language Support**: Native execution for Python 3, C++17, and Java 17.
*   **Isolated Execution**: Runs code in temporary directories to prevent conflicts.
*   **Concurrent Benchmarking**: Optimized test case execution with controlled concurrency.
*   **Test Generation**: Special endpoint for generating test cases using C++ generators.
*   **Health Monitoring**: Built-in health check endpoint for uptime monitoring.
*   **Production Ready**: Docker-ready with a `bullseye` base image containing all necessary runtimes.

## 🛠️ Tech Stack

*   **Runtime**: [Node.js](https://nodejs.org/) (Express.js)
*   **Containerization**: [Docker](https://www.docker.com/)
*   **Languages**:
    *   Python 3
    *   C++ (g++)
    *   Java (OpenJDK 17)

## 🏁 Getting Started

### Prerequisites

*   Node.js v14+ (for local development)
*   Docker (recommended for deployment)

### Local Development

1.  **Install Dependencies**
    ```bash
    npm install
    ```

2.  **Start the Server**
    ```bash
    npm run dev
    ```
    The server will start on port `4001` (or the port defined in `PORT` env var).

### Docker Deployment

1.  **Build the Image**
    ```bash
    docker build -t judge-api .
    ```

2.  **Run the Container**
    ```bash
    docker run -p 4001:4001 judge-api
    ```

## 🔌 API Reference

### 1. Execute Code
**Endpoint**: `POST /submit`

Executes the submitted code against a set of input/output test cases.

**Request Body**:
```json
{
  "language": "python" | "cpp" | "java",
  "code": "print(input())",
  "input": ["test_input_1", "test_input_2"],
  "output": ["expected_output_1", "expected_output_2"],
  "timeLimit": 5000, 
  "memoryLimit": 256
}
```

*   `input`: Array of input strings (stdin).
*   `output`: Array of expected output strings (stdout).
*   `timeLimit`: Execution time limit in milliseconds (default: 5000ms).

**Response**:
```json
{
  "summary": {
    "total": 2,
    "passed": 2,
    "failed": 0
  },
  "results": [
    {
      "index": 0,
      "passed": true,
      "received": "test_input_1",
      "expected": "expected_output_1",
      "timeOut": false
    },
    ...
  ]
}
```

### 2. Generate Tests
**Endpoint**: `POST /generate-tests`

Compiles and runs a C++ generator to produce test inputs and expected outputs programmatically. A common pattern in Competitive Programming.

**Request Body**:
```json
{
  "language": "cpp",
  "code": "// C++ code that prints input JSON to stdout and output JSON to stderr"
}
```

**Response**:
Returns the generated input and output arrays.

### 3. Health Check
**Endpoint**: `GET /health`

**Response**:
```json
{ "status": "ok" }
```

## ⚙️ Configuration

| Variable | Description | Default |
| :--- | :--- | :--- |
| `PORT` | The port the server listens on | `4001` |
| `JUDGE_PORT` | Alternative port variable | - |
| `NODE_ENV` | Environment mode | `development` (local) / `production` (docker) |

## 🛡️ Security Note

This service executes arbitrary code submitted by users. While it runs in a Docker container, for a public-facing production environment, it is highly recommended to implement further sandboxing (e.g., `nsjail` or specialized execution drivers) to enforce strict memory and CPU limits and prevent network access.