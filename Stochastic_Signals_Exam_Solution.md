# Stochastic Signals and Systems — Final Exam Solution (July 202x)

This is a full, step-by-step ("baby steps") solution to all three tasks. Wherever a formula is used, it's stated first, then applied.

---

# Task #1 — Random Process and Moments (40 points)

## Step 0: Understand the random variable a(ζ)

The density is written a bit confusingly, but it says:

$$f_a(a) = \frac{1}{3}\delta(a) + \frac{2}{3}\delta(a+1)$$

A delta at `a = 0` with weight 1/3, and a delta at `a = -1` with weight 2/3. This just means **a(ζ) is a discrete random variable with two outcomes**:

| Outcome | Value | Probability |
|---|---|---|
| Event 1 | a = 0  | 1/3 |
| Event 2 | a = -1 | 2/3 |

That's it — it's a simple two-valued (Bernoulli-type) random variable. Everything else follows from this.

We will need these building blocks over and over, so compute them once:

- E[a] = (0)(1/3) + (-1)(2/3) = **-2/3**
- E[a²] = (0)²(1/3) + (-1)²(2/3) = **2/3**
- Var(a) = E[a²] - E[a]² = 2/3 - 4/9 = **2/9**
- E[1+a] = 1 + E[a] = 1 - 2/3 = **1/3**
- E[(1+a)²]: when a=0 → (1)²=1 (prob 1/3); when a=-1 → (0)²=0 (prob 2/3) → **E[(1+a)²] = 1/3**
- E[a(1+a)]: when a=0 → 0×1=0; when a=-1 → (-1)×0=0 → **E[a(1+a)] = 0** (always zero, in *every* outcome, not just on average — this is the key simplification for the whole problem)

---

## a) Pattern functions [6]

A "pattern function" (Musterfunktion) is just x(t) for one fixed outcome of ζ. Since a(ζ) only takes 2 values, there are exactly **2 distinct pattern functions**.

**Pattern 1 — outcome a = 0 (probability 1/3):**

Substitute a = 0 into both branches:
- t < 0: x₁(t) = (1+0)·sin(2πt/T) = sin(2πt/T)
- t ≥ 0: x₁(t) = 0·(t/T) + (1+0)·sin(2πt/T) = sin(2πt/T)

Both branches give the *same* expression → **x₁(t) = sin(2πt/T) for all t.** A clean, continuous sine wave running through all time, amplitude 1, period T.

**Pattern 2 — outcome a = -1 (probability 2/3):**

Substitute a = -1:
- t < 0: x₂(t) = (1-1)·sin(2πt/T) = 0
- t ≥ 0: x₂(t) = (-1)·(t/T) + (1-1)·sin(2πt/T) = -t/T

**x₂(t) = 0 for t<0, and x₂(t) = -t/T for t≥0.** This is flat at zero for negative time, then turns into a downward-sloping ramp starting at the origin.

**Sketch description:**
- Pattern 1: an ordinary sine wave, unbroken, oscillating between +1 and -1, for all t (−∞ to +∞).
- Pattern 2: a horizontal line at 0 for t<0, with a kink at t=0 where it turns into a straight line of slope −1/T going down to −∞ as t→∞.

---

## b) Mean m_x(t) [8]

$$m_x(t) = E[x(t)]$$

Because the mean is linear, we don't need to re-derive anything — just take expectation of each branch using the building blocks from Step 0.

**For t < 0:**
$$m_x(t) = E[(1+a)]\sin\left(2\pi \frac{t}{T}\right) = \frac{1}{3}\sin\left(2\pi \frac{t}{T}\right)$$

**For t ≥ 0:**
$$m_x(t) = E[a]\frac{t}{T} + E[1+a]\sin\left(2\pi \frac{t}{T}\right) = -\frac{2}{3}\frac{t}{T} + \frac{1}{3}\sin\left(2\pi \frac{t}{T}\right)$$

**Sanity check** using the pattern functions directly: m_x(t) = (1/3)x₁(t) + (2/3)x₂(t). For t≥0: (1/3)sin(2πt/T) + (2/3)(-t/T) — same result. ✓

---

## c) Autocorrelation s_xx(t₁,t₂) [14]

$$s_{xx}(t_1,t_2) = E[x(t_1)x(t_2)]$$

We must consider t₁, t₂ each being either <0 or ≥0. There are 4 combinations, but 2 of them (mixed sign) turn out identical by symmetry, so really 3 distinct results.

**Case A: both t₁<0 and t₂<0**

$$x(t_1)x(t_2) = (1+a)^2 \sin\left(2\pi\frac{t_1}{T}\right)\sin\left(2\pi\frac{t_2}{T}\right)$$

Taking expectation, only E[(1+a)²] = 1/3 is needed:

$$s_{xx}(t_1,t_2) = \frac{1}{3}\sin\left(2\pi\frac{t_1}{T}\right)\sin\left(2\pi\frac{t_2}{T}\right)$$

**Case B: both t₁≥0 and t₂≥0**

Multiply out the two branches:
$$x(t_1)x(t_2) = a^2\frac{t_1 t_2}{T^2} + a(1+a)\frac{t_1}{T}\sin\left(2\pi\frac{t_2}{T}\right) + a(1+a)\frac{t_2}{T}\sin\left(2\pi\frac{t_1}{T}\right) + (1+a)^2\sin\left(2\pi\frac{t_1}{T}\right)\sin\left(2\pi\frac{t_2}{T}\right)$$

Now take expectation term by term. Recall from Step 0 that **E[a(1+a)] = 0** — this instantly kills the two middle cross-terms. We're left with:

$$s_{xx}(t_1,t_2) = E[a^2]\frac{t_1 t_2}{T^2} + E[(1+a)^2]\sin\left(2\pi\frac{t_1}{T}\right)\sin\left(2\pi\frac{t_2}{T}\right)$$

$$\boxed{s_{xx}(t_1,t_2) = \frac{2}{3}\frac{t_1 t_2}{T^2} + \frac{1}{3}\sin\left(2\pi\frac{t_1}{T}\right)\sin\left(2\pi\frac{t_2}{T}\right)} \quad (t_1,t_2\geq 0)$$

**Case C: mixed sign (t₁<0, t₂≥0 or vice versa)**

Say t₁<0, t₂≥0:
$$x(t_1)x(t_2) = (1+a)a\frac{t_2}{T}\sin\left(2\pi\frac{t_1}{T}\right) + (1+a)^2\sin\left(2\pi\frac{t_1}{T}\right)\sin\left(2\pi\frac{t_2}{T}\right)$$

Again E[a(1+a)]=0 kills the first term, leaving:

$$s_{xx}(t_1,t_2) = \frac{1}{3}\sin\left(2\pi\frac{t_1}{T}\right)\sin\left(2\pi\frac{t_2}{T}\right)$$

(same value if the roles of t₁,t₂ swap — symmetric, as it must be, since s_xx(t₁,t₂)=s_xx(t₂,t₁) always.)

**Compact single formula covering all four cases** (using the unit step u(t), which is 1 for t≥0 and 0 for t<0):

$$s_{xx}(t_1,t_2) = \frac{1}{3}\sin\left(2\pi\frac{t_1}{T}\right)\sin\left(2\pi\frac{t_2}{T}\right) + \frac{2}{3}\frac{t_1 t_2}{T^2}\,u(t_1)\,u(t_2)$$

The ramp-product term only survives when *both* times are ≥ 0.

---

## d) Variance σ_x²(t) [10]

$$\sigma_x^2(t) = s_{xx}(t,t) - m_x(t)^2$$

**Shortcut (recommended):** notice that in both branches, x(t) is a *linear* function of the single random variable a, of the form x(t) = a·g(t) + h(t) for some deterministic functions g(t), h(t). For a linear transformation, Var(x) = g(t)²·Var(a), and we already know **Var(a) = 2/9**. This avoids repeating the whole subtraction.

- For t<0: x(t) = (1+a)sin(2πt/T) = a·sin(2πt/T) + sin(2πt/T) → g(t) = sin(2πt/T)
$$\sigma_x^2(t) = \frac{2}{9}\sin^2\left(2\pi\frac{t}{T}\right), \quad t<0$$

- For t≥0: x(t) = a(t/T) + (1+a)sin(2πt/T) = a·[t/T + sin(2πt/T)] + sin(2πt/T) → g(t) = t/T + sin(2πt/T)
$$\sigma_x^2(t) = \frac{2}{9}\left[\frac{t}{T} + \sin\left(2\pi\frac{t}{T}\right)\right]^2, \quad t\geq 0$$

**Verification the long way** (subtracting s_xx(t,t) − m_x(t)²) gives exactly the same two expressions — the shortcut is just faster and less error-prone.

---

## e) Is the process stationary? [2]

**No.** Two independent reasons, either one is sufficient:

1. The mean m_x(t) is **not constant** — it explicitly depends on t (it even has a different functional form for t<0 vs t≥0).
2. The autocorrelation s_xx(t₁,t₂) depends on the **individual** values t₁ and t₂ (through the product t₁t₂ and through u(t₁)u(t₂)), not only on the time difference τ = t₁−t₂.

Either condition alone violates the requirements for (wide-sense) stationarity, so the process is **non-stationary**.

---

# Task #2 — Kalman Filter (40 points)

## The idea in plain words before the code

We build a noisy signal = sine wave + constant offset (8) + random noise. The Kalman filter's job is to **recover the constant (8)** from this noisy, oscillating signal — pretending the constant is a hidden "state" that doesn't change over time, and everything else (the sine wave riding on top plus the noise) is treated as measurement disturbance around that constant.

State-space model used by the filter:
- **State equation:** x_k = x_{k-1} + w_k  (the true constant barely moves — process noise w_k is tiny/zero)
- **Measurement equation:** z_k = x_k + v_k  (what we observe is the constant plus disturbance v_k)

Standard scalar Kalman recursion, for each time step k:
1. **Predict:** x̂⁻ₖ = x̂ₖ₋₁,  P⁻ₖ = Pₖ₋₁ + Q
2. **Compute gain:** Kₖ = P⁻ₖ / (P⁻ₖ + R)
3. **Update (correct with measurement):** x̂ₖ = x̂⁻ₖ + Kₖ(zₖ − x̂⁻ₖ),  Pₖ = (1−Kₖ)P⁻ₖ

Where Q = process noise variance (how much we believe the constant can drift — set very small, since it's truly constant), R = measurement noise variance (how noisy/uncertain our observations are).

```matlab
%% Task 2: Kalman Filter for estimating a constant hidden in a noisy sinusoid
% Author: <your name>
clear; clc; close all;

%% a) Generate sinusoidal signal: A = 2.5, f = 5 Hz, t = 0..10 s, fs = 100 Hz
A  = 2.5;          % amplitude
f0 = 5;             % frequency [Hz]
fs = 100;           % sampling frequency [Hz]
t  = 0:1/fs:10;      % time vector, 0 s to 10 s
x_sine = A * sin(2*pi*f0*t);   % pure sinusoid

%% b) Add a constant offset of 8
c = 8;
x_plus_c = x_sine + c;

%% c) Add zero-mean Gaussian noise with (approx.) max amplitude 0.5
% We model this as Gaussian noise with standard deviation chosen so that
% +/-0.5 roughly bounds the noise (about +/-3*sigma), i.e. sigma = 0.5/3.
noise_std = 0.5/3;
noise = noise_std * randn(size(t));
z = x_plus_c + noise;          % this is the Kalman filter INPUT (measurement)

%% d) Kalman filter to estimate the constant c
N = length(z);
x_hat = zeros(1,N);   % state estimate (estimate of the constant) over time
P     = zeros(1,N);   % estimation error variance over time

% --- Initialization ---
x_hat(1) = 0;          % initial guess of the constant (no prior knowledge)
P(1)     = 1;          % initial uncertainty (large, since guess is poor)

Q = 1e-6;              % process noise variance: constant barely changes
R = noise_std^2;        % measurement noise variance (matches added noise)
% Note: in practice R would also need to absorb the sinusoidal "disturbance",
% but since the sinusoid has zero mean over a full period, the filter still
% converges to the true constant on average.

for k = 2:N
    % ---- Predict step ----
    x_pred = x_hat(k-1);      % state doesn't change (F = 1)
    P_pred = P(k-1) + Q;      % uncertainty grows slightly

    % ---- Update step ----
    K = P_pred / (P_pred + R);            % Kalman gain
    x_hat(k) = x_pred + K * (z(k) - x_pred);  % correct prediction with measurement
    P(k) = (1 - K) * P_pred;              % update uncertainty
end

%% e) Plot: noisy signal from c) vs. Kalman estimate
figure;
plot(t, z, 'Color', [0.7 0.7 0.7]); hold on;
plot(t, x_hat, 'r', 'LineWidth', 2);
yline(c, '--k');
xlabel('Time [s]');
ylabel('Amplitude');
legend('Noisy signal z(t) (input)', 'Kalman estimate of constant', 'True constant = 8');
title('Kalman Filter Estimation of a Constant Hidden in a Noisy Sinusoid');
grid on;
```

**What to expect when you run it:** the gray trace oscillates around 8 with noise riding on top; the red trace starts at 0, and within a fraction of a second converges and settles very close to 8, with the fluctuations from the sinusoid smoothed out.

---

# Task #3 — Text Questions (20 points)

**a) The PSD of a pure sinusoidal signal is …**
A pair of Dirac delta impulses (a discrete line spectrum) located at ±f₀ (the signal frequency), each weighted by A²/4:
$$S_{xx}(f) = \frac{A^2}{4}\left[\delta(f-f_0) + \delta(f+f_0)\right]$$
(All the signal's power is concentrated at exactly one frequency — nothing is spread out.)

**b) The PSD of a delta pulse is …**
**Flat (constant) over all frequencies** — a delta function in the time/lag domain transforms (via Fourier transform) into a constant in the frequency domain. This is the definition of "white" — infinite bandwidth, equal power density at every frequency.

**c) Wiener–Khintchine theorem**
$$S_{xx}(f) = \int_{-\infty}^{\infty} s_{xx}(\tau)\, e^{-j2\pi f\tau}\, d\tau \qquad \Longleftrightarrow \qquad s_{xx}(\tau) = \int_{-\infty}^{\infty} S_{xx}(f)\, e^{j2\pi f\tau}\, df$$
It states that for a (wide-sense) stationary random process, the power spectral density is the Fourier transform of the autocorrelation function (and the ACF is the inverse Fourier transform of the PSD). It's the bridge that lets you move between the time-domain correlation description of a random signal and its frequency-domain power distribution.

**d) Gaussian noise at input of linear system → output shows …**
The output is **also Gaussian**. Any linear operation (filtering, weighted sums, integration) applied to a Gaussian process produces another Gaussian process — the mean and variance (and spectrum, if the system isn't memoryless) change according to the system, but the Gaussian *shape* of the distribution is preserved.

**e) Poles of a stable system's transfer function are located …**
In the **left half of the s-plane**, i.e. Re{s} < 0 (strictly negative real part), for continuous-time systems. (Discrete-time equivalent: strictly inside the unit circle, |z| < 1.)

**f) ACF of a periodic signal with limited mean power — integration over … is sufficient**
Integration over **one period (T₀)** is sufficient (then normalize by dividing by T₀), instead of integrating from −∞ to +∞, because the signal — and hence its correlation properties — simply repeats every T₀.

**g) The Fourier transform of the ACF of a random signal is called …**
The **power spectral density (PSD)**.

**h) Advantage of a Hamming window vs. a rectangular window**
A Hamming window has much **lower sidelobes** (less spectral leakage) than a rectangular window — it tapers the signal smoothly to near-zero at the edges instead of cutting it off abruptly. The trade-off is a **wider main lobe** (slightly worse frequency resolution), but in most practical spectral-analysis situations the reduced leakage is worth it.

**i) When can the ACF be used for noise suppression?**
The ACF is useful for noise suppression when the **noise is uncorrelated (e.g. white)** and the **signal of interest is correlated/periodic** (e.g. a sinusoid or any deterministic/narrowband signal). White noise's autocorrelation collapses to essentially a spike at lag τ=0 and (ideally) vanishes for all τ≠0, whereas a periodic or correlated signal's ACF stays significant even at larger lags. So computing the ACF of the combined (signal+noise) data suppresses the noise contribution away from τ=0 while preserving/revealing the underlying periodic/correlated signal structure — this is exactly why the ACF is a standard tool for pulling a hidden periodicity out of noisy data.
