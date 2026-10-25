---
theme: seriph
background: /images/cover.png
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Learning to Ride
  Reproducing and Extending Reinforcement-Learning Control of a Benchmark Bicycle
drawings:
  persist: false
transition: slide-left
title: Learning to Ride - Bicycle RL
mdc: true
---

# Learning to Ride

Reproducing and Extending Reinforcement-Learning Control of a Benchmark Bicycle

<div class="pt-12">
  <span class="text-sm opacity-75">
    Maple Shao · October 25, 2026
  </span>
</div>

<div class="abs-br m-6 text-sm opacity-50">
  ISAC Group Meeting
</div>

<!--
Good morning. Today I will show how a neural network learns to balance and steer a bicycle with no rider and no training wheels. I will reproduce a published result and extend it in three directions: better learning algorithms, a more realistic tyre model, and better visualisation.
-->

---

# Background & Setup

<div class="grid grid-cols-2 gap-6">
<div>

## The Challenge

**Why bicycles are hard:**
- Statically unstable
- Non-minimum-phase (steer right to turn left)
- Speed-dependent dynamics
- Underactuated (steering only)

## Reference Work
**Weyrer et al. (2024)** - arXiv:2407.17156
- SAC + 4-stage curriculum
- Path following at 2–7 m/s

</div>
<div>

## Our System

**Plant:** Whipple benchmark bicycle
- Meijaard (2007) parameters
- Multibody simulation (Exudyn)
- Self-stable: 4.26–6.02 m/s ✓

**Learning Problem:**
- Observation: lean, steer, heading + 5 preview points
- Action: steering angle (±70°)
- Reward: distance to path
- 4-part curriculum: gentle → tight curves

</div>
</div>

<!--
A bicycle is a classic hard control problem. It's unstable when stationary and requires counter-intuitive steering. We start from a 2024 paper that used SAC with a curriculum to learn path following at 2 to 7 metres per second. Our plant is the standard Whipple benchmark bicycle. The agent sees the bicycle state plus preview points ahead and outputs a steering angle.
-->

---

# Contribution 1: Better Learning Algorithms

<div class="grid grid-cols-2 gap-6">
<div>

## Methods Compared

**SAC (baseline)**
- Soft Actor-Critic
- Single value estimate
- Tends to overestimate

**TQC** ⭐
- Distributional ensemble critics
- Drops optimistic estimates
- Counters overestimation

**CrossQ**
- Batch-normalized critics
- Sample efficient

</div>
<div>

## Training Results

| Method | Steps | Pass Rate |
|--------|-------|-----------|
| SAC | 1.36 M | 81/100 |
| TQC | **0.82 M** | **98/100** ⭐ |
| CrossQ | 0.80 M | 91/100 |

<div class="mt-4 p-3 bg-blue-500/10 rounded text-sm">
✅ TQC: ~40% faster, 21% more accurate
</div>

<div class="mt-3 p-3 bg-orange-500/10 rounded text-sm">
⚠️ SAC: only 1/3 seeds completed curriculum
</div>

</div>
</div>

<!--
First contribution: better learning algorithms. We compared three methods on the exact same task. SAC, the baseline, is slow and fragile - only one of three random seeds finished training after 1.36 million steps. TQC finished in 820,000 steps and passes 98 of 100 test rides. That's 40% faster and 21% more accurate. CrossQ is similarly fast but slightly less accurate. The key is TQC's distributional critics that counter overestimation.
-->

---

# Contribution 2: Control Performance (1/2)

<div class="text-center mb-4">
<span class="text-xl font-bold">TQC (Red) vs SAC (Blue) - Easy to Moderate</span>
</div>

<div class="grid grid-cols-2 gap-8">

<div>

### Easy Warm-up (Part 0, 3 m/s)
<video controls class="w-full" style="max-height: 400px;">
  <source src="/videos/01_P0_random_v3.0.mp4" type="video/mp4">
</video>
<div class="text-center mt-2">
  <p class="text-sm font-bold">TQC: 0.07 m | SAC: 0.14 m</p>
  <p class="text-lg text-blue-600 font-bold">2× better</p>
</div>

</div>

<div>

### Tight U-turns (r=5m, 4 m/s)
<video controls class="w-full" style="max-height: 400px;">
  <source src="/videos/05_u_turns_r5_v4.0.mp4" type="video/mp4">
</video>
<div class="text-center mt-2">
  <p class="text-sm font-bold">TQC: 0.05 m | SAC: 0.29 m</p>
  <p class="text-lg text-green-600 font-bold">6× better ⭐</p>
</div>

</div>

</div>

<!--
Second contribution: control performance. We put both agents on the same challenging paths. Watch the videos - red is TQC, blue is SAC. On the left, an easy warm-up path where TQC is twice as accurate. On the right, tight U-turns with 5-meter radius - tighter than anything in training. Here TQC is six times better, staying within 5 centimeters while SAC swings wide.
-->

---

# Contribution 2: Control Performance (2/2)

<div class="text-center mb-4">
<span class="text-xl font-bold">TQC (Red) vs SAC (Blue) - Hard Challenges</span>
</div>

<div class="grid grid-cols-2 gap-8">

<div>

### Tightening Spiral (5.5 m/s)
<video controls class="w-full" style="max-height: 260px;">
  <source src="/videos/06_spiral_in_v5.5.mp4" type="video/mp4">
</video>
<div class="text-center mt-2">
  <p class="text-sm font-bold">TQC: 0.07 m | SAC: 0.66 m</p>
  <p class="text-lg text-red-600 font-bold">9× better 🏆</p>
</div>

</div>

<div>

### Hardest Path (7 m/s)
<video controls class="w-full" style="max-height: 260px;">
  <source src="/videos/08_P3_random_v7.0.mp4" type="video/mp4">
</video>
<div class="text-center mt-2">
  <p class="text-sm font-bold">TQC: 0.39 m | SAC: 1.08 m</p>
  <p class="text-lg text-purple-600 font-bold">3× better</p>
</div>

</div>

</div>

<div class="mt-4 text-center p-3 bg-purple-500/10 rounded">
<span class="font-bold">Key Finding:</span> Performance gap grows with difficulty - from 2× to 9×
</div>

<!--
On harder challenges, the gap widens dramatically. On the left, a spiral that continuously tightens from 14 to 5 meters radius. Watch SAC drift outward as the curve closes - its error is ten times larger than TQC's. On the right, the hardest of 400 random paths at maximum speed. SAC comes within 60 centimeters of the failure limit while TQC stays three times more accurate. The performance gap clearly grows with task difficulty.
-->

---

# Difficulty Ladder: Complete Analysis

<div class="text-sm">

| Path | Speed | Radius | TQC | SAC | Ratio |
|------|-------|--------|-----|-----|-------|
| Easy warm-up | 3.0 m/s | Part 0 | 0.07 m | 0.14 m | 2× |
| Double hairpin | 4.5 m/s | 7 m | 0.04 m | 0.14 m | 3.5× |
| **U-turns** | 4.0 m/s | **5 m** | **0.05 m** | **0.29 m** | **6×** ⭐ |
| Random Part 2 | 5.0 m/s | ≥10 m | 0.10 m | 0.32 m | 3.2× |
| Slalom | 5.0 m/s | 5 m | 0.06 m | 0.24 m | 4× |
| **Spiral** | 5.5 m/s | **14→5 m** | **0.07 m** | **0.66 m** | **9×** 🏆 |
| Hardest path | 7.0 m/s | Part 3 | 0.39 m | 1.08 m | 3× |

</div>

<div class="mt-4 p-3 bg-gradient-to-r from-blue-500/10 to-purple-500/10 rounded text-center">
<span class="font-bold">Summary:</span> Eight paths tested, accuracy advantage grows from 2× (easy) to 9× (tightest curves)
</div>

<!--
Here's the complete difficulty ladder with all eight test paths. Notice the clear pattern: on gentle curves at moderate speed, TQC is about twice as accurate. As we tighten the curves and increase speed, the gap widens systematically. The U-turns at 5-meter radius show six-to-one advantage. The continuously tightening spiral reveals the largest gap at nine-to-one. This demonstrates that superior learning translates to superior control, especially under stress.
-->

---

# Contribution 3: Classical + Learned Control

<div class="grid grid-cols-2 gap-6">
<div>

## Classical Baseline

**Lean-Steer Controller**
- Pure pursuit + steer-into-fall
- Gains tuned by evolution strategy
- Reads same 16 observations

**Performance:** 59% pass rate (Part 3)

<div class="mt-4 text-sm opacity-75">
Respectable baseline, but below learned agents
</div>

</div>
<div>

## Residual RL

**Concept:**  
steering = controller + learned correction

**Results:**

| Method | Pass Rate | Improvement |
|--------|-----------|-------------|
| SAC alone | 81/100 | baseline |
| **Residual SAC** | **96/100** | **+15%** ⭐ |
| Controller alone | 59/100 | -22% |

<div class="mt-4 p-3 bg-green-500/10 rounded text-sm">
✅ Good prior helps weak learner significantly
</div>

</div>
</div>

<!--
Third contribution: combining classical and learned control. We built a classical lean-steer controller that passes 59% of hard test rides. Then we used residual RL - the controller does the basic job, the agent learns a correction on top. For SAC, this boosted performance from 81% to 96%. This shows that a good classical prior can significantly improve a weaker learning algorithm.
-->

---

# Contribution 4: Sim-to-Sim Transfer

<div class="grid grid-cols-2 gap-4">
<div>

## The Challenge

**Ideal model → Realistic tyres**

Second simulator (MuJoCo):
- Measured tyre data (Bulsink 2016)
- Magic Formula model
- Slip, camber, relaxation

**Physics impact:**

| Model | Stable window |
|-------|--------------|
| Ideal | 4.3–6.0 m/s |
| Real | >7.4 m/s |

⚠️ Unstable across 2–7 m/s range!

</div>
<div>

## Transfer Results

**Trained on ideal → Tested on real**

| Method | Train | Test | Drop |
|--------|-------|------|------|
| SAC | 81% | 61% | -20% |
| **TQC** | **98%** | **95%** | **-3%** |

<div class="mt-3 p-2 bg-green-500/10 rounded text-sm">
✅ More accurate = more robust
</div>

<div class="mt-2 text-xs opacity-75">
Both still balance (falls ≤5%)<br>
TQC maintains without retraining
</div>

</div>
</div>

<!--
Fourth contribution: robustness to realistic physics. We built a second simulator with measured real tyre data. The tyres completely change the dynamics - the bicycle becomes unstable across our entire training range. We tested both agents zero-shot, without any retraining. SAC's performance drops from 81% to 61%. TQC barely notices the change: 98% down to 95%. The more accurate policy is inherently more robust.
-->

---

# Contribution 5: CARLA Visualisation

<div class="grid grid-cols-2 gap-6">
<div>

## Implementation

**CARLA 0.9.16 as renderer only**
- All physics in our simulator
- Bicycle posed at 20 Hz
- Physics disabled in CARLA

**Enables:**
- Realistic 3D rendering
- Chase camera views
- Side-by-side comparison
- Live telemetry overlay

<div class="mt-4 p-3 bg-blue-500/10 rounded text-sm">
Turns numbers into visible behaviour
</div>

</div>
<div>

## Example: Double Hairpin

<video controls class="w-full">
  <source src="/videos/03_hairpin_r7_v4.5.mp4" type="video/mp4">
</video>

<div class="mt-3 text-sm">
<p><strong>Test:</strong> r=7m, 4.5 m/s</p>
<p><strong>TQC:</strong> 0.04 m error</p>
<p><strong>SAC:</strong> 0.14 m error</p>
</div>

</div>
</div>

<!--
Fifth contribution: high-quality visualisation. We integrated CARLA as a renderer while keeping all physics in our simulator. This lets us film multiple agents on the same path with chase cameras and live telemetry. Watch this double hairpin example - you can immediately see TQC tracking tighter through the curves. Visualisation is crucial - it turns abstract tracking errors into visible behaviour differences.
-->

---

# Key Results Summary

<div class="grid grid-cols-2 gap-6 mt-6">

<div>

## Training Efficiency
- **TQC: 40% faster** than SAC
- 98/100 vs 81/100 pass rate
- More reliable (SAC: 1/3 seeds failed)

## Control Accuracy  
- **2-9× better** tracking on hard paths
- Gap grows with curvature
- Tighter than training: still robust

</div>

<div>

## Robustness
- **Zero-shot transfer:** 95% on real tyres
- SAC drops to 61% on same task
- More accurate → more robust

## Hybrid Control
- **Residual SAC: 96%** vs 81% pure RL
- Classical prior helps weak learner
- Best of both worlds

</div>

</div>

<div class="mt-6 p-4 bg-gradient-to-r from-blue-500/10 to-purple-500/10 rounded">
<span class="text-lg font-bold">Bottom Line:</span> Better critics (TQC) learn faster, control better, and transfer more robustly
</div>

<!--
Let me summarize the key results. TQC learns 40% faster and achieves 98% success versus 81% for SAC. On challenging paths, it's two to nine times more accurate. It transfers to realistic tyres with almost no performance loss while SAC drops 20 percentage points. Combining classical control with learning helps weaker algorithms significantly.
-->

---

# Next Steps

## Ongoing Work

<div class="grid grid-cols-2 gap-6">
<div>

1. **Multi-seed statistics**
   - ≥3 seeds per method
   - Confidence intervals

2. **Full evaluation protocol**
   - Tracking error at 21 speeds
   - Direct comparison with paper

</div>
<div>

3. **Tyre-plant training**
   - Train directly on realistic model
   - Randomised properties

4. **Path projection fix**
   - Handle figure-eight paths
   - Local search algorithm

</div>
</div>

<!--
Next steps: everything shown today is single-seed results. We need at least three seeds per method for proper statistics. We'll run the paper's full evaluation protocol for direct comparison. We're training on the realistic tyre model with randomisation. And we'll fix the path projection issue for figure-eight paths.
-->

---
layout: center
class: text-center
---

# Summary

<div class="text-left max-w-4xl mx-auto mt-8 text-lg">

**We reproduced and extended bicycle RL control:**

<div class="grid grid-cols-2 gap-6 mt-6">

<div>

✅ **Better algorithms**  
TQC: 40% faster, 98% success

✅ **Superior control**  
2-9× more accurate on hard paths

</div>

<div>

✅ **Robust transfer**  
95% on realistic tyres (zero-shot)

✅ **Effective visualisation**  
CARLA rendering + telemetry

</div>

</div>

<div class="mt-6 p-4 bg-blue-500/10 rounded">
<span class="font-bold">Core contribution:</span> Modern critics make bicycle control learning faster, more accurate, and more robust
</div>

</div>

<div class="mt-10 text-xl opacity-75">
Thank you! Questions?
</div>

<!--
To summarize: We independently reproduced the bicycle learning result and pushed it in three directions. Modern critic algorithms like TQC learn 40% faster and achieve 98% success. They're two to nine times more accurate on challenging paths. They transfer to realistic tire models with minimal performance loss. And we provide high-quality visualization to make all these differences visible.
-->
