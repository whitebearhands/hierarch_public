# Hierarch — Architecture Overview

> **Hierarch** is an agentic automation platform that synthesizes workflows into executable code via **Harness Engineering.**.

> Describe your workflow in plain language — Hierarch designs the multi-agent architecture, determines each agent's execution strategy, and generates a production-ready program using **Harness Engineering**: every agent is compiled independently and assembled into a single, self-contained executable that runs your automation end-to-end.  
> You define the goal. Hierarch writes and runs the code.

> ⚠️ **Active development.** Server and client application are maintained in **private repositories**.  
> The same technology is currently **under patent examination**.  
> Inquiries: **hwansys@naver.com**

---

## Vision

Every day, people repeat the same tasks: checking news, researching markets, summarizing documents, monitoring competitors.  
Hierarch automates these recurring workflows — describe what you need once, and a team of AI agents handles it from then on.

```mermaid
flowchart LR
    A["💬 Describe your\nworkflow in plain language"] --> B(["⚙️ Hierarch\nDesign · Build · Execute"])
    B --> C["🤖 Autonomous\nMulti-Agent Workflow"]
    C --> D["📊 Automated\nResults — daily, weekly"]

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
| 4 | **Dual Execution Mode** | Run workflows on-demand locally, or schedule them on the cloud for fully hands-free automation |
| 5 | **Token Budget & Circuit Breaker** | Every run has a hard token spending limit — costs are predictable and capped |
| 6 | **Semantic Tool Discovery** | External MCP tools are discovered via embedding-based vector search and automatically bound to the agents that need them |

---

## How It Works

Hierarch operates as a **three-phase pipeline**: Design → Build → Execute.

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

    subgraph EXEC["⚡ Phase 3 · Execute"]
        direction LR
        DB2 --> LOCAL["🖥️ Local Execution\n(Desktop App)"]
        DB2 --> SERVER["☁️ Cloud Execution\n(Scheduled)"]
    end

    Phase1 --> Phase2 --> EXEC

    style CLOUD fill:#f8faff,stroke:#6366f1
    style EXEC fill:#ecfdf5,stroke:#10b981
    style Phase1 fill:#eff6ff,stroke:#3b82f6
    style Phase2 fill:#fdf4ff,stroke:#a855f7
    style LOCAL fill:#f0fdf4,stroke:#22c55e
    style SERVER fill:#eff6ff,stroke:#6366f1
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

    User->>Architect: "I need a daily news briefing on AI trends"
    Architect->>User: "What topics? Any specific sources?"
    User->>Architect: "Tech + economy, summarized by 8 AM"
    Architect->>Store: Save draft Blueprint
    Architect-->>User: "Should I schedule it daily, or run on demand?"
    User->>Architect: "Daily at 8 AM"
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
        LLM1 -->|"execution_type:\nllm · mcp · python"| PBP["Physical Blueprint\n(enriched)"]
    end

    subgraph Synthesize["Synthesizer · Harness Engineering"]
        direction TB
        PBP2["Physical Blueprint"] --> CG["Per-Agent\nCode Generator"]
        CG --> ASM["Assembly"]
        ASM --> SCR["Self-contained\nExecutable Script"]
    end

    Compile --> Synthesize

    style Compile fill:#fdf4ff,stroke:#a855f7
    style Synthesize fill:#eff6ff,stroke:#3b82f6
```

**Compiler decisions per agent:**

```mermaid
flowchart TD
    D{"Agent task type?"}
    D -->|"Reasoning /\nanalysis / creation"| LLM["🤖 LLM Mode\nPure language model inference"]
    D -->|"MCP tool available\n& relevant"| MCP["🔌 MCP Mode\nReAct loop over tool server"]
    D -->|"File I/O / calculation /\ndeterministic logic"| CODE["⚙️ Python Mode\nDeterministic code snippet"]

    style LLM fill:#eff6ff,stroke:#6366f1
    style MCP fill:#fdf4ff,stroke:#a855f7
    style CODE fill:#f0fdf4,stroke:#22c55e
```

By deciding execution strategy at **compile time** rather than runtime, Hierarch avoids unnecessary LLM calls — deterministic tasks run as plain code, saving both cost and latency.

---

## Harness Engineering — Code Synthesis Strategy

### What is Agent Harness Engineering?

In conventional multi-agent frameworks (LangChain, AutoGen, CrewAI), a **harness** is the runtime scaffolding that wires agents together while the program is running.

```mermaid
flowchart LR
    subgraph CONVENTIONAL["Conventional Approach"]
        direction TB
        FW["Framework\n(harness)"]
        AG1["Agent A\n(config)"]
        AG2["Agent B\n(config)"]
        AG3["Agent C\n(config)"]
        FW -->|"loads at runtime"| AG1 & AG2 & AG3
        DEP["Framework must be\ninstalled at runtime"]
    end

    style CONVENTIONAL fill:#fef2f2,stroke:#ef4444
    style DEP fill:#fee2e2,stroke:#ef4444
```

Agents are instances of framework classes, behavior is controlled by config passed at runtime, and the framework itself is always a runtime dependency.

---

### Hierarch's Approach — Harness as Code Generator

Hierarch inverts this model. The harness is used **only at build time** as a code generator. At runtime, the generated script is entirely self-contained — the harness package is never imported.

```mermaid
flowchart TD
    subgraph BUILD["Build Time (Server)"]
        direction LR
        HB["harness/base.py\nharness/react.py"]
        HC["harness/codegen.py"]
        BP["Physical Blueprint"]

        BP --> HC
        HB -->|"source embed"| ASM
        HC -->|"per-agent class\ncode generation"| ASM
        ASM["Assembler\n(synthesizer.py)"] --> SCRIPT
    end

    subgraph SCRIPT["Generated Script (self-contained)"]
        direction TB
        I["imports"]
        CL["HierarchLogger\nTokenCircuitBreaker\nLLMClient\n— source embedded —"]
        RA["ReActLoop\n— source embedded —"]
        BA["BaseAgent\n— source embedded —"]
        A1["Agent_step1_executor\n(LLMAgent)"]
        A2["Agent_step2_sender\n(MCPAgent)"]
        A3["Agent_step3_parser\n(PythonAgent)"]
        S1["run_step_step1()"]
        S2["run_step_step2()"]
        S3["run_step_step3()"]
        M["main()\nnext_step chain runner"]
    end

    subgraph RUNTIME["Run Time (anywhere)"]
        SCRIPT -->|"python generated_run.py"| EX["Execution\n— no framework needed —"]
    end

    style BUILD fill:#fdf4ff,stroke:#a855f7
    style SCRIPT fill:#eff6ff,stroke:#3b82f6
    style RUNTIME fill:#f0fdf4,stroke:#22c55e
```

---

### Comparison

| Aspect | Conventional Harness | Hierarch Harness |
|--------|---------------------|-----------------|
| **Harness role** | Runtime orchestrator | Build-time code generator |
| **Agent definition** | Class instance + config dict | Generated class with logic baked in |
| **Framework at runtime** | Required | Not present |
| **Portability** | Needs framework installed | Any Python environment |
| **Agent code** | Shared template, differentiated by config | Each agent gets its own generated class |
| **Introspection** | Opaque — behavior hidden in framework | Transparent — all logic readable in one file |
| **Execution entry point** | Framework `.run()` method | Plain `python script.py` |
| **Modification** | Change config → re-run | Regenerate or edit script directly |

---

### Per-Agent Independent Code Generation

The key innovation is that **each agent's code is generated independently** from its compiled spec, then assembled into a single file.

```mermaid
flowchart LR
    subgraph BLUEPRINT["Physical Blueprint"]
        SA1["SubAgent\nid: news-fetcher\ntype: mcp\nmcp_config: {...}"]
        SA2["SubAgent\nid: summarizer\ntype: llm\ndescription: ..."]
        SA3["SubAgent\nid: slack-sender\ntype: python\ncode_snippet: ..."]
    end

    subgraph CODEGEN["codegen.py — Independent Generation"]
        G1["generate_mcp_agent()\n→ MCPAgent class code"]
        G2["generate_llm_agent()\n→ LLMAgent class code"]
        G3["generate_python_agent()\n→ PythonAgent class code"]
    end

    subgraph OUTPUT["Assembled Script"]
        C1["class Agent_step1_news_fetcher(BaseAgent):\n    async def run(...):\n        # ReAct loop over MCP server\n        ..."]
        C2["class Agent_step1_summarizer(BaseAgent):\n    async def run(...):\n        # LLM inference\n        ..."]
        C3["class Agent_step2_slack_sender(BaseAgent):\n    async def run(...):\n        # exec(code_snippet)\n        ..."]
    end

    SA1 --> G1 --> C1
    SA2 --> G2 --> C2
    SA3 --> G3 --> C3

    style BLUEPRINT fill:#fef9c3,stroke:#ca8a04
    style CODEGEN fill:#fdf4ff,stroke:#a855f7
    style OUTPUT fill:#eff6ff,stroke:#3b82f6
```

Because each agent class is generated independently, each one can be:
- **Reviewed** in isolation before the full script is deployed
- **Tested** independently by instantiating the class directly
- **Regenerated** individually if the compiled spec changes, without touching other agents

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

## Agent Execution — ReAct Loop

When an agent is bound to an MCP tool server, it does not call the tool blindly.  
Instead, it runs a **Planning → Tool Call → Observation → Re-plan** cycle until it has enough information to produce a final answer.

```mermaid
sequenceDiagram
    participant LLM as LLM
    participant RT as Agent Runtime
    participant MCP as MCP Tool Server

    RT->>MCP: Connect & list available tools
    MCP-->>RT: Tool catalog (name + schema)

    RT->>LLM: Prompt + tool schemas
    LLM-->>RT: Plan → Tool call decision

    loop Until final answer (max 10 iterations)
        RT->>MCP: Call tool(name, args)
        MCP-->>RT: Observation (result)
        RT->>LLM: Re-plan with observation
        LLM-->>RT: Next action or final answer
    end

    RT-->>RT: Return final text answer
```

| Property | Detail |
|----------|--------|
| Tool discovery | Available tools are fetched live from the MCP server at runtime |
| Multi-turn | Full conversation history is preserved across iterations |
| Termination | Loop ends when the LLM produces a plain-text response with no tool call, or after max iterations |
| Fallback | If the MCP server is unreachable, the agent falls back to pure LLM inference |
| Parallelism | Multiple agents within the same workflow step run their ReAct loops concurrently |

---

## Phase 3 — Dual Execution Mode

Hierarch supports **two execution modes** from the same synthesized script. The script itself is environment-agnostic — it runs identically regardless of where it executes.

```mermaid
flowchart TD
    SCRIPT["📄 Synthesized Script\n(environment-agnostic)"]

    SCRIPT --> LOCAL
    SCRIPT --> SERVER

    subgraph LOCAL["🖥️ Local Execution"]
        L1["Manual trigger\n(user clicks 'Run')"]
        L2["User's own API keys"]
        L3["MCP tools supported"]
    end

    subgraph SERVER["☁️ Cloud Execution"]
        S1["Scheduled trigger\n(daily / weekly cron)"]
        S2["Platform-managed keys"]
        S3["LLM-only workflows"]
    end

    LOCAL --> R1["📊 Result\n(displayed in app)"]
    SERVER --> R2["📊 Result\n(delivered via notification)"]

    style SCRIPT fill:#f0f4ff,stroke:#6366f1
    style LOCAL fill:#f0fdf4,stroke:#22c55e
    style SERVER fill:#eff6ff,stroke:#6366f1
```

| | Local Execution | Cloud Execution |
|---|---|---|
| **Trigger** | Manual (click to run) | Scheduled (cron — daily, weekly, etc.) |
| **LLM API Key** | User's own key | Platform-managed |
| **MCP Tools** | ✅ Supported | LLM-only workflows |
| **Best for** | Testing, MCP workflows, privacy-sensitive tasks | Recurring automation — news, research, monitoring |

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

## Platform Access

Hierarch is accessible via **Web** and **Desktop App**, sharing the same interface.

```mermaid
flowchart LR
    subgraph UI["🎨 Shared React UI"]
        CHAT["Chat with\nAI Architect"]
        VIZ["Blueprint\nVisualization"]
        LOG["Execution\nLog Viewer"]
    end

    UI --> WEB["🌐 Web App\n(browser)"]
    UI --> DESK["🖥️ Desktop App\n(Tauri)"]

    WEB --> CLOUD_EXEC["☁️ Cloud Execution"]
    DESK --> LOCAL_EXEC["🖥️ Local Execution"]
    DESK --> CLOUD_EXEC

    style UI fill:#f0f4ff,stroke:#6366f1
    style WEB fill:#eff6ff,stroke:#3b82f6
    style DESK fill:#f0fdf4,stroke:#22c55e
```

| | Web App | Desktop App |
|---|---|---|
| **Access** | Browser — no install | Download (Windows / macOS) |
| **Design & Build** | ✅ | ✅ |
| **Cloud Execution** | ✅ | ✅ |
| **Local Execution** | — | ✅ |
| **MCP Tools** | Cloud-supported tools only | All MCP tools |

The Web App provides **zero-friction onboarding** — try Hierarch instantly in the browser.  
The Desktop App adds **local execution** for users who want full control over their API keys and data.

---

## System Components

```mermaid
flowchart TB
    subgraph CLIENT["🖥️ Client (Web or Desktop)"]
        APP["React UI\n(shared codebase)"]
        RT["Local Runtime\n(Desktop only)"]
        APP -->|"launch & stream logs"| RT
    end

    subgraph SERVER["☁️ Cloud Server"]
        API["Backend API\n(Python · FastAPI)"]
        SYNTH["AI Architect\n+ Compiler\n+ Synthesizer"]
        SCHED["Scheduler\n(cron-based)"]
        SRT["Server Runtime\n(sandboxed)"]
        API --> SYNTH
        SCHED --> SRT
    end

    subgraph STORE["🗄️ Cloud Storage"]
        FS["Firestore\n· Sessions · Blueprints\n· MCP Registry · Vectors"]
    end

    subgraph TOOLS["🔌 External Services"]
        LLM["LLM API\n(Gemini / OpenAI)"]
        MCP["MCP Tool Servers\n(Smithery)"]
    end

    APP <-->|"Chat / Build\nREST + SSE"| API
    API <-->|"Read / Write"| FS
    RT -->|"LLM calls\n(user's API key)"| LLM
    RT -->|"Tool calls"| MCP
    SRT -->|"LLM calls\n(platform key)"| LLM
    SCHED -->|"Read schedules"| FS

    style CLIENT fill:#f0fdf4,stroke:#22c55e
    style SERVER fill:#f8faff,stroke:#6366f1
    style STORE fill:#fef9c3,stroke:#ca8a04
    style TOOLS fill:#fdf4ff,stroke:#a855f7
```

| Component | Technology | Responsibility |
|-----------|-----------|----------------|
| **Web / Desktop UI** | React (shared) · Tauri (desktop shell) | Chat UI · Blueprint visualization · Log display |
| **Backend Server** | Python · FastAPI · async SSE | Blueprint design · Compilation · Code synthesis · Scheduling |
| **Database** | Google Cloud Firestore | Sessions, blueprints, MCP registry, vector index |
| **Local Runtime** | Subprocess (inside Tauri) | Execute synthesized script · Token circuit breaker |
| **Server Runtime** | Subprocess (on server) | Scheduled execution · Token circuit breaker |


---

## Security Design

```mermaid
flowchart TD
    subgraph Isolation["Process Isolation"]
        P["Runtime"] -->|"spawn"| CH["Child Process\n(synthesized script)"]
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

1. **Process Isolation** — each workflow runs in an isolated subprocess, separate from the host process
2. **Credential Scoping** — API keys are injected via environment variables and never persisted in scripts
3. **Automatic Cleanup** — temporary script files are deleted immediately after execution completes
4. **Token Budget** — per-project spending limits prevent runaway LLM costs
5. **User Isolation** — all cloud data is partitioned by user identity at the storage layer
6. **Local-first Privacy** — desktop users' API keys and data never pass through the server during execution

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Web App | React |
| Desktop App | Tauri · Rust · React |
| API Server | Python · FastAPI · async SSE |
| AI / LLM | Google Gemini (multi-model) · LangChain |
| Embeddings | Gemini Embedding API · 768-dim vectors |
| Vector Search | Google Cloud Firestore native vector index |
| State Store | Google Cloud Firestore |
| Tool Ecosystem | MCP (Model Context Protocol) · Smithery Registry |
| Code Synthesis | Harness Engineering — per-agent independent code generation + assembly |
| Runtime | Isolated subprocess · token-aware circuit breaker |
| Scheduling | Cron-based scheduler (server-side) |

---

## Use Cases

| Use Case | Description | Execution |
|----------|-------------|-----------|
| 📰 **Daily News Briefing** | Summarize top news by category every morning | ☁️ Scheduled |
| 📊 **Market Research** | Monitor competitors and generate weekly trend reports | ☁️ Scheduled |
| 📄 **Paper Curation** | Find and summarize latest academic papers in your field | ☁️ Scheduled |
| 🎬 **Content Pipeline** | Automate YouTube production workflow from ideation to editing | 🖥️ Local (MCP) |
| 🔍 **Ad-hoc Research** | Deep-dive into a topic with multi-agent analysis | 🖥️ Local |

---

## Project Status

| Item | Status |
|------|--------|
| AI Architect (Design) | ✅ Complete |
| Compiler + Synthesizer (Build) | ✅ Complete |
| Desktop App (UI) | 🔧 In progress |
| Execution Engine | 🔧 In progress |
| Web App | 📋 Planned |
| Scheduling | 📋 Planned |
| Development | 🔧 Active development |
| Source code | 🔒 Private repositories (server + client app) |
| Patent | 📋 Under examination |

---

## Contact

Questions, collaboration, or licensing inquiries:

**✉️ hwansys@naver.com**

---

*Built with ❤️ by WhiteBearHands*
