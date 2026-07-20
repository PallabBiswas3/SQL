# IMC Technical Interview — Deep Dive with Examples
*(Companion to IMC_Interview_QuickPrep.md — same structure, expanded with worked examples)*

---

## 1. OS Fundamentals

### 1.1 Process vs Thread — concrete example

```cpp
// Process: fork() — separate address space, child gets a COPY of memory
pid_t pid = fork();
if (pid == 0) { /* child: own copy of parent's variables, changes don't affect parent */ }

// Thread: pthread/std::thread — SHARED address space
int shared_counter = 0;
std::thread t([&](){ shared_counter++; }); // directly mutates parent's memory — needs synchronization
```
**Interview angle**: "why threads, not processes, for a matching engine?" → threads share memory, so passing a tick from feed-handler thread to strategy thread is just a pointer/reference, not a serialize-copy-deserialize across process boundary (which fork/IPC would require). That memory-copy cost is exactly what you're trying to eliminate at the microsecond scale.

### 1.2 Cache locality & false sharing — concrete example

```cpp
// BAD: two threads' counters sit on the same 64-byte cache line
struct Counters {
    std::atomic<long> buyCount;   // thread A writes this
    std::atomic<long> sellCount;  // thread B writes this
}; // both fields likely share one cache line → every write invalidates
   // the OTHER core's cached copy, forcing a re-fetch = "false sharing"

// FIXED: pad so each counter owns its own cache line
struct Counters {
    alignas(64) std::atomic<long> buyCount;
    alignas(64) std::atomic<long> sellCount;
};
```
**Why it matters**: this is a real, commonly-asked HFT interview question — two threads updating logically-unrelated variables can still contend if those variables share a cache line. Padding to 64 bytes (typical line size) fixes it.

### 1.3 Priority inversion — the classic story to know
Mars Pathfinder (1997): a low-priority task held a mutex needed by a high-priority task, but a medium-priority task kept preempting the low-priority one — so the high-priority task starved indefinitely even though it "outranked" everyone. Fixed with **priority inheritance**: the low-priority task temporarily inherits the high priority while holding the lock, so the medium-priority task can't cut in line. Good example to have ready if asked about scheduling + locking together.

### 1.4 Concurrency primitives — code you should be able to write cold

**Spinlock** (preferred over mutex in a pinned-core hot loop):
```cpp
class Spinlock {
    std::atomic_flag flag = ATOMIC_FLAG_INIT;
public:
    void lock()   { while (flag.test_and_set(std::memory_order_acquire)) {} }
    void unlock() { flag.clear(std::memory_order_release); }
};
```

**Mutex + condition variable producer/consumer** (standard interview whiteboard question):
```cpp
std::queue<Tick> q;
std::mutex m;
std::condition_variable cv;

void producer(Tick t) {
    { std::lock_guard<std::mutex> lk(m); q.push(t); }
    cv.notify_one();
}
void consumer() {
    std::unique_lock<std::mutex> lk(m);
    cv.wait(lk, [](){ return !q.empty(); });
    Tick t = q.front(); q.pop();
}
```

**CAS (compare-and-swap)** — the building block of lock-free structures:
```cpp
std::atomic<int> value{0};
int expected = value.load();
int desired = expected + 1;
while (!value.compare_exchange_weak(expected, desired)) {
    desired = expected + 1; // retry with the freshest 'expected'
}
```

### 1.5 Deadlock — example and the fix
```cpp
// Thread 1                      // Thread 2
lock(A);                         lock(B);
lock(B);  // waits on B          lock(A);  // waits on A  → DEADLOCK (circular wait)
```
**Fix — enforce a global lock ordering** (always acquire A before B, everywhere in the codebase), or use `std::lock(A, B)` which acquires both atomically without the circular-wait risk. If asked "how do you prevent deadlock," lock ordering is the answer they're fishing for 90% of the time.

### 1.6 Memory pool — why and a minimal example
```cpp
// Hot-path code avoids `new`/`malloc` because the allocator may take a lock,
// walk a free list, or trigger a page fault — all unpredictable latency.
class OrderPool {
    std::vector<Order> pool;
    std::stack<Order*> free_list;
public:
    OrderPool(size_t n) : pool(n) {
        for (auto& o : pool) free_list.push(&o);
    }
    Order* acquire() { Order* o = free_list.top(); free_list.pop(); return o; }
    void release(Order* o) { free_list.push(o); }
};
// All memory is pre-allocated once at startup; acquire/release are O(1), no syscalls.
```

---

## 2. Networking

### 2.1 TCP handshake + order-entry socket sketch
```
Client                     Server (exchange gateway)
  |----SYN-------------------->|
  |<---SYN-ACK------------------|
  |----ACK-------------------->|   connection established
  |----[ORDER: BUY 100 @ 50]-->|   reliable, in-order delivery
```
```cpp
int sock = socket(AF_INET, SOCK_STREAM, 0);   // TCP
int flag = 1;
setsockopt(sock, IPPROTO_TCP, TCP_NODELAY, &flag, sizeof(flag)); // disable Nagle
connect(sock, ...);
send(sock, orderBuffer, len, 0);
```
**Why `TCP_NODELAY` matters**: Nagle's algorithm buffers small packets to send fewer, larger ones — great for throughput, terrible for latency, since your single-order packet might sit waiting to be batched with the next one. Every low-latency trading system disables it.

### 2.2 UDP multicast — market data feed sketch
```cpp
int sock = socket(AF_INET, SOCK_DGRAM, 0);    // UDP
ip_mreq mreq;
mreq.imr_multiaddr.s_addr = inet_addr("239.1.1.1"); // exchange's multicast group
mreq.imr_interface.s_addr = INADDR_ANY;
setsockopt(sock, IPPROTO_IP, IP_ADD_MEMBERSHIP, &mreq, sizeof(mreq));
// now this socket receives every tick the exchange broadcasts to that group,
// same as every other subscribed firm — no per-client TCP connection needed
```
**Why UDP here**: the exchange has thousands of subscribers. TCP would mean thousands of individual reliable streams (huge overhead, and if one client is slow, TCP backpressure could theoretically slow the sender). UDP multicast broadcasts once; if you drop a packet, you just wait for the next tick rather than blocking everyone else for a retransmit.

### 2.3 A worked latency budget (good to have a mental model like this)
```
Exchange matching engine emits tick
   → NIC receives packet                      ~1-2 µs (co-located) to ~ms (far away)
   → kernel network stack parses it            ~1-5 µs   (kernel bypass avoids this)
   → your feed-handler thread processes it      ~100s ns  (if lock-free, cache-friendly)
   → strategy computes a decision                ~ns-µs   (depends on complexity)
   → order serialized + sent over TCP            ~1-2 µs  (with TCP_NODELAY)
   → back to exchange                            wire latency again
```
Being able to talk through *where* time goes in this pipeline — and which layer each of your OS/networking answers optimizes — is a strong signal in an IMC interview.

---

## 3. OOP

### 3.1 Polymorphism — Order hierarchy example
```cpp
class Order {
public:
    virtual void execute() = 0;   // vtable dispatch — one indirection per call
    virtual ~Order() = default;
};
class LimitOrder : public Order {
public:
    void execute() override { /* only fill at price or better */ }
};
class MarketOrder : public Order {
public:
    void execute() override { /* fill immediately at best available price */ }
};
// Order management code just calls order->execute() without caring which subtype it is
```
**Follow-up they may ask**: "what's the cost of that `virtual` call?" — one pointer indirection through the vtable, which also defeats some compiler inlining/branch prediction. In a hot loop processing millions of orders/sec, some shops avoid this via templates or CRTP instead (see 3.4).

### 3.2 Observer pattern — market data example
```cpp
class MarketDataSubscriber {
public:
    virtual void onTick(const Tick& t) = 0;
};
class MarketDataPublisher {
    std::vector<MarketDataSubscriber*> subscribers;
public:
    void subscribe(MarketDataSubscriber* s) { subscribers.push_back(s); }
    void publish(const Tick& t) {
        for (auto* s : subscribers) s->onTick(t);   // every strategy gets the update
    }
};
class MomentumStrategy : public MarketDataSubscriber {
    void onTick(const Tick& t) override { /* update signal, maybe fire an order */ }
};
```
This is literally how a feed handler fans a single incoming tick out to N independent strategies without them knowing about each other.

### 3.3 Strategy pattern — swappable trading logic
```cpp
class SignalStrategy {
public:
    virtual Signal compute(const MarketState&) = 0;
};
class MeanReversion : public SignalStrategy { Signal compute(const MarketState&) override; };
class Momentum      : public SignalStrategy { Signal compute(const MarketState&) override; };

class TradingEngine {
    std::unique_ptr<SignalStrategy> strategy;
public:
    void setStrategy(std::unique_ptr<SignalStrategy> s) { strategy = std::move(s); }
    void onTick(const MarketState& m) { auto sig = strategy->compute(m); /* act on sig */ }
};
// swap MeanReversion <-> Momentum at runtime without touching TradingEngine at all
```

### 3.4 CRTP — polymorphism without vtable cost (a "senior" answer if asked to optimize 3.1)
```cpp
template <typename Derived>
class OrderBase {
public:
    void execute() { static_cast<Derived*>(this)->executeImpl(); } // resolved at compile time
};
class LimitOrder : public OrderBase<LimitOrder> {
public:
    void executeImpl() { /* ... */ }
};
// no vtable, no runtime indirection — the compiler knows the exact type at compile time
```
Mentioning this unprompted after explaining virtual functions is a nice way to show you understand the latency trade-off, not just the OOP definition.

### 3.5 SOLID — one concrete violation + fix
```cpp
// Violates Single Responsibility: this class both computes signals AND sends orders
class BadStrategy {
    void computeAndTrade(const MarketState& m) {
        double signal = /* compute */;
        if (signal > threshold) sendOrder(/* ... */);  // mixing concerns
    }
};
// Fix: separate the signal computation from execution (as in 3.3) so each can be
// tested, replaced, or reused independently.
```

---

## 4. Correlation — putting the pieces into one pipeline

A realistic mental model to describe end-to-end if asked "design a simple trading system":

```
[Exchange] --UDP multicast--> [Feed Handler thread, kernel-bypass NIC, cache-friendly parsing]
                                        |
                         lock-free ring buffer (CAS-based, single-producer/single-consumer)
                                        |
                          [Strategy threads, pinned to cores, Observer-subscribed to ticks]
                                        |
                          decision → [Order Pool: pre-allocated Order objects, no malloc]
                                        |
                         [Order Gateway] --TCP + TCP_NODELAY--> [Exchange]
                                        |
                    [Risk Engine: pre-trade checks run inline before send — see Section 5]
```
Every box in that diagram maps directly back to a concept from Sections 1–3. If you can draw and narrate something like this in an interview, it demonstrates you're not just reciting definitions.

---

## 5. Compliance — worked examples

### 5.1 Pre-trade risk check (pseudocode)
```cpp
bool preTradeCheck(const Order& o, const Position& pos, const RiskLimits& limits) {
    if (o.quantity > limits.maxOrderSize) return false;         // fat-finger protection
    if (pos.netExposure() + o.notional() > limits.maxPosition) return false;
    if (std::abs(o.price - lastTradedPrice) / lastTradedPrice > limits.priceCollarPct)
        return false;                                            // reject wild price
    return true;
}
// This check runs INLINE, before the order leaves the firm — not after.
```

### 5.2 What spoofing/layering actually looks like (so you can explain it, not just name it)
A trader places a large **sell** order they never intend to execute, to make the book look heavy on the sell side and push other participants' algos to sell too — then cancels it and buys at the now-lower price. It's illegal because it creates a false impression of supply/demand. Detection typically looks at high cancel-to-fill ratios on large orders placed near the touch.

### 5.3 Algo-ID tagging (SEBI framework, India)
Every algo order gets a unique exchange-issued ID attached, e.g. conceptually:
```
Order { ..., algoId: "STRAT_7734", ... }
```
This lets the exchange trace any anomalous market event back to the exact strategy/code version that generated it — the audit-trail idea, made concrete.

### 5.4 Kill switch — a scenario to have ready
Your momentum strategy has a bug and starts sending orders in a runaway loop. A kill switch is a single control (often literally a monitored flag or dead-man's-switch heartbeat) that, when triggered — manually or automatically on anomaly detection (e.g. order rate exceeds N/sec, or P&L drawdown exceeds a threshold) — immediately halts all outbound order flow from that strategy/firm. Good to mention that this is usually implemented at multiple layers: in the strategy process itself, at the gateway, and sometimes enforced by the exchange/broker independently so a single point of failure can't disable it.

---

## Sample drill questions (talk through these out loud once tonight)
1. "Why would you use a spinlock instead of a mutex here?" → pinned core, avoid sleep/wake context switch cost.
2. "Walk me through what happens when a tick arrives and your strategy decides to trade." → use the Section 4 pipeline.
3. "What's wrong with this two-lock code?" → spot the circular wait, fix with lock ordering.
4. "Why UDP for market data but TCP for orders?" → reliability requirement differs; a missed tick self-heals, a missed order does not.
5. "How would you prevent a runaway algo from blowing up the firm?" → pre-trade checks + kill switch, layered.


# Project Interview Talking Points

---

## 1. FTIR Saliva Diagnostics (Research Internship)

**30-sec pitch**: Built a complex-valued OLS model for non-invasive diabetes detection from FTIR saliva spectra. Used a Hilbert transform to convert real spectra into complex signals, capturing phase information real-valued baselines throw away. On 1,040 samples, hit 91.8% accuracy / 95.4% recall, beating KNN/SVC/real-OLS. Fingerprint region gave the best signal (AUC 0.971). Stress-tested under 4 noise types and stayed >80% accurate.

**Likely follow-ups → answer angle**
- *Why complex OLS over real-valued models?* → Phase carries information amplitude alone doesn't — two spectra can have identical amplitude but different phase due to molecular absorption shifts. Hilbert transform reconstructs that phase from the real signal.
- *Why is the fingerprint region best?* → 500–1500 cm⁻¹ is where most biomolecular functional groups (glucose-related C-O, C-H bonds) show characteristic absorption — it's the most information-dense region for this task.
- *What does AUC 0.971 actually mean?* → 97.1% chance the model ranks a random diabetic sample's score above a random non-diabetic one's.
- *How did you validate robustness, and why does it matter?* → Injected Gaussian noise, Mie scattering, saturation, and ghost-peak artifacts synthetically — these are real failure modes in FTIR hardware, so a model that only works on clean lab spectra isn't clinically useful.
- *What's Complex SHAP doing here?* → Standard SHAP explains real-valued feature contributions; Complex SHAP extends attribution to complex features so you can say *which wavenumber region and phase component* drove a prediction — needed for a diagnostic tool.
- *What was your actual contribution vs. the professor's guidance?* → Be ready to state this precisely and honestly — e.g., "I designed and implemented the complex-OLS pipeline and noise-robustness experiments; the research direction was set jointly with my advisor." Don't overclaim, don't underclaim.

---

## 2. ARC-AGI Visual Reasoning (Tiny Recursive Model)

**30-sec pitch**: Built a ~367K-parameter recursive Transformer for ARC-AGI — orders of magnitude smaller than typical LLM-based approaches, which run in the billions. Used weight sharing across 3 outer/6 inner reasoning cycles (same weights applied repeatedly, like iterative refinement), 3D RoPE to encode row/column/pair spatial structure, and deep supervision (loss applied at every cycle, not just the end) to keep gradients flowing through the full reasoning trajectory. Solved 1/400 held-out tasks with 8-fold augmentation.

**Likely follow-ups → answer angle**
- *1/400 is very low — why present this?* → ARC-AGI is deliberately adversarial to pattern-matching; most sub-million-parameter models score near 0. The point of this project was testing whether recursive, parameter-efficient reasoning could get *any* traction where scale-based approaches dominate — architecture and efficiency were the objective, not leaderboard rank. Say this plainly if asked, don't get defensive.
- *Why weight sharing instead of stacking more layers?* → Forces the model to learn a genuinely iterative reasoning operator rather than memorizing a fixed-depth transformation — closer to how a human re-examines a grid.
- *What does 3D RoPE buy you over standard 2D positional encoding?* → ARC grids have row, column, *and* cross-example pair structure (input/output pairs); a third rotary axis lets the model distinguish "same position, different example" from "different position."
- *What's deep supervision doing mechanically?* → Cross-entropy loss applied after every recursive cycle, not just the final one, so early cycles get direct gradient signal instead of relying on it propagating back through all 6 steps.
- *What would you do differently / to improve accuracy?* → Have 1–2 honest answers ready — e.g., curriculum ordering of task difficulty, more augmentation variety, or scaling inner-cycle count with early stopping per task.

---

## 3. Credit Card Fraud Detection

**30-sec pitch**: Processed 285K transactions at 0.17% fraud rate — a severe imbalance problem. Engineered 38 features including graph-based degree centrality (via NetworkX) and velocity features (rolling stats over 10/50/100-event windows) to capture behavioral patterns fraud detection actually relies on. Trained XGBoost/LightGBM with SMOTE + StratifiedKFold, tuned thresholds for F2 (recall-weighted) since missing fraud is costlier than a false alarm. Got 0.29 PR-AUC / 0.96 ROC-AUC — a 180x improvement over random baseline.

**Likely follow-ups → answer angle**
- *ROC-AUC is 0.96 but PR-AUC is only 0.29 — is that a problem?* → Expected, not a red flag: with 0.17% positive rate, ROC-AUC is inflated because true negatives are trivially easy to get right. PR-AUC is the honest metric here since it only looks at precision/recall on the rare positive class — 0.29 vs. a random baseline near 0.0017 is actually a large improvement.
- *Why F2 instead of F1 or accuracy?* → In fraud, a missed fraud (false negative) is far costlier than a false alarm (false positive) — F2 weights recall 2x over precision, matching that business cost asymmetry.
- *What's "leak-free" imputation mean, and why does it matter?* → Imputing missing values using only information available *before* the transaction's timestamp — using future data (even from the same feature's global mean) would leak information the model wouldn't have at real inference time, inflating offline metrics falsely.
- *Why velocity/graph features specifically?* → Fraud rings show up as behavioral anomalies over time (sudden burst of transactions) and as connectivity patterns (shared devices/merchants) — raw transaction-level features miss both.
- *Why SMOTE and not just class weighting?* → Be ready to discuss the trade-off honestly — SMOTE generates synthetic minority examples which can help gradient boosting see more decision boundary detail, but risks synthetic artifacts; class weighting is simpler and sometimes just as effective. Good if you can say why you chose one over the other rather than just "it's standard."

---

## 4. ESP32 HC-SR04 TinyML Anomaly Detector

**30-sec pitch**: Trained a small Keras autoencoder (4-8-2-8-4) on ultrasonic sensor data, then hand-wrote the forward pass in pure C++ instead of using TFLite Micro — eliminating runtime/interpreter overhead entirely. Result: 19 microsecond inference, 512 bytes of flash, zero heap allocation. Engineered temporal features (distance, jitter, variance) from an 8-sample circular buffer, and flagged mechanical anomalies in real time using MSE reconstruction error against a >2 std threshold.

**Likely follow-ups → answer angle**
- *Why an autoencoder specifically, not a classifier?* → No labeled anomaly data exists in a real deployment — autoencoder learns to reconstruct *normal* patterns only; anomalies produce high reconstruction error simply because the model was never trained to compress them well. Fully unsupervised.
- *Why write raw C++ instead of using TFLite Micro?* → TFLite's interpreter carries overhead (op dispatch, tensor allocation) that's wasteful for a network this small; exporting weights as static C arrays and hand-writing the matrix multiplies means zero heap use and a fixed, predictable inference time — critical on a microcontroller with tight flash/RAM budgets.
- *What issues did you actually hit building this?* → Good honest details to have ready: fixing a blocking sensor read inside an ISR (interrupt service routine) — originally the ISR was doing work that could stall other interrupts, moved to a non-blocking flag-and-defer pattern; and a dying ReLU problem in the small network, fixed by switching to LeakyReLU so units don't permanently zero out during training.
- *How did you pick the >2 std threshold?* → Set from the validation set's reconstruction error distribution on known-normal data — 2 std balances false positive rate against catching genuine jitter.
- *96 MAC ops, 19 microseconds — how do you know that's fast enough?* → Frame it against the sensor's own sampling rate — if HC-SR04 gives readings every few ms, a 19µs inference is a tiny fraction of the budget, leaving headroom for other real-time tasks on the same MCU.

---

## 5. Delhivery ETA Optimization

**30-sec pitch**: Modeled 141K trip segments as a directed weighted graph (1,588 nodes) to find bottleneck hubs via network centrality. Engineered 69 features and built a two-tier stacking ensemble (XGBoost + LightGBM + RandomForest as base learners, meta-learner on top) to push ETA accuracy to 74.2% and cut MAE to 27.9 minutes. Flagged the Gurgaon hub as high-risk, translating to an estimated ₹149 Lakhs of revenue-at-risk mitigated and a 5.5pp reduction in SLA breaches.

**Likely follow-ups → answer angle**
- *What is "network centrality" doing here, concretely?* → Identifies nodes (hubs) that sit on a disproportionate number of shortest paths between other nodes — a bottleneck hub isn't just busy, it's structurally hard to route around, so delays there cascade further than delays at a peripheral node.
- *Why two-tier stacking instead of a single model?* → Different base learners (XGB, LGB, RF) capture different error patterns; a meta-learner trained on their outputs can correct for each one's blind spots — typically buys a few points of accuracy over any single model, at the cost of more complexity/inference time.
- *How did you translate MAE into a rupee figure?* → Be ready to explain the actual chain of reasoning you used — e.g., linking predicted SLA breaches to contractual penalty/refund costs per shipment, then aggregating over affected volume at the flagged hub. If this was an estimate/assumption-based calculation (common in case-study style projects), say so plainly rather than presenting it as measured fact.
- *Why did the Gurgaon hub specifically stand out?* → Have the actual reason ready — e.g., highest betweenness centrality combined with high volume throughput, meaning both structurally critical *and* heavily loaded.
- *74.2% ETA accuracy — accuracy relative to what threshold?* → Clarify whether this means "within X minutes of actual" or a classification-style bucket accuracy — interviewers will push on this if the number sounds vague.
