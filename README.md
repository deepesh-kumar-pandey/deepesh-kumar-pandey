<div align="center">

# Deepesh Kumar Pandey

### Systems & Backend Engineer &nbsp;|&nbsp; Go · C++ · Infrastructure

**Durable storage · Concurrent pipelines · LLM agent tooling**

Computer Science undergraduate (2023–2027) focused on performance, reliability, and clean, testable code.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/deepesh-kumar-pandey-013233288)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/deepesh-kumar-pandey)
[![LeetCode](https://img.shields.io/badge/LeetCode-FFA116?style=for-the-badge&logo=leetcode&logoColor=black)](https://leetcode.com/u/Deepesh_Kumar_Pandey)
[![HackerRank](https://img.shields.io/badge/HackerRank-2EC866?style=for-the-badge&logo=hackerrank&logoColor=white)](https://hackerrank.com/profile/deepesh040505)

[About](#about) · [Projects](#featured-projects) · [Skills](#skills) · [Competencies](#core-competencies) · [Code](#selected-code-patterns) · [Roadmap](#roadmap) · [Contact](#contact)

</div>

---

## About

I build backend and infrastructure software with a focus on **correctness under failure**: crash recovery, thread safety, bounded resource use, and measurable performance. I started in modern C++ (rate limiters, health monitors, async task engines) and now work primarily in **Go** on storage engines, concurrent pipelines, and an LLM agent runtime.

Across languages the priorities stay the same: **thread safety**, **crash resilience**, **cross-platform compatibility**, and a **minimal resource footprint**.

```go
type Engineer struct {
    Name        string
    Focus       string
    Specialties []string
}

func Me() Engineer {
    return Engineer{
        Name:  "Deepesh Kumar Pandey",
        Focus: "Systems Programming & Infrastructure",
        Specialties: []string{
            "Go & C++ systems development",
            "Durable storage and caching engines",
            "LLM agent runtimes, MCP and tooling",
            "Concurrent programming",
            "DevOps automation and observability",
        },
    }
}
```

### What I Work On

| Area | Focus |
|---|---|
| **Systems programming** | Low-level infrastructure in Go and C++11/14/17 |
| **Durable storage** | Write-ahead logs, fsync semantics, crash recovery, compaction, atomic writes |
| **Agent infrastructure** | Orchestration loops, tool registries, MCP integration, session persistence |
| **DevOps automation** | Dockerized services, multi-stage builds, CI checks, multi-platform deployment |
| **Security-minded design** | Thread-safe code, encrypted audit logs, signed requests, validated inputs |
| **Observability** | Health checks, metrics endpoints, real-time system monitoring and alerting |
| **Performance** | Algorithmic efficiency, benchmarking, low-overhead hot paths |

---

## Featured Projects

### 1. [Agent Harness](https://github.com/deepesh-kumar-pandey/Agent-harness) &nbsp;·&nbsp; Go

> **A runtime for building and running tool-using AI agents: the model supplies the intelligence, the harness supplies the tools, state, and execution loop.**

**Status:** Active development. Core runtime, MCP integration, and session persistence are implemented.

#### Architecture

```text
 User / CLI
     │
     ▼
Orchestrator ──► runs the agent loop, feeds tool results back to the model
     │
     ▼
   Agent ──────► tool access + conversation history
     │
     ▼
Tool Registry ─► built-in tools (calculator, shell, filesystem)
     │
     └────────► MCP ToolAdapter ─► external MCP servers and their tools
```

#### Implemented

| Capability | Details |
|---|---|
| **Orchestrator** | Agent loop that executes each tool call through the Agent and returns results to the model |
| **Agent** | Resolves tools through the registry; maintains in-memory conversation history |
| **Tool layer** | Common `Name() / Description() / Execute()` contract and a registry for registering and discovering tools |
| **Built-in tools** | Calculator, Shell (validated `exec.LookPath` execution with captured output), Filesystem (read/write/list/search/delete) |
| **MCP client** | Built on the official MCP SDK; identifies itself to servers by implementation name and version |
| **MCP ToolAdapter** | Wraps each discovered MCP tool behind the harness tool interface so the Agent treats it like any other tool |
| **MCP CLI** | `mcp list`, `mcp add <name> <command> [args...]`, `mcp remove <name>` with persisted config and duplicate-name validation |
| **Sessions** | Active-session switching on create; file-backed store with atomic writes (temp file + rename); SQLite-backed store with transactions and foreign-key cascade deletes |
| **Providers** | Ollama chat provider with request validation and an injectable HTTP client |
| **Config** | JSON provider config with a `config.example.json` that contains no local credentials |
| **Quality** | Unit and integration tests; GitHub Actions workflow for formatting, tests, and `go vet` |

#### Usage

```bash
# Manage MCP servers from the CLI
agent-harness mcp list
agent-harness mcp add <name> <command> [args...]
agent-harness mcp remove <name>
```

```go
type Tool interface {
    Name() string
    Description() string
    Execute(args map[string]any) (any, error)
}
```

**Stack:** `Go` · `Ollama` · `MCP` · `SQLite (modernc.org/sqlite)` · `GitHub Actions`

**Use cases:** local-LLM tool-calling agents, extensible agent runtimes, testable provider and tool abstractions

---

### 2. [Durable Log Cache Engine](https://github.com/deepesh-kumar-pandey/durable-log-cache-engine) &nbsp;·&nbsp; Go

> **A crash-resilient log ingestion and caching engine with zero data loss guarantees**

**Status:** Complete

Low-latency log ingestion backed by a write-ahead log (WAL), a concurrent worker pipeline, an LRU in-memory cache, and a token-bucket rate limiter.

**Key features**
- Binary-framed, fsync-backed WAL: data is durable before any acknowledgement
- Crash recovery: replays the WAL on boot, restores payloads into the LRU cache, then compacts the log
- O(1) LRU cache index with hit-rate telemetry
- Configurable worker pool with automatic panic recovery and self-healing replacement
- Token-bucket rate limiter with configurable burst depth; excess load shed with `429`
- HTTP API: `/submit`, `/lookup/{id}`, `/metrics`, `/health`
- Multi-stage, minimal Alpine Docker image running as `nobody`
- 20 tests across cache and pipeline packages, including race-detector coverage

**Benchmarked performance** (AMD Ryzen 5 5600H)

| Component | Operation | Throughput | Latency |
|---|---|---|---|
| LRU Cache | Get (hit) | ~5.4M ops/sec | 183 ns |
| WAL (disk I/O) | Write + fsync | ~348K ops/sec | 2.8 µs |
| Rate Limiter | Allow (concurrent) | ~5.2M ops/sec | 189 ns |

```bash
curl -X POST http://localhost:8080/submit \
  -H "Content-Type: application/json" \
  -d '{"id":"TX-001","payload":"user_login_event"}'
# → 202 Accepted, WAL write in flight
```

**Stack:** `Go` · `Docker` · `Alpine Linux`

**Use cases:** durable event ingestion, crash-safe log pipelines, low-latency lookup caches

---

### 3. [Automation Engine](https://github.com/deepesh-kumar-pandey/Automation-Engine) &nbsp;·&nbsp; C++ / React

> **An asynchronous task orchestration system that automates workflows across applications, shell environments, and infrastructure, driven entirely by JSON**

**Status:** Core complete (backend, security worker, dashboard, and run pipeline working end-to-end)

Routines are defined as JSON workflows made of polymorphic, non-blocking tasks, from shell automation to a live threat-detection worker that talks to a rate limiter over an HMAC-signed API.

**Key features**
- **Async core:** non-blocking task execution via `std::async` / `std::future`
- **Polymorphic workers:** extend through an abstract `Task` base class
- **Threat analyzer:** C++ TCP server that parses incoming requests for SQL injection and blocks offending IPs in real time
- **Security integration:** `BlockIPTask` calls a rate limiter service directly, with requests signed using HMAC-SHA256
- **Encrypted auditing:** logs encrypted on disk with AES-256-GCM to detect tampering
- **JSON-driven workflows:** routines defined declaratively in `routine.json`
- **React dashboard:** real-time task and routine monitoring with a live packet inspector
- **CMake + vcpkg:** modern dependency management; fully Dockerized (backend, frontend, relay)

**Benchmarked performance** (300K requests, 64 concurrent workers)

| Mode | Throughput | Success rate | Avg latency | p99 latency |
|---|---|---|---|---|
| Steady (sustained) | 1,258 req/s | 100.0% | 50.58 ms | 128.97 ms |
| Burst (10K waves) | 1,258 req/s | 100.0% | 193.56 ms | 731.01 ms |

```cpp
class MyTask : public AutomationEngine::Task {
public:
    std::future<TaskStatus> execute(const json& input) override {
        return std::async(std::launch::async, [this, input]() {
            // your logic here
            return TaskStatus::Completed;
        });
    }
    std::string getName() const override { return "MyTask"; }
};
```

**Stack:** `C++` · `CMake` · `vcpkg` · `OpenSSL` · `React` · `TypeScript` · `Python` · `Node.js` · `Docker`

**Use cases:** workflow orchestration, intrusion detection and auto-blocking, encrypted audit logging, CI/CD-style automation

---

### 4. [Gatekeeper: Rate Limiting API](https://github.com/deepesh-kumar-pandey/API-project) &nbsp;·&nbsp; C++

> **A thread-safe rate limiter that protects backend services from traffic spikes and API abuse**

**Key features**
- Fixed-window counter algorithm with O(1) lookups
- Thread-safe concurrent operations with mutex-based locking
- Persistent state management (survives crashes and restarts)
- Docker-ready with an Alpine Linux base
- Zero external dependencies (pure C++ STL)

```cpp
RateLimiter limiter(100, 60);  // 100 requests per 60 seconds
if (limiter.is_request_allowed("user123")) {
    // Process request
} else {
    // Return 429 Too Many Requests
}
```

**Stack:** `C++11` · `POSIX Threads` · `Docker` · `Alpine Linux`

**Use cases:** API protection, brute-force prevention, cost control, fair-usage policies

---

### 5. [DeepGuard: Health Monitoring Service](https://github.com/deepesh-kumar-pandey/Health-Monitoring-Service) &nbsp;·&nbsp; C++

> **Cross-platform system monitoring with encrypted logging and real-time alerts**

**Key features**
- Real-time CPU and RAM monitoring (Windows and Linux)
- Disk usage analytics with configurable thresholds
- Database connectivity health checks (MySQL / PostgreSQL)
- XOR-encrypted log storage (AES-256-GCM upgrade in progress)
- Native system notifications (Windows MessageBox / Linux `notify-send`)
- Thread-safe concurrent operations

```cpp
Monitor monitor(80.0, "alerts.log", encryption_key);
monitor.run_monitoring_cycle(5);  // check every 5 seconds
```

**Stack:** `C++17` · `CMake` · `Docker` · `WinAPI` · `POSIX`

**Use cases:** server monitoring, DevOps automation, infrastructure observability

---

### 6. Additional Projects

| Project | Description | Stack |
|---|---|---|
| [Interactive To-Do List](https://github.com/deepesh-kumar-pandey/To_do_list) | Task management app with a clean UI and persistent storage | JavaScript, HTML5, CSS3, Local Storage API |
| [GUI Application](https://github.com/deepesh-kumar-pandey/GUI) | Desktop application demonstrating Python interface development | Python, Tkinter / PyQt |

---

## Skills

### Languages

![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white)
![C++](https://img.shields.io/badge/C++-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)
![C](https://img.shields.io/badge/C-A8B9CC?style=for-the-badge&logo=c&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)

### Infrastructure & DevOps

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)

### Build, Data & Frameworks

![CMake](https://img.shields.io/badge/CMake-064F8C?style=for-the-badge&logo=cmake&logoColor=white)
![GCC](https://img.shields.io/badge/GCC-A42E2B?style=for-the-badge&logo=gnu&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-000000?style=for-the-badge&logo=ollama&logoColor=white)

### Development Environment

| Category | Tools |
|---|---|
| **IDEs** | Visual Studio Code, Visual Studio, CLion, GoLand |
| **Build systems** | CMake, Make, MinGW, Go Modules, vcpkg |
| **Version control** | Git, GitHub |
| **Containerization** | Docker, Docker Compose |
| **Operating systems** | Windows, Linux (Ubuntu / Alpine), macOS |

---

## Core Competencies

### Systems & Storage Engineering
- **Write-ahead logging:** binary framing, fsync durability, crash recovery, compaction
- **Atomic persistence:** temp-file-and-rename writes, transactional SQLite stores
- **Concurrent programming:** goroutines and channels in Go; mutexes and lock guards in C++
- **Memory management:** RAII patterns in C++; allocation profiling in Go
- **Performance optimization:** complexity analysis, benchmarking, low-allocation hot paths
- **Cross-platform development:** Windows, Linux, and macOS compatibility

### Agent & Tooling Architecture
- **Tool interfaces:** uniform `Name() / Description() / Execute()` contracts for extensible tools
- **Orchestration:** separating "what to do" (orchestrator) from "how to do it" (agent and tools)
- **MCP integration:** client implementation, tool discovery, adapter layer, CLI-managed server config
- **Provider abstractions:** swappable LLM providers with injectable, testable dependencies
- **Session management:** persistent conversation state across file and database backends

### DevOps Practices
- **Containerization:** multi-stage Docker builds, Alpine optimization, non-root runtime
- **CI/CD:** GitHub Actions for formatting, test, and vet checks
- **Monitoring:** health checks, logging, metrics endpoints, alerting
- **Documentation:** architecture docs, API references, design-decision records

---

## Engineering Approach

```yaml
principles:
  code_quality:
    - "Clean, readable, maintainable code"
    - "Comprehensive error handling"
    - "Documentation that matches the code"

  performance:
    - "Algorithmic efficiency (O(1) where possible)"
    - "Minimal memory footprint"
    - "Benchmark before optimizing"

  reliability:
    - "Durable by design (WAL, fsync, atomic writes, crash recovery)"
    - "Thread- and goroutine-safe by design"
    - "Graceful failure handling"

  portability:
    - "Cross-platform compatibility"
    - "Zero or minimal dependencies"
    - "Standards-compliant Go and C++"
```

### What Sets My Projects Apart

1. **Built for failure:** crash recovery, atomic writes, and panic recovery are designed in rather than added later
2. **Measured:** benchmarked throughput and latency, with tests including race-detector runs
3. **Secure by default:** thread safety, encryption, signed requests, validated inputs, non-root containers
4. **Portable:** Docker deployment and cross-platform builds out of the box
5. **Documented:** architecture, crash-recovery, and design-decision notes for the major projects

---

## Selected Code Patterns

### Durable Write-Ahead Logging (Go)

```go
// Binary frame: [len:4][ts:8][payload:N], fsync'd before ack
func (w *WAL) Write(payload []byte) error {
    frame := encodeFrame(payload, time.Now().UnixNano())
    if _, err := w.file.Write(frame); err != nil {
        return err
    }
    return w.file.Sync() // durable before acknowledgement
}
```

### Tool Interface Abstraction (Go)

```go
type Tool interface {
    Name() string
    Description() string
    Execute(args map[string]any) (any, error)
}
```

### Thread-Safe Design (C++)

```cpp
class RateLimiter {
private:
    std::mutex global_mutex_;
    std::map<std::string, UserLimit> user_limits_;

public:
    bool is_request_allowed(const std::string& user_id) {
        std::lock_guard<std::mutex> lock(global_mutex_);
        // Thread-safe operations...
    }
};
```

---

## Currently Working On

- **Agent Harness:** expanding orchestration, adding tools, and broadening provider support
- **DeepGuard:** upgrading log encryption from XOR to AES-256-GCM
- **Observability:** Prometheus-style metrics export across projects
- **Testing:** unit tests, benchmarking, and performance profiling
- **Infrastructure as Code:** Terraform for automated cloud deployments

---

## Roadmap

### Short-term
- [ ] Add further tools and provider integrations to Agent Harness
- [ ] Implement an HTTP REST API for Gatekeeper
- [ ] Add AES-256-GCM encryption to DeepGuard
- [ ] Add Prometheus metrics export across projects

### Medium-term
- [ ] Distributed mode for the Durable Log Cache Engine (replication, Raft-backed WAL)
- [ ] Kubernetes operator for DeepGuard
- [ ] Web dashboard for monitoring
- [ ] Terraform-managed infrastructure deployments

### Long-term
- [ ] Production-grade observability platform
- [ ] Fully orchestrated, tool-using agent runtime
- [ ] Contributions to major open-source projects
- [ ] Educational content on systems programming

---

## Collaboration

I'm interested in:

- Systems programming projects (Go, C++, Rust)
- Agent tooling and LLM infrastructure
- DevOps tooling and infrastructure automation
- Monitoring and observability
- Open-source contributions

**Looking for:** code reviews on systems projects, feedback on architecture decisions, and collaborators on infrastructure and agent tooling.

---

## Contact

Open to internships and collaboration.

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/deepesh-kumar-pandey-013233288)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/deepesh-kumar-pandey)
[![LeetCode](https://img.shields.io/badge/LeetCode-FFA116?style=for-the-badge&logo=leetcode&logoColor=black)](https://leetcode.com/u/Deepesh_Kumar_Pandey)
[![HackerRank](https://img.shields.io/badge/HackerRank-2EC866?style=for-the-badge&logo=hackerrank&logoColor=white)](https://hackerrank.com/profile/deepesh040505)

*If you find these projects useful, a star is always appreciated.*

</div>
