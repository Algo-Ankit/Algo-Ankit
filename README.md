<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:1F8ACB,50:5E5CE6,100:0F2027&height=170&section=header&text=Ankit%20Anand%20Singh&fontSize=50&fontColor=ffffff&fontAlignY=34&desc=AI%20Engineer%20%C2%B7%20Data%20Scientist%20%C2%B7%20Gurgaon%2C%20India&descSize=16&descAlignY=56&animation=fadeIn" alt="Ankit Anand Singh: AI Engineer, Data Scientist" />
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/ankit-anand-singh-27076b262/"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" /></a>
  <a href="https://x.com/AlgoAnkit"><img src="https://img.shields.io/badge/@AlgoAnkit-000000?style=for-the-badge&logo=x&logoColor=white" alt="X" /></a>
</p>

<h3 align="center">AI engineer building LLM systems where a wrong answer has a price.</h3>

<p align="center">
  Trading infrastructure · regulatory Q&A · small-business agents
</p>

### Hi, I'm Ankit

I'm an AI engineer and data scientist, an NIT Jaipur graduate living in Gurgaon. I build LLM systems for places where a wrong answer costs something real: money in a brokerage account, a student's visa deadline, a shop's stock.

Right now I'm running the answer-quality evals for Niyam and building a startup around AI agents for small shops (private while it's in development). I'm open to AI engineering roles and contract work.

<br/>

## How I build

> **The model proposes. Code decides.**

LLMs extract, draft and explain. The parts that have to be right (an order, a legal deadline, a stock change) live in plain, tested code, behind a gate the model can't talk its way past. Each project below is a different place to apply that rule, and each lists where it still falls short.

<br/>

## Selected work

| Project | What it does | Evidence |
|---|---|---|
| [**Niyam**](https://github.com/Algo-Ankit/Niyam) | Answers F-1 OPT questions with the regulation paragraph cited and every deadline computed in code | Retrieval hit@1 0.857 vs 0.086 for keyword · 108 tests · `mypy --strict` · [results write-up](https://github.com/Algo-Ankit/Niyam/blob/main/RESULTS.md) |
| [**AlphaSwarm**](https://github.com/Algo-Ankit/AlphaSwarm) | An LLM writes a trading strategy, a sandbox runs it, and a risk gate decides whether it can reach your broker | 9-check risk gate on every order path · 117 tests (28 on the sandbox) · CI · [architecture doc](https://github.com/Algo-Ankit/AlphaSwarm/blob/main/ARCHITECTURE.md) · [quant maths](https://github.com/Algo-Ankit/AlphaSwarm/blob/main/docs/QUANT_MATH.md) |
| [**agent-towhatsapp**](https://github.com/Algo-Ankit/agent-towhatsapp) | Lets a shop owner manage inventory over WhatsApp, with risky changes held for a "YES" | Webhook signature checked before parsing · retry de-duplication · demo-grade |
| [**TradeProject**](https://github.com/Algo-Ankit/TradeProject) | Event-driven research stack for NSE India (paper only) | Deflated Sharpe validation · 45 tests · early stage |
| [**claude-code-local-models-runtime**](https://github.com/Algo-Ankit/claude-code-local-models-runtime) | Runs Claude Code on local Ollama models in an isolated config | VRAM sizing worked through for 6–16 GB cards |

<br/>

### Niyam: OPT work-rules assistant

F-1 students describe their situation in plain words. Niyam returns the rules that apply from 8 CFR 214.2(f), with the paragraph cited, every deadline as a date, and the unemployment days used against the 90-day limit. A model never produces a date: it extracts facts and writes the explanation, and code computes the dates and checks the output.

```mermaid
flowchart LR
    Q["Student question<br/>plain words"]

    subgraph LLM1["Model: extract"]
        EX["Fact extraction<br/>typed StudentFacts"]
    end

    subgraph RULES["Deterministic core"]
        RULE{"Rules engine<br/>deadlines · day counts"}
    end

    subgraph RET["Retrieval"]
        VEC[("pgvector<br/>181 CFR provisions")]
        RR["Reranker<br/>top 10 to top 5"]
    end

    subgraph LLM2["Model: explain"]
        EXP["Explanation<br/>from given provisions only"]
    end

    GATE{"Output check<br/>citations and dates must match"}
    OUT["Cited answer"]

    Q --> EX
    EX --> RULE
    Q --> VEC
    VEC --> RR
    RR --> EXP
    RULE -->|"findings + citations"| EXP
    EXP --> GATE
    GATE -->|"pass"| OUT
    GATE -.->|"reject"| EXP

    classDef client fill:#1F8ACB,stroke:#0B3C5D,color:#fff
    classDef agent fill:#5E5CE6,stroke:#2A2870,color:#fff
    classDef exec fill:#FF6F00,stroke:#7A3500,color:#fff
    classDef data fill:#336791,stroke:#12293D,color:#fff
    classDef ext fill:#37814A,stroke:#12331D,color:#fff
    class Q client
    class EX,EXP agent
    class RULE,GATE exec
    class VEC,RR data
    class OUT ext
```

- **Measured:** on a 35-question test split I wrote myself, vector retrieval gets hit@1 0.857 against 0.086 for keyword search. Hybrid fusion did worse than vector alone, and the write-up says so. Reranking lifts hit@1 from 0.900 to 0.983 on a 60-question dev split, but the bootstrap interval's lower bound is zero, so I call that gain marginal.
- **Known limits:** the output check validates citations and dates, not whether every sentence is supported by the provisions. The eval sets are mine, not an external benchmark, and the whole-pipeline evals are built but not yet run.

<sub>Python 3.12 · Gemini · pgvector · PostgreSQL · pytest · mypy</sub>

<br/>

### AlphaSwarm: an LLM writes the strategy, a risk gate decides whether it trades

A multi-tenant trading platform. You describe a strategy in English, an agent writes it as Python, a hardened sandbox runs it, and a backtester tests it against realistic fills. Only then can it reach your own broker account. AlphaSwarm never holds funds.

```mermaid
flowchart LR
    UI["Next.js 14 Terminal<br/>candles · indicators · live PnL"]

    subgraph CONTROL["Control Plane"]
        API["FastAPI<br/>RS256 JWT · tenant scoped"]
        VAULT["Credential Vault<br/>HKDF + Fernet envelope"]
    end

    subgraph AGENT["Agent Layer"]
        LLM["Strategy Builder<br/>Microsoft Agent Framework · BYOK"]
        SBX["RestrictedPython Sandbox<br/>SIGALRM timeout · escape guards"]
    end

    subgraph EXEC["Execution Plane"]
        CELERY["Celery Workers<br/>pg advisory locks"]
        BT["Backtester<br/>participation cap · sqrt impact"]
        RISK{"Risk Gate<br/>verify_order_intent"}
    end

    subgraph DATA["Data Plane"]
        PG[("PostgreSQL 16")]
        RD[("Redis 7")]
    end

    BROKER["Your Own Broker<br/>Upstox · Alpaca"]

    UI -->|"plain English"| API
    API --> LLM
    LLM -->|"generated Python"| SBX
    SBX -->|"validated"| BT
    BT -.->|"same gate"| RISK
    API --> CELERY
    CELERY --> RISK
    RISK -->|"approved orders only"| BROKER
    VAULT -.->|"decrypt on use"| BROKER
    CELERY --> PG
    CELERY <--> RD

    classDef client fill:#1F8ACB,stroke:#0B3C5D,color:#fff
    classDef control fill:#009688,stroke:#00443F,color:#fff
    classDef agent fill:#5E5CE6,stroke:#2A2870,color:#fff
    classDef exec fill:#FF6F00,stroke:#7A3500,color:#fff
    classDef data fill:#336791,stroke:#12293D,color:#fff
    classDef ext fill:#37814A,stroke:#12331D,color:#fff
    class UI client
    class API,VAULT control
    class LLM,SBX agent
    class CELERY,BT,RISK exec
    class PG,RD data
    class BROKER ext
```

- **One gate, no bypass:** `verify_order_intent()` runs 9 checks, including a 3% gap-safety margin when no live tick exists. The backtester, the live executor and the worker all call the same function.
- **Backtests that don't flatter:** fills are capped at 10% of volume, gapped stops fill at the adverse open, short-borrow cost accrues, and Sortino uses true downside RMS.
- **Known limits:** tenant isolation is enforced by repository convention rather than by the database, and there's no public demo yet.

<sub>FastAPI · Celery · PostgreSQL · Redis · Microsoft Agent Framework · Next.js · Docker · GitHub Actions</sub>

<br/>

### agent-towhatsapp: a small shop's inventory over WhatsApp

A FastAPI bridge between the WhatsApp Cloud API and two handlers, one for inventory and one for support. Risky changes (price edits, zero stock) wait for the owner's "YES".

```mermaid
flowchart LR
    META["WhatsApp Cloud API<br/>POST /webhook"]

    subgraph EDGE["Webhook edge"]
        SIG{"Signature check<br/>X-Hub-Signature-256"}
        DEDUPE["Message-ID de-dupe<br/>ack fast, work later"]
    end

    subgraph CORE["Handlers"]
        ROUTER{"Keyword router"}
        INV["Inventory handler"]
        SUP["Support handler"]
    end

    PEND[("Pending confirmations<br/>price · zero stock")]
    SEND["Reply via<br/>WhatsApp send API"]

    META --> SIG
    SIG --> DEDUPE
    DEDUPE --> ROUTER
    ROUTER -->|"stock / price"| INV
    ROUTER -->|"everything else"| SUP
    ROUTER -->|"YES / NO"| PEND
    INV -->|"risky change"| PEND
    INV --> SEND
    SUP --> SEND
    PEND -->|"confirmed"| SEND

    classDef client fill:#1F8ACB,stroke:#0B3C5D,color:#fff
    classDef control fill:#009688,stroke:#00443F,color:#fff
    classDef agent fill:#5E5CE6,stroke:#2A2870,color:#fff
    classDef data fill:#336791,stroke:#12293D,color:#fff
    classDef ext fill:#37814A,stroke:#12331D,color:#fff
    class META client
    class SIG,DEDUPE control
    class ROUTER,INV,SUP agent
    class PEND data
    class SEND ext
```

- **Careful edge:** the signature is checked on the raw body, so a forged request gets a 401 before anything is parsed. The webhook acknowledges at once and works in the background, so a slow handler never triggers Meta's retries.
- **Known limits:** routing is keyword-based, media messages are acknowledged but not processed, and there are no tests yet.

<sub>Python · FastAPI · WhatsApp Cloud API</sub>

<br/>

<details>
<summary><b>More architecture:</b> TradeProject and claude-code-local-models-runtime</summary>

<br/>

#### [TradeProject](https://github.com/Algo-Ankit/TradeProject): NSE India research stack (paper only)

The feed, features, Deflated Sharpe check and backtester work end to end. The kill switch and pre-trade checks are built and unit-tested but not yet wired into a live order path, so they're shown dashed. About 7 commits and no CI yet.

```mermaid
flowchart LR
    FEED["Zerodha / Shoonya<br/>WebSocket feeds"]
    VAL{"Tick validator<br/>stale · crossed · 5% jump"}
    BUS["Event bus<br/>asyncio"]

    subgraph FEAT["Features"]
        VPIN["VPIN<br/>50 volume buckets"]
        OFI["Order-flow imbalance"]
        GRK["Black-Scholes Greeks<br/>Newton IV"]
    end

    STRAT["Pairs strategy<br/>Kalman hedge ratio"]

    subgraph RESEARCH["Research loop"]
        CH[("ClickHouse ticks")]
        BT["Backtester<br/>sqrt impact · depth-capped fills"]
        DSR["Walk-forward +<br/>Deflated Sharpe"]
    end

    subgraph LIVE["Built, not yet wired"]
        KILL{"Kill switch<br/>5% daily drawdown"}
        EXEC["Order manager"]
    end

    FEED --> VAL
    VAL --> BUS
    BUS --> VPIN
    BUS --> OFI
    BUS --> GRK
    VPIN --> STRAT
    OFI --> STRAT
    GRK --> STRAT
    CH --> BT
    STRAT --> BT
    BT --> DSR
    STRAT -.-> KILL
    KILL -.-> EXEC

    classDef client fill:#1F8ACB,stroke:#0B3C5D,color:#fff
    classDef control fill:#009688,stroke:#00443F,color:#fff
    classDef agent fill:#5E5CE6,stroke:#2A2870,color:#fff
    classDef exec fill:#FF6F00,stroke:#7A3500,color:#fff
    classDef data fill:#336791,stroke:#12293D,color:#fff
    class FEED client
    class VAL,BUS control
    class VPIN,OFI,GRK,STRAT agent
    class KILL,EXEC exec
    class CH,BT,DSR data
```

#### [claude-code-local-models-runtime](https://github.com/Algo-Ankit/claude-code-local-models-runtime): Claude Code on a local GPU

```mermaid
flowchart LR
    USER["claude-local<br/>--fast / --heavy"]
    WRAP{"Wrapper script<br/>scoped env · separate config dir"}
    CC["Claude Code CLI"]
    OLL[("Ollama :11434<br/>32K ctx · q8 KV cache")]
    CLEAN["On exit<br/>unset env · ollama stop"]

    USER --> WRAP
    WRAP --> CC
    CC <-->|"Anthropic-compatible API"| OLL
    WRAP -.->|"finally / trap"| CLEAN

    classDef client fill:#1F8ACB,stroke:#0B3C5D,color:#fff
    classDef control fill:#009688,stroke:#00443F,color:#fff
    classDef agent fill:#5E5CE6,stroke:#2A2870,color:#fff
    classDef data fill:#336791,stroke:#12293D,color:#fff
    classDef exec fill:#FF6F00,stroke:#7A3500,color:#fff
    class USER client
    class WRAP control
    class CC agent
    class OLL data
    class CLEAN exec
```

</details>

<br/>

## Building

I'm building a startup around AI agents for small shops. It's private while it's in development, so there's no repo to link yet. If you run a small shop or sell to them, I'd like to hear what's painful.

<br/>

## Get in touch

If you're hiring for AI engineering, or you have a problem where LLM output has to be right, a [LinkedIn message](https://www.linkedin.com/in/ankit-anand-singh-27076b262/) is the fastest way to reach me. A short description of the problem is enough.

<br/>

<sub>**Stack:** Python, TypeScript, C++ · FastAPI, Celery, Next.js · PostgreSQL, pgvector, Redis · Microsoft Agent Framework, Gemini, Ollama · pytest, mypy, Docker, GitHub Actions. I also practise competitive programming in C++ ([LeetCode](https://github.com/Algo-Ankit/Leetcode_Solutions), [DSA](https://github.com/Algo-Ankit/DSA)) and play tennis.</sub>
