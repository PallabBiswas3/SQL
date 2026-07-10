# Internship Preparation Roadmap
### Master Skill Analysis across 11 Job Descriptions
Prepared for: Pallab Biswas (B.Tech Hons. EE + M.Tech SPML, IIT Kharagpur)

**Source JDs analyzed (11 total):**
1. PlusWealth Capital — Quant Developer + Researcher Intern
2. PlusWealth Capital — Intern (Core Engineering)
3. Google — Software Engineering Intern, Summer 2027
4. IMC Trading — Quant Research Intern
5. IMC Trading — Software Engineering Intern
6. Microsoft — Software Engineering Intern
7. Millennium — Treasury Analytics Intern
8. Nomura — Wholesale Strategy Division Intern
9. Nomura — Global Markets Division (Structuring / QIS / Securitized Products / Equity Derivatives)
10. Stripe — Software Engineer Intern
11. Trexquant — Quantitative Research Intern

**Important framing note before the data:** this set spans two genuinely different tracks — *software engineering* (Google, Microsoft, Stripe, IMC-SWE) and *quant research/trading/finance* (PlusWealth, IMC-QR, Trexquant, Millennium, Nomura). A few JDs (PlusWealth Core) sit in between. Because of that split, almost nothing hits a >60% "everyone wants this" threshold — that's a real finding, not a gap in the analysis, and it directly shapes how you should prioritize (see Part 9).

---

## Part 1: Skill Extraction Table

| Skill | # JDs | % | Companies | Exact / near-exact wording used |
|---|---|---|---|---|
| Python | 6 | 55% | PlusWealth-QD, PlusWealth-Core, IMC-QR, Millennium, Trexquant, Google | "working knowledge of Python", "Python preferred", "programming in Python" |
| Financial Markets Knowledge | 6 | 55% | Millennium, Nomura-Strat, Nomura-GM, PlusWealth-QD, IMC-QR, Trexquant | "understanding of financial markets", "market structure" |
| Communication & Presentation | 6 | 55% | PlusWealth-QD, PlusWealth-Core, IMC-QR, Millennium, Nomura-Strat, Stripe | "strong communication and interpersonal skills", "present findings" |
| Quant Finance Concepts (options, pricing, strategy formalization) | 5 | 45% | PlusWealth-QD, IMC-QR, Nomura-GM, Millennium, Trexquant | "options theory", "pricing structured payoffs", "optimization problems" |
| Problem Solving / Analytical Thinking | 5 | 45% | PlusWealth-QD, PlusWealth-Core, Google, IMC-QR, Trexquant | "exceptional analytical and problem-solving skills" |
| C++ | 4 | 36% | PlusWealth-QD, PlusWealth-Core, IMC-QR (alt.), Google (example) | "Proficient in C++", "strong foundational knowledge of C++" |
| Statistics & Probability / Statistical Modeling | 3 | 27% | PlusWealth-QD, IMC-QR, Trexquant | "sophisticated grasp of statistical theory", "statistical modelling" |
| Machine Learning | 3 | 27% | Google (pref.), IMC-QR, Trexquant | "machine learning techniques", "cutting-edge ML" |
| AI-integrated / GenAI tooling | 3 | 27% | Google, Microsoft, Millennium | "AI-integrated software", "leverage AI-powered tools" |
| Data Structures & Algorithms | 3 | 27% | PlusWealth-Core, Google, IMC-SWE | "data structures, algorithms, and computational complexity" |
| Linux | 2 | 18% | PlusWealth-QD, PlusWealth-Core | "working knowledge of Linux" |
| Shell Scripting / Bash | 2 | 18% | PlusWealth-QD, PlusWealth-Core | "shell scripting" |
| System Design | 2 | 18% | Stripe, Google | "systems design and testing", "designing...a complex system" |
| Testing / Code Review / PR Quality | 2 | 18% | Stripe, IMC-SWE | "good test coverage", "code reviews" |
| Risk Management Systems | 2 | 18% | PlusWealth-Core, Nomura-GM | "risk-management systems", "risk monitoring" |
| Java / Go / Scala / Ruby / JS (general-purpose langs) | 2 | 18% | Stripe, Google | "Java, Ruby, JavaScript, Scala, and Go" |
| SQL | 1 | 9% | Millennium | "Proficient in SQL" |
| Computer Networks (TCP/IP, UDP) | 1 | 9% | PlusWealth-Core | "basic networking concepts like TCP/IP and UDP" |
| Distributed Systems / Concurrency | 1 | 9% | Google | "concurrency, multi-threading, or synchronization" |
| Low-Latency / HPC Optimization | 1 | 9% | PlusWealth-Core | "nanosecond-level performance improvements" |
| Data Visualization (Tableau/dashboards) | 1 | 9% | Millennium | "Tableau dashboards, Web UI" |
| Research Skills (reading/implementing papers) | 1 | 9% | Trexquant | "implement cutting-edge academic research" |
| Business/Macro Analysis (regulation, M&A) | 1 | 9% | Nomura-Strat | "impact of changing regulations", "acquisition screening" |
| Compilers (niche CS) | 1 | 9% | Google | "compilers" (preferred qualification) |

**Notably absent across all 11 JDs:** Git/GitHub, Docker, Kubernetes, Cloud (AWS/GCP/Azure), CI/CD, REST APIs, React/frontend frameworks. None of these companies wrote them explicitly — see Part 6, because that doesn't mean you can skip them.

---

## Part 2: Frequency Ranking

**Tier A (>60%): None.**
This is the honest result of mixing SWE and quant-finance JDs — the highest scorers (Python, Financial Markets Knowledge, Communication) all sit at 55%. If you were only looking at the 4 pure SWE JDs, DSA/C++/Python would be Tier A; if only the 5 quant JDs, Financial Markets + Communication would be. Treat Part 9 (80/20) as the more useful cut for a mixed application strategy.

**Tier B (30–60%):**
Python · Financial Markets Knowledge · Communication & Presentation · Quant Finance Concepts · Problem Solving · C++

**Tier C (10–30%):**
Statistics & Probability · Machine Learning · AI-integrated tooling · Data Structures & Algorithms · Linux · Shell Scripting · System Design · Testing/Code Review · Risk Management Systems · Java/Go/Scala/JS

**Tier D (<10%):**
SQL · Computer Networks · Distributed Systems · Low-Latency/HPC · Data Visualization · Research Skills · Business/Macro Analysis · Compilers

---

## Part 3: Skill Dependency Graph

Two parallel tracks converge at the top, because your target companies split cleanly into two hiring pipelines.

```
                    Programming Fundamentals (pick Python + C++)
                                    ↓
                    ┌───────────────┴───────────────┐
                    ↓                                 ↓
          SOFTWARE ENGINEERING TRACK          QUANT / FINANCE TRACK
          (Google, Microsoft, Stripe,          (PlusWealth, IMC-QR,
           IMC-SWE)                             Trexquant, Millennium, Nomura)
                    ↓                                 ↓
          Data Structures & Algorithms         Probability & Statistics
                    ↓                                 ↓
          Object-Oriented Programming          Linear Algebra (for ML/pricing)
                    ↓                                 ↓
          Operating Systems / Linux            Financial Markets Fundamentals
                    ↓                                 ↓
          Computer Networks                    Options Theory / Derivatives Pricing
                    ↓                                 ↓
          Concurrency & Distributed Systems    Statistical Modeling / ML for Finance
                    ↓                                 ↓
          System Design                        Backtesting & Strategy Formalization
                    ↓                                 ↓
                    └───────────────┬───────────────┘
                                    ↓
                    Communication & Presentation
                                    ↓
                    Domain-specific polish:
                    SQL (Millennium) · Low-latency C++ (PlusWealth-Core)
                    · Tableau (Millennium) · Compilers/Distributed (Google)
```

**Why this order:**
- **Programming fundamentals first** — every single JD assumes fluency in at least one language before anything else is testable.
- **DSA before OOP before OS/Networks (SWE track)** — interview loops at Google/Stripe/IMC-SWE test DSA in round 1; OS/networking questions only appear once you clear that bar.
- **Probability/Stats before Financial Markets before Options Theory (quant track)** — you cannot reason about option pricing or risk models without the underlying stats; markets fundamentals give you the vocabulary options theory assumes you already have.
- **Both tracks converge on Communication** — every JD in this set, without exception, requires presenting findings or collaborating cross-functionally. It's the one truly universal skill.
- **Domain-specific polish is last and role-specific** — SQL only matters if you're pursuing Millennium; low-latency C++ only matters for PlusWealth-Core. Don't front-load these.

---

## Part 4: Learning Roadmap

### Detailed roadmaps (Tier B/C skills that appear across multiple companies)

#### Data Structures & Algorithms
- **What/Why:** The default screening filter for every SWE JD (Google, Stripe, IMC-SWE) and implicitly assumed for quant roles too — PlusWealth-Core explicitly names "computational complexity."
- **Topics:** Arrays, strings, hashing, two-pointer/sliding window → trees, graphs, heaps, tries → DP, greedy, backtracking → complexity analysis (Big-O in practice, not just theory).
- **Progression:** Beginner — arrays/strings/hashmaps, ~150 easy problems. Intermediate — trees/graphs/DP, ~150 medium problems, timed. Advanced — hard DP, graph algorithms (Dijkstra, union-find), mock interviews under 35-min constraints.
- **Common interview Qs:** reverse a linked list, LRU cache, top-K elements, longest substring problems, graph traversal + shortest path, DP on subsequences.
- **Mini-project:** Implement a rate limiter or LRU cache from scratch with a test suite — signals you understand DS beyond leetcode pattern-matching.
- **Common mistakes:** memorizing solutions instead of patterns; skipping complexity analysis out loud in interviews; not practicing under time pressure.
- **Est. time:** 8–10 weeks at 2hrs/day for someone at your stated "intermediate DSA" level to reach interview-ready.

#### C++ (Proficiency + Low-Level)
- **What/Why:** Explicit hard requirement at both PlusWealth JDs; your strongest differentiator given the ESP32 project already on your resume.
- **Topics:** Core syntax, STL (vectors, maps, sets) → OOP (RAII, virtual functions, operator overloading) → memory management (stack vs heap, smart pointers) → templates, move semantics → for PlusWealth-Core specifically: cache-friendly data layout, lock-free structures, latency profiling.
- **Progression:** Beginner — STL fluency, pointers/references. Intermediate — RAII, smart pointers, custom allocators. Advanced — lock-free queues, memory alignment, profiling with perf/valgrind.
- **Interview Qs:** virtual destructor pitfalls, copy vs move constructor, false sharing, when to use unique_ptr vs shared_ptr, implement a thread-safe queue.
- **Mini-project:** Extend your ESP32 forward-pass work into a benchmark harness comparing naive vs cache-optimized matrix multiply — directly reusable in PlusWealth-Core interviews.
- **Common mistakes:** treating C++ like Python with different syntax (ignoring ownership semantics); not profiling before optimizing.
- **Est. time:** 6–8 weeks to go from "proficient" to "interview-sharp" given your existing embedded C++ base.

#### Python
- **What/Why:** Table-stakes across nearly every quant JD and a stated advantage for PlusWealth; you're already strong here from your ML projects.
- **Topics:** Language fluency (already solid) → NumPy/Pandas vectorization → writing production-quality Python (typing, testing, packaging) → for quant roles: vectorized backtesting, avoiding lookahead bias.
- **Interview Qs:** vectorize a rolling-window calculation without loops, explain GIL implications, pandas groupby-apply performance traps.
- **Mini-project:** Convert one loop-heavy section of your Fraud Detection or Delhivery pipeline into fully vectorized NumPy and benchmark the speedup — concrete, quantifiable story for interviews.
- **Common mistakes:** shipping notebook-style code with no error handling in take-home tests; not knowing Big-O of pandas operations.
- **Est. time:** 2–3 weeks of targeted practice (you're past the "learning Python" stage).

#### Statistics & Probability / Statistical Modeling
- **What/Why:** Backbone of PlusWealth-QD, IMC-QR, and Trexquant; you already have real depth here from the FTIR project (Complex OLS, SHAP) and your probability problem-solving background.
- **Topics:** Hypothesis testing, MLE/MAP → regression theory (OLS assumptions, regularization) → time series (stationarity, autocorrelation, cointegration — you've touched this in your pairs-trading work) → Bayesian methods.
- **Interview Qs:** derive OLS estimator, explain p-hacking/multiple testing, what breaks IID assumptions in financial time series, bias-variance tradeoff.
- **Mini-project:** You already have this — your Statistical Arbitrage Pairs Trading Backtester (Engle-Granger, OU half-life) is exactly this skill demonstrated. Make sure it's interview-ready to discuss in depth.
- **Common mistakes:** quoting a formula without being able to derive it when pushed; not connecting theory to why a specific model was chosen over alternatives.
- **Est. time:** You're closer to "interview-ready" here than any other skill — 2–3 weeks of derivation drilling, not new learning.

#### Quant Finance Concepts (Options Theory, Pricing, Strategy Formalization)
- **What/Why:** Central to PlusWealth-QD, IMC-QR, Nomura-GM, Trexquant. This is the one area with almost zero overlap with your current resume — your quant portfolio work (asset allocation, options pricing challenge) is a start but needs deepening.
- **Topics:** Black-Scholes derivation and Greeks → put-call parity → market microstructure (order books, bid-ask spread) → factor models (Fama-French) → portfolio construction and risk (Sharpe, max drawdown, VaR).
- **Interview Qs:** derive Black-Scholes assumptions, explain delta-hedging, what happens to option Greeks near expiry, why does implied vol smile exist.
- **Mini-project:** Build a Black-Scholes pricer + Greeks calculator with a Monte Carlo cross-check — natural extension of your existing options-pricing MLE/KS-test work.
- **Common mistakes:** memorizing the Black-Scholes formula without understanding the no-arbitrage argument behind it; ignoring market microstructure entirely (common gap for ML-background candidates).
- **Est. time:** 6–8 weeks — this is your biggest genuine gap for the quant-track companies.

#### SQL
- **What/Why:** Only explicit for Millennium, but SQL fluency is assumed everywhere data lives in tables — worth building regardless of which company you land.
- **Topics:** Joins, aggregations, window functions → CTEs, subqueries → query optimization basics (indexes, EXPLAIN).
- **Interview Qs:** rank-within-group with window functions, running totals, self-joins for hierarchical data.
- **Mini-project:** Rebuild one of your pandas aggregation pipelines (e.g., fraud detection feature engineering) as equivalent SQL — shows you can move between tools.
- **Common mistakes:** defaulting to pandas-style thinking instead of set-based SQL logic; not knowing window functions.
- **Est. time:** 2 weeks — MySQL is already on your resume, this is sharpening, not learning from scratch.

#### System Design
- **What/Why:** Explicit at Stripe and Google, implicitly tested at Microsoft/IMC-SWE for penultimate-year candidates.
- **Topics:** Scaling basics (load balancing, caching, sharding) → API design → database choice tradeoffs (SQL vs NoSQL) → for intern level: mostly "design a URL shortener / rate limiter"-tier problems, not FAANG-senior-level distributed systems.
- **Interview Qs:** design a URL shortener, design a rate limiter, design a notification service.
- **Mini-project:** Write a one-page design doc for scaling your AeroShield IQ system if it needed to serve 1M requests/day — reuses a project you already know cold.
- **Common mistakes:** jumping to technology names (Kafka, Redis) without justifying tradeoffs; ignoring back-of-envelope capacity estimation.
- **Est. time:** 3–4 weeks at intern-appropriate depth (not senior-level distributed systems depth).

### Condensed entries (Tier C/D — role-specific, lower breadth priority)

| Skill | Core topics | Key resource | Est. time |
|---|---|---|---|
| Machine Learning | Bias-variance, regularization, tree ensembles, basic NN | You're already strong — polish interview articulation | 1–2 weeks |
| Linux / Shell | Process management, permissions, piping, grep/awk/sed | *The Linux Command Line* (free book) | 1 week |
| Networking (TCP/IP, UDP) | Handshakes, latency vs throughput, socket basics | Kurose & Ross Ch. 1–3 | 1–2 weeks |
| Distributed Systems/Concurrency | Threads vs processes, race conditions, mutexes, CAP theorem | MIT 6.824 lecture notes (free) | 2–3 weeks |
| Low-Latency/HPC | Cache lines, branch prediction, lock-free basics | *Agner Fog's optimization manuals* (free PDF) | 2 weeks, pairs with C++ track |
| Testing/Code Review | Unit vs integration tests, PR etiquette, test coverage | Google's Testing Blog | 1 week |
| Risk Management Systems | VaR, stress testing, exposure limits | Millennium/Nomura JD context only | 1 week overview |
| Data Visualization (Tableau) | Dashboards, calculated fields | Tableau free public training | 1 week |
| Research Skills | Reading arXiv papers, reproducing results | Practice on 2–3 papers relevant to your TRM project | Ongoing habit, not a sprint |
| Business/Macro Analysis | Reading 10-Ks, regulatory impact framing | *Financial Times* + company 10-Ks | Ongoing habit |
| Compilers | Lexing/parsing basics, only if pursuing Google's preferred track | CS143 (Stanford, free) | Skip unless targeting Google specifically |

---

## Part 5: Best Learning Resources

**DSA**
- Free: NeetCode 150 (YouTube + tracker), CP-Algorithms.com
- Practice: LeetCode, Codeforces (Div 2 for contest reps)
- Book: *Cracking the Coding Interview* (still solid for patterns)

**C++**
- Free: learncpp.com (best free reference, bar none), CppCon talks on YouTube for advanced topics
- Book: *Effective Modern C++* by Scott Meyers
- Practice: implement STL containers from scratch

**Python**
- Free: Real Python (free tier), official docs
- Practice: Advent of Code for language fluency reps

**Statistics/Probability**
- Free: Harvard Stat 110 (Joe Blitzstein, full course free on YouTube)
- Book: *All of Statistics* by Wasserman (rigorous, matches your existing level)

**Quant Finance / Options Theory**
- Free: QuantStart.com articles, Khan Academy options basics
- Book: *Options, Futures, and Other Derivatives* by Hull (the standard text)
- Practice: implement Black-Scholes + Monte Carlo pricer yourself, don't just read

**SQL**
- Free: SQLBolt, Mode Analytics SQL tutorial
- Practice: LeetCode SQL, StrataScratch

**System Design**
- Free: "System Design Primer" GitHub repo, ByteByteGo YouTube channel
- Note: at intern level, interviewers calibrate expectations down — don't over-invest relative to DSA

**Linux/Networking**
- Free: OverTheWire Bandit (hands-on Linux), Neso Academy Networking playlist

**Communication/Presentation**
- No shortcut resource — the highest-leverage move is mock-presenting your existing projects (FTIR, TRM, ETA optimization) to a peer or mentor and taking real-time feedback.

---

## Part 6: Hidden Requirements

These didn't appear explicitly in any JD text, but you'd be under-prepared without them:

- **Git/GitHub** → Not one JD names it, but "collaborate through code reviews" (Stripe, IMC-SWE) is functionally impossible without it. Assumed baseline everywhere.
- **Linear Algebra & Calculus** → Underpins both "machine learning techniques" (IMC-QR) and derivatives pricing (Nomura-GM, PlusWealth-QD). You have this from coursework — make sure it's fluent, not just passed.
- **NumPy/Pandas** → Every "programming in Python" mention (Millennium, IMC-QR, Trexquant) assumes vectorized numerical work, not vanilla Python loops.
- **Excel/spreadsheet modeling** → Never named, but Nomura-Strat's "consolidate/analyze quarterly financial results" and Millennium's Treasury work are Excel-heavy in practice at every real desk, regardless of what the JD emphasizes.
- **Basic HTTP/REST concepts** → Stripe's preferred qualifications mention "how a service handles an HTTP request" almost in passing — treat it as a real requirement, not a footnote.
- **OOP fundamentals** → Assumed prerequisite for "software design" (Google) and C++ (both PlusWealth JDs) — never spelled out as a standalone item because it's considered too basic to list.
- **Reading academic/financial documents fast** → Trexquant explicitly wants paper-reading; Nomura-Strat implicitly wants 10-K/regulatory-filing reading speed. Different documents, same underlying skill: extracting signal from dense text quickly.

---

## Part 7: Company-wise Analysis

| Company (Role) | Main Skills | Difficulty | Core Focus | Nice-to-Have |
|---|---|---|---|---|
| PlusWealth (Quant Dev+Researcher) | C++, Python, stats, optimization | High | Research → production trading models | Linux, shell scripting |
| PlusWealth (Core) | C++, DS&A, networking, low-latency | Very High | Nanosecond-level systems engineering | Risk system design |
| Google (SWE Intern) | DS&A, distributed systems, general-purpose lang | Very High | Scalable production software | AI/ML exposure, compilers |
| IMC (Quant Research) | Python, stats, market structure, ML | High | Alpha/signal research | C++, presentation skills |
| IMC (SWE Intern) | DS&A, algorithm complexity, collaboration | High | Fluent, mentored software craft | Business domain knowledge |
| Microsoft (SWE Intern) | General SWE fundamentals, teamwork | High | Cross-team production software | AI-integrated tooling |
| Millennium (Treasury Analytics) | SQL, Python, financial modeling | Medium-High | Liquidity/margin quant models | Tableau, AI tools |
| Nomura (Wholesale Strategy) | Financial analysis, presentation, business acumen | Medium | Market/competitive strategy work | Regulatory/M&A exposure |
| Nomura (Global Markets) | Derivatives pricing, risk monitoring, market analytics | High | Structured products / QIS / risk | Presentation, macro fluency |
| Stripe (SWE Intern) | General-purpose lang, systems design, testing | Very High | Production-shipping real features | Frontend/HTTP fundamentals |
| Trexquant (Quant Research) | ML, feature engineering, paper reading | High | Alpha discovery from novel data | Portfolio construction |

---

## Part 8: Top 20 Skills — Priority Ranked

Ranked by a composite of (a) breadth across JDs, (b) how hard it is to fake in an interview, and (c) how much it gates access to a whole category of roles.

1. Data Structures & Algorithms
2. Python (production-quality, not just scripting)
3. Statistics & Probability (deep, derivation-level)
4. C++ (with real memory/ownership fluency)
5. Financial Markets Fundamentals
6. Communication / Presenting Technical Work
7. Options Theory & Derivatives Pricing
8. Machine Learning (applied, not just theory)
9. SQL
10. Object-Oriented Programming
11. System Design (intern-level depth)
12. Operating Systems / Linux
13. Git & Collaborative Workflow
14. Computer Networks
15. Low-Latency/Performance Engineering
16. Distributed Systems & Concurrency
17. Testing & Code Review Practice
18. Data Visualization
19. Risk Management Concepts
20. Business/Macro Analysis Literacy

---

## Part 9: 80/20 Analysis

The 20% of effort that unlocks ~80% of these internships:

**→ DSA + Python + Statistics/Probability + Communication**

Why these four specifically:
- **DSA** is the hard gate for all 4 SWE-adjacent JDs (Google, Microsoft, Stripe, IMC-SWE) and implicitly tested at PlusWealth-Core. Fail this, and nothing else in your resume matters for those rounds.
- **Python** is the shared language across every quant JD (6/11) — it's the one skill where PlusWealth, IMC-QR, Millennium, and Trexquant all converge.
- **Statistics/Probability** is what separates "can code ML" from "understands why the model works" — the exact distinction quant interviewers probe for, and an area where your FTIR/pairs-trading work already gives you real ammunition.
- **Communication** appears in 6/11 JDs explicitly and is silently required in all 11 — every interview, technical or not, is ultimately you explaining your reasoning out loud.

Everything else (C++ depth, options theory, SQL, system design) is high-value but role-*specific* — worth building once you've picked which 3–4 companies you're actually prioritizing, not before.

---

## Part 10: Week-by-Week Plan

I don't have your current daily hour budget, so this is laid out as a **topic sequence by week**, not hours — you can compress it into 4 weeks at higher intensity or stretch to 12 at a lighter pace. Tell me your hours/day and target interview date and I'll convert this into an exact daily schedule (similar to the 11-hr/day CDC plan you'd built before).

| Week | Focus | Deliverable to prove it |
|---|---|---|
| 1–2 | DSA fundamentals (arrays → trees → graphs) | 100 LeetCode easy/medium solved, timed |
| 3 | Statistics/Probability derivation drilling | Can derive OLS, MLE, hypothesis tests from scratch on a whiteboard |
| 4 | C++ ownership/memory + STL fluency | Rewrite one existing Python project's core logic in C++ |
| 5 | Options theory + Black-Scholes | Build a working BS pricer + Greeks calculator |
| 6 | SQL + one system design pattern/week | 20 SQL problems + 2 design docs (URL shortener, rate limiter) |
| 7 | DSA hard problems + mock interviews | 2 timed mock interviews (DSA), 1 mock quant-brainteaser session |
| 8 | Project polish + behavioral/communication prep | 90-second pitch nailed for each of your 5 projects, company-specific tailoring done |

---

## Part 11: Final Summary

**1. Master Skill Checklist**
☐ DSA (arrays/trees/graphs/DP) ☐ Python (vectorized, production-style) ☐ Statistics/Probability (derivation-level) ☐ C++ (ownership/STL/memory) ☐ Financial Markets fundamentals ☐ Options theory/Black-Scholes ☐ SQL ☐ OOP ☐ System design (intern-level) ☐ Linux/shell ☐ Git workflow ☐ Communication/presentation drilling

**2. Skill Priority Ranking** — see Part 8.

**3. Learning Sequence** — Programming → DSA/Stats split by track → domain-specific layer (options theory or system design) → communication polish. See Part 3 diagram.

**4. Estimated Total Prep Time** — 8–10 weeks for genuine interview-readiness across both tracks, assuming 3–4 focused hours/day; compressible to ~5–6 weeks at higher daily intensity.

**5. Highest ROI Topics** — DSA, Python, Statistics/Probability, Communication (Part 9).

**6. Skills common across almost every JD** — Python, Financial Markets Knowledge, Communication (all 55%); none reach the 60% "universal" bar, which is itself the key insight for this batch.

**7. Skills specific to only a few companies** — SQL (Millennium only), Networking/Low-latency (PlusWealth-Core only), Research-paper reading (Trexquant only), Compilers (Google only), Business/macro analysis (Nomura-Strat only).

**8. Biggest weaknesses to address first, based on your resume vs. this list:**
- **Options theory/derivatives pricing** — your only prior exposure is the asset allocation/options-pricing challenge; this needs to go from "touched it once" to "can derive Black-Scholes under pressure" for PlusWealth-QD, IMC-QR, Nomura-GM.
- **Low-latency/systems C++** — your ESP32 work proves you can write tight embedded C++, but PlusWealth-Core wants network-level, nanosecond-scale optimization specifically — a different (though related) muscle.
- **SQL** — listed on your resume but not demonstrated in any project; Millennium will likely test it directly.
- **DSA interview reps** — none of your resume projects are DSA-interview-style problems; this is pure grinding you haven't started yet based on what's here.

Everything else — Python, ML, statistics, communication about technical work — you're already carrying real, demonstrable project evidence for. The gap isn't breadth, it's depth in the quant-finance-specific layer and interview-format DSA reps.
