# fuzzy-controller

Two Fuzzy Inference Systems (FIS) for adaptive traffic-light timing at a two-street intersection (a main street and a side street), built with `scikit-fuzzy` as part of a Soft Computing course assessment.

## Problem

A traffic light needs to decide how long to keep each street's green light on based on how many cars are currently queued. This project compares two different design philosophies for that decision:

- **FIS 1 — Fairness-oriented:** minimises the *difference* in average waiting time between the main street and the side street.
- **FIS 2 — Main-street priority:** minimises average waiting time *on the main street specifically*, at the side street's expense.

Both systems take the same two inputs and produce the same two outputs:

| Type | Variable | Range | Meaning |
|---|---|---|---|
| Input | `waiting` | 1–100 | Cars queued on the side street |
| Input | `incoming` | 0–100 | Cars queued on the main street |
| Output | `wait duration` | 0–120 | Green time given to the main street |
| Output | `open duration` | 0–120 | Green time given to the side street |

## Repository contents

| File | Description |
|---|---|
| `FIS_1.ipynb` | Fairness-oriented controller: membership functions, 9 rules, simulation, and results |
| `FIS_2.ipynb` | Main-priority controller: membership functions, 6 rules, simulation, and results |
| `trafficSimulator.pyc` | Compiled traffic queue simulator used by both notebooks |

Each notebook is self-contained: it defines the fuzzy antecedents/consequents, sets membership functions (mostly triangular, one trapezoidal), defines the rule base, compiles the `ControlSystem`, runs it through `trafficSimulator.simulate()`, and reports summary metrics.

## Results

Both FIS were run through the same simulator. Lower `mean_wait_*` and `diff_wait` are better; higher `fairness_index` (Jain's fairness index) and `total_throughput` are better.

| Metric | FIS 1 (Fairness) | FIS 2 (Main Priority) |
|---|---|---|
| mean_wait_main | 226.63 | 230.75 |
| mean_wait_side | 57.65 | 51.86 |
| diff_wait | 168.98 | 178.89 |
| max_wait_main | 442.30 | 449.14 |
| max_wait_side | 123.98 | 100.86 |
| throughput_main | 946 | 941 |
| throughput_side | 110 | 102 |
| total_throughput | 1056 | 1043 |
| fairness_index | 0.74 | 0.71 |
| avg_queue_main | 339.68 | 370.31 |
| avg_queue_side | 3.46 | 2.62 |
| max_queue_main | 657 | 674 |
| max_queue_side | 16 | 15 |

**Takeaway (as written up in the notebooks):** FIS 2 does successfully bias service toward the main street relative to FIS 1 (higher `diff_wait`), but neither controller hits its stated goal well — FIS 1's `diff_wait` of ~169 is still a large gap for a "fairness" controller, and main-street congestion (`avg_queue_main`) is high in both cases, suggesting the rule base reacts too little or too late to growing main-street queues.

## Requirements

Not currently pinned in the repo, but the notebooks import:
- `numpy`
- `scikit-fuzzy`
- `matplotlib`
- `trafficSimulator` (bundled as a compiled `.pyc`, source not included)

## Known limitations (current state)

- `trafficSimulator.pyc` ships as compiled bytecode with no source in the repo, so the simulation logic isn't inspectable or reviewable from the repo alone.
- No `README` previously existed, no `requirements.txt`, and no license.
- The two notebooks duplicate the `evaluate_metrics()` function and plotting code rather than sharing it.
- No baseline controller (fixed-time, PID, etc.) is included, so the metrics above can only be compared between FIS 1 and FIS 2, not against a non-fuzzy alternative.
- No automated tests.

## Future improvements

Planned/possible extensions, roughly in priority order:

1. **Publish the simulator source.** Decompile or rewrite `trafficSimulator.pyc` as plain `.py` and commit it, so the whole pipeline is inspectable.
2. **Refactor out of notebooks.** Move membership function definitions, rule bases, and `evaluate_metrics()` into a shared `src/` package (`membership.py`, `rules.py`, `simulator.py`, `evaluate.py`), with the notebooks reduced to experiment runners.
3. **Add `requirements.txt` and a license.**
4. **Add baseline controllers for comparison:**
   - Fixed-time (non-adaptive) signal, as a control group.
   - A PID or bang-bang controller, to compare classical control against fuzzy control directly.
   - A simple tabular Q-learning controller, to compare a learned policy against both.
5. **Visualise the controllers, not just the metrics:**
   - Plot each membership function.
   - Plot the control surface (output vs. both inputs) for each FIS.
   - Build a small interactive demo (Streamlit/Gradio) with sliders for `waiting`/`incoming` so the controller's behaviour can be explored live.
6. **Automated tests** covering rule-base edge cases (e.g. both streets empty, both streets saturated, one saturated/one empty).
7. **Stretch: swap the bespoke simulator for SUMO** (Simulation of Urban Mobility), the standard open-source traffic simulation tool, for more realistic and credible evaluation.
8. **Stretch: extend to a multi-intersection grid** to study coordination between adjacent lights rather than a single isolated intersection.
9. **Stretch: port the controller to a ROS2 node** for use as an obstacle-avoidance/priority controller in a small robotics context, reusing the same fuzzy inference approach.
