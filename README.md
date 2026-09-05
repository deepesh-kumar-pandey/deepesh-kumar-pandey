<div align="center">

# 👋 Hi, I'm Deepesh Kumar Pandey

### Systems Engineer | C++ & Go Developer | Building Production-Grade Infrastructure


**Building robust systems that solve real-world problems** | **Focused on performance, reliability, and clean code**

[View Projects](#-featured-projects) • [Tech Stack](#-tech-stack)

</div>

---

## 🚀 About Me

I'm a **systems engineer** passionate about building **high-performance, production-grade infrastructure**. I started out deep in modern C++ — rate limiters, health monitors, thread-safe services — and have since picked up **Go**, which is now my primary language for backend and infrastructure work: durable storage engines, concurrent pipelines, and (most recently) an LLM agent harness.

Across languages, the focus stays the same: **thread safety**, **crash resilience**, **cross-platform compatibility**, and **minimal resource footprint**.

```go
package main

type Developer struct {
    Name           string
    Focus          string
    Specializations []string
}

func Me() Developer {
    return Developer{
        Name:  "Deepesh Kumar Pandey",
        Focus: "Systems Programming & Infrastructure",
        Specializations: []string{
            "Go & C++ Systems Development",
            "Durable Storage & Caching Engines",
            "LLM Agent Harnesses & Tooling",
            "Concurrent Programming",
            "DevOps Tooling",
        },
    }
}
```

### 💡 What I Do

- 🔧 **Systems Programming** - Building low-level infrastructure in Go and C++11/14/17
- 🗄️ **Durable Storage Engines** - Write-ahead logs, crash recovery, in-memory caching
- 🤖 **Agent Tooling** - Harnesses that connect LLMs to tools and execution loops
- 🐳 **DevOps Automation** - Dockerized applications, multi-platform deployment
- 🔒 **Security-First Design** - Thread-safe, encrypted, production-ready code
- 📊 **Monitoring & Observability** - Real-time system health tracking
- ⚡ **Performance Optimization** - Efficient algorithms, minimal overhead

---

## 🎯 Featured Projects

### 1️⃣ [Durable Log Cache Engine](https://github.com/deepesh-kumar-pandey/durable-log-cache-engine) — Go

> **A production-grade, crash-resilient log ingestion and caching engine with zero data loss guarantees**

**✅ Status: Complete**

Ultra-low-latency log ingestion backed by a Write-Ahead Log (WAL), a concurrent worker pipeline, an LRU in-memory cache, and a token-bucket rate limiter.

**🔑 Key Features:**
- ✅ Binary-framed, fsync-backed WAL — data is durable before any acknowledgement
- ✅ Crash recovery: replays the WAL on boot, restores payloads into the LRU cache, then compacts it
- ✅ O(1) LRU cache index with hit-rate telemetry
- ✅ Configurable worker pool with automatic panic recovery and self-healing replacement
- ✅ Token-bucket rate limiter with configurable burst depth; excess load shed with `429`
- ✅ HTTP API — `/submit`, `/lookup/{id}`, `/metrics`, `/health`
- ✅ Multi-stage, minimal Alpine Docker image running as `nobody`
- ✅ 20 tests across cache/pipeline packages, including race-detector coverage

**📈 Benchmarked Performance** (AMD Ryzen 5 5600H):
| Component | Operation | Throughput | Latency |
|---|---|---|---|
| LRU Cache | Get (Hit) | ~5.4M ops/sec | 183 ns |
| WAL (Disk I/O) | Write + fsync | ~348K ops/sec | 2.8 µs |
| Rate Limiter | Allow (Concurrent) | ~5.2M ops/sec | 189 ns |

**🛠️ Tech Stack:** `Go` • `Docker` • `Alpine Linux`

**💼 Use Cases:** durable event ingestion, crash-safe log pipelines, low-latency lookup caches

```bash
curl -X POST http://localhost:8080/submit \
  -H "Content-Type: application/json" \
  -d '{"id":"TX-001","payload":"user_login_event"}'
# → 202 Accepted, WAL write in flight
```

---

### 2️⃣ [Automation Engine](https://github.com/deepesh-kumar-pandey/Automation-Engine) — C++26 / React

> **A high-performance, asynchronous task orchestration system that automates workflows across applications, shell environments, and infrastructure — driven entirely by JSON**

**✅ Status: Core Complete** (backend, security worker, dashboard, and full run pipeline all working end-to-end)

Define "routines" as JSON-configured workflows made of polymorphic, non-blocking tasks — from shell automation to a live threat-detection worker that talks to a rate limiter over an HMAC-signed API.

**🔑 Key Features:**
- ✅ **Async Core** - non-blocking task execution via `std::async` / `std::future`
- ✅ **Polymorphic Workers** - extend via an abstract `Task` base class for any capability
- ✅ **Threat Analyzer** - a C++ TCP server that parses incoming requests for SQL injection and blocks bad IPs in real time
- ✅ **Security Integration** - `BlockIPTask` calls a Rate Limiter service directly, requests signed with HMAC-SHA256
- ✅ **Encrypted Auditing** - logs encrypted on disk with AES-256-GCM to detect tampering
- ✅ **JSON-Driven Workflows** - routines defined declaratively in `routine.json`
- ✅ **React Dashboard** - real-time task/routine monitoring with a live packet inspector
- ✅ **CMake + vcpkg** - modern dependency management; fully Dockerized (backend, frontend, relay)

**📈 Benchmarked Performance** (300K requests, 64 concurrent workers):
| Mode | Throughput | Success Rate | Avg Latency | p99 Latency |
|---|---|---|---|---|
| Steady (sustained) | 1,258 req/s | 100.0% | 50.58 ms | 128.97 ms |
| Burst (10K waves) | 1,258 req/s | 100.0% | 193.56 ms | 731.01 ms |

**🛠️ Tech Stack:** `C++26` • `React` • `TypeScript` • `Python` • `Node.js` • `OpenSSL` • `Docker`

**💼 Use Cases:** workflow orchestration, intrusion detection & auto-blocking, encrypted audit logging, CI/CD-style automation

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

---

### 3️⃣ [Agent Harness](https://github.com/deepesh-kumar-pandey/Agent-harness) — Go

> **A runtime framework connecting an LLM to external tools via a registry-and-execution-loop architecture**

**🚧 Status: In Progress**

An agent harness is the scaffolding that turns a language model into an autonomous agent: the model supplies the intelligence, the harness supplies the environment, the tool interface, and the execution loop.

**🔑 Implemented So Far:**
- ✅ **Config Layer** — loads and validates provider config (model, base URL, endpoint) from JSON
- ✅ **Provider Layer** — Ollama chat provider with request validation, injectable HTTP client, and unit + integration tests
- ✅ **Tool Interface** — a common `Name() / Description() / Execute()` contract for all tools
- ✅ **Tool Registry** — `Register`, `Get`, `Has`, `List`, `Remove` for managing available tools
- ✅ **Agent** — resolves a named tool from the registry and executes it via `Run`
- ✅ **Concrete Tools** — Calculator (`add`/`subtract`/`multiply`/`divide`/`modulus`), Shell (validated `exec.LookPath` execution with captured output), Filesystem (read/write/list/search/delete)

**🔄 Planned:**
- ⏳ **Orchestrator** — coordinates workflow decisions and tool selection (separating "what to do" from "how/when")
- ⏳ **Search Tool** — retrieval across external data sources
- ⏳ Task planning and full LLM-driven orchestration loop

```go
type Tool interface {
    Name() string
    Description() string
    Execute(args map[string]any) (any, error)
}
```

**🛠️ Tech Stack:** `Go` • `Ollama` • `net/http` • `os/exec`

**💼 Use Cases:** local-LLM tool-calling agents, extensible agent runtimes, testable provider/tool abstractions

---

### 4️⃣ [Gatekeeper - Rate Limiting API](https://github.com/deepesh-kumar-pandey/API-project)

> **Thread-safe rate limiter for protecting backend services from traffic spikes and API abuse**

**🔑 Key Features:**
- ✅ Fixed-window counter algorithm with O(1) lookups
- ✅ Thread-safe concurrent operations with mutex-based locking
- ✅ Persistent state management (survives crashes/restarts)
- ✅ Docker-ready with Alpine Linux base
- ✅ Zero external dependencies (pure C++ STL)

**🛠️ Tech Stack:** `C++11` • `POSIX Threads` • `Docker` • `Alpine Linux`

**💼 Use Cases:** API protection, brute-force prevention, cost control, fair usage policies

```cpp
// Simple, powerful API
RateLimiter limiter(100, 60);  // 100 requests per 60 seconds
if (limiter.is_request_allowed("user123")) {
    // Process request
} else {
    // Return 429 Too Many Requests
}
```

---

### 5️⃣ [DeepGuard - Health Monitoring Service](https://github.com/deepesh-kumar-pandey/Health-Monitoring-Service)

> **Cross-platform system monitoring with encrypted logging and real-time alerts**

**🔑 Key Features:**
- ✅ Real-time CPU/RAM monitoring (Windows & Linux)
- ✅ Disk usage analytics with configurable thresholds
- ✅ Database connectivity health checks (MySQL/PostgreSQL)
- ✅ XOR-encrypted log storage
- ✅ Native system notifications (Windows MessageBox / Linux notify-send)
- ✅ Thread-safe concurrent operations

**🛠️ Tech Stack:** `C++17` • `CMake` • `Docker` • `WinAPI` • `POSIX`

**💼 Use Cases:** Server monitoring, DevOps automation, infrastructure observability

```cpp
Monitor monitor(80.0, "alerts.log", encryption_key);
monitor.run_monitoring_cycle(5);  // Check every 5 seconds
```

---

### 6️⃣ [Interactive To-Do List](https://github.com/deepesh-kumar-pandey/To_do_list)

> **Modern task management application with clean UI and persistent storage**

**🛠️ Tech Stack:** `JavaScript` • `HTML5` • `CSS3` • `Local Storage API`

---

### 7️⃣ [GUI Application](https://github.com/deepesh-kumar-pandey/GUI)

> **Desktop GUI application demonstrating Python interface development**

**🛠️ Tech Stack:** `Python` • `Tkinter/PyQt`

---

## 🛠️ Tech Stack

### Languages & Core Technologies

```text
Go      ████████████████░░░░  80%  (Primary focus: storage engines, agent tooling)
C++     ████████████████████  95%  (C++11/14/17 systems programming)
Python  ████████░░░░░░░░░░░░  40%  (GUI applications, scripting)
JS/TS   ███████░░░░░░░░░░░░░  35%  (Web development)
Bash    ██████░░░░░░░░░░░░░░  30%  (Shell scripting, automation)
HCL     ███░░░░░░░░░░░░░░░░░  15%  (Terraform, learning IaC)
```

### Systems & Storage

<table>
<tr>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/go/go-original.svg" width="48" height="48" alt="Go" />
<br>Go
</td>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/cplusplus/cplusplus-original.svg" width="48" height="48" alt="C++" />
<br>C++
</td>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/c/c-original.svg" width="48" height="48" alt="C" />
<br>C
</td>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/cmake/cmake-original.svg" width="48" height="48" alt="CMake" />
<br>CMake
</td>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/gcc/gcc-original.svg" width="48" height="48" alt="GCC" />
<br>GCC
</td>
</tr>
</table>

### DevOps & Infrastructure

<table>
<tr>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/docker/docker-original.svg" width="48" height="48" alt="Docker" />
<br>Docker
</td>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/linux/linux-original.svg" width="48" height="48" alt="Linux" />
<br>Linux
</td>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/bash/bash-original.svg" width="48" height="48" alt="Bash" />
<br>Bash
</td>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/git/git-original.svg" width="48" height="48" alt="Git" />
<br>Git
</td>
<td align="center" width="96">
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/terraform/terraform-original.svg" width="48" height="48" alt="Terraform" />
<br>Terraform
</td>
</tr>
</table>

### Development Tools

- **IDEs:** Visual Studio Code, Visual Studio, CLion, GoLand
- **Build Systems:** CMake, Make, MinGW, Go Modules
- **Version Control:** Git, GitHub
- **Containerization:** Docker, Docker Compose
- **Operating Systems:** Windows, Linux (Ubuntu/Alpine), macOS

---

## 💼 Core Competencies

### Systems & Storage Engineering
- **Write-Ahead Logging** - binary framing, fsync durability, crash recovery, compaction
- **Concurrent Programming** - goroutines/channels in Go; mutex/lock guards in C++
- **Cross-Platform Development** - Windows, Linux, macOS compatibility
- **Memory Management** - RAII patterns in C++; careful allocation profiling in Go
- **Performance Optimization** - algorithm complexity analysis, benchmarking, zero-allocation hot paths

### Agent & Tooling Architecture
- **Tool Interfaces** - uniform `Name()/Description()/Execute()` contracts for extensible tools
- **Provider Abstractions** - swappable LLM providers with injectable, testable dependencies
- **Registry Patterns** - centralized tool discovery decoupled from orchestration logic

### DevOps Practices
- **Containerization** - Docker multi-stage builds, Alpine optimization, non-root runtime
- **CI/CD** - build automation, testing pipelines
- **Monitoring** - health checks, logging, metrics endpoints, alerting
- **Documentation** - technical writing, architecture docs, API references

---

## 🎓 Technical Approach

```yaml
principles:
  code_quality:
    - "Clean, readable, maintainable code"
    - "Comprehensive error handling"
    - "Extensive documentation"

  performance:
    - "Algorithmic efficiency (O(1) when possible)"
    - "Minimal memory footprint"
    - "Benchmark before optimizing"

  reliability:
    - "Durable by design (WAL, fsync, crash recovery)"
    - "Thread/goroutine-safe by design"
    - "Graceful failure handling"

  portability:
    - "Cross-platform compatibility"
    - "Zero/minimal dependencies"
    - "Standards-compliant Go and C++"
```

### What Sets My Projects Apart

1. **Production-Ready** - not demos; built with crash recovery, benchmarks, and Docker deployment in mind
2. **Well-Documented** - comprehensive READMEs, architecture docs, API references
3. **Performance-Conscious** - benchmarked throughput/latency, algorithmic analysis
4. **Security-Minded** - thread safety, encryption, validated inputs
5. **Cross-Platform** - works on Windows, Linux, Docker out-of-the-box

---

## 🌱 Currently Learning

- 🤖 **Agent Orchestration** - building the Orchestrator layer and full LLM-driven execution loop for Agent Harness
- 🔐 **AES-256-GCM Encryption** - upgrading DeepGuard from XOR to production-grade crypto
- 📊 **Prometheus Integration** - metrics export for monitoring systems
- 🧪 **Advanced Testing** - unit tests, benchmarking, performance profiling
- ☁️ **Terraform** - Infrastructure as Code for automated cloud deployments

---

## 🏆 Achievements & Highlights

- 🔧 **9 Public Repositories** - active open-source contributor, now spanning Go, C++, and full-stack projects
- 🗄️ **Durable Systems** - shipped a WAL-backed cache engine with crash recovery and benchmarked performance
- 🛡️ **Security-Grade Automation** - built an async C++26 orchestration engine with encrypted auditing, HMAC-signed requests, and a live threat-blocking worker, load-tested to 300K requests
- 🤖 **Agent Tooling** - building an extensible LLM agent harness from the ground up
- 🐳 **Docker Expertise** - all major projects containerized
- 📚 **Comprehensive Documentation** - architecture, crash-recovery, and design-decision docs for every major project

---

## 💬 Notable Code Patterns

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

func (a *Agent) Run(name string, args map[string]any) (any, error) {
    tool, err := a.registry.Get(name)
    if err != nil {
        return nil, err
    }
    return tool.Execute(args)
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

## 🎯 Project Roadmap

### Short-term Goals
- [ ] Build the Orchestrator layer for Agent Harness (tool selection, workflow coordination)
- [ ] Add a Search tool to Agent Harness
- [ ] Implement HTTP REST API for Gatekeeper
- [ ] Add AES-256-GCM encryption to DeepGuard
- [ ] Add Prometheus metrics export across projects

### Medium-term Goals
- [ ] Full LLM-driven planning/orchestration loop in Agent Harness
- [ ] Distributed mode for Durable Log Cache Engine (replication / Raft-backed WAL)
- [ ] Build Kubernetes operator for DeepGuard
- [ ] Create a web dashboard for monitoring
- [ ] Deploy infrastructure using Terraform

### Long-term Vision
- [ ] Build a production-grade observability platform
- [ ] Ship a fully orchestrated, tool-using agent runtime
- [ ] Contribute to major open-source projects
- [ ] Create educational content on systems programming

---

## 🤝 Let's Collaborate!

I'm always interested in:

- 🔧 **Systems programming projects** (Go, C++, Rust)
- 🤖 **Agent tooling & LLM infrastructure**
- 🐳 **DevOps tooling** and infrastructure automation
- 📊 **Monitoring & observability** solutions
- 📚 **Open source** contributions

### Looking For

- **Code reviews** on systems programming projects
- **Collaboration** on infrastructure tools and agent harnesses
- **Feedback** on architecture decisions
- **Contributions** to existing projects

---

## 📫 Connect With Me

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/deepesh-kumar-pandey)

</div>

---

## 🌟 Project Philosophy

I build projects that:
- ✅ Solve real problems
- ✅ Are production-ready
- ✅ Have minimal dependencies
- ✅ Work cross-platform
- ✅ Include comprehensive docs
- ✅ Follow best practices

---

<div align="center">

### ⭐ If you find my projects useful, consider giving them a star!

### 💻 Open for collaboration and always learning

**Thanks for visiting!** 🚀

---

</div>
