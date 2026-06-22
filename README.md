# jujulabz101# JuJuLabz Platform

**Platform Identifier:** `JCH-2026-JUJU-001`  
**Author:** Julius Hill  
**License:** [Choose your license—see `LICENSE`](LICENSE)  
**Status:** 🚧 Prototype / Early Open Source

***

## ⚡ What Is This?

JuJuLabz is a **monolithic, self-contained AI orchestration platform** that fuses:

- **Zig WASM guest sandbox** for edge-layer payload mutation
- **Go host runtime** with five-layer substrate architecture:
  - **Cortex:** Meta-learning engine with neural architecture search (NAS)
  - **Prometheus:** Self-healing orchestration supervisor with Kafka failure logging
  - **Mneme:** Three-tier memory (Postgres+pgvector, Redis cache, local exp)
  - **Provenance:** Pedersen zero-knowledge audit chain with cryptographic commitments
  - **Edge:** WASM isolation runtime via wazero

It runs **zero external dependencies** beyond Docker containers for Postgres, Redis, and Kafka—everything else is compiled into a single binary.

***

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│           JuJuLabz Monolithic Platform                      │
├─────────────┬─────────────┬─────────────┬───────────────────┤
│   Cortex    │  Prometheus │    Mneme    │     Provenance    │
│  (NAS +     │  (Supervisor│   (3-tier   │   (ZK Commit-     │
│  Self-Play) │   + Kafka)  │   Memory)   │    ments)         │
├─────────────┴─────────────┴─────────────┴───────────────────┤
│                    Edge Layer (WASM)                         │
│              Zig Guest → wazero Host Runtime                 │
└─────────────────────────────────────────────────────────────┘
```

**Runtime Stack:**
- **Zig:** `wasm32-wasi` guest for unsafe payload mutation in sandbox
- **Go:** `wazero` WASM runtime, `pgx/v5` Postgres, `go-redis/v9`, `franz-go` Kafka
- **Docker:** `pgvector/pgvector:pg16`, `redis:7-alpine`, `confluentinc/cp-kafka:7.5.0` (KRaft)

***

## 🚀 Quick Start

```bash
# Clone the repo
git clone https://github.com/jujulabz/jujulabz-core.git
cd jujulabz-core

# Run the monolithic deployment script
./deploy.sh
```

The script will:
1. Spin up Docker containers (Postgres+pgvector, Redis, Kafka KRaft)
2. Compile Zig WASM guest → `bin/main.wasm`
3. Compile Go monolith → `bin/jujulabz`
4. Launch the platform on port `9000` (override with `JUJU_PORT`)

### Environment Variables

| Variable        | Default   | Description                      |
|-----------------|-----------|----------------------------------|
| `JUJU_PORT`     | `9000`    | HTTP server port                 |
| `DATABASE_URL`  | (built)   | Postgres connection string       |
| `REDIS_URL`     | (built)   | Redis connection string          |
| `KAFKA_BROKERS` | (built)   | Kafka broker address             |

***

## 📁 Project Structure

```
jujulabz-core/
├── deploy.sh              # Monolithic deployment script
├── main.zig               # Zig WASM guest (edge layer)
├── main.go                # Go host (5-layer substrate)
├── go.mod                 # Go module definition
├── bin/
│   ├── main.wasm          # Compiled Zig guest
│   └── jujulabz           # Compiled Go binary
└── README.md
```

***

## 🧠 Key Features

### Cortex Layer: Meta-Learning Engine
- **Neural Architecture Search (NAS):** Evolutionary population with fitness scoring
- **Self-Play Tournament:** Challenger vs. current config comparison with delta tracking
- **Fitness Function:** Parameter count penalty + learning rate + depth penalty

### Prometheus Layer: Self-Healing Supervisor
- **Failure Logging:** Kafka topic `jujulabz-failures` for component errors
- **System Health Check:** GC-triggered heap monitor (>512MB threshold)
- **State Machine:** `Idle → Learning → SelfPlay → Healing → Idle`

### Mneme Layer: Three-Tier Memory
- **Postgres+pgvector:** `experiential_memory` table with 64-dim embeddings
- **Redis Cache:** 24-hour TTL heuristic caching
- **Local Exp:** In-memory ring buffer (10K entries)

### Provenance Layer: Zero-Knowledge Audit
- **Pedersen Commitments:** Elliptic curve (P256) hash-based commitments
- **Cryptographic Head:** Chain-linked `headX/headY` for audit trail

### Edge Layer: WASM Isolation
- **Zig Guest:** `alloc`/`dealloc`/`handle` FFI for safe memory mutation
- **wazero Host:** WASI snapshot preview1 for sandboxed execution

***

## 🛠️ Build Manual

If you want to build components separately:

### Zig WASM Guest
```bash
zig build-exe main.zig -target wasm32-wasi -O ReleaseFast
mv main.wasm bin/main.wasm
```

### Go Monolith
```bash
go mod init jujulabz-core
go get github.com/tetratelabs/wazero@v1.7.2
go get github.com/jackc/pgx/v5@v5.6.0
go get github.com/redis/go-redis/v9@v9.5.1
go get github.com/twmb/franz-go@v1.16.1
go build -o bin/jujulabz main.go
```

***

## 🧪 API Endpoints

### `GET /`
Returns platform status:

```json
{
  "platform": "JUJULABZ",
  "specification": "JCH-2026-JUJU-001",
  "uptime": "5m32s",
  "active_generation": 47,
  "cached_local_count": 892,
  "cortex_best_nas": {
    "layers": [128, 64],
    "activation": "gelu",
    "lr": 0.0011,
    "score": 0.0847,
    "generation": 47
  }
}
```

Headers:
- `Content-Type: application/json`
- `X-Powered-By: JuJuLabz-Platform-JCH-2026`

***

## ⚠️ Known Issues

- **Zig ABI Fragility:** Pointer packing in `handle` assumes 32-bit alignment; may break on 64-bit hosts
- **Kafka KRaft Bootstrapping:** Single-broker KRaft requires exact env var alignment (see `deploy.sh`)
- **No Shutdown Path:** Ticker loop has no graceful stop; send `SIGTERM` to kill
- **pgvector Column Unused:** Embedding column exists but is not populated

***

## 🤝 Contributing

This is early open source. Expect breaking changes.

1. Fork the repo
2. Create a feature branch
3. Submit a PR with tests if you touch core layers

**Areas that need help:**
- Graceful shutdown + signal handling
- pgvector embedding generation
- Zig ABI safety guarantees
- Kafka client cleanup on exit

***

## 📜 License



***

## 👤 Author

**Julius Hill**  
📍 Brentwood, Tennessee, US  
🔗 [GitHub](https://github.com/jujulabz)  
🎵 Music production + AI integration

***

## 🏷️ Tags

`ai-orchestration` `wasm` `zig` `go` `neural-architecture-search` `zero-knowledge` `monolith` `self-healing` `kafka` `pgvector` `edge-computing`
