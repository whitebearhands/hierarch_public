# Hierarch — Architecture Overview

> **Hierarch** is a generative AI agent orchestration platform that transforms natural language requirements into fully autonomous, multi-agent workflows — no code required.

> ⚠️ **Active development.** Server and client application are maintained in private repositories.  
> The same technology is currently **under patent examination**.  
> Inquiries: **hwansys@naver.com**

---

## Vision

Traditional automation requires engineers to write, deploy, and maintain complex pipelines.  
Hierarch eliminates that gap: describe what you need in plain language, and the platform designs, builds, and runs a team of AI agents to accomplish it.

```mermaid
flowchart LR
    A["💬 Natural Language\nRequirement"] --> B(["⚙️ Hierarch\nPlatform"])
    B --> C["🤖 Autonomous\nMulti-Agent Workflow"]
    C --> D["📊 Result"]

    style A fill:#f0f4ff,stroke:#6366f1
    style B fill:#6366f1,stroke:#4f46e5,color:#fff
    style C fill:#f0f4ff,stroke:#6366f1
    style D fill:#ecfdf5,stroke:#10b981
```

---

## Core Differentiators

| # | Feature | Description |
|---|---------|-------------|
| 1 | **Conversational Blueprint Design** | An AI architect conducts a structured interview to extract requirements and produces an executable workflow specification |
| 2 | **Automatic Execution Strategy** | Each agent's optimal execution mode (AI reasoning vs. deterministic code) is decided automatically at compile time |
| 3 | **Dynamic Code Synthesis** | The workflow specification is compiled into a runnable program on-the-fly — no static templates, no manual wiring |
| 4 | **Isolated Execution + Token Budget** | Each run is sandboxed in an isolated process with a hard token spending limit to guarantee cost control |
| 5 | **Semantic Tool Discovery** | External MCP tools are discovered via embedding-based vector search and automatically bound to the agents that need them |

---

## System Architecture

Hierarch operates as a **three-phase pipeline**: Design → Build → Execute.

> **Key principle:** Design and code synthesis happen on the **server**. Execution happens **locally on the user's machine** via the desktop client app.

```mermaid
flowchart TD
    subgraph CLOUD["☁️ Cloud Server"]
        subgraph Phase1["Phase 1 · Design"]
            U["User"] -->|natural language| A["AI Architect"]
            A -->|clarifying questions| U
            A -->|Blueprint| DB1[("State Store")]
        end

        subgraph Phase2["Phase 2 · Build"]
            DB1 --> C["Compiler\n(strategy decision)"]
            C -->|tool search| R[("MCP\nRegistry")]
            R --> C
            C --> S["Synthesizer\n(code generation)"]
            S -->|synthesized script| DB2[("Script Store")]
        end
    end

    subgraph LOCAL["🖥️ Client Machine (Tauri App)"]
        subgraph Phase3["Phase 3 · Execute"]
            DB2 -->|download script| RT["Local\nSandboxed Runtime"]
            RT -->|real-time logs| UX["User Interface"]
            RT --- CB["Token\nCircuit Breaker"]
        end
    end

    Phase1 --> Phase2 --> Phase3

    style CLOUD fill:#f8faff,stroke:#6366f1
    style LOCAL fill:#f0fdf4,stroke:#22c55e
    style Phase1 fill:#eff6ff,stroke:#3b82f6
    style Phase2 fill:#fdf4ff,stroke:#a855f7
    style Phase3 fill:#dcfce7,stroke:#16a34a
```

---

## Phase 1 — AI Architect (Design)

The platform begins with a **conversational interview**.  
The AI Architect asks targeted questions to understand the goal, data sources, constraints, and approval requirements — then produces a structured **Blueprint**.

```mermaid
sequenceDiagram
    actor User
    participant Architect as AI Architect
    participant Store as State Store

    User->>Architect: "I need a report on trending GitHub repos"
    Architect->>User: "What time range? Any specific language?"
    User->>Architect: "Past 7 days, Python only"
    Architect->>Store: Save draft Blueprint
    Architect-->>User: "Got it. Any approval step before sending the report?"
    User->>Architect: "No, just run it"
    Architect->>Store: Save final Blueprint (is_complete = true)
    Architect-->>User: ✅ Design complete — ready to build
```

### Hierarchical Agent Structure

Every Blueprint step uses a **leader + sub-agent** team model:

```mermaid
flowchart TB
    subgraph Step["Workflow Step"]
        direction TB
        L["👑 Leader Agent\n(finalizes & synthesizes)"]
        S1["🔵 Sub-Agent A\n(parallel task)"]
        S2["🔵 Sub-Agent B\n(parallel task)"]
        S3["🔵 Sub-Agent C\n(parallel task)"]

        S1 & S2 & S3 -->|partial results| L
    end

    Prev["Previous Step\nOutput"] -->|context injection| Step
    Step -->|final output| Next["Next Step"]

    style L fill:#fef9c3,stroke:#ca8a04
    style S1 fill:#dbeafe,stroke:#3b82f6
    style S2 fill:#dbeafe,stroke:#3b82f6
    style S3 fill:#dbeafe,stroke:#3b82f6
```

Sub-agents run **in parallel**. The leader receives all partial results and produces the step's final output, which flows into subsequent steps.

---

## Phase 2 — Compiler + Synthesizer (Build)

The Blueprint is transformed into a runnable program through two stages.

```mermaid
flowchart LR
    subgraph Compile["Compiler"]
        direction TB
        BP["Blueprint\n(logical)"] --> VEC["Semantic\nTool Search"]
        VEC --> LLM1["Strategy\nDecision LLM"]
        LLM1 -->|execution_type:\nai · code| PBP["Physical Blueprint\n(enriched)"]
    end

    subgraph Synthesize["Synthesizer"]
        direction TB
        PBP2["Physical Blueprint"] --> TMP["Template\nEngine"]
        TMP --> SCR["Executable\nScript"]
    end

    Compile --> Synthesize

    style Compile fill:#fdf4ff,stroke:#a855f7
    style Synthesize fill:#eff6ff,stroke:#3b82f6
```

**Compiler decisions per agent:**

```mermaid
flowchart TD
    D{"Agent task type?"}
    D -->|"Reasoning / analysis /\nweb search / MCP tool"| AI["🤖 AI Execution\nLLM inference + optional tool call"]
    D -->|"File I/O / API call /\nparsing / calculation"| CODE["⚙️ Code Execution\nDeterministic Python snippet"]

    style AI fill:#eff6ff,stroke:#6366f1
    style CODE fill:#f0fdf4,stroke:#22c55e
```

### MCP Tool Discovery

Hierarch maintains a **registry of external MCP servers** (sourced from Smithery).  
During compilation, agent descriptions are embedded and matched to relevant tools via vector similarity search.

```mermaid
flowchart LR
    DESC["Agent\nDescription"] -->|embed| VEC["768-dim\nVector"]
    VEC -->|cosine search| REG[("MCP\nRegistry\n(Firestore)")]
    REG -->|best match| TOOL["Matched Tool\n+ auth config"]
    TOOL --> AGENT["Agent gets\ntool endpoint\n& credentials"]

    style REG fill:#fef9c3,stroke:#ca8a04
    style TOOL fill:#ecfdf5,stroke:#10b981
```

---

## Phase 3 — Runtime (Execute)

The synthesized script is **downloaded to the user's machine** and executed locally inside the Tauri desktop app.  
All LLM API calls and MCP tool connections originate from the **client**, keeping user credentials entirely off the server.

```mermaid
sequenceDiagram
    participant SRV as Cloud Server
    participant APP as Tauri App
    participant RT as Local Runtime
    participant AG as Agent Process
    participant CB as Token Circuit Breaker

    APP->>SRV: Request synthesized script
    SRV-->>APP: Download script + required credential list
    APP->>APP: User provides API keys (stored locally)

    APP->>RT: Execute script locally
    RT->>AG: Spawn isolated subprocess
    activate AG

    loop For each workflow step
        AG->>AG: Inject previous step context
        AG->>AG: Run sub-agents in parallel
        AG-)SRV: LLM API calls (direct, with user's key)
        AG->>CB: Report token usage
        CB-->>AG: ✅ Within budget
        AG->>AG: Leader finalizes step output
        AG-->>RT: Stream JSON log event
        RT-->>APP: Real-time log display
    end

    alt Normal completion
        AG-->>RT: exit 0
        RT-->>APP: ✅ Workflow complete
    else Token budget exceeded
        CB->>AG: ⛔ STOP
        RT-->>APP: ⚠️ Budget limit reached
    else Timeout
        RT->>AG: Force kill
        RT-->>APP: ⏱ Timed out
    end

    deactivate AG
```

### Token Circuit Breaker

Every project sets a **maximum token budget** at design time.  
The circuit breaker monitors cumulative token consumption across all agents in real time — if the limit is exceeded, execution halts immediately, preventing runaway costs.

```mermaid
xychart-beta
    title "Token Budget Enforcement"
    x-axis ["Step 1", "Step 2", "Step 3", "Step 4 (tripped)"]
    y-axis "Cumulative Tokens" 0 --> 60000
    line [8000, 22000, 39000, 50000]
    bar  [8000, 22000, 39000, 50000]
```

---

## API Interface

All interactions are available via a simple REST + SSE API.

```mermaid
flowchart LR
    subgraph Design
        C1["POST /chat"]
    end
    subgraph Build
        C2["GET /build/stream/{thread_id}"]
    end
    subgraph Execute
        C3["GET /execute/stream/{script_id}"]
    end
    subgraph Utility
        C4["GET /threads"]
        C5["GET /users/usage"]
        C6["GET /mcp"]
    end

    Design --> Build --> Execute
```

| Phase | Endpoint | Transport | Description |
|-------|----------|-----------|-------------|
| Design | `POST /chat` | SSE stream | Conversational blueprint design |
| Build | `GET /build/stream/{id}` | SSE stream | Real-time compile + code synthesis |
| Execute | `GET /execute/stream/{id}` | SSE stream | Sandboxed workflow execution |
| — | `GET /threads` | REST | List user sessions |
| — | `GET /users/usage` | REST | Token consumption tracking |
| — | `GET /mcp` | REST | Available MCP tool catalog |

---

## System Components

Hierarch is composed of three independently managed components.  
The division of responsibility between server and client is intentional: **heavy AI work on the server, execution on the user's machine**.

```mermaid
flowchart TB
    subgraph CLIENT["🖥️ Client Machine"]
        APP["Tauri Desktop App\n(Windows / macOS)"]
        RT["Local Runtime\n(sandboxed subprocess)"]
        APP -->|"launch & stream logs"| RT
    end

    subgraph SERVER["☁️ Cloud Server"]
        API["Backend API\n(Python · FastAPI)"]
        SYNTH["AI Architect\n+ Compiler\n+ Synthesizer"]
        API --> SYNTH
    end

    subgraph STORE["🗄️ Cloud Storage"]
        FS["Firestore\n· Sessions · Blueprints\n· MCP Registry · Vectors"]
    end

    subgraph TOOLS["🔌 External Services"]
        LLM["LLM API\n(Gemini / OpenAI)"]
        MCP["MCP Tool Servers\n(Smithery)"]
    end

    APP <-->|"① Chat / Build\nREST + SSE"| API
    API <-->|"Read / Write"| FS
    APP -->|"② Download script"| API
    RT -->|"③ LLM calls\n(user's API key)"| LLM
    RT -->|"③ Tool calls"| MCP

    style CLIENT fill:#f0fdf4,stroke:#22c55e
    style SERVER fill:#f8faff,stroke:#6366f1
    style STORE fill:#fef9c3,stroke:#ca8a04
    style TOOLS fill:#fdf4ff,stroke:#a855f7
```

| Component | Technology | Responsibility |
|-----------|-----------|----------------|
| **Desktop App** | Tauri (Rust + WebView) | Chat UI · Build trigger · Script execution · Log display |
| **Backend Server** | Python · FastAPI | Blueprint design · Compilation · Code synthesis |
| **Database** | Google Cloud Firestore | Sessions, blueprints, MCP registry, vector index |
| **Local Runtime** | Subprocess (inside Tauri) | Execute synthesized script · Token circuit breaker |

### Why run locally?

- **Privacy** — user API keys and data never pass through the server during execution
- **Flexibility** — users can connect to any LLM provider or local model
- **No server cost** — LLM inference costs go directly from user to provider

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Desktop App | Tauri · Rust |
| API Server | Python · FastAPI · async SSE |
| AI / LLM | Google Gemini (multi-model) · LangChain |
| Embeddings | Gemini Embedding API · 768-dim vectors |
| Vector Search | Google Cloud Firestore native vector index |
| State Store | Google Cloud Firestore |
| Tool Ecosystem | MCP (Model Context Protocol) · Smithery Registry |
| Code Synthesis | Template-based dynamic program generation |
| Runtime | Isolated subprocess · token-aware circuit breaker |

---

## Security Design

```mermaid
flowchart TD
    subgraph Isolation["Process Isolation"]
        P["Parent API Server"] -->|"spawn"| CH["Child Process\n(synthesized script)"]
        P -. "credentials\nvia env only" .-> CH
        CH -->|"stdout logs"| P
        CH -->|"auto-deleted\nafter run"| TMP["Temp Script File"]
    end

    subgraph Budget["Cost Control"]
        CB2["Token\nCircuit Breaker"] -->|"hard stop"| CH
    end

    subgraph Tenancy["Multi-Tenancy"]
        U1["User A data"] -. isolated .-> U2["User B data"]
    end

    style Isolation fill:#fef2f2,stroke:#ef4444
    style Budget fill:#fffbeb,stroke:#f59e0b
    style Tenancy fill:#f0fdf4,stroke:#22c55e
```

1. **Local Execution** — synthesized code runs entirely on the user's machine; the server never executes user workloads
2. **Credential Scoping** — API keys are stored and used locally in the desktop app, never transmitted to the server
3. **Process Isolation** — each workflow runs in an isolated subprocess inside the Tauri app, separate from the UI process
4. **Automatic Cleanup** — temporary script files are deleted immediately after execution completes
5. **Token Budget** — per-project spending limits are enforced locally, preventing runaway LLM costs
6. **User Isolation** — all cloud data is partitioned by user identity at the storage layer

---

---

## Project Status

| Item | Status |
|------|--------|
| Development | 🔧 Active development |
| Source code | 🔒 Private repositories (server + client app) |
| Patent | 📋 Under examination (same technology) |

---

## Contact

Questions, collaboration, or licensing inquiries:

**✉️ hwansys@naver.com**

---

*Built with ❤️ by WhiteBearHands*
