# Skill Deep-Dive: Concepts, Q&A & Tasks

Covers all 8 requested skills:

- **Financial Markets Knowledge**
- **Quant Finance Concepts** (options, pricing, strategy formalization)
- **Linux**
- **Shell Scripting / Bash**
- **Computer Networks** (TCP/IP, UDP)
- **Distributed Systems / Concurrency**
- **Low-Latency / HPC Optimization**
- **Compilers** (niche CS)

**How to use this:** read the concept notes once, then close the file and answer the Q&A cold — that's the real test. "Medium proficiency" here means: you can answer variations of these questions fluently in a live interview, *and* you have working code from the task you can pull up and explain. A few tasks are chained together on purpose (BS pricer → benchmark it → the networking task feeds the concurrency task), so the second half of this guide compounds if you do it in order.

---

## 1. Financial Markets Knowledge

**Why this matters:** interviewers at quant/trading-adjacent firms assume you already speak this language before they'll talk strategy or pricing with you.

### Concept notes
- **Primary vs secondary market:** primary = company raises capital directly (IPO, bond issuance); secondary = investors trade existing securities among themselves (this is what "the market" usually means day to day).
- **Exchanges vs OTC:** exchanges (NYSE, NSE) are centralized, order-book-driven, standardized contracts. OTC (most bonds, FX, most derivatives) are negotiated bilaterally, less transparent, more customizable.
- **Order types:** market order (fill now, at whatever price is available), limit order (fill only at your price or better), stop order (dormant until price crosses a trigger, then becomes a market/limit order).
- **Bid-ask spread** compensates the market maker for providing immediacy and for the risk of being caught holding inventory as prices move against them. Spreads widen when volatility rises or liquidity dries up — the market maker demands more compensation for the same risk.
- **Buy-side vs sell-side:** sell-side (banks, broker-dealers) makes markets, underwrites, and sells research/products. Buy-side (hedge funds, asset managers, pension funds) is the client — it consumes liquidity and takes directional/relative-value positions.
- **Order book / market depth:** the stack of resting limit orders at each price level on both sides. "Thin" books move a lot on small orders; "deep" books absorb size without much price impact.
- **Yield curve:** yield vs maturity for bonds of similar credit quality (usually government bonds). Normal = upward sloping (investors demand more yield for locking up money longer). **Inverted** = short-term yields exceed long-term — historically a recession signal, because it implies the market expects the central bank to cut rates in the future.
- **Bond price vs yield:** inverse relationship. Price rises → yield falls, and vice versa, because a bond's cash flows are fixed and yield is just the discount rate that reprices them.
- **Settlement:** T+1/T+2 — the lag between trade date and when cash/securities actually change hands. Clearing houses sit in the middle to reduce counterparty risk.
- **Margin & leverage:** borrowing to control a larger position than your capital alone allows. A margin call happens when your equity falls below a maintenance threshold, forcing you to post more collateral or get liquidated.

### Q&A

**Q: What's the practical difference between a market order and a limit order, and what's the risk of each?**
A: A market order guarantees execution but not price — you take whatever the book offers, which can be bad in a fast-moving or thin market (slippage). A limit order guarantees price but not execution — it may never fill if the market doesn't come to you.

**Q: Why does the bid-ask spread widen during volatile periods?**
A: Market makers face more inventory risk when prices move fast — the odds they get "picked off" (someone trades with them right before the price moves against them) go up, so they widen the spread to be compensated for that risk.

**Q: What does an inverted yield curve signal, and why?**
A: It signals the market expects the central bank to cut short-term rates in the future, usually because growth/inflation is expected to slow — historically a leading recession indicator (though the timing and reliability are debated).

**Q: What's the difference between the buy-side and sell-side?**
A: Sell-side firms (banks, broker-dealers) provide liquidity, underwrite securities, and sell research/execution services. Buy-side firms (hedge funds, asset managers) are the clients — they deploy capital and consume that liquidity to express views.

**Q: Why might a company's bond yield and its stock's implied cost of equity move differently after the same news?**
A: Bondholders only care about getting paid back (default risk), while equity holders bear the full upside/downside of the business. Bad news that raises default risk without threatening solvency can hit the bond yield hard while equity reacts less, or vice versa for news that's dilutive/growth-negative but doesn't touch credit quality.

**Q: What's basis risk?**
A: The risk that a hedge doesn't move perfectly in line with the position it's meant to protect — e.g., hedging jet fuel exposure with crude oil futures leaves you exposed to the crack spread (the price gap between crude and refined products) moving against you.

### Task
Pick one index (NIFTY 50 or S&P 500) and, for 5 trading days, keep a short "market journal":
1. Note the opening gap vs. previous close.
2. Find one instance where you can see a bid-ask spread widen (an options chain around a news event is easiest — NSE weekly index options or any free US quote site).
3. Pick one macro data point/central bank event that week and write 3–4 sentences on how the yield curve or index reacted and why.
4. Open an options chain and, for near/mid/far expiries on the same strike, note how the bid-ask spread and the time-value component of the premium change with time-to-expiry. This bridges directly into Section 2.

---

## 2. Quant Finance Concepts (Options, Pricing, Strategy Formalization)

**Why this matters:** this is the explicit differentiator for quant-research and quant-desk roles — usually the area where a strong CS/CP background doesn't automatically transfer.

### Concept notes
- **Option payoff at expiry:** call = max(S − K, 0), put = max(K − S, 0), where S = spot, K = strike.
- **Premium = intrinsic value + time value.** Intrinsic value is what you'd get if exercised right now; time value reflects the chance the option moves further in your favor before expiry, and decays to zero at expiration (theta decay).
- **Moneyness:** ITM (in the money — has intrinsic value), ATM (spot ≈ strike), OTM (out of the money — pure time value).
- **Put-call parity (European, no dividends):** C − P = S − K·e^(−rT). This is a no-arbitrage identity — if it doesn't hold, you can construct a riskless profit by buying the cheap side and selling the expensive side.
- **Black-Scholes assumptions:** underlying follows geometric Brownian motion, constant volatility and risk-free rate, no dividends, frictionless markets (no transaction costs), continuous trading, no arbitrage. Every one of these assumptions is violated in real markets to some degree — that's *why* implied vol has a smile/skew instead of being flat.
- **Black-Scholes formula (call):**
  - C = S·N(d1) − K·e^(−rT)·N(d2)
  - d1 = [ln(S/K) + (r + σ²/2)T] / (σ√T)
  - d2 = d1 − σ√T
- **The Greeks:**
  - **Delta** (∂price/∂S): hedge ratio — how much the option price moves per $1 move in the underlying. Also ≈ probability the option finishes ITM (rough intuition, not exact).
  - **Gamma** (∂delta/∂S): convexity — how fast delta itself changes. High near ATM, close to expiry.
  - **Vega** (∂price/∂σ): sensitivity to implied volatility.
  - **Theta** (∂price/∂t): time decay — options are a depreciating asset, all else equal.
  - **Rho** (∂price/∂r): sensitivity to interest rates — usually the smallest effect for short-dated options.
- **Implied vs. historical volatility:** historical vol is computed from realized past returns; implied vol is backed out of the market price of an option — it's the market's forward-looking estimate (and includes a risk premium, so implied tends to run above realized on average).
- **Volatility skew/smile:** OTM puts trade at higher implied vol than OTM calls on equity indices — the market prices in crash risk (fat left tail) that lognormal Black-Scholes doesn't capture.
- **Formalizing a strategy:** state the hypothesis precisely (what signal, what edge, over what horizon), define entry/exit rules unambiguously, backtest with realistic frictions (transaction costs, slippage, no lookahead bias — you must only use information available at that point in time), then evaluate with risk-adjusted metrics, not raw return.
- **Sharpe ratio:** (Rp − Rf) / σp — return per unit of total volatility. Sortino uses only downside deviation in the denominator (doesn't penalize upside volatility). Max drawdown measures the worst peak-to-trough loss — often more relevant to how a strategy actually *feels* to run.
- **Kelly criterion (discrete form):** f* = (bp − q) / b, where p = win probability, q = 1−p, b = odds. Full Kelly is aggressive (high variance); practitioners commonly use half-Kelly or quarter-Kelly.

### Q&A

**Q: State put-call parity and explain why it must hold.**
A: C − P = S − K·e^(−rT) for European options with no dividends. It must hold because a portfolio of long call + short put has the exact same payoff as a forward contract on the underlying — if the prices diverged, you could buy the cheap combination and sell the expensive one for a riskless profit (arbitrage).

**Q: What do each of the five main Greeks measure, in one line each?**
A: Delta = sensitivity to underlying price; gamma = sensitivity of delta itself (convexity); vega = sensitivity to implied vol; theta = time decay; rho = sensitivity to interest rates.

**Q: Why does implied volatility typically show a skew for equity index options rather than being flat across strikes?**
A: Black-Scholes assumes lognormal returns, but real equity markets have fatter left tails — crashes happen more often and more violently than a lognormal distribution predicts. The market prices that in by bidding up OTM puts (crash insurance) relative to OTM calls.

**Q: What's the difference between historical and implied volatility?**
A: Historical vol is backward-looking, computed from realized price changes. Implied vol is forward-looking, extracted from current option prices — it reflects the market's expectation of future volatility plus a risk premium (which is why implied tends to average higher than subsequently realized vol).

**Q: How would you formalize a trading strategy so it's actually testable, not just a hunch?**
A: Write down the exact signal (what data, what transformation, what threshold), the exact entry/exit rule, the exact position sizing rule, then backtest using only information that would have been available at each point in time (no lookahead), including realistic transaction costs and slippage — then report risk-adjusted performance, not just cumulative return.

**Q: Why is Sharpe ratio more useful than raw return for comparing two strategies?**
A: Raw return says nothing about the risk taken to get there. A strategy returning 20% with wild swings can be strictly worse to actually run than one returning 12% smoothly. Sharpe normalizes return by volatility so you're comparing risk-adjusted performance, not just the headline number.

### Task
1. Build a Black-Scholes pricer in Python that returns price, delta, gamma, and vega for a call and a put given (S, K, T, r, σ). Skeleton to get you started (theta/rho intentionally left for you):

```python
import math
from scipy.stats import norm

def black_scholes(S, K, T, r, sigma, option='call'):
    d1 = (math.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*math.sqrt(T))
    d2 = d1 - sigma*math.sqrt(T)
    if option == 'call':
        price = S*norm.cdf(d1) - K*math.exp(-r*T)*norm.cdf(d2)
        delta = norm.cdf(d1)
    else:
        price = K*math.exp(-r*T)*norm.cdf(-d2) - S*norm.cdf(-d1)
        delta = norm.cdf(d1) - 1
    gamma = norm.pdf(d1) / (S*sigma*math.sqrt(T))
    vega  = S*norm.pdf(d1)*math.sqrt(T)
    # TODO: implement theta and rho yourself
    return price, delta, gamma, vega
```

2. Sanity check: verify put-call parity holds numerically for your outputs (C − P should equal S − K·e^(−rT) to floating-point precision).
3. Extend it: take a simple signal (e.g. an SMA-crossover or RSI+MACD rule) — instead of trading the underlying, use the signal to decide "buy calls" vs "buy puts," price the position at entry/exit with your BS pricer, and compute the Sharpe ratio of the *option* strategy vs. buy-and-hold on the underlying. This is the strategy-formalization skill in action, not just a pricer that sits unused.

---

## 3. Linux

**Why this matters:** quant/trading infra and most backend/SWE stacks run on Linux almost universally — this is assumed baseline knowledge, not a bonus.

### Concept notes
- **Filesystem hierarchy:** `/` root, `/home` user directories, `/etc` system configs, `/var` logs and variable data, `/proc` a virtual filesystem exposing live process/kernel info, `/tmp` scratch space.
- **"Everything is a file"** — devices, sockets, and pipes are all addressed through the filesystem interface.
- **Permissions:** `rwx` for owner / group / others. `chmod 755` = owner has read+write+execute, group and others have read+execute. `chown` changes the owner; `sudo` runs a command with elevated privileges.
- **Process management:** `ps aux` lists processes, `top`/`htop` shows live resource usage. Every process has a PID and a PPID (parent). States include running, sleeping, zombie (finished but not yet reaped by its parent), and stopped.
- **Signals:** `SIGTERM` (15) politely asks a process to stop and can be caught/handled for graceful shutdown; `SIGKILL` (9) terminates immediately and cannot be caught or ignored; `SIGINT` (2) is what Ctrl+C sends.
- **Job control:** `&` backgrounds a job, `jobs` lists them, `fg`/`bg` move a job between foreground and background, `nohup command &` detaches the process from the terminal so it survives you logging out, `disown` removes it from the shell's job table.
- **I/O streams:** stdin (0), stdout (1), stderr (2). Redirection: `>` overwrite, `>>` append, `<` read from file, `2>&1` redirect stderr into wherever stdout is currently going.
- **Disk usage:** `df -h` shows filesystem-level usage; `du -sh <dir>` shows how much a specific directory is consuming.
- **find vs grep:** `find` locates files by name/type/size/date; `grep` searches *inside* file contents for a pattern.

### Q&A

**Q: What's the difference between a hard link and a symbolic link?**
A: A hard link is another directory entry pointing directly at the same inode (same underlying data) — it survives deletion of the original name and can't cross filesystems. A symbolic link is a separate file that just stores a path to the target — it breaks if the target is moved/deleted, but can point across filesystems and to directories.

**Q: What do the three digits in `chmod 755` mean?**
A: Each digit is a bitmask of read(4)+write(2)+execute(1) for owner, group, others respectively. 7 = rwx (owner), 5 = r-x (group), 5 = r-x (others) — owner can modify the file, everyone else can read and execute but not write.

**Q: How would you find which process is listening on a specific port?**
A: `sudo lsof -i :<port>` or `sudo ss -tulnp | grep <port>` — both show the PID and process name bound to that port.

**Q: What's the difference between SIGTERM and SIGKILL?**
A: SIGTERM is a graceful request — the process can catch it and clean up (close files, flush buffers) before exiting. SIGKILL is immediate and unconditional — the kernel terminates the process directly and it cannot intercept or ignore it, so it's a last resort when a process is unresponsive.

**Q: What does `nohup some_command &` accomplish?**
A: It runs the command in the background (`&`) and makes it immune to the SIGHUP signal that would normally be sent when your terminal session closes (`nohup`), so the process keeps running after you log out.

**Q: How would you check which directory is consuming the most disk space under `/var`?**
A: `du -sh /var/* | sort -rh | head` — sum usage per subdirectory, sort descending by human-readable size, show the top offenders.

### Task
Do this hands-on (WSL, a Linux VM, or a lab machine) — reading isn't enough:
1. Navigate the filesystem and identify what's in `/etc`, `/var/log`, `/proc/<your-shell-pid>`.
2. Create a file, set it to `chmod 644`, then try to execute it and explain why it fails; fix it and re-test.
3. Start a long-running process (`sleep 300 &`), find its PID, send it SIGTERM, confirm it's gone.
4. Use `top`/`htop` while running something CPU-heavy (e.g., a brute-force solution on a large input) and note CPU%/memory in real time.
5. On any log file (or generate a fake one), use `grep`, `find`, and `du` to answer: "which files were modified in the last 24 hours, and which is the largest?"

---

## 4. Shell Scripting / Bash

**Why this matters:** pairs directly with Linux above, and is genuinely useful for automating your own workflow, not just interview trivia.

### Concept notes
- **Variables:** no spaces around `=`; always quote (`"$var"`) to prevent word-splitting/glob expansion on values with spaces.
- **Control flow:** `if [[ condition ]]; then ... fi`, `for i in ...; do ... done`, `while [[ condition ]]; do ... done`.
- **Functions:** `name() { ... }` — `$1`, `$2`, `$@` work as positional args inside functions too.
- **`$@` vs `$*`:** `"$@"` expands to each argument as a separate quoted string (preserves spaces inside individual args) — almost always what you want. `"$*"` joins all arguments into a single string. This matters the moment an argument contains a space.
- **Text processing:** `grep` (pattern search; `-i` ignore case, `-v` invert match, `-c` count, `-E` extended regex), `sed` (stream editor; `s/find/replace/g` is the classic substitution), `awk` (field-based processing — `$1`, `$2`, ... are columns, `-F','` sets the delimiter — the tool of choice for CSV/log column extraction and aggregation), `cut`, `sort`, `uniq -c`.
- **Exit codes:** `0` = success, non-zero = failure. `$?` holds the exit code of the last command.
- **Robust scripting:** `set -e` (exit immediately on any command failure), `set -u` (error on referencing an unset variable), `set -o pipefail` (a pipeline fails if *any* stage fails, not just the last one), `trap 'cleanup' EXIT` (run cleanup code no matter how the script exits).
- **Cron:** `crontab -e` to edit, syntax is `minute hour day month weekday command` (e.g., `0 6 * * *` = every day at 6:00 AM).

### Q&A

**Q: What's the practical difference between `"$@"` and `"$*"` when forwarding arguments to another command?**
A: `"$@"` preserves each argument as its own separate word (correct for args with spaces), while `"$*"` collapses everything into one string. If a filename has a space in it, `"$*"` will mangle it into two words while `"$@"` won't.

**Q: How do you make a bash script fail loudly instead of silently continuing after an error?**
A: `set -euo pipefail` at the top — `-e` exits on any failing command, `-u` catches unset-variable typos, `pipefail` makes a failure anywhere in a pipe propagate instead of being masked by the last command's success.

**Q: How would you sum the 3rd column of a comma-separated file using awk?**
A: `awk -F',' '{sum += $3} END {print sum}' file.csv`

**Q: What does `2>&1` do, and why does order matter (`cmd > out.log 2>&1` vs `cmd 2>&1 > out.log`)?**
A: It redirects file descriptor 2 (stderr) to wherever file descriptor 1 (stdout) is currently pointing. Order matters because redirections are processed left to right: `> out.log 2>&1` sends both stdout and stderr to the file; `2>&1 > out.log` redirects stderr to the *terminal* (wherever stdout was pointing at that moment) and then redirects stdout alone to the file.

**Q: How do you schedule a script to run every day at 6 AM?**
A: Add a cron entry: `0 6 * * * /path/to/script.sh`.

**Q: What's the difference between running a script with `bash script.sh` vs. `source script.sh`?**
A: `bash script.sh` runs it in a new subshell — variable/directory changes inside don't affect your current shell. `source script.sh` (or `. script.sh`) runs it in your *current* shell, so `cd`, exported variables, and function definitions persist afterward.

### Task
Write a bash script that automates a competitive-programming test loop: given a folder of `.cpp` solutions and a matching folder of `.in`/`.out` test cases, it compiles each solution and reports pass/fail per test case with timing. Skeleton:

```bash
#!/usr/bin/env bash
set -euo pipefail
SRC_DIR="./solutions"
TEST_DIR="./tests"

for src in "$SRC_DIR"/*.cpp; do
    name=$(basename "$src" .cpp)
    g++ -O2 -o "/tmp/$name" "$src"
    pass=0; total=0
    for infile in "$TEST_DIR/$name"/*.in; do
        total=$((total+1))
        expected="${infile%.in}.out"
        actual=$(/tmp/"$name" < "$infile")
        expected_content=$(cat "$expected")
        if [[ "$actual" == "$expected_content" ]]; then
            pass=$((pass+1))
        fi
    done
    echo "$name: $pass/$total passed"
done
```

Extend it yourself: add per-test timing (wrap the run with `time`, or capture start/end with `date +%s%N`), flag any test that exceeds a 2-second limit (TLE simulation), and use `trap` to clean up the `/tmp` binaries on exit.

---

## 5. Computer Networks (TCP/IP, UDP)

**Why this matters:** foundational for anything low-latency/trading-infra or backend-systems adjacent, and it feeds directly into Sections 6 and 7 below.

### Concept notes
- **Simplified layering:** Application (HTTP, DNS) → Transport (TCP/UDP) → Network (IP) → Link (Ethernet/WiFi). Each layer only needs to know about the layer immediately below it.
- **TCP** is connection-oriented and reliable:
  - **Three-way handshake:** SYN → SYN-ACK → ACK establishes the connection before any data flows.
  - **Reliability:** every byte is numbered (sequence numbers); the receiver ACKs what it got; unACK'd data gets retransmitted.
  - **Flow control:** a sliding window caps how much unACK'd data the sender can have in flight, so a slow receiver isn't overwhelmed.
  - **Congestion control:** algorithms (Cubic is the Linux default, BBR is a newer alternative) back off the send rate when packet loss is detected, to avoid overwhelming the *network*, not just the receiver.
- **UDP** is connectionless: no handshake, no ordering guarantee, no retransmission — the OS just fires the packet and moves on. Lower overhead → lower latency, at the cost of reliability. Used where speed matters more than occasional loss: DNS queries, video/audio streaming, gaming, and many market-data multicast feeds (where an application-level sequence number lets you *detect* a gap and decide whether to request a retransmit, rather than paying TCP's overhead on every single packet).
- **Sockets:** an endpoint is (IP address, port). The port identifies which application on the host should receive the traffic.
- **HTTP:** request (method + URL + headers + body) / response (status code + headers + body), normally carried over TCP (HTTP/3 moves to QUIC, which runs over UDP).
- **DNS resolution (simplified):** browser cache → OS cache → resolver → root server → TLD server → authoritative server → IP address returned.
- **Latency vs. bandwidth vs. throughput:** latency = time for one packet to make the trip; bandwidth = the link's maximum theoretical data rate; throughput = what you actually achieve, which can fall short of bandwidth because of latency, loss, or protocol overhead.

### Q&A

**Q: Walk through the TCP three-way handshake.**
A: Client sends SYN (synchronize, "I want to connect, here's my starting sequence number"). Server replies SYN-ACK (acknowledges the client's SYN and sends its own). Client replies ACK (acknowledges the server's SYN). Only after this does either side send actual data — it's how both sides agree they can both send and receive before committing resources.

**Q: Why does UDP have lower latency than TCP, and what do you give up?**
A: No handshake before sending, no waiting for ACKs, no retransmission logic, no congestion-control back-off — the packet just goes. You give up delivery guarantees, ordering, and duplicate protection; the application has to handle any of that itself if it needs it.

**Q: What problem is TCP congestion control solving, and name one algorithm?**
A: It prevents a sender from flooding the *network* (not just the receiver) faster than it can carry the traffic, which would cause cascading packet loss for everyone sharing that path. Cubic is the default on Linux; it grows the send rate aggressively until loss is detected, then backs off and probes again.

**Q: What's the difference between a socket and a port?**
A: A port is just a number identifying an application on a host. A socket is the full addressable endpoint — (IP address, port, protocol) — plus the OS-level handle/state used to actually send and receive on that connection.

**Q: Why might a low-latency trading system prefer UDP over TCP for market data?**
A: TCP's reliability machinery (ACKs, retransmission, congestion back-off) adds latency and unpredictable jitter — exactly what you don't want when every microsecond counts. UDP multicast lets the exchange broadcast the same feed to many subscribers cheaply, and if you occasionally lose a packet, an application-level sequence number lets you detect the gap and decide how to handle it, rather than paying TCP's overhead on every packet just to guard against a rare loss.

**Q: At a high level, what happens between typing a URL and seeing the page?**
A: DNS resolves the hostname to an IP → TCP handshake establishes a connection to that IP on port 443 (or 80) → (for HTTPS) a TLS handshake negotiates encryption → the browser sends an HTTP request → the server sends back an HTTP response → the browser parses and renders it, likely firing off more requests for embedded resources along the way.

### Task
Write a TCP echo server/client pair in Python that measures round-trip latency, then do the same over UDP and compare.

```python
# tcp_server.py
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('localhost', 9999)); s.listen(1)
conn, _ = s.accept()
while True:
    data = conn.recv(1024)
    if not data:
        break
    conn.sendall(data)  # echo back
```

```python
# tcp_client.py
import socket, time
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('localhost', 9999))
times = []
for _ in range(1000):
    start = time.perf_counter()
    s.sendall(b'ping')
    s.recv(1024)
    times.append(time.perf_counter() - start)
times.sort()
print(f"avg RTT: {sum(times)/len(times)*1e6:.1f} us, p99: {times[int(len(times)*0.99)]*1e6:.1f} us")
```

Rewrite both using `socket.SOCK_DGRAM` (UDP) instead of `SOCK_STREAM`. Compare average *and* p99 (tail) latency between the two — tail latency is usually the more interesting number for this kind of system. Keep this server/client pair; you'll extend it in Section 6.

---

## 6. Distributed Systems / Concurrency

**Why this matters:** a real gate in many SWE loops (concurrency, multithreading, synchronization), not a nice-to-have.

### Concept notes
- **Concurrency vs. parallelism:** concurrency is about *structuring* a program to deal with multiple things that can be in progress at once (may interleave on a single core); parallelism is *actually* executing multiple things simultaneously (requires multiple cores). Concurrency is a design property; parallelism is a runtime property.
- **Threads vs. processes:** threads share the same memory space (cheap to create/switch, but unsynchronized access to shared data is dangerous); processes are isolated (safer, but heavier — no shared memory by default, need explicit IPC).
- **Race condition:** the outcome depends on the unpredictable timing/interleaving of concurrent operations — classic example: two threads doing `counter++` on a shared variable without synchronization lose updates, because `counter++` is actually read-modify-write, not atomic.
- **Synchronization primitives:** mutex (mutual exclusion — exactly one owner at a time), semaphore (a counting mutex — allows up to N concurrent holders, useful for limiting concurrent access to a pool of resources), condition variable (lets a thread sleep until another thread signals that some condition changed — the basis of producer-consumer patterns).
- **Deadlock — the four Coffman conditions, all of which must hold simultaneously:** mutual exclusion, hold-and-wait, no preemption, circular wait. Breaking any single one prevents deadlock (e.g., always acquiring locks in the same global order breaks circular wait).
- **CAP theorem, stated precisely:** during a network partition, a distributed system must choose between Consistency (every read sees the latest write) and Availability (every request gets a response). You cannot have both *during* a partition — the common "pick 2 of 3" phrasing oversells it; in the absence of a partition you can have both C and A, the real tradeoff only bites when the network splits.
- **Consensus:** the problem of getting distributed nodes to agree on a single value despite failures/network issues. Paxos and Raft are the canonical named algorithms — you don't need to implement them, but you should know they solve leader election + replicated log agreement, and *why* that's hard (nodes can fail or messages can be delayed/lost, and you can't always tell the difference).
- **Idempotency:** an operation that produces the same result no matter how many times it's applied. Critical for safe retries over unreliable networks — if a client doesn't get an ACK, it doesn't know if the request was lost or just the *response* was lost, so it retries; idempotency means that retry is safe even if the original actually succeeded.
- **Delivery semantics:** at-least-once (may duplicate, never loses), at-most-once (may lose, never duplicates), exactly-once (the ideal, expensive/hard to guarantee in general — usually approximated as at-least-once + idempotency on the receiving side).

### Q&A

**Q: What's the difference between concurrency and parallelism, concretely?**
A: A single-core CPU running a web server that juggles many open connections via an event loop is concurrent but not parallel — nothing is literally happening at the same instant. A multi-core machine running four threads that are each actually executing at the same moment is parallel.

**Q: What is a race condition, and how do you prevent one on a shared counter?**
A: The final result depends on the timing of unsynchronized concurrent access — e.g., two threads both read the counter as 5, both compute 6, both write 6 back, so an increment is lost. Fix it with a mutex around the read-modify-write, or use a language-level atomic type (`std::atomic<int>` in C++) which guarantees the operation is indivisible.

**Q: State the four Coffman conditions for deadlock.**
A: Mutual exclusion (a resource can only be held by one thread at a time), hold-and-wait (a thread holds one resource while waiting for another), no preemption (a resource can't be forcibly taken away), circular wait (a cycle of threads each waiting on a resource held by the next). All four must hold for deadlock to occur.

**Q: State the CAP theorem correctly — what's the common misconception?**
A: During a network partition, you must choose Consistency or Availability — you can't guarantee both. The misconception is treating it as "pick any 2 of 3 at all times" — in normal operation without a partition, a system *can* be both consistent and available; the tradeoff is specifically about behavior under partition.

**Q: What's the difference between a mutex and a semaphore?**
A: A mutex allows exactly one owner at a time and typically must be released by the same thread that acquired it (ownership semantics). A semaphore is a counter that allows up to N concurrent holders and has no ownership concept — any thread can signal/release it, which makes it suited to resource-pool limiting rather than simple mutual exclusion.

**Q: Why does idempotency matter for retrying a failed payment API call?**
A: If a client times out waiting for a response, it can't tell whether the request never arrived, arrived but the server crashed before responding, or succeeded and only the response was lost. If the retry isn't idempotent, retrying could charge the customer twice. Making the endpoint idempotent (e.g., via a client-generated request ID the server deduplicates on) makes "just retry" a safe default.

### Task
Write a C++ program that demonstrates the race condition, then fixes it, then times both:

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>
#include <atomic>

long counter = 0;
std::mutex mtx;

void increment(int n) {
    for (int i = 0; i < n; i++) {
        counter++;  // TODO: run this version first, unguarded
        // std::lock_guard<std::mutex> lock(mtx); counter++;  // then this version
        // or: replace 'long counter' with 'std::atomic<long> counter' entirely
    }
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 4; i++) threads.emplace_back(increment, 1000000);
    for (auto& t : threads) t.join();
    std::cout << counter << " (expected 4000000)\n";
}
```

Run it unguarded first and confirm the count comes out wrong (and non-deterministic — run it a few times). Fix it two ways: (1) a `std::mutex` around the increment, (2) `std::atomic<long>` instead of a plain `long`. Time all three versions and note the synchronization overhead of each approach relative to the (incorrect) unguarded baseline.

**Bonus — ties Sections 5 and 6 together:** take your TCP client/server from Section 5. Make the client send the same "increment" request up to 3 times if it doesn't get an ACK within a short timeout (simulating an unreliable network), and modify the server to be idempotent — dedupe by a request ID so retries don't double-count the increment. That's the retry + idempotency pattern from real distributed systems, built from scratch.

---

## 7. Low-Latency / HPC Optimization

**Why this matters:** the most demanding, narrowest skill on this list — where hand-wavy understanding gets caught fastest in an interview, since it's easy to ask a quick follow-up that exposes a shallow answer.

### Concept notes
- **Cache hierarchy:** L1/L2/L3 are progressively larger and slower than the CPU registers but much faster than RAM. A **cache line** (typically 64 bytes) is the unit fetched from memory at a time. Sequential access (arrays) is cache-friendly because it pulls in nearby data you're about to use anyway; pointer-chasing (linked lists, trees with scattered nodes) is cache-*unfriendly* because each access can be a cache miss.
- **False sharing:** two threads modifying *different* variables that happen to sit on the *same* cache line force the cache-coherency protocol to keep invalidating and re-fetching that line between cores, even though the threads aren't logically touching each other's data. Fixed by padding/aligning shared data so hot variables used by different threads land on separate cache lines.
- **Branch prediction:** the CPU speculatively executes down the predicted path of a branch before it's confirmed. A misprediction flushes the speculative work — costly (tens of cycles). Reduce it with predictable branch patterns, or rewrite hot branches as branchless arithmetic (e.g., `result = (a > b) * a + (a <= b) * b` instead of an if/else) when the branch is genuinely unpredictable.
- **Heap allocation cost:** `malloc`/`new` involve bookkeeping (and sometimes syscalls) that's expensive relative to a single arithmetic operation. Hot loops should avoid allocating at all — use stack allocation, `reserve()` on containers ahead of time, or pre-allocated memory pools/arenas.
- **SIMD (Single Instruction, Multiple Data):** modern CPUs can apply one instruction to multiple data elements at once (e.g., AVX processes several floats in one instruction). Compilers can auto-vectorize simple, predictable loops — this is one reason simple array loops often outperform "clever" code with branches or indirection.
- **Lock-free structures:** use atomic CPU instructions (like compare-and-swap) instead of locks, avoiding the overhead of blocking/context-switching. Harder to get correct (memory-ordering bugs are notoriously subtle), but common in low-latency trading systems where a thread blocking on a lock is unacceptable.
- **Profiling before optimizing:** tools like `perf` on Linux let you measure where cycles are actually going instead of guessing. The rule is: measure, then optimize the actual bottleneck — intuition about what's "slow" is wrong more often than people expect.
- **Correct benchmarking:** warm up first (cold caches/branch predictors skew early runs), prevent the compiler from optimizing away your benchmark entirely (it can prove a loop with no observable side effect is dead code and delete it — use something like `volatile` or a "do not optimize" sink), run multiple trials and look at the distribution (not just the mean — tail latency matters), and watch for noisy-neighbor effects from other processes or CPU frequency scaling.
- **Latency vs. throughput:** batching improves throughput (more total work per second) but adds latency to any individual item waiting in the batch. Low-latency trading systems typically care about *tail* latency (worst case, e.g., p99.9) far more than average throughput — one slow trade can be worse than the system being slightly less efficient on average.

### Q&A

**Q: What is a cache line, and why does false sharing hurt multithreaded performance even when threads touch logically unrelated data?**
A: A cache line (usually 64 bytes) is the minimum unit of data moved between memory and cache. If two threads' *different* variables happen to sit on the same line, every write by one thread invalidates the other core's cached copy of that whole line, forcing a re-fetch — so you pay cache-coherency traffic for data you never actually shared logically.

**Q: Why is heap allocation expensive in a hot loop, and what's a common fix?**
A: `malloc`/`new` carry bookkeeping overhead (finding free blocks, updating allocator metadata, potential locking in a multithreaded allocator) that dwarfs a simple arithmetic op. Fix: pre-allocate once outside the loop (`reserve()`, a memory pool/arena), and reuse that memory instead of allocating fresh each iteration.

**Q: What does a branch misprediction cost, and how do you write code that avoids it?**
A: The CPU speculatively executes the predicted branch path; a misprediction means flushing that speculative work and restarting down the correct path — tens of cycles wasted. For genuinely unpredictable branches in hot code, rewriting the logic as branchless arithmetic (using multiplication/masking instead of if/else) can avoid the misprediction entirely, at the cost of doing both branches' work unconditionally — a tradeoff you profile, not assume.

**Q: Can you optimize for latency and throughput at the same time, or is it always a tradeoff?**
A: Often a tradeoff: batching and pipelining improve throughput but add per-item latency (waiting for the batch to fill); processing one item at a time immediately minimizes its latency but may waste throughput on per-item overhead. Some optimizations (better cache locality, removing unnecessary allocations) genuinely help both — the tradeoff isn't universal, but assuming there's no tension is naive.

**Q: What's a lock-free data structure, and why might it be preferred in a trading system?**
A: One that uses atomic hardware instructions (like compare-and-swap) to coordinate access instead of a lock, so no thread ever blocks waiting for another to release a mutex. Preferred in latency-sensitive systems because a thread blocking on a lock introduces unpredictable delay (and potential priority inversion) — at the cost of the code being significantly harder to write and verify correct.

**Q: What mistakes make a microbenchmark lie to you?**
A: Not warming up first (measuring cold-cache/cold-branch-predictor effects), letting the compiler optimize away the code under test because it has no observable effect, measuring only the mean instead of the distribution (hiding tail latency), and not controlling for other load on the machine or CPU frequency scaling during the run.

### Task
Take the Black-Scholes pricer from Section 2. Write a benchmark harness that calls it a large number of times (e.g., 1,000,000 calls with randomized inputs) using `time.perf_counter()` in a loop (with a warm-up phase you discard), and report the median and p99 per-call latency. Then optimize it — avoid repeated `math.log`/`sqrt` calls where the same value is reused, precompute constants outside the hot path — and report before/after numbers. If you want to push further: port just the core pricing loop to C++ (which you already have from Section 6) and compare Python vs. C++ per-call latency directly — this is one of the most concrete "I understand low-latency tradeoffs" stories you can walk into an interview with.

---

## 8. Compilers (Niche CS)

**Why this matters:** the narrowest item on this list, usually a "preferred" rather than required qualification. Treat this as a differentiator, not a gate: enough to hold a conversation and show you understand what's happening under the hood, not deep expertise.

### Concept notes
- **Compilation pipeline:** source code → **lexer** (breaks text into tokens: keywords, identifiers, numbers, operators) → **parser** (checks the tokens follow the language's grammar, builds an **AST**) → **semantic analysis** (type checking, scope resolution — does this program actually make sense, not just parse) → **intermediate representation (IR)** (a lower-level, machine-independent form the optimizer works on) → **optimization passes** → **code generation** (emits actual machine code for the target architecture).
- **Compiler vs. interpreter vs. JIT:** a compiler (g++, clang) translates the whole program ahead of time into machine code before it ever runs. An interpreter (pure Python's CPython, mostly) executes source (or bytecode) line by line at runtime. A JIT (V8 for JavaScript, the JVM's HotSpot, PyPy) starts by interpreting, then compiles the "hot" code paths (the ones actually running a lot) to machine code at runtime — most languages people casually call "interpreted" today actually use some JIT layer.
- **AST (Abstract Syntax Tree):** a tree representation of the program's structure with syntactic noise (parentheses, semicolons) stripped out — e.g., `3 + 4 * 2` becomes a tree with `+` as the root, `3` as one child, and a `*` subtree (`4`, `2`) as the other, which directly encodes operator precedence. This structure is what makes subsequent analysis and codegen tractable.
- **Common optimizations:**
  - *Constant folding* — compute constant expressions (`2 + 3`) at compile time instead of at runtime.
  - *Dead code elimination* — remove code that's unreachable or whose result is never used.
  - *Inlining* — replace a function call with the function's body directly, avoiding call/return overhead (at the cost of larger code size).
  - *Loop unrolling* — replicate a loop body multiple times to reduce the overhead of the loop-control instructions relative to the actual work.
- **Static vs. dynamic typing:** static typing (C++, Java) checks types at compile time — catches a class of bugs before the program ever runs, and gives the compiler more information to optimize aggressively. Dynamic typing (Python) checks types at runtime — more flexible and faster to write, but type errors only surface when that code path actually executes.
- **Why this matters practically, even outside compiler-writing jobs:** understanding what the compiler will and won't do for you (does it auto-vectorize this loop? will it inline this function? does `-O2` actually change this benchmark?) directly informs how you write genuinely fast C++, and understanding the pipeline helps you reason about segfaults and undefined behavior by thinking about what code the compiler actually generated, not just what you wrote.

### Q&A

**Q: What are the main phases of a compiler pipeline?**
A: Lexing (source → tokens), parsing (tokens → AST, checking grammar), semantic analysis (type checking, scope resolution), IR generation, optimization passes on the IR, and code generation (IR → target machine code).

**Q: What's the difference between a compiler, an interpreter, and a JIT?**
A: A compiler translates the entire program to machine code ahead of time. An interpreter executes source/bytecode directly, line by line, at runtime. A JIT starts interpreting but compiles frequently-executed ("hot") code paths to native machine code on the fly, combining the interpreter's flexibility with near-compiled speed for the code that actually matters.

**Q: What is an AST, and why do you need one instead of working directly with the token stream?**
A: An Abstract Syntax Tree is a hierarchical representation of the program's structure, with precedence and grouping made explicit as tree structure instead of implicit in a flat sequence of tokens. It's what makes semantic analysis, optimization, and code generation tractable — you're recursing over a tree that already encodes "what belongs to what," rather than re-parsing precedence rules at every later stage.

**Q: Name two compiler optimizations and briefly explain each.**
A: Constant folding — evaluating constant expressions at compile time instead of generating code to compute them at runtime. Dead code elimination — removing code the compiler can prove has no observable effect on the program's output, so it's never emitted at all.

**Q: What's the core tradeoff between static and dynamic typing?**
A: Static typing catches a class of bugs at compile time and gives the compiler more information to optimize with, at the cost of more upfront ceremony writing the code. Dynamic typing is faster to write and more flexible (duck typing, easy generics-by-default), at the cost of type errors only surfacing when that exact code path runs, and less information available for the runtime to optimize with ahead of time.

**Q: Why would understanding compilers help you write faster C++ or debug it better?**
A: Knowing what the compiler will auto-vectorize, inline, or fold at compile time tells you which "clever" hand-optimizations are actually redundant (the compiler already does them) versus genuinely necessary. It also helps you reason about crashes and undefined behavior by thinking in terms of what machine code actually got generated from your source, rather than treating the compiler as an opaque black box.

### Task
Write a minimal expression evaluator in Python: tokenize an arithmetic string, build an AST via recursive descent respecting operator precedence, then evaluate it.

```python
import re

def tokenize(expr):
    return re.findall(r'\d+\.?\d*|[+\-*/()]', expr)

# Grammar (standard precedence, lowest to highest):
#   expr   -> term (('+'|'-') term)*
#   term   -> factor (('*'|'/') factor)*
#   factor -> NUMBER | '(' expr ')'
#
# TODO: implement parse_expr(tokens), parse_term(tokens), parse_factor(tokens)
# as a recursive-descent parser over the token list, evaluating directly
# as you parse (or building an explicit AST first, then a separate eval step
# -- try the AST version if you want the fuller "compiler" experience).

print(tokenize("3 + 4 * (2 - 1)"))  # should print ['3','+','4','*','(','2','-','1',')']
```

Implement the three functions following the grammar in the comments. Test against expressions where naive left-to-right evaluation (no precedence) would give the wrong answer — e.g., `"3 + 4 * 2"` should evaluate to 11, not 14 — to confirm your parser is actually respecting precedence via the AST structure, not just coincidentally getting simple cases right.

---

## Proficiency Checklist

| Skill | Concept notes read | Q&A answered cold | Task completed |
|---|---|---|---|
| Financial Markets Knowledge | ☐ | ☐ | ☐ |
| Quant Finance Concepts | ☐ | ☐ | ☐ |
| Linux | ☐ | ☐ | ☐ |
| Shell Scripting / Bash | ☐ | ☐ | ☐ |
| Computer Networks | ☐ | ☐ | ☐ |
| Distributed Systems / Concurrency | ☐ | ☐ | ☐ |
| Low-Latency / HPC Optimization | ☐ | ☐ | ☐ |
| Compilers | ☐ | ☐ | ☐ |

**Suggested order:** Sections 1–2 (Markets, Quant Finance) pair naturally together and can be tackled first. Sections 3–4 (Linux/Bash) are low-lift and can run in parallel with anything else. Sections 5–7 (Networks → Concurrency → HPC) are designed to be done in that sequence, since each task builds on the previous one's code. Compilers is the lowest-priority item here — fine to leave for last or do only a light pass if you're tight on time.
