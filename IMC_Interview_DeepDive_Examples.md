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
