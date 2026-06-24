# Disney World Monorail — Queueing Theory Analysis

A Continuous-Time Markov Chain (CTMC) analysis of the Disney World Monorail boarding queue, modeled under the M/M/c framework. Steady-state metrics are derived from first principles, validated against 30,108 observed operating intervals, and extended with a Pointwise Stationary Approximation to capture time-of-day demand variation that the baseline model misses.

---

## Results at a Glance

| Metric | Value |
|--------|:-----:|
| Arrival rate (λ̂) | 9.11 guests/min |
| Service rate per unit (μ̂) | 0.86 guests/min |
| Active units (c) | 11 |
| Utilization (ρ) | 96.2% |
| Mean wait time (Wq) | 144.6 sec |
| Probability of delay (Erlang-C) | 86.0% |
| **Peak-hour finding** | **System unstable (ρ > 100%) at 9–10 AM** |
| Recommended capacity | c=12 baseline, c=13 during morning peak |

---

## Why This Project Is Different From a Standard M/M/c Exercise

Most textbook queueing exercises stop at fitting a global M/M/c model and reporting Wq. This analysis goes further in three ways:

1. **Tests the Poisson assumption rather than assuming it.** A chi-squared goodness-of-fit test rejects the homogeneous Poisson arrival hypothesis (VMR = 7.76) — and instead of discarding the model, this is used as direct motivation for a time-varying extension.
2. **Applies the Pointwise Stationary Approximation (Green & Kolesar, 1991)** to recompute queueing metrics using hour-specific arrival rates. This reveals that the system is formally **unstable** (ρ > 100%) during the 9–10 AM peak — a finding the 24-hour-average model cannot surface.
3. **Explicitly addresses what "server" means in a batch-service system.** A monorail train doesn't process one passenger at a time; the white paper includes a dedicated discussion of how the M/M/c "server" abstraction maps onto a batch-dispatch transit system.

---

## Repository Structure

```
disney-monorail-queue-analysis/
├── queue_system_analysis.ipynb         # Full analysis (27 cells, 12 sections)
├── monorail_analysis.png               # 6-panel results figure
├── whitepaper_monorail_queue.pdf       # Full technical write-up
├── .gitignore                          # Excludes waiting_times.csv (374 MB)
└── README.md
```

---

## Dataset

**Source:** [Disney World Ride Wait Times — Kaggle](https://www.kaggle.com/datasets/ayoubimrich/disney-world-wait-times)

- **Full dataset:** 1,317,071 rows × 14 columns, 37 attractions, Jan 2018 – Mar 2020
- **Filtered to Monorail only:** 30,108 records
- **Key fields used:**

| Field | Role |
|-------|------|
| `DEB_TIME` / `FIN_TIME` | Interval timestamps |
| `GUEST_CARRIED` | Used to estimate arrival rate λ |
| `NB_UNITS` | Used to estimate number of servers c (mode = 11) |
| `UP_TIME` / `OPEN_TIME` | Used to construct an operational availability proxy for ρ |
| `WAIT_TIME_MAX` | Posted wait time, used for external validation only |

> **Note:** `waiting_times.csv` (374 MB) is not included in this repository due to size.

---

## Methodology Pipeline

```
Raw Data (1,317,071 records, 37 rides)
        │
        ▼
┌─────────────────────────────────┐
│   FILTER & PREPROCESS           │
│  Monorail only → 30,108 records │
│  Drop non-positive intervals    │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│   PARAMETER ESTIMATION (MLE)    │
│  λ̂ = 9.1056 guests/min          │
│  c = mode(NB_UNITS) = 11        │
│  ρ̂ = UP_TIME / OPEN_TIME        │
│  μ̂ = λ̂ / (c · ρ̂) = 0.8602       │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│   GOODNESS-OF-FIT TEST          │
│  χ² test vs Poisson             │
│  Rejected (VMR = 7.76)          │
│  → motivates PSA extension      │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│   CTMC / STEADY-STATE METRICS   │
│  Generator matrix (birth-death) │
│  P₀, Erlang-C, Lq, Wq, P(n)     │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│   PSA EXTENSION (hourly)        │
│  Recompute ρ, Wq per hour       │
│  → ρ > 100% at 9–10 AM          │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│   VALIDATION & CAPACITY         │
│  Wq vs WAIT_TIME_MAX (r=0.02)   │
│  Sensitivity: c ∈ {9,...,15}    │
│  → c=12 baseline, c=13 peak     │
└─────────────────────────────────┘
```

---

## Key Design Decisions

**Why extend to a Pointwise Stationary Approximation instead of stopping at the global M/M/c model?**
The chi-squared goodness-of-fit test rejects the Poisson arrival assumption (VMR = 7.76), driven by a clear morning peak in demand. Rather than treating this as a limitation to footnote, the analysis applies the PSA method (Green & Kolesar, 1991) to recompute steady-state metrics using hour-specific arrival rates. This surfaces a finding the global model conceals entirely: the system is formally unstable during the 9–10 AM peak.

**Why interpret ρ̂ = UP_TIME/OPEN_TIME as an availability proxy rather than a direct utilization measurement?**
No data dictionary accompanies the source dataset. Based on standard naming conventions in theme-park operational systems, `UP_TIME`/`OPEN_TIME` most plausibly captures operational uptime (not in breakdown) rather than instantaneous server-busy fraction. The white paper makes this distinction explicit rather than presenting μ̂ as a directly measured quantity.

**Why does μ̂ ≈ 0.86 guests/min/server not represent a literal single-passenger boarding time?**
The Monorail dispatches guests in batches of hundreds per train, not one at a time. The white paper includes a dedicated section explaining that c and μ in this model are a mathematical convention mapping a batch-dispatch system onto the standard M/M/c framework — the aggregate throughput (c × μ) is what carries operational meaning, not the per-server figure in isolation.

---

## Result Highlights

**Steady-state baseline (c=11):**
- ρ = 96.23% — near-critical utilization, virtually no buffer
- Erlang-C: 86.02% of arriving guests face a queue
- Wq = 144.6 sec, Lq = 21.94 guests

**Time-varying (PSA) extension:**
- ρₕ > 100% at 09:00 and 10:00 — system is formally unstable during morning peak
- This is invisible to the 24-hour-average model

**Capacity sensitivity:**
- c=12 reduces baseline Wq by 80.1%
- c=13 restores stability during the morning peak
- Diminishing returns beyond c=13

See `monorail_analysis.png` for the full 6-panel results figure (guest distribution vs. Poisson fit, hourly demand, steady-state distribution P(n), and capacity sensitivity curves).

---

## Installation

```bash
pip install numpy pandas scipy matplotlib
```

**Python version:** 3.x

No external queueing libraries are used — all core formulas (P₀, Erlang-C, steady-state distribution P(n), generator matrix) are implemented from scratch in the notebook.

---

## Reproducibility

The notebook runs top-to-bottom once `waiting_times.csv` is placed in the expected path. All estimation steps are deterministic given the source data — no random sampling or seeded processes are involved in this analysis, unlike stochastic-simulation-based queueing studies.

---

## Limitations

- `UP_TIME`/`OPEN_TIME`-derived ρ̂ is an operational availability proxy, not a directly observed server busy fraction.
- The Poisson arrival assumption is rejected at the aggregate level; the PSA extension mitigates but does not fully resolve within-hour non-stationarity.
- Aggregated 15-minute interval data precludes direct testing of the exponential service-time assumption.
- `WAIT_TIME_MAX` validation is necessarily indirect, given the absence of individual-guest wait-time records.

Full discussion in the [white paper](whitepaper_monorail_queue.pdf).

---

## References

1. A. Imrich, "Disney World Ride Wait Times," Kaggle, 2022.
2. D. Gross, J. F. Shortle, J. M. Thompson, and C. M. Harris, *Fundamentals of Queueing Theory*, 5th ed. Wiley, 2018.
3. L. Green and P. Kolesar, "The Pointwise Stationary Approximation for Queues with Nonstationary Arrivals," *Management Science*, vol. 37, no. 1, pp. 84–97, 1991.

---

## Author

**Ilham Khadafi**
ilhamkhadafi.dkh@gmail.com
[github.com/hamkhadafi](https://github.com/hamkhadafi)
