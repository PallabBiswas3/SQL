# ESP32 HC-SR04 TinyML Anomaly Detector — Interview Question Bank (with Answers)

Organized by resume bullet. Answers are written the way you should say them out loud — concise, honest about the project's actual gaps, and specific to your code.

---

## Bullet 1: "Trained a Keras autoencoder (4-8-2-8-4) on 3K HC-SR04 samples, exporting weights as C arrays to eliminate TFLite runtime overhead"

**Q1. Why autoencoder over a supervised classifier?**
A: I didn't have labeled examples of every possible failure mode a sensor could see. An autoencoder only needs "normal" data — it learns to reconstruct the normal manifold, and anything that reconstructs poorly (high MSE) is flagged. That means it generalizes to failure types I never explicitly trained on, which a classifier can't do since it's bounded by its label set.

**Q2. What does the 2-node bottleneck represent? Why not 1 or 4?**
A: It's a compressed latent representation of the 4 input features — analogous to 2 learned principal components describing a "healthy bounce profile." 1 node would likely be too restrictive to separate the different anomaly types (too-close vs. too-far vs. jittery) in a useful way; 4 (no compression) wouldn't force the network to learn anything — it could just copy the input through and reconstruction error would be near-zero for everything, including anomalies. 2 was a reasonable middle ground for a 4-feature input, not derived from a rigorous sweep.

**Q3. Was your training data real or simulated?**
A: Honestly, simulated — `simulate_window_features()` in `train.py` generates Gaussian-distributed distance/jitter/rate-of-change/variance for a "normal" condition (30cm, low noise) and two anomaly conditions (too close, too far). I haven't yet validated against a long capture of real HC-SR04 noise characteristics on physical hardware. That's a real limitation I'd want to close before calling this production-ready — real ultrasonic sensors have quantization noise and multipath effects that a Gaussian model doesn't capture.

**Q4. Why 3000 normal vs. 400 anomaly samples?**
A: The imbalance is intentional and matches the deployment reality — anomalies are rare, so I don't want the autoencoder to "learn" anomalies as normal by training on them at all. The 400 anomaly samples are used only at evaluation time (to compute recall and pick a threshold), never for training — `X_train`/`X_val` are built exclusively from `normal_data`.

**Q5. Why min-max normalization over z-score?**
A: Min-max bounds every feature to [0,1], which pairs naturally with the sigmoid output activation on the decoder — sigmoid's output range is also [0,1], so the reconstruction target and output range match exactly. Z-score standardization would leave inputs roughly zero-centered and unbounded, which doesn't match a sigmoid output as cleanly.

**Q6. Why sigmoid on the output layer given normalized inputs?**
A: Same reason as above — since inputs are scaled to [0,1] by `normalize()`, sigmoid gives the decoder an output range that can actually match the input range. Using linear output would let the reconstruction overshoot outside [0,1], which doesn't correspond to any valid normalized input.

**Q7. Could ReLU zero out the bottleneck entirely (dead latent space)?**
A: Yes, that's a real risk with ReLU — if a bottleneck neuron's pre-activation goes negative for all inputs during training, it dies and stays at zero permanently ("dying ReLU"), effectively throwing away one of your two latent dimensions. I didn't specifically audit for this, but it's a legitimate thing to check by looking at `latent` activations across the validation set — if one column is always ~0, you've effectively got a 1-node bottleneck without realizing it.

**Q8. Why 4→8→2→8→4 specifically, expanding before compressing?**
A: The expansion to 8 gives the network more capacity to find useful nonlinear combinations of the 4 raw features before compressing — a monotonic 4→3→2 shrink gives it less room to separate patterns before the bottleneck forces a decision. This wasn't the result of an architecture search, though — it's a reasonable default I picked, and I'd defend it as "reasonable" rather than "optimal," since I didn't sweep alternatives.

**Q9. What does "eliminating TFLite runtime overhead" mean at the bytes level?**
A: TFLite Micro needs a tensor arena (a static memory block sized for intermediate activations), an interpreter that walks the model graph at runtime, and op resolvers that map op codes to kernel implementations — that's tens of KB of flash and runtime dispatch overhead even for a tiny model. By hand-writing the forward pass as fixed C arrays and a fixed sequence of `dense()` calls, there's no graph interpretation at all — just direct function calls, which the compiler can inline and optimize, landing at ~512 bytes total.

**Q10. Did you consider pruning or knowledge distillation instead?**
A: Not seriously — at this model size (118 total weights) there's nothing meaningful to prune, and distillation is solving a different problem (compressing a large model into a small one). The overhead here was never the model size, it was the *runtime* — which is why bypassing the interpreter mattered more than shrinking the model further.

**Curveball: "If TFLite Micro supports INT8 at a few KB, why not just use it?"**
A: It would work fine here — I want to be fair about that. The reasons I went hand-rolled C++ instead: deterministic execution time (no interpreter dispatch jitter run to run), truly zero heap allocation, and the smallest possible flash footprint. But at this model's tiny size, TFLite Micro's overhead wouldn't have been prohibitive — this was a deliberate demonstration of understanding what's inside the runtime, not a strict necessity.

**Curveball: "train.py generates a full INT8 .tflite model that inference.cpp never loads — why?"**
A: That's a leftover from an earlier iteration where I was going the TFLite Micro route before switching to hand-written C++. I kept the export code as a reference/fallback path rather than deleting it, but it's genuinely dead weight in the current pipeline — good catch, and something I'd clean up.

### Live coding — answers

**Dense layer forward pass:**
```cpp
void dense_layer(const float* input, const float* weights, const float* biases,
                  float* output, int input_dim, int output_dim) {
    for (int i = 0; i < output_dim; i++) {
        float sum = biases[i];
        for (int j = 0; j < input_dim; j++) {
            sum += input[j] * weights[i * input_dim + j];
        }
        output[i] = (sum > 0.0f) ? sum : 0.0f; // ReLU
    }
}
```
Note this indexes `weights[i * input_dim + j]` — output-major flattening. My actual `dense()` in `inference.cpp` indexes `W_flat[i * out_n + j]` (input-major, matching Keras's `(input_dim, output_dim)` kernel shape) — if asked to reconcile the two, the key point is: **the indexing scheme must match how the weights were flattened at export time**, and getting this backwards silently produces plausible-looking-but-wrong output rather than a crash. That's why I'd always sanity-check with a known input/output pair from Keras before trusting the C++ port.

**C export function (row-major flatten):**
```python
def arr2c_define(arr, name):
    flat = arr.flatten().tolist()  # row-major by default (C order)
    vals = ", ".join(f"{v:.6f}f" for v in flat)
    return f"#define {name} {{{vals}}}"
```
`arr.flatten()` on a NumPy array defaults to row-major (`'C'` order), which is why `W_flat[i * out_n + j]` in `dense()` correctly walks it — row `i`, column `j` — as long as the Keras kernel shape is `(input_dim, output_dim)`, which it is by default.

**Bug hunt (mismatched in_n/out_n):** If you call `dense(h1, 8, W_enc2_flat, B_enc2, latent, 2, ...)` but accidentally pass `in_n=2, out_n=8` (swapped), the loop bounds `j < in_n` and the indexing `W_flat[i * out_n + j]` both go wrong — you'd read past the end of a smaller weight array (undefined behavior, likely garbage or a crash on a platform with memory protection) or silently read the wrong subset of weights on a platform without bounds checking, like the ESP32. The fix is asserting `in_n`/`out_n` match the actual declared array dimensions, ideally via `static_assert` or at minimum a runtime bounds check during development.

---

## Bullet 2: "Engineered temporal features (distance, jitter, variance) from an 8-sample circular window on an ESP32 using ultrasonic timing"

**Q1. Why a circular buffer instead of std::vector or a shifting array?**
A: A shifting array means every new sample costs an O(N) memmove to shift everything down. A circular buffer makes insertion O(1) — just write to `window_[idx % WINDOW_SIZE]` and increment `idx`. `std::vector` adds heap allocation and potential reallocation, which I explicitly wanted to avoid for deterministic, zero-heap embedded behavior.

**Q2. Why exactly 8 samples?**
A: At 5Hz sampling (`SAMPLE_INTERVAL=200ms`), 8 samples covers about 1.6 seconds of history — long enough to characterize short-term jitter/vibration but short enough that the window fills almost instantly on boot and stays responsive to genuine step changes. A much larger window (say 1000) would smooth out real anomalies and take minutes to warm up; a much smaller window (say 2-3) wouldn't give a meaningful variance estimate.

**Q3. Walk through jitter and variance — why is variance ≈ jitter²?**
A: `jitter` is `window_std()` (standard deviation), and `variance` is `window_variance()` — so yes, variance is literally jitter squared by definition, computed independently rather than derived from each other in code. That does mean the two features are highly correlated and somewhat redundant as inputs to the autoencoder — I kept both because the raw squared term changes the "shape" of how large deviations are weighted (variance penalizes big jitter spikes more than std does), giving the model slightly different information, but I'd concede a more careful feature-selection pass might drop one.

**Q4. What's rate_of_change computing, and why divide by delta_sec?**
A: It's `|current_distance - previous_distance| / elapsed_seconds` — normalizing by actual elapsed time rather than assuming a fixed interval makes it robust to jitter in the sample loop timing itself (e.g., if `SAMPLE_INTERVAL` isn't hit exactly on time). Dividing by a fixed assumed interval would silently misreport rate if the loop timing drifted.

**Q5. What happens before the window is full?**
A: `window_count()` returns `window_idx_` (the actual number of samples written so far) until `window_full_` flips true, at which point it always returns `WINDOW_SIZE`. `window_mean()`/`window_variance()` correctly use this partial count rather than dividing by a fixed `WINDOW_SIZE`, so early readings are legitimate — just computed over fewer samples, meaning higher variance in the variance estimate itself during warm-up.

**Q6. Single-pass vs. two-pass variance — which do you use, and stability tradeoff?**
A: Two-pass: `window_mean()` computes the mean first, then `window_variance()` re-loops using that mean. At N=8 this costs nothing extra and two-pass is numerically more stable than the naive single-pass "sum of squares minus square of sum" formula, which can suffer catastrophic cancellation when the mean is large relative to the variance. Since distance values (~2–400cm) aren't huge, single-pass would probably be fine too, but two-pass was the safer default with negligible extra cost at this N.

**Q7. Explain Welford's algorithm and when you'd use it.**
A: Welford's algorithm updates a running mean and sum-of-squared-deviations incrementally, one sample at a time, in O(1) per update and O(1) memory — no need to store or re-loop over the raw samples at all. I'd reach for it if N were large enough that O(N) recomputation per sample became costly, or if I needed variance over an unbounded/streaming history rather than a fixed window. At N=8 it's unnecessary — brute force is simpler and cheap enough.

**Q8. Does correlation between jitter and variance hurt the autoencoder?**
A: Not fundamentally — autoencoders can handle correlated inputs fine, it just means the effective information content of the 4-feature vector is a bit less than 4 independent dimensions, so the "true" complexity the 2-node bottleneck needs to capture is somewhat lower than it looks. It's mostly a minor inefficiency, not a correctness bug.

**Curveball: "You divide by n, not n-1 — population vs. sample variance?"**
A: Correct catch — `window_variance()` divides by `n`, which is population variance, not the Bessel-corrected sample variance (`n-1`). I think of it as "the variance of the window I actually have," not an estimate of some larger population's variance, so population variance is the more literal interpretation — but I'll be honest, this wasn't a deliberate statistical choice, more a default I didn't question. At N=8 the difference between dividing by 8 vs. 7 is small but not negligible (~14%), and it's a fair thing to flag as worth revisiting.

**Curveball: "Discontinuity when window_full_ flips true?"**
A: There's no code-level discontinuity — `window_count()` already handles the partial-window case correctly, so variance/jitter transition smoothly as more real samples accumulate; there's no special-case jump the moment the buffer fills. The bigger practical concern is that variance estimates during warm-up (with n=1 or 2) are statistically noisy, not that there's a hard discontinuity — worth distinguishing those two claims if pressed.

### Live coding — answers

**Circular buffer with mean/variance:**
```cpp
class SensorBuffer {
private:
    static const int SIZE = 8;
    float buffer[SIZE] = {};
    int head = 0;
    int count = 0;
public:
    void add_sample(float distance) {
        buffer[head] = distance;
        head = (head + 1) % SIZE;
        if (count < SIZE) count++;
    }
    float mean() const {
        if (count == 0) return 0.0f;
        float sum = 0.0f;
        for (int i = 0; i < count; i++) sum += buffer[i];
        return sum / count;
    }
    float variance() const {
        if (count < 2) return 0.0f;
        float m = mean(), sum = 0.0f;
        for (int i = 0; i < count; i++) {
            float d = buffer[i] - m;
            sum += d * d;
        }
        return sum / count;
    }
};
```

**Welford's online algorithm:**
```cpp
struct Welford {
    int n = 0;
    float mean = 0.0f, m2 = 0.0f;
    void update(float x) {
        n++;
        float delta = x - mean;
        mean += delta / n;
        m2 += delta * (x - mean);
    }
    float variance() const { return n > 1 ? m2 / n : 0.0f; }
};
```

**Windowed (sliding) O(1) variance** — the harder version. Maintain running sum and sum-of-squares, subtracting the evicted sample when the ring buffer wraps:
```cpp
class SlidingStats {
    static const int SIZE = 8;
    float buffer[SIZE] = {};
    int head = 0, count = 0;
    float sum = 0.0f, sum_sq = 0.0f;
public:
    void add_sample(float x) {
        if (count == SIZE) {
            float evicted = buffer[head];
            sum -= evicted;
            sum_sq -= evicted * evicted;
        } else {
            count++;
        }
        buffer[head] = x;
        sum += x;
        sum_sq += x * x;
        head = (head + 1) % SIZE;
    }
    float mean() const { return count ? sum / count : 0.0f; }
    float variance() const {
        if (count < 2) return 0.0f;
        float m = mean();
        // E[x^2] - (E[x])^2, guarded against tiny negative float error
        float v = (sum_sq / count) - (m * m);
        return v > 0.0f ? v : 0.0f;
    }
};
```
Worth flagging live: this sum-of-squares approach can suffer the same cancellation issue as naive single-pass variance if values are large relative to their spread — acceptable here given distance magnitudes, but worth naming as the tradeoff versus the two-pass version already in `sensor.cpp`.

**Rate of change with first-sample edge case:**
```cpp
float compute_rate_of_change(float dist, uint32_t ts_ms,
                              float& prev_distance, uint32_t& prev_timestamp) {
    float roc = 0.0f;
    if (prev_distance >= 0.0f && ts_ms > prev_timestamp) {
        float delta_cm  = fabsf(dist - prev_distance);
        float delta_sec = (ts_ms - prev_timestamp) / 1000.0f;
        roc = delta_cm / delta_sec;
    }
    prev_distance = dist;
    prev_timestamp = ts_ms;
    return roc;
}
```

---

## Bullet 3: "Developed pure C++ forward pass with 96 MAC ops; achieved 19 microseconds inference using 512 bytes of flash and zero heap usage"

**Q1. Derive the 96 MACs live.**
A: 4→8 is 4×8=32 MACs, 8→2 is 8×2=16, 2→8 is 2×8=16, 8→4 is 8×4=32. Total: 32+16+16+32 = 96.

**Q2. How did you measure 19µs?**
A: `micros()` before the encoder call and after the decoder call in `run()`, subtracting the two — `micros()` on ESP32 has microsecond resolution backed by a hardware timer, so it's a legitimate measurement, though there's some small fixed call overhead to `micros()` itself that isn't subtracted out.

**Q3. Does 19µs include normalization and MSE?**
A: No — looking at my own code, `t0 = micros()` is set right after the normalization block and `result.inference_us = micros() - t0` is read right after the decoder's last `dense()` call, before `compute_mse()` runs. So the 19µs figure is specifically the 4 dense-layer forward pass, not normalization or MSE computation.

**Q4. How did you verify 512 bytes flash / zero heap?**
A: Honestly, 512 bytes is closer to a computed estimate from the weight array sizes than a value read off a linker map — 118 total floats × 4 bytes ≈ 472 bytes, plus small overhead, rounds to "about 512." I haven't pulled the actual `.map` file from a PlatformIO build to confirm the linked `.rodata` size precisely, which I'd want to do before stating it as a hard measured number rather than an estimate.

**Q5. Where do the static const float arrays live — flash or RAM?**
A: `static const float W_enc1[4][8] = ...` at file scope with `const` should be placed in flash (`.rodata`/`.rodata.data` depending on toolchain), not copied into RAM — `static` here gives it internal linkage (each translation unit gets its own instance, not exported), and `const` is what allows the linker to place it in read-only flash instead of RAM.

**Q6. Are h1, latent, h2 heap or stack allocated?**
A: They're declared as local arrays inside `run()` (`float h1[8], latent[2];` etc.) — that's stack allocation, not heap. No `new`, `malloc`, or STL containers appear anywhere in the forward pass, which is what backs the zero-heap claim for this specific function.

**Q7. FLOPs and effective FLOPS/sec?**
A: Each MAC is one multiply + one add = 2 FLOPs, so 96 MACs = 192 FLOPs. At 19µs, that's 192 / 19e-6 ≈ 10.1 million FLOPS (~10 MFLOPS) — a tiny number in absolute terms, but the point isn't throughput, it's that a 240MHz single core can do this specific tiny workload with no measurable latency impact on the rest of the loop.

**Q8. Why float32 instead of the INT8 model train.py already builds?**
A: Simplicity and precision headroom — float32 avoids needing a quantization-aware inference path (scale/zero-point bookkeeping) in the C++ code, and at this model's tiny size, the flash savings from INT8 weren't the bottleneck; determinism and avoiding an interpreter were. It's a legitimate follow-up question whether INT8 would meaningfully shrink the 512 bytes further — it would, roughly 4x on the weight arrays — but I prioritized simplicity of the hand-written forward pass over squeezing that further.

**Curveball: "Is printf really heap-free?"**
A: Not something I verified rigorously — `Serial.printf`'s underlying implementation can vary by platform/libc, and some printf implementations do allocate internally for formatting, especially for floats. The "zero heap" claim I can defend confidently is specifically about the `run()` inference path itself (which touches no printf), not the surrounding logging code in `main.cpp`/`sensor.cpp` — worth being precise about which part of the system the claim applies to.

**Curveball: "Did you verify row-major indexing matches Keras's kernel shape?"**
A: Not with an automated byte-for-byte test bench, no — I verified it logically (Keras Dense kernel shape is `(input_dim, output_dim)`, NumPy's default `flatten()` is row-major, and my `dense()` indexes `W_flat[i*out_n+j]` which walks that layout correctly) but I didn't build a script that feeds the same input vector through both Keras and the compiled C++ and diffs the output. That's the honest gap, and the right answer if pushed: I'd build that test bench before trusting this in a real deployment.

### Live coding — answers

**calculate_mse:**
```cpp
float calculate_mse(const float* original_input, const float* reconstructed_output, int dim) {
    float sum_sq_error = 0.0f;
    for (int i = 0; i < dim; i++) {
        float error = original_input[i] - reconstructed_output[i];
        sum_sq_error += error * error;
    }
    return sum_sq_error / dim;
}
```

**Transposed matmul bug hunt:** given
```cpp
// BUGGY: indexes as if weights were (output_dim, input_dim) row-major
sum += in[i] * W_flat[j * in_n + i];   // should be in[i] * W_flat[i*out_n+j]
```
The fix is matching the index formula to how the array was actually flattened at export time. This produces no crash — just silently wrong numbers, because both indexing schemes are in-bounds, just reading different weight values than intended. The way to catch it in practice: feed a known input through both the Keras model and the C++ port and diff the outputs, since visually the two indexing formulas look "reasonable" and won't be caught by a code reviewer skimming the loop.

**Cycle-count instrumentation:**
```cpp
uint32_t c0 = ESP.getCycleCount();
// ... forward pass ...
uint32_t cycles = ESP.getCycleCount() - c0;
float time_us = (float)cycles / (ESP.getCpuFreqMHz());  // cycles / MHz = microseconds
```

**Flash byte-count math:** weight/bias element counts: WEIGHTS_ENC1=32, BIAS_ENC1=8, WEIGHTS_ENC2=16, BIAS_ENC2=2, WEIGHTS_DEC1=16, BIAS_DEC1=8, WEIGHTS_DEC2=32, BIAS_DEC2=4. Total = 32+8+16+2+16+8+32+4 = 118 floats × 4 bytes = 472 bytes. That's close to but not exactly 512 — the extra ~40 bytes is reasonable to attribute to struct/array padding, alignment, or rounding in how the estimate was originally stated; worth saying "approximately 512" rather than defending it as an exact figure if pressed on the arithmetic.

---

## Bullet 4: "Deployed unsupervised anomaly detection using MSE reconstruction, flagging mechanical jitter in real-time at >2 std validation threshold"

**Q1. How was the threshold computed?**
A: `threshold = mean(val_err) + 2*std(val_err)` where `val_err` is the reconstruction MSE over the held-out validation split of *normal* data only. This is a standard statistical approach — assuming validation MSE is roughly normally distributed, mean+2σ covers about 95% of normal-condition reconstruction error, so readings beyond that are flagged as unusual.

**Q2. Why 2 std, not 1.5 or 3?**
A: 2σ is a common default balancing sensitivity and false-positive rate — smaller (like 1.5σ) would catch anomalies faster but with more false alarms on normal noise; larger (3σ) would be more conservative but might miss subtler anomalies. I didn't run a sweep to optimize this against a labeled cost function (e.g., cost of a missed anomaly vs. cost of a false alarm) — it's a reasonable statistical default, not a tuned hyperparameter, and I'd be upfront about that if asked whether it was optimized.

**Q3. Validation split or training split?**
A: Validation split (`X_val`, the held-out 20% of normal data) — using the training split itself would risk the threshold being biased toward the exact data the model was fit to, understating real-world reconstruction error and making the threshold too tight.

**Q4. "Real-time" relative to what? Where does the rest of the 200ms go?**
A: Sample rate is 5Hz (`SAMPLE_INTERVAL=200ms`), and inference itself is 19µs — so inference consumes roughly 0.01% of each sample period. Echo capture no longer blocks the CPU either — it's ISR-driven — so the rest of the 200ms is genuinely available for other work rather than spent parked inside `pulseIn()`. HC-SR04's own physical constraint still applies regardless of read strategy: you shouldn't trigger faster than about every 60ms, since a stale echo from the previous pulse can bleed into the next reading.

**Q5. What if the object is intentionally moved to a new stable position?**
A: The autoencoder was trained on "normal" meaning specifically ~30cm ± small noise — a deliberate move to 50cm would initially register as anomalous (high reconstruction error on the distance feature) even though nothing is actually wrong, and it would stay flagged indefinitely since there's no retraining or threshold adaptation happening online. That's a real limitation of a static, train-once deployment — it doesn't distinguish "new normal" from "genuine fault."

**Q6. Any online retraining or threshold updates?**
A: No — this is a static, train-once-and-flash deployment. The weights and threshold are compiled constants in `model_weights.h`/`model_data.h`. A more production-grade version might periodically recompute a rolling baseline or support an explicit "recalibrate" mode triggered after a known-good repositioning.

**Q7. Distinguishing sensor fault vs. genuine environmental anomaly?**
A: Both currently produce the same signal — high MSE, no differentiation. In practice you'd want a secondary check: a dead/disconnected sensor tends to produce a consistent timeout or a suspiciously constant reading with near-zero jitter, whereas a genuine environmental anomaly (object moved, vibration) tends to still show plausible physical variation. I don't currently implement that distinction — it's a legitimate "what would you add" answer, not something the current code does.

**Curveball: "Does the threshold generalize to a different physical environment?"**
A: Honestly, probably not without recalibration — it's fit to the specific simulated scenarios (30cm stable / 6cm close / 150cm far). A different install (different mounting height, different surface reflectivity, different ambient noise) would likely need its own threshold calibration pass rather than reusing this constant as-is.

**Curveball: "How would you turn is_anomaly into a confidence score?"**
A: Instead of a hard threshold cutoff, report the reconstruction MSE itself (or a normalized version of it, like `(mse - mean_normal) / std_normal`, a z-score-like quantity) as a continuous severity signal, and let the downstream consumer decide its own cutoff or trend on it over time. That's more useful for something like a dashboard, where you'd rather see "creeping toward abnormal" than a flat binary flag.

### Live coding — answers

**Threshold computation from an array of MSE values:**
```cpp
float compute_threshold(const float* val_err, int n, float k = 2.0f) {
    float sum = 0.0f;
    for (int i = 0; i < n; i++) sum += val_err[i];
    float mean = sum / n;

    float sq_sum = 0.0f;
    for (int i = 0; i < n; i++) {
        float d = val_err[i] - mean;
        sq_sum += d * d;
    }
    float std_dev = sqrtf(sq_sum / n);
    return mean + k * std_dev;
}

int count_false_positives(const float* val_err, int n, float threshold) {
    int fp = 0;
    for (int i = 0; i < n; i++) if (val_err[i] > threshold) fp++;
    return fp;
}
```

**Hysteresis/debounce wrapper:**
```cpp
class AnomalyDebouncer {
    int consecutive_ = 0;
    static const int REQUIRED = 3;
public:
    // Feed each raw is_anomaly flag; returns the debounced/confirmed flag
    bool update(bool raw_is_anomaly) {
        consecutive_ = raw_is_anomaly ? consecutive_ + 1 : 0;
        return consecutive_ >= REQUIRED;
    }
};
```
Worth noting live: this trades detection latency for false-alarm suppression — REQUIRED=3 at 5Hz means a genuine anomaly takes at least 600ms to confirm, which is a real tradeoff to name explicitly rather than presenting debouncing as a free win.

---

## Cross-cutting / system-design questions

**Q1. Walk through the entire pipeline end to end — where is precision or information lost?**
A: Physical echo pulse (analog timing) → ISR-captured `micros()` timestamp on rise and fall edges (quantization to µs resolution; unsigned subtraction so a `micros()` overflow every ~71 minutes doesn't corrupt the reading) → distance in cm via a fixed speed-of-sound constant (0.01715, which ignores temperature/humidity effects on actual sound speed) → pushed into an 8-sample window (only the last 8 readings are ever considered — older history is fully discarded) → normalized via min/max fit on training data (any real-world reading outside that original min/max range gets clipped or extrapolated oddly) → float32 forward pass (Python/Keras trained in float64 by default, so there's a precision mismatch not formally verified) → MSE compared to a static threshold. Each stage is a legitimate place to lose fidelity, and I'd walk through this list if asked to be thorough.

**Q2. Supporting 10 HC-SR04 sensors on one ESP32?**
A: Each sensor needs its own trigger/echo pin pair and its own `SensorManager` instance (or the class refactored to be multi-instance-aware, since right now the ISR trampoline uses a single global instance pointer, which wouldn't scale to multiple sensors as written). Triggering all 10 simultaneously risks acoustic crosstalk (one sensor's echo triggering another's receiver), so you'd likely stagger triggers in time or use different sensors' known operating frequencies. Memory-wise, 10× the window buffers and 10× the ISR bookkeeping is still trivial on ESP32's RAM budget.

**Q3. What's missing to ship this as an actual product?**
A: Real hardware validation of the training data (not simulated), per-install threshold calibration, OTA firmware/model update capability, real INA219 power measurement instead of the current hardcoded 120mA estimate, watchdog/brownout recovery handling, and a way to distinguish sensor faults from genuine anomalies. I'd list these unprompted if asked "is this production ready" rather than waiting to be caught on them.

**Q4. How would you unit-test the C++ forward pass without hardware?**
A: Build a host-side (non-Arduino) target that mocks out `Serial`/`micros()`/pin functions, then feed the same input vectors through both the Keras model (Python) and the compiled C++ `dense()` chain, diffing outputs within a small float tolerance. This is exactly the test bench I flagged earlier as a gap I haven't built yet.

**Q5. How do you keep model_weights.h in sync with train.py, avoiding silent drift?**
A: Currently, nothing enforces this automatically — it's a manual copy step (`train.py` writes to `output/`, then you manually copy into `firmware/include/`), which is exactly the kind of process that silently goes stale. A better setup would hash the weight file or embed a version/training-timestamp constant that's checked at boot and logged, plus a CI step that fails the build if `model_weights.h` predates the latest `train.py` run.

---

## The ISR / non-blocking gap — full honest answer (updated)

If asked to open `sensor.cpp` live: the version I originally wrote used `pulseIn()`, which blocks the CPU for up to the timeout duration while waiting for the echo. I identified this as a real limitation and **implemented the fix** — an ISR (`attachInterrupt` on `CHANGE`) that timestamps the rising and falling edges of the ECHO pin asynchronously, paired with a state machine (`IDLE → WAITING → READY/TIMED_OUT`) driven from `poll()`, so `loop()` never stalls. This is what's actually in the code now, not just a plan — I'd present it as "here's the limitation I found in my own design, and here's the redesign I shipped to fix it."

**Q. Walk me through a bug you found in your own ISR redesign after building it.**
A: The `READY` case originally computed echo duration as `(fall >= rise) ? (fall - rise) : 0` — a guard meant to protect against `micros()` overflow. That guard was actually backwards: unsigned integer subtraction is modular, so `fall - rise` already computes the correct elapsed time even when `fall` numerically wraps below `rise` across an overflow. The guard was silently discarding a small number of otherwise-valid readings (roughly once every ~71 minutes of continuous uptime, whenever a reading happened to straddle the wraparound) by treating them as zero-duration/out-of-range. I removed the guard and now just use the raw unsigned subtraction, which is correct as-is.

**Q. Any other cleanup after the redesign?**
A: Two dead-code removals: a global `SensorManager*` instance pointer that was assigned in `begin()` but never actually read anywhere — the ISR only touches `static` class members, so no instance pointer was ever needed, it was a leftover from an earlier draft of the design. And a `RawReading` struct that was left over from the old `read_hcsr04()` function, which no longer exists now that echo capture happens in the ISR instead.

**Q. Is the ISR/state-machine design fully race-free now?**
A: No, and I'd say that directly rather than overclaim it. There's one narrow, low-probability window I've identified but not fixed: if a reading is marked `TIMED_OUT` right as the real echo is about to arrive, the ISR can still fire microseconds later — right around when the next cycle's `start_trigger()` resets `echo_ready_`. In that narrow window a stale echo could get misattributed to the new cycle. It's timing-dependent and rare, and I've chosen to document and accept it rather than add locking complexity for a fringe case — but I wouldn't claim the design has zero races if pushed on it.

**Q. Your earlier simulation-mode scenario cycling in `read()` — is that still there?**
A: No, I removed it as part of the ISR migration. `poll()` now always reads the real ECHO pin through the ISR — there's no built-in synthetic-data path anymore. If I needed to bench-test the model without hardware wired up, the honest answer is I'd add a temporary `#ifdef`-gated branch that injects a synthetic distance instead of reading `echo_rise_us_`/`echo_fall_us_`, or just feed pre-recorded CSV rows straight into `InferenceEngine::run()` and bypass `SensorManager` entirely — neither of which is currently built, so I'd frame it as "here's how I'd add it back" rather than implying it still exists.
