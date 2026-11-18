<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:1F8ACB,50:5E5CE6,100:0F2027&height=170&section=header&text=Ankit%20Anand%20Singh&fontSize=50&fontColor=ffffff&fontAlignY=34&desc=AI%20Engineer%20%C2%B7%20Data%20Scientist%20%C2%B7%20Gurgaon%2C%20India&descSize=16&descAlignY=56&animation=fadeIn" alt="Ankit Anand Singh: AI Engineer, Data Scientist" />
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/ankit-anand-singh-27076b262/"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" /></a>
  <a href="https://x.com/AlgoAnkit"><img src="https://img.shields.io/badge/@AlgoAnkit-000000?style=for-the-badge&logo=x&logoColor=white" alt="X / Twitter" /></a>
</p>

### Hi, I'm Ankit

I'm an AI engineer and data scientist, an NIT Jaipur graduate living in Gurgaon. I build LLM systems for places where a wrong answer costs something real: money in a brokerage account, a student's visa deadline, a shop's stock.

> **The model proposes. Code decides.**
> I let LLMs extract, draft and explain. The parts that have to be right (an order, a date, a stock change) stay in plain, tested code.

The clearest examples are AlphaSwarm, where a risk gate sits in front of every broker call, and Niyam, where deadlines are computed in tested code and citations are checked before an answer ships. agent-towhatsapp applies the same idea at a smaller scale: price and zero-stock changes wait for the owner's "YES".

**Right now** I'm running Niyam's answer-quality evals and building a TypeScript Telegram agent for small shops (private for now).

<br/>

## Selected work

### [AlphaSwarm](https://github.com/Algo-Ankit/AlphaSwarm): an LLM writes the strategy, a risk gate decides whether it trades

A multi-tenant trading platform. You describe a strategy in English. An agent writes it as Python, a hardened sandbox runs it, and a backtester tests it against realistic fills. Only then can it reach your own broker account. AlphaSwarm never holds funds.

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

- `verify_order_intent()` runs 9 checks before any order, including a 3% gap-safety margin when no live tick exists. The backtester, the live executor and the worker all call the same function, so there is no path that skips it.
- The backtester caps fills at 10% of volume, fills gapped stops at the adverse open, accrues short-borrow cost and uses a true downside-RMS Sortino.
- The sandbox blocks str-subclass and `format` escapes. 28 of the 117 tests target it directly. CI runs lint and pytest on every push.

<sub>FastAPI · Celery · PostgreSQL · Redis · Microsoft Agent Framework · Next.js · Prophet · Docker · GitHub Actions</sub>

<br/>

### [Niyam](https://github.com/Algo-Ankit/Niyam): OPT work-rules assistant that cites the regulation and never computes a date with a model

F-1 students describe their situation in plain words. Niyam returns the rules that apply from 8 CFR 214.2(f), with the paragraph cited, every deadline as a date, and the unemployment days used against the 90-day limit.

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

- On a 35-question test split I wrote myself, vector retrieval reaches hit@1 0.857 against 0.086 for keyword search. Hybrid fusion scored below vector alone, and the write-up reports that.
- Reranking lifts hit@1 from 0.900 to 0.983 on a 60-question dev split. The bootstrap interval is published with it, and the lower bound is zero. There's no external benchmark yet.
- 108 tests, `mypy --strict`, and a pre-commit hook that runs lint, types and tests.

<sub>Python 3.12 · Gemini · Gemma · pgvector · PostgreSQL · SQLite · pytest · mypy</sub>

<br/>

### [agent-towhatsapp](https://github.com/Algo-Ankit/agent-towhatsapp): run a small shop's inventory from WhatsApp

A FastAPI bridge between the WhatsApp Cloud API and two handlers, one for inventory and one for support. A shop owner texts in plain language, and risky changes are held until they reply "YES". It's demo-grade: routing is keyword-based and there are no tests yet.

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

- The webhook signature is verified on the raw body, so a forged request gets a 401 before anything is parsed. Meta's retries are de-duplicated by message ID.
- The webhook acknowledges at once and the work runs in the background, so a slow handler never triggers redelivery.

<sub>Python · FastAPI · WhatsApp Cloud API</sub>

<br/>

### More projects

<details>
<summary><b><a href="https://github.com/Algo-Ankit/TradeProject">TradeProject</a></b>: research stack for systematic trading on NSE India (paper only). Click for architecture.</summary>

<br/>

The feed, features, Deflated Sharpe check and backtester work end to end. The kill switch and pre-trade checks are built and unit-tested but not yet wired into a live order path, so those are shown dashed. It's an early-stage repo: about 7 commits, 45 tests and no CI yet.

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

</details>

<details>
<summary><b><a href="https://github.com/Algo-Ankit/claude-code-local-models-runtime">claude-code-local-models-runtime</a></b>: run Claude Code on local Ollama models with an isolated config and VRAM sizing maths. Click for architecture.</summary>

<br/>

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

## Toolbox

<p>
<img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python" />
<img src="https://img.shields.io/badge/LLM_Agents_%26_RAG-5E5CE6?style=flat-square" alt="LLM agents and RAG" />
<img src="https://img.shields.io/badge/Microsoft_Agent_Framework-5E5CE6?style=flat-square&logo=microsoft&logoColor=white" alt="Microsoft Agent Framework" />
<img src="https://img.shields.io/badge/Gemini-8E75B2?style=flat-square&logo=googlegemini&logoColor=white" alt="Gemini" />
<img src="https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white" alt="FastAPI" />
<img src="https://img.shields.io/badge/PostgreSQL_+_pgvector-336791?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL and pgvector" />
<img src="https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white" alt="Redis" />
<img src="https://img.shields.io/badge/Celery-37814A?style=flat-square&logo=celery&logoColor=white" alt="Celery" />
<img src="https://img.shields.io/badge/pandas_·_Polars-150458?style=flat-square&logo=pandas&logoColor=white" alt="pandas and Polars" />
<img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker" />
<img src="https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=next.js&logoColor=white" alt="Next.js" />
<img src="https://img.shields.io/badge/C++-00599C?style=flat-square&logo=cplusplus&logoColor=white" alt="C++" />
</p>

## Off the clock

I keep my algorithm skills sharp in C++ ([LeetCode](https://github.com/Algo-Ankit/Leetcode_Solutions), [DSA and CSES](https://github.com/Algo-Ankit/DSA)). Away from the keyboard I play tennis and FPS games.

The fastest way to reach me is a LinkedIn DM. I'm happy to talk about AI engineering roles, applied LLM systems and quant tooling.

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F2027,50:5E5CE6,100:1F8ACB&height=100&section=footer" alt="" />
</p>
