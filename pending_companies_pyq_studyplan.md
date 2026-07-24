# PYQ Compilation & Study Plan — 15 Pending Companies (CDC)

Sources: IIT Kharagpur SWG "FORESIGHT" series (Medium), GeeksforGeeks interview-experience posts, Glassdoor, Wall Street Oasis, TheJobOverflow, LeetCode Discuss, and firsthand blog write-ups. Every company below has been cross-checked against IIT Kharagpur/campus-specific experiences where available; where only generic (non-KGP) data exists, that's flagged.

---

## 0. Companies grouped by prep track (do this first)

| Track | Companies | Core prep overlap |
|---|---|---|
| **A. Quant/Prop/Trading-tech** | AlphaGrip (AlphaGrep) Securities, Fin Mechanics (FinMechanics), Neo Wealth and Asset Management, Wells Fargo (Quant Analyst track) | Probability puzzles, mental math, C++/Python DSA, options/derivatives basics |
| **B. Product/SDE (pure coding)** | Atlassian, Salesforce, UiPath, KLA, GreenSearch Technologies (= Glean), **Databricks**, **NatWest Group** | DSA (medium-hard), OOPs, CS fundamentals (OS/DBMS/CN), LLD for senior-style rounds |
| **C. Fintech/Full-stack + resume-weighted** | Corridor Technologies (= Corridor Platforms), Capital One Services | Full-stack basics + light DSA, resume-story fluency > pure CP |
| **D. Analyst/Consulting/Case-based** | Piramal Finance, Hindustan Unilever Limited (HUL) | Case studies, GD, SQL/Python basics, HR-fit narrative |

Note on the two new additions: **Databricks** is the single highest-bar pure-DSA company on this whole list (medium→hard LeetCode, back-to-back rounds, deep CS-fundamentals expectations) — treat it as your Track B stretch target. **NatWest** is comparatively the most "checklist-able" — a fixed-format MCQ+coding OA followed by one combined tech/HR round — so it rewards broad-but-not-deep CS fundamentals revision more than grind depth.

This maps well onto your existing prep — Track A leans on the same probability/expected-value muscle you're building for Trexquant alphas, and Track B is a direct extension of your LeetCode DP work.

---

## 1. AlphaGrip Securities (AlphaGrep Securities)

**Role at KGP:** Analyst — Trading Systems / SDE (C++, low-latency infra) or Python Developer (HFT firm, Mumbai/Bangalore).

**Selection process (consistent across years):**
1. Online Assessment (HackerRank) — 2–3 coding questions + ~15–20 MCQs, OR 4 Python-only coding questions in 90 min for the Python Developer track.
2. Technical interview(s) — heavy on sockets/buffer management, OOPs, multithreading, OS basics, C++ under-the-hood questions.
3. Puzzle/probability round — e.g., coin-flip probability, DP-style puzzles.
4. HR/culture fit.

**Real PYQs found:**
- Graph connectivity: *N palaces, M tunnels (weighted edges) — determine if all palaces can be connected with minimum total tunnel distance* (DFS/MST-style, print YES/NO).
- Variant: *find size of the largest connected sub-graph of size ≥ K.*
- Buffer management: *read data via sockets, write 50 bytes at a time.*
- String manipulation (Python role): *alter exactly one letter of a string to make it the smallest possible string that is NOT a palindrome.*
- Probability: coin-flipping puzzles; "what happens under the hood in Python" type questions.
- Design/orderbook coding: *create an order book supporting new/cancel/amend orders (amend must preserve priority) and handle fills* (from Glassdoor — likely more senior-level but good practice for SDE-Trading-Systems).

**Prep priority:** Graphs (DFS/BFS/Union-Find), buffer/socket-level system programming concepts, C++ OOPs, basic probability puzzles. If Python role: multithreading + Python internals (GIL, memory model).

**Sources:** [FORESIGHT 2023 – Pavan Sai Chandra](https://medium.com/@swgiitkgp/foresight-2023-placement-at-alphagrep-pavan-sai-chandra-dbdaa6d6bd64), [Mayank Bhushan's interview](https://medium.com/@miku.bhushan/my-interview-experience-with-alphagrep-a13fa4e5e544), [GeeksforGeeks](https://www.geeksforgeeks.org/alphagrep-securities-interview-experience/), [Sahma – Python Dev Intern](https://medium.com/@sahmaanwar61/alphagrep-python-developer-interview-on-campus-5385e535caf8), [Glassdoor](https://www.glassdoor.co.in/Interview/AlphaGrep-Securities-Interview-Questions-E2164049.htm)

---

## 2. Atlassian

**Role:** SDE (Intern/FTE), Bangalore.

**Selection process:**
1. Online Assessment on HackerRank — typically 2 coding questions, medium→hard, ~60 min. (CGPA cutoff seen historically: 7.5)
2. Technical interview(s) — DSA + some system design for senior roles, occasionally proctored via the **Karat** platform (trained external interviewers).
3. Behavioral/values-fit round (Atlassian leans heavily on cultural-fit signaling — know their values: "Open company, no bullshit," "Build with heart and balance," etc.)

**Real PYQ style:** LeetCode medium-to-hard level, standard DSA (arrays, trees, graphs). No AlphaGrep-style puzzle rounds reported — pure coding + behavioral.

**Prep priority:** LeetCode medium/hard grind (you're well-covered with 700+ solved), STAR-format behavioral stories, be ready to discuss *why Atlassian* with specifics about their products (Jira/Trello/Confluence).

**Sources:** [Navpreet Kaur](https://navpreetk12.medium.com/interview-experience-of-atlassian-a1da3b070125), [Olingo – Intern OA](https://medium.com/@olllingooo/atlassian-intern-interview-experience-a84ac617207d), [Cuvette guide](https://cuvette.tech/blog/atlassian-interview-experience), [Gagandeep Singh (Karat)](https://gagan93.me/blog/2024/05/04/atlassian-interview-experience.html)

---

## 3. Capital One Services

**Role:** Associate Business Analyst (DataLabs, Bangalore) — a tech-adjacent business/analytics role, not pure SWE.

**Selection process:**
1. CV screen (CGPA > 7).
2. Screening test at CDC — **aptitude, logical reasoning, data interpretation, probability & statistics** (no heavy DSA reported for the ABA track).
3. Interview: **4 rounds total** — 3 case-interview rounds + 1 HR round.

**Real PYQ notes:** No hard coding questions surfaced for ABA role — it's a **case-round-heavy** process (business cases, not LeetCode). If you're going for a tech-titled Capital One role instead, expect standard OA coding (2 problems) — see TheJobOverflow tag for that variant.

**Prep priority:** Case-interview frameworks (market sizing, profitability cases), probability/stats brush-up, data interpretation speed (bar/pie charts, tables). Since your resume leans DS/ML, foreground that — it's explicitly valued for this role even though it's a "business" title.

**Sources:** [FORESIGHT 2024 – Aman Kedia](https://medium.com/@swgiitkgp/foresight-2024-placement-at-capital-one-aman-kedia-112d88743852), [TheJobOverflow – Capital One tag](https://www.thejoboverflow.com/company_tag/95), [Glassdoor](https://www.glassdoor.com/Interview/Capital-One-Interview-Questions-E3736.htm)

---

## 4. Corridor Technologies (= **Corridor Platforms**)

**Role:** Full-Stack Developer — US-based fintech firm (analytics/AI/governance automation for banking).

**Selection process:**
1. Online Assessment.
2. Round 1: CV-based + technical — **basic DSA + CS fundamentals** (not deep CP).
3. Rounds 2–3: heavily CV/HR-based — deep dive into your past internships/projects, "why this project," "what challenges did you face."

**Key insight from an actual selected candidate:** *"Prior experience and projects seem to carry more weight than CP/DSA... a solid grasp of DSA is still important, but be thoroughly prepared to discuss every item on your CV."* Society/PoR involvement (esp. tech societies) was explicitly noted as a positive signal.

**Prep priority:** Given your SvelTech (full-stack) and AeroShield IQ projects, this is a strong fit — rehearse clean, structured narratives for both (problem → your specific contribution → a bug/tradeoff you handled → impact). Keep DSA at "solve 3–5/day" maintenance level, not a fresh grind.

**Source:** [FORESIGHT 2025 – Saroja Bamra](https://medium.com/@swgiitkgp/foresight-2025-internship-at-corridor-platforms-saroja-bamra-f0968fbb31df)

---

## 5. Fin Mechanics (**FinMechanics** — Singapore-HQ banking software/treasury tech firm)

**Role:** Associate Consultant / Software / Analytics (product development for treasury management, banking tech).

**Selection process:**
1. Written/online test (often Google Forms based) — 4 sections: (a) subjective/behavioral ("biggest achievement"), (b) math — sequences & series, general quant/probability, (c) coding — OOPs, SQL, syntax debugging, (d) finance basics — interest rates, equity, bonds, FX.
2. Round 1 interview: CV + math + 1 DSA/coding question + probability puzzle.
3. Round 2: HR-heavy panel (5–6 people), situational/behavioral grilling.

**Real PYQs found:**
- Probability: *expected number of coin tosses needed to get 3 consecutive heads.*
- Puzzle style: "five thieves" / game-show logic puzzles.
- Coding: OOPs + SQL + debugging (not competitive-programming-hard).
- Finance-consultant-track questions (if resume shows finance interest): option pricing via **Binomial Tree Method** (1-period and 2-period), BTM vs. Finite Difference Method, deriving risk-neutral probability, delta of an ATM call, 2008 financial crisis discussion.

**Prep priority:** Expected-value/probability puzzles (classic ones — coin tosses, dice, hat-check problems), light SQL, sequences & series (JEE-level), and — since you have quant-finance interest on your resume — brush up Binomial Tree option pricing basics (this directly overlaps your Trexquant/derivatives prep).

**Sources:** [WSO company page](https://www.wallstreetoasis.com/company/finmechanics/interview), [Glassdoor](https://www.glassdoor.com/Interview/FinMechanics-Interview-Questions-E1144480.htm), [Ayushi Garg (BSP IITD)](https://www.bspiitd.com/post/ayushi-garg-finmechanics), [Yash Kulkarni (IIT Bombay)](https://summerblog.insightiitb.org/yash-kulkarni-finmechanics/)

---

## 6. GreenSearch Technologies → this is **Glean** (legal entity: "Glean Search Technologies India Pvt Ltd")

**Role:** SDE (backend/full-stack) — Glean is the enterprise AI search company (Palo Alto HQ, Bengaluru office opened 2024, $2.2B valuation).

**Selection process (from Glean's general engineering loop, not KGP-specific yet since it's a new recruiter):**
1. Recruiter call → Hiring-manager screen (~45 min, past projects + why Glean).
2. **Practical coding assignment — ~2 hours to build a small working system**, NOT a LeetCode-style timed puzzle round. Interviewers grade on how much *working, readable* code you produce and whether a teammate could pick it up.
3. System-design round — tilts toward **search, retrieval, ranking, and permission-aware access control** (Glean indexes company docs/Slack/Jira/GitHub/Salesforce and must enforce per-user permissions at query time).
4. Loop is fast: 3–5 weeks typically.

**Prep priority:** This is different from a standard grind — practice building a small end-to-end feature quickly (e.g., a mini search/ranking service) rather than only isolated LeetCode problems. Review basics of inverted indices, ranking (TF-IDF/BM25 intuition), and access-control/permission models. Given your ML/NLP background (BPE tokenizer project) this plays to a real strength — mention it.

**Sources:** [Tracxn legal-entity record confirming Glean=GreenSearch Technologies India](https://tracxn.com/d/legal-entities/india/glean-search-technologies-india-private-limited/), [techinterview.org — what Glean's loop tests](https://www.techinterview.org/post/3233476794/glean-engineering-interview/), [Glean careers](https://job-boards.greenhouse.io/gleanwork)

---

## 7. Hindustan Unilever Limited (HUL)

**Role at KGP:** Multiple — R&D, Tech R&D, Supply Chain, Tech Supply Chain (this year HUL opened tech profiles too, not just core FMCG roles).

**Selection process (Supply Chain Trainee / Tech track):**
1. Resume shortlisting (CGPA-based + relevant projects).
2. Psychometric test.
3. Values/behavioral questionnaire (past experiences + "what leadership means to you," alignment with HUL values).
4. **Video case-study round** — timed case questions (basic supply-chain case scenarios), strict per-question time limits.
5. GD (conducted as a case-study discussion, not open-topic) + Technical interview + HR interview + (sometimes) telephonic round.

No written aptitude test for the classic FMCG track — HUL relies on GD + case rounds + psychometric instead.

**Prep priority:** Case-interview practice (specifically FMCG/supply-chain flavored — inventory, distribution, pricing cases), a tight personal "leadership story," and comfort answering under a hard time limit (their case rounds are strictly timed). If applying to the **Tech** track, standard SQL/Python + light DSA also shows up — confirm which JD you're targeting from the PPT.

**Sources:** [FORESIGHT 2024 – Saheb Lohakare](https://medium.com/@swgiitkgp/foresight-2024-internship-at-hindustan-unilever-limited-saheb-lohakare-1fcad86f59c3), [FORESIGHT 2023 – Sravani Gopi](https://medium.com/@swgiitkgp/foresight-2023-summer-internship-at-hul-hari-g-db74a20049a5), [Communiqué IIT KGP interview](https://cq-iitkharagpur.medium.com/cq-fmcg-luminaries-barchha-dhawal-samir-hindustan-unilever-ltd-3e2a9649a3dd)

---

## 8. KLA (KLA-Tencor)

**Role:** ML/SWE Internship (semiconductor process-control/metrology company).

**Selection process (KGP-specific, seen across years):**
1. Resume shortlist (no CGPA cutoff seen in one recent cycle).
2. Online test on HackerRank — **26 questions across 5 sections in 80 minutes** (aptitude + logic + core CS, mixed with a couple of coding problems) — heavy time pressure, so speed matters more than depth per question.
3. In an older on-campus cycle: 6-stage process — Resume → Aptitude Test (20 Qs/30 min) → **Group Discussion** (10 people/group, ~2 selected) → **Picture Perception Test** (unusual — described as unique to KLA) → Technical Interview (1:1) → HR.

**Prep priority:** Speed-focused aptitude practice (this is a high-question-count, low-time-per-question format — practice under a strict per-question timer, not just accuracy). If GD/PPT rounds are still part of their current process, prep general GD frameworks and be ready for an unconventional round (KLA's Picture Perception Test is atypical — stay adaptable, articulate a clear structured narrative for whatever image/prompt you get).

**Sources:** [Shanmukha Sainath – ML Internship OA](https://medium.com/@shanmukh05/kla-on-campus-machine-learning-internship-interview-experience-2021-72518adac034), [Glassdoor – KLA @ IIT Kgp](https://www.glassdoor.com/Interview/KLA-Interview-Questions-E1557_P21.htm)

---

## 9. Neo Wealth and Asset Management

**Role:** Quantitative Trader / Quant Strategist / Quant Developer-Researcher — founded by an IIT Kharagpur alumnus (Nitin Jain, ex-Edelweiss/Goldman Sachs), fast-growing systematic trading + wealth platform (~$3B AUA).

**Selection process:** No KGP-specific written interview breakdown surfaced yet (this looks like a newer/smaller-volume recruiter on campus), but based on the role descriptions:
- Focus areas for the Quant Strategist/Trader role: systematic strategies across **equity derivatives, index futures/options, factor-based equity portfolios**, statistical arbitrage, quantamental strategies, low-to-mid-frequency research.
- Expect standard quant-hiring format: OA with probability/mental-math/logic → technical interview with market-microstructure + coding + stats → HR/fit.

**Prep priority:** Since concrete KGP PYQs aren't public yet, prepare generically for quant-trading interviews: probability & expected value (classic brainteasers), statistics (regression, hypothesis testing basics), options/derivatives fundamentals (Greeks, payoff diagrams), Python/C++ for backtesting-style coding, and be ready to discuss any of your own quant projects (pairs-trading backtester, asset-allocation work) — this overlaps directly with your Trexquant Alpha work, which is a strong talking point here.

**Sources:** [Foundit – Quant Trader JD](https://www.foundit.in/job/quantitative-trader-neo-wealth-and-asset-management-india-53599593), [Neo Asset Management](https://neoassetmanagement.com/), [Glassdoor (limited data)](https://www.glassdoor.co.in/Interview/Neo-Wealth-Management-Interview-Questions-E7125889.htm)

*Caveat: this is the thinnest-data company on your list — treat the above as directional, not confirmed PYQs, and check with seniors/CDC WhatsApp groups closer to the date.*

---

## 10. Piramal Finance (Piramal Capital & Housing Finance / Piramal Enterprises)

**Role:** Data/Analytics intern-FTE (Business Intelligence Unit) or Software track — open to all departments.

**Selection process (Data/BIU track, most relevant to you):**
1. Online Assessment — 3 sections: **English & Grammar, Logical Reasoning & Quantitative Aptitude, Coding.**
2. Interviews: CV-based rounds (multiple) + final HR round.

**Selection process (Software track, alternate variant seen):**
1. Coding round — MCQs + **2 competitive-programming-style coding questions** (own language choice).
2. Technical rounds: DSA (medium/hard) + OOPs + SQL + OS core concepts.
3. HR.

**Real PYQ notes:** "Find mean and median from a dataset in Python," "highest marks in each subject" (SQL query), general Python/SQL basics for the FTE-Management-Trainee variant. Aptitude test + GD were seen in an older campus-pool-hiring cycle.

**Prep priority:** Given your ML/data background, the Data/BIU track fits well — brush up basic-to-intermediate SQL (GROUP BY, window functions), Pandas basics (mean/median/groupby), and standard quant aptitude. Keep 2 DSA problems/day as backup in case you land the software-track OA instead.

**Sources:** [FORESIGHT 2025 – Telu Amruthavarshini (BIU)](https://medium.com/@swgiitkgp/foresight-2025-internship-at-piramal-housing-finance-limited-telu-amruthavarshini-3536053763e5), [FORESIGHT 2024 – Sampreeth Sivakoti / Nikhil Thogaru (software)](https://medium.com/@swgiitkgp/foresight-2024-internship-at-piramal-housing-and-finance-sampreeth-sivakoti-2887d892b374), [Glassdoor](https://www.glassdoor.co.in/Interview/Piramal-Finance-Interview-Questions-E6977947.htm)

---

## 11. Salesforce

**Role:** SDE Intern/FTE (Hyderabad/Bangalore).

**Selection process (very consistent across many KGP years):**
1. OA on HackerRank/HackerEarth — typically **2–3 coding questions** (medium level: greedy, BFS on matrix, min-heap+DP have all appeared) + sometimes 10 MCQs on DS/OS/DBMS. Duration ~60–90 min.
2. Technical interview(s) — DSA (trees, heaps, LCS, parenthesis-matching type problems have recurred), sometimes OOPs deep-dive (they've asked candidates to literally write polymorphism code even off-language).
3. HR/culture round — standard strengths/weaknesses + "why Salesforce."

**Real PYQs found (recurring across years — good signal these repeat):**
- Check if a matrix is symmetric.
- Longest Common Subsequence.
- Print all balanced parenthesis combinations of length n.
- Tree question: given a tree where each node has a number, [variant tasks around subtree sums/paths].
- Cricket-simulation style question: *given per-ball scores for two teams, binary-search-based comparison logic* (then extended live in-interview with new constraints — sixes limit, wide balls — to test if you can adapt existing code).
- OOPs: explain class/object, inheritance types, write Java code demonstrating polymorphism.

**Prep priority:** Medium-level LeetCode (you're strong here already) — specifically revise **heaps, LCS/DP-on-strings, BFS-on-grid, and parenthesis/backtracking problems**, since these repeat almost every year. Also rehearse OOPs fundamentals verbally (they like asking you to explain, not just code).

**Sources:** [GfG – multiple Salesforce experience posts](https://www.geeksforgeeks.org/salesforce-interview-experience-internship-on-campus/), [GfG Set 3](https://www.geeksforgeeks.org/salesforce-interview-experience-set-3-campus/), [GfG SDE Intern 2024](https://www.geeksforgeeks.org/salesforce-interview-experience-sde-intern-on-campus-2024/), [Ankit Saurabh](https://medium.com/@ankitsaurabh/salesforce-interview-experience-42aba277126b)

---

## 12. Wells Fargo

**Role at KGP:** Two distinct tracks seen — (a) **Technology/SDE Analyst**, (b) **Quantitative Program Analyst** (rotation across Quantitative Analytics Centres of Excellence — Risk & AI Model Development).

**Selection process (Quant Program Analyst — most relevant given your quant-finance interest):**
1. CV shortlist.
2. Online Test — 3 sections: **Data Interpretation, Math + Probability & Statistics, 2 coding questions.**
3. Interview rounds (typically 2 technical + 1 combined technical/HR).

**Selection process (Technology Analyst variant, newer format seen):**
1. OA — no backtracking allowed, sections: **10 GenAI-conceptual/scenario questions, Data Interpretation (pie charts etc.), Verbal English, DSA coding.**
2. 2 technical rounds + 1 combined technical/HR round.

**Real PYQ notes:** Probability + basic options questions have appeared in some OA variants (general Wells Fargo pattern, not KGP-confirmed). DSA questions tend to be standard medium-level (arrays/strings/graphs).

**Prep priority:** Data-interpretation speed (charts/tables under time pressure), probability & statistics (expected value, distributions — same muscle as your Trexquant work), 2 solid medium-level DSA problems worth of practice, and — new this cycle — be ready for **GenAI-conceptual MCQs** (basic LLM/AI literacy: what is a transformer, prompt engineering basics, hallucination, RAG at a conceptual level — no need for deep math here, just familiarity).

**Sources:** [FORESIGHT 2025 – Samanway Ray (Quant Program Analyst)](https://medium.com/@swgiitkgp/foresight-2025-placement-at-wells-fargo-samanway-ray-71d20c7d410c), [CodingKaro – 2026 OA breakdown](https://www.codingkaro.in/jobs-internships/leetcode-interview-experience/Wells%20fargo), [TheJobOverflow – Wells Fargo tag](https://thejoboverflow.com/?tag=wells-fargo), [Naukri Code360 experience](https://www.naukri.com/code360/interview-experiences/wells-fargo/wells-fargo-interview-experience-aug-2021-exp-0-2-years)

---

## 13. UiPath

**Role:** SDE / Product Support Intern (Bangalore) — RPA (robotic process automation) company.

**Selection process:**
1. On-campus OA (HackerRank) — **3 sections: Aptitude & Reasoning, Communication/English, Programming Fundamentals (C/Java-based)**. ~300 shortlisted from resume → top ~25 from OA.
2. Technical interview — resume-based (internships/projects/skills) + behavioral (customer-facing role) + light CN/SQL/OS questions.
3. (For SDE track, off-campus/lateral data) — Coding & Problem-Solving rounds (Trees/DSA, Two-Pointer/Linked List), a **Low-Level Design (LLD)** round, and a Culture-fit round discussing HLD concepts and past project challenges.

**Real PYQs found:**
- DSA: LCA on binary trees, two-pointer/linked-list problems.
- Classic DP: **House Robber problem** ("can't rob two adjacent houses — maximize amount stolen").
- LLD: *design a service that replicates data from upstream services and forwards it to consuming microservices.*

**Prep priority:** Standard medium DSA (trees, DP, two-pointer) — you're well-drilled here. Add a light LLD pass (class-diagram-level system design — e.g., practice designing a rate limiter, notification service, or the replication-service prompt above) since UiPath's SDE-2+ loop leans on it. Also brush up basic CN/SQL/OS (your existing CN/OS reference doc directly covers this).

**Sources:** [GfG – Product Support Intern](https://www.geeksforgeeks.org/uipath-interview-experience-for-product-support-intern-on-campus/), [Naukri Code360 – SWE Intern](https://www.naukri.com/code360/interview-experiences/uipath/interview-experience-on-campus-aug-2022-2-8778), [enginebogie – SDE-2](https://enginebogie.com/interview/experience/uipath-software-development-enginner-2/135), [Glassdoor](https://www.glassdoor.co.in/Interview/UiPath-Bangalore-Interview-Questions-EI_IE1102519.0,6_IL.7,16_IM1091.htm)

---

## 14. NatWest Group

**Role:** Software Engineer (Gurgaon/Chennai/Bangalore/Pune, Global Capability Centre banking-tech).

**Selection process (highly consistent across campuses — this is one of the most "fixed format" companies on your list):**
1. Eligibility screen: 10th ≥70%, 12th ≥70%, CGPA ≥6 with no standing arrears; shortlist further narrowed by CGPA + resume.
2. **Online Test** — platform varies by year (AMCAT seen in one cycle, standalone OA in another) but the structure is consistent: **4 sections** — English/Verbal, Technical MCQs (OOPs, DSA, DBMS, C/output-prediction), SQL, and 2 Coding questions. In one full breakdown: 50 total questions — 48 MCQs (4 sections × 12) + 2 programming questions.
3. **Single combined Technical + HR interview** (~25–35 min) — no separate rounds. Covers resume/projects/internships in depth, a DSA-fundamentals discussion, then folds directly into HR questions ("what do you know about NatWest," "why should we hire you," "what values can you provide to NatWest").

**Real PYQs found:**
- Coding: *find the number of perfect squares in an array.*
- Coding: *find the intersection area of two circles* given two centers (x1,y1),(x2,y2) and radii r1,r2 — return floor of the overlap area.
- Coding (other cycle): *sum of elements of an array* (easy warm-up-style question — difficulty varies a lot by set).
- MCQs: DBMS/SQL-heavy, OOPs concepts, C output-prediction, English vocabulary/passage-reading.
- Interview: linked-list complexities, "which basic data structure underlies a Tree," real-world applications of linked list/stack/queue/tree/graph, explain Logistic Regression/Naïve Bayes/Decision Tree briefly (if ML is on your resume), why CNNs suit image classification, **write a SQL query for the 3rd-highest salary**, OOPs (polymorphism, inheritance), **Diamond Problem in C++ & virtual functions** (recurring favorite).

**Prep priority:** This is a breadth-over-depth company — solid MCQ-level command of DBMS/SQL/OOPs/CN/OS (your CN/OS Microsoft-prep doc and STL reference directly cover this), the classic "Nth highest salary" SQL pattern, and C++ Diamond Problem/virtual-function mechanics specifically (it's come up more than once). Coding questions themselves are easy-medium — don't over-invest there relative to the MCQ sections, since candidates who nailed both coding questions but weak on MCQs were reported as not advancing.

**Sources:** [Vishwajeet Anand – Selected](https://medium.com/@krishnaram7755/natwest-group-interview-experience-selected-cf84170fae96), [GfG – On-Campus SDE](https://www.geeksforgeeks.org/interview-experiences/natwest-interview-experience-for-software-engineer-on-campus/), [LeetCode Discuss – Summer Trainee OA](https://leetcode.com/discuss/post/7229292/), [Glassdoor](https://www.glassdoor.co.in/Interview/NatWest-Group-Interview-Questions-E10222.htm)

---

## 15. Databricks

**Role:** Software Engineer (Intern/FTE) — this is the most CP/DSA-intensive company on your full list; treat it as your ceiling target.

**Selection process — Internship track:**
1. Online coding test on **CodeSignal** — **4 DSA questions** (first 2 easy, last 2 medium), 90 min. Shortlisting appears to combine "solved all 4" + submission time + department/CGPA.
2. 3 interview rounds same day: **2 coding + 1 HR**.
 - Round 1: design a small class with functions to store/retrieve elements per queries — brute force (set+map, O(n log n)) is a valid start, but they push you toward an optimized **linear-time** solution, then ask you to implement it and spot bugs *without running the code*.
 - Round 2: DFS-on-tree question, implement + run against a test case, followed by a harder follow-up on the same concept (may not require full implementation if time runs out).
 - Round 3: deep resume/project dive + role-specific follow-ups (e.g., TCP/UDP questions if a networking project is listed) + standard HR ("why Databricks," teamwork/collaboration).

**Selection process — FTE track:**
1. Direct CV shortlist (reported cutoff one year: CS department, 9+ CGPA — a strong CV/CGPA bar).
2. **3 rounds**: 2 technical (medium→hard LeetCode-level DSA, standard patterns, no puzzles) + 1 HR/leadership round (~30–45 min, conversational but requires sharp, clear communication — interviewers can be very senior, 15+ years experience).
3. Rounds run **back-to-back with only 2–3 minutes between them** — the whole loop can take ~2.5 hours in one sitting. No guarantee of breaks. Core CS subjects (OS/DBMS/CN) were *not* asked in one FTE cycle but "can vary by year" per the candidate — don't skip them.

**Prep priority (this is your best-covered company given existing prep, but also your hardest bar):**
- DSA: medium→hard LeetCode, with real signal from a selected candidate that **Codeforces 1800+ problems**, **CSES** (DP/graphs/range-queries/tree-algorithms sections), and **GOC intern-test archives** (last 3–4 years) were the most useful — beyond what LeetCode alone gives you.
- Be ready to **optimize live and reason about correctness without running code** — this recurs in both the intern and FTE loops.
- CS fundamentals: go deep, not just topic-name-level — one candidate specifically regretted not knowing **exact normal-form conditions** (not just names) for a sibling company's interview in the same season; Databricks itself has asked networking (TCP/UDP) when it's on your resume. This is a direct match for your existing CN/OS prep doc — just push it one level deeper (exact definitions, not just concepts).
- OOPs/C++: comfort with STL internals (what a vector does under the hood, etc.) was explicitly called out as more useful than passive reading.
- Practical logistics: interviews are same-day and back-to-back — check your laptop/charger the night before (one candidate nearly missed their slot after forgetting their laptop).

**Sources:** [FORESIGHT 2025 – Abir Roy (Internship, full round-by-round detail)](https://medium.com/@swgiitkgp/foresight-2025-internship-at-databricks-abir-roy-dd7efb40cd97), [FORESIGHT 2025 – Mayukha Marla (FTE, full prep breakdown)](https://medium.com/@swgiitkgp/foresight-2025-placement-at-databricks-mayukha-marla-6dc1238f04fd), [Soukhin Nayek – Internship prep journey](https://medium.com/@soukhinkgp2/databricks-internship-code-effort-and-offer-6cad5c4c6736)

---

## 16. Master Study Plan

Since test dates aren't fixed for all 13 yet, this is a **3-week rolling plan** organized by track, so you can compress or expand based on when each company's slot actually lands. Run the "Night-before checklist" for whichever company is up next, regardless of where you are in the main plan.

### Week 1 — Track A + B foundation (Quant + Product/SDE)
| Day | Focus | Why |
|---|---|---|
| 1–2 | Probability/expected-value puzzle set (coin tosses, dice, hat-check, ballot problems) | Feeds AlphaGrip, FinMechanics, Neo Wealth, Wells Fargo |
| 3 | Options basics: Binomial Tree Method (1 & 2 period), risk-neutral probability, Greeks intuition | FinMechanics, Neo Wealth, Wells Fargo — direct overlap with your derivatives/Trexquant work |
| 4–5 | LeetCode medium sprint: **heaps, DP-on-strings (LCS-family), BFS/graphs-on-grid, parenthesis/backtracking** | These exact patterns recur in Salesforce/Atlassian/UiPath PYQs above |
| 6 | Graph algorithms refresh: DFS connectivity, Union-Find, MST intuition | Direct match to the AlphaGrep "palaces & tunnels" question type |
| 7 | Mock timed aptitude set (80 Q in 80 min style) | Matches KLA's high-volume, low-time-per-question format |

### Week 2 — Track B depth + Track C
| Day | Focus | Why |
|---|---|---|
| 8–9 | OOPs oral fluency: be able to *explain* (not just code) inheritance, polymorphism, encapsulation with a live example in 2 languages | Salesforce has literally asked candidates to write polymorphism code in an unfamiliar language |
| 10 | Light LLD pass: rate limiter, pub-sub/replication service, notification system (class diagrams, not full code) | UiPath SDE-2+ loop |
| 11 | Search/retrieval fundamentals: inverted index, basic ranking intuition (TF-IDF/BM25), permission-aware access control concept | Glean (GreenSearch Technologies) — different from standard LeetCode prep, don't skip |
| 12 | 2-hour timed "build something small and working" exercise (e.g., a mini search-and-rank CLI tool) | Glean's actual practical-coding-assignment format |
| 13 | Project-narrative rehearsal: SvelTech + AeroShield IQ — structured 90-second pitch each (problem → your contribution → a bug/tradeoff → impact) | Corridor Platforms weighs CV/project depth over CP |
| 14 | SQL practice: GROUP BY, window functions, joins, Nth-highest-salary pattern; Pandas mean/median/groupby drills | Piramal Finance BIU track + NatWest's classic "3rd highest salary" query |
| 15 | CS-fundamentals depth pass: exact DBMS normal-form conditions (not just names), C++ Diamond Problem + virtual functions, TCP/UDP basics | Databricks (deep-knowledge expectation) + NatWest (Diamond Problem is a recurring favorite) |
| 16 | Codeforces 1800+ / CSES sprint (DP, graphs, range queries, tree algos) — 2-3 problems, timed, no hints | Databricks-specific signal: this outperformed pure LeetCode prep for a selected candidate |

### Week 3 — Track D + integration + GenAI literacy
| Day | Focus | Why |
|---|---|---|
| 17 | Case-interview framework practice: 3 FMCG/supply-chain cases + 1 profitability case, timed | HUL case rounds, Capital One case rounds |
| 18 | Data interpretation speed drills (pie/bar charts, tables) under a strict per-question timer | Capital One, Wells Fargo, HUL |
| 19 | GenAI conceptual literacy pass: transformers, RAG, hallucination, prompt engineering — at MCQ-depth, not research depth | Wells Fargo Technology Analyst OA (new-format) |
| 20 | Full mixed mock OA: aptitude + 2 coding + 1 probability puzzle, single sitting, timed | Simulates the general CDC test format across most of these |
| 21 | Databricks-style back-to-back mock: 2 timed coding rounds + 1 HR/leadership round, no break between | Databricks loop is uniquely compressed (~2.5 hrs, 2-3 min gaps) — nothing else on this list needs this specific stamina rehearsal |
| 22 | Resume rehearsal against all 4 tracks: know which project/story to lead with per company type | Everyone probes CV — get the mapping automatic |
| 23–24 | Rest + light review + re-run your weakest 2 topics from the week | Avoid burnout going into test season |

### Night-before checklist (use before *any* of these 15 tests)
1. Reread this company's section above once.
2. Solve 2 problems in that company's dominant pattern (see table below).
3. Rehearse your 90-second project pitch tailored to the role (data/quant/SDE/consulting).
4. Sleep — CDC test-day performance correlates more with alertness than last-minute cramming, per multiple FORESIGHT accounts above.

| Company | Dominant pattern to drill night-before |
|---|---|
| AlphaGrip Securities | Graph connectivity (DFS/Union-Find) + 1 probability puzzle |
| Atlassian | 1 LeetCode medium/hard + 1 behavioral story |
| Capital One | 1 case framework + data-interpretation set |
| Corridor Technologies | Project pitch rehearsal (not DSA) |
| Databricks | 1 Codeforces 1800+/CSES problem + rehearse "optimize live without running code" out loud; pack your laptop/charger tonight |
| Fin Mechanics | 1 probability puzzle + Binomial Tree option pricing recap |
| GreenSearch Technologies (Glean) | Inverted-index/ranking concept recap |
| HUL | 1 timed case scenario |
| KLA | Timed aptitude set (high volume, low time/question) |
| NatWest Group | SQL Nth-highest-salary query + C++ Diamond Problem/virtual functions recap + skim DBMS/OOPs MCQ topics |
| Neo Wealth | Probability + options/derivatives recap |
| Piramal Finance | SQL query set + Pandas mean/median drill |
| Salesforce | Heap + LCS + balanced-parenthesis problem |
| Wells Fargo | Data interpretation + 1 probability + GenAI-concept skim |
| UiPath | 1 DP problem (House-Robber family) + LLD sketch |

---

*Note: Neo Wealth's KGP-specific process wasn't publicly documented at the time of this search — verify with CDC notices/seniors closer to their visit date. NatWest's data above is cross-campus (not KGP-confirmed) but the format is reported as highly consistent year-to-year and campus-to-campus. All other companies have at least one direct IIT Kharagpur account cited above.*
