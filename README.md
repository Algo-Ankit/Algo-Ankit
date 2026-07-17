<!-- ─────────────────────────  HEADER  ───────────────────────── -->

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:1F8ACB,50:5E5CE6,100:0F2027&height=180&section=header&text=Ankit%20Anand%20Singh&fontSize=52&fontColor=ffffff&fontAlignY=32&desc=AI%20Systems%20%C2%B7%20Quant%20Infrastructure%20%C2%B7%20Data%20Science&descSize=17&descAlignY=54&animation=fadeIn" alt="Ankit Anand Singh" />
</p>

<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=600&size=20&pause=1200&color=5E5CE6&center=true&vCenter=true&width=850&height=45&lines=Building+AlphaSwarm%2C+an+AI+quant+team+in+a+browser+tab;LLM+agents+that+write%2C+sandbox+and+trade+real+strategies;Gen+AI+on+Azure+%7C+Microsoft+Agent+Framework+%7C+Python" alt="What I build" />
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/ankit-anand-singh-27076b262/" target="_blank"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" /></a>
  &nbsp;
  <a href="https://twitter.com/AlgoAnkit" target="_blank"><img src="https://img.shields.io/badge/X-000000?style=for-the-badge&logo=x&logoColor=white" alt="X" /></a>
</p>

<p align="center">
  <b>Data Scientist</b> and <b>AI Engineer</b>. NIT Jaipur graduate.<br/>
  <em>I build LLM agent systems, quantitative infrastructure, and the production plumbing that holds them up.</em>
</p>

<br/>

<!-- ─────────────────────  CURRENTLY BUILDING  ───────────────────── -->

<div align="center">

<img src="https://img.shields.io/badge/%20-CURRENTLY%20BUILDING-5E5CE6?style=for-the-badge&labelColor=0F2027" alt="Currently building" />

# AlphaSwarm

### *Your AI quant team, in a browser tab.*

<img src="https://img.shields.io/badge/Status-MVP%20in%20development-success?style=flat-square" alt="Status" />
<img src="https://img.shields.io/badge/Repo-Private%20%C2%B7%20access%20on%20request-8A8A8E?style=flat-square" alt="Private" />
<img src="https://img.shields.io/badge/Demo-recording%20coming%20soon-1F8ACB?style=flat-square" alt="Demo soon" />

</div>

> A **multi-tenant SaaS platform** where you describe a trading strategy in plain English, or set a goal like *"retire by 50"*, and an LLM agent writes the Python, runs it inside a hardened sandbox, backtests it against institutional-grade slippage and market-impact modelling, and trades it through **your own broker**.
>
> Built as a pro trading terminal, a wealth-management copilot, and a GitHub-style commons of forkable strategies, with zero infrastructure to run and no Bloomberg subscription.
>
> **It never touches your money.** Funds stay in your own Zerodha / Upstox / Angel One / Alpaca account. It's software that sends orders on your behalf: not a broker, not a fund. That is the entire trust model.

```mermaid
flowchart LR
    UI["Next.js 14 Terminal<br/>candles · indicators · live P&amp;L"]

    subgraph CONTROL["Control Plane"]
        API["FastAPI"]
        VAULT["Credential Vault<br/>HKDF envelope"]
    end

    subgraph AGENT["Agent Layer"]
        LLM["Strategy Compiler<br/>Microsoft Agent Framework"]
        SBX["RestrictedPython Sandbox<br/>exec timeout · escape guards"]
    end

    subgraph EXEC["Execution Plane"]
        CELERY["Celery Workers"]
        BT["Backtester<br/>slippage + market impact"]
        RISK["Risk Engine<br/>verify_order_intent"]
    end

    subgraph DATA["Data Plane"]
        PG[("PostgreSQL 16<br/>tenant-scoped")]
        RD[("Redis 7")]
    end

    BROKER["Your Own Broker<br/>Zerodha · Upstox<br/>Angel One · Alpaca"]

    UI -->|"plain English"| API
    API --> LLM
    LLM -->|"generated Python"| SBX
    SBX -->|"validated"| BT
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
    classDef broker fill:#37814A,stroke:#12331D,color:#fff

    class UI client
    class API,VAULT control
    class LLM,SBX agent
    class CELERY,RISK,BT exec
    class PG,RD data
    class BROKER broker
```

<br/>

<table>
<tr>
<td width="50%" valign="top">

#### The AI Layer
- **LLM strategy compiler** turning plain English into validated, executable Python
- **Microsoft Agent Framework** driving a ReAct loop with a sandbox-validate tool
- **Bring-your-own-key**, so the LLM provider, key and cost stay yours
- **AI investment advisor** with RAG grounded in your own portfolio
- **Goal Wizard** that turns *"₹50L in 20 years"* into a real SIP portfolio

</td>
<td width="50%" valign="top">

#### The Trust Layer
- **Hardened sandbox** with str-subclass guards, a format denylist and SIGALRM exec timeouts
- **Risk is sacred**: `verify_order_intent()` runs before *every* broker call, with no bypasses
- **Daily notional caps** and market-hours gating on every order
- **HKDF envelope encryption** for broker credentials, ~21,000× faster than the PBKDF2 it replaced
- **Multi-tenant isolation** with `tenant_id` enforced on every query

</td>
</tr>
<tr>
<td width="50%" valign="top">

#### The Quant Layer
- **Prop-grade backtester** modelling slippage *and* market impact rather than fantasy fills
- **Out-of-sample honesty scoring** so strategies cannot lie about their own results
- **Monte Carlo wealth forecaster** (GBM) across a unified multi-broker portfolio
- **Multi-broker CAS ingestion** for one view of everything you own
- **pandas-ta indicators**: RSI, MACD, Bollinger, ATR, EMA, VWAP

</td>
<td width="50%" valign="top">

#### The Product Layer
- **Bloomberg-style terminal** with candles, overlays, news, AI forecasts and live P&L over WebSockets
- **Strategy Commons** offering GitHub-style fork graphs and immutable pinned versions
- **Swarm copy-trading** on a leaderboard of forkable, honestly-scored strategies
- **Creator monetization** via a 1% Swarm Tax, split 50/50
- **SIPs** you can pause, resume, top up and rebalance

</td>
</tr>
</table>

<div align="center">

<sub><b>Under the hood</b></sub><br/>
<img src="https://img.shields.io/badge/Python_3.11+-3776AB?style=flat-square&logo=python&logoColor=white" />
<img src="https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white" />
<img src="https://img.shields.io/badge/Next.js_14-000000?style=flat-square&logo=next.js&logoColor=white" />
<img src="https://img.shields.io/badge/PostgreSQL_16-336791?style=flat-square&logo=postgresql&logoColor=white" />
<img src="https://img.shields.io/badge/Redis_7-DC382D?style=flat-square&logo=redis&logoColor=white" />
<img src="https://img.shields.io/badge/Celery-37814A?style=flat-square&logo=celery&logoColor=white" />
<img src="https://img.shields.io/badge/Microsoft_Agent_Framework-5E5CE6?style=flat-square&logo=microsoft&logoColor=white" />
<img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" />
<img src="https://img.shields.io/badge/Sentry-362D59?style=flat-square&logo=sentry&logoColor=white" />

</div>

<!-- DEMO RECORDING: drop the GIF/MP4 in here when it's ready, e.g.
<div align="center">
  <img src="demo.gif" alt="AlphaSwarm demo" width="80%" />
</div>
-->

<br/>

<!-- ─────────────────────────  SKILLS  ───────────────────────── -->

<div align="center">

## What I Work With

</div>

<table>
<tr>
<td valign="top" width="33%" align="center">

**AI / Gen AI**

<img src="https://img.shields.io/badge/Microsoft_Agent_Framework-5E5CE6?style=flat-square&logo=microsoft&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/Azure_AI-0078D4?style=flat-square&logo=microsoftazure&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/Azure_ML-0078D4?style=flat-square&logo=microsoftazure&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/LLM_Agents-5E5CE6?style=flat-square&logo=openai&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/RAG-FF6F00?style=flat-square&logo=databricks&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/No--Code_Workflows-1F8ACB?style=flat-square&logo=zapier&logoColor=white" />

</td>
<td valign="top" width="33%" align="center">

**Data Science**

<img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/pandas-150458?style=flat-square&logo=pandas&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=numpy&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikit-learn&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/TensorFlow-FF6F00?style=flat-square&logo=tensorflow&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/Jupyter-F37626?style=flat-square&logo=jupyter&logoColor=white" />

</td>
<td valign="top" width="33%" align="center">

**Systems & Infra**

<img src="https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/Celery-37814A?style=flat-square&logo=celery&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" /><br/>
<img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white" />

</td>
</tr>
</table>

<div align="center">
<sub><b>Also fluent in</b></sub><br/>
<img src="https://img.shields.io/badge/C++-00599C?style=flat-square&logo=cplusplus&logoColor=white" />
<img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white" />
<img src="https://img.shields.io/badge/Java-007396?style=flat-square&logo=openjdk&logoColor=white" />
<img src="https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black" />
<img src="https://img.shields.io/badge/Django-092E20?style=flat-square&logo=django&logoColor=white" />
<img src="https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white" />
</div>

<br/>

<!-- GitHub stats cards intentionally omitted: the public github-readme-stats
     instance is rate-limited and was serving 503s, which renders as a broken
     image on the profile. Re-add if you self-host it. -->

<!-- ─────────────────────  COMPETITIVE PROGRAMMING  ───────────────────── -->

<div align="center">

## Competitive Programming

<a href="https://github.com/Algo-Ankit/DSA" target="_blank"><img src="https://img.shields.io/badge/Codeforces_Solutions-1F8ACB?style=for-the-badge&logo=codeforces&logoColor=white" alt="Codeforces" /></a>
&nbsp;
<a href="https://github.com/Algo-Ankit/Leetcode_Solutions" target="_blank"><img src="https://img.shields.io/badge/LeetCode_Solutions-FFA116?style=for-the-badge&logo=leetcode&logoColor=white" alt="LeetCode" /></a>

</div>

<br/>

<!-- ─────────────────────  BEYOND THE KEYBOARD  ───────────────────── -->

<div align="center">

## Beyond the Keyboard

</div>

<p align="center">
<b>FPS gaming.</b> Strategy, teamwork and split-second decisions, which map to debugging better than they should.<br/>
<b>Tennis.</b> Played as a hobby.
</p>

<br/>

<p align="center">
  <em>"I love cracking tough problems — whether it's a puzzle, a program, or a process."</em>
</p>

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F2027,50:5E5CE6,100:1F8ACB&height=120&section=footer" alt="" />
</p>
