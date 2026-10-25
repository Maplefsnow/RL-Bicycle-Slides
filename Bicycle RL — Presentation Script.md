# Bicycle RL — Presentation Script

Oct 25, 2026 · @Maplef

## Slide plan

The talk is planned as 16 slides in about 20 minutes, plus backup slides for questions. Each slide below has three parts: **On the slide**

&#32;(what to put on it), **Say** (the spoken script), and **Video** (which clip to play, if any). All numbers are single-seed results as of 25 Oct 2026; the script says so wherever it matters.

| # | Slide | Time | Video |
| --- | --- | --- | --- |
| 1 | Title | 0:30 | — |
| 2 | Why a bicycle is hard to control | 1:30 | — |
| 3 | The reference paper and our goals | 1:30 | — |
| 4 | The plant: Whipple benchmark bicycle | 1:30 | — |
| 5 | The learning problem | 1:30 | — |
| 6 | Curriculum and validation gate | 1:15 | — |
| 7 | Reproduction result | 1:30 | Paper-SAC slalom, 5 m/s |
| 8 | Better critics: TQC and CrossQ | 1:45 | — |
| 9 | Classical controller and residual RL | 1:30 | — |
| 10 | Head-to-head: the difficulty ladder | 2:30 | Ladder clips 1, 5, 6, 8 |
| 11 | A more realistic plant: measured tyres | 1:30 | — |
| 12 | Sim-to-sim transfer | 1:15 | — |
| 13 | CARLA as a renderer | 0:45 | Ladder clip 3 |
| 14 | A limitation we found | 1:00 | — |
| 15 | Expected outcomes | 1:30 | — |
| 16 | Summary | 0:45 | — |

Video files (all in `runs/`):

- **Ladder clips**: `videos/20261025_tqc_vs_sac_ladder/01…08_*.mp4` — CARLA split view, TQC (red) vs paper SAC (blue), same path and start, 15–20 s each.
- **Paper-SAC benchmark clips**: `paper_faithful_n4_seed19260817_lr1e4/videos/<2–7>mps/{slalom,lane_change,full_circle,random_seed101,random_seed202}.mp4`.

## Part 1 — Background (slides 1–3)

### Slide 1 — Title

**On the slide:** "Learning to Ride: Reproducing and Extending Reinforcement-Learning Control of a Benchmark Bicycle" · your name · date · a still frame from ladder clip 8.

**Say:** "Good morning. Today I will show how a neural network learns to balance and steer a bicycle with no rider and no training wheels. I will start from a published result, reproduce it, and then push it in three directions: better learning algorithms, a more realistic tyre model, and better visualisation."

### Slide 2 — Why a bicycle is hard to control

**On the slide:**

- Statically unstable: it falls over unless it is steered
- Non-minimum-phase: to turn left, you first steer right
- Speed-dependent: the benchmark bicycle balances itself only between 4.3 and 6.0 m/s
- Underactuated: one input (steering) controls lean, heading and position

**Say:** "A bicycle is a classic hard control problem. Standing still, it simply falls over. When it moves, the only way to stay upright is to steer into the fall. That creates a counter-intuitive effect: to turn left, you must first steer briefly to the right, so that the bicycle leans left. Control engineers call this non-minimum-phase behaviour.

The dynamics also change with speed. The standard benchmark bicycle we use is self-stable only in a narrow window, roughly 4.3 to 6 metres per second. Below that, it falls; above that, it slowly capsizes. And we have only one input, the steering angle, to control lean, heading and position on the road at the same time."

### Slide 3 — The reference paper and our goals

**On the slide:**

- Weyrer, Manzl, Schwab, Gerstmayr (2024), arXiv:2407.17156 — Innsbruck and TU Delft
- SAC agent + four-part curriculum → path following and balance at 2–7 m/s, steering only
- Our goals: (1) independent reproduction, (2) better RL methods, (3) a more realistic plant, (4) visualisation

**Say:** "Our starting point is a 2024 paper from the University of Innsbruck and TU Delft. Weyrer and colleagues trained a Soft Actor-Critic agent, SAC for short, to follow a path while keeping the bicycle upright. The agent only outputs a steering angle. With a four-stage curriculum it rides full circles, slaloms and lane changes between 2 and 7 metres per second.

We asked four questions. Can we reproduce this independently? Can newer RL algorithms learn it faster or better? Does the learned policy survive a more realistic tyre model? And can we make the behaviour visible in a way people understand at a glance? The rest of the talk follows these four questions."

## Part 2 — The reproduction (slides 4–6)

### Slide 4 — The plant: Whipple benchmark bicycle

**On the slide:**

- Meijaard et al. (2007) benchmark parameters: rear frame, front fork, two wheels
- Multibody model in Exudyn: 4 rigid bodies, slip-free rolling-disc contacts
- Verified: simulated self-stable window 4.26–6.02 m/s vs. theory 4.29–6.02 m/s
- Visual: a sketch of the bicycle with lean angle φ and steer angle δ marked

**Say:** "The bicycle is the Whipple model with the Meijaard benchmark parameters. This is the standard reference bicycle in bicycle dynamics. It has four rigid bodies: rear frame, front fork and two wheels. The wheels roll without slipping. We simulate it as a multibody system in Exudyn, the same tool the original authors used.

Before training anything, we checked the plant. We let the bicycle coast without control at speeds from 2 to 8 metres per second and measured where it balances by itself. Our simulation is self-stable from 4.26 to 6.02 metres per second. The linear theory says 4.29 to 6.02. So the plant matches the textbook."

### Slide 5 — The learning problem

**On the slide:**

- Observation (16 values): lean, steer, heading, their rates, wheel speed + 5 preview points of the path ahead (0.4 s)
- Action: steering set point, up to ±70°, sent to a PD steering servo
- Control at 20 Hz; physics at 200 Hz
- Reward: 1 on the path, falling linearly to 0 at 3.5 m lateral error
- Episode ends if the bicycle leans past 45° or leaves the path by 3.5 m

**Say:** "Here is the learning problem. Twenty times per second, the agent sees the state of the bicycle: lean, steering, heading and their rates. It also sees five preview points of the path ahead, like a rider looking down the road. Speed is not given separately; the agent infers it from the wheel rotation rate.

The agent answers with a steering angle. A PD servo turns that into a steering torque, just as in the paper. The reward is simple: one point per step when the bicycle is exactly on the path, decreasing to zero at 3.5 metres off. There is no explicit reward for staying upright. The agent learns to balance only because a fall ends the episode and stops the reward."

### Slide 6 — Curriculum and validation gate

**On the slide:**

- Part 0: 4 m/s, gentle arcs (radius 20–28 m), 2 m lane shifts
- Part 1: 2–7 m/s, radius ≥ 14 m, shifts up to 4 m
- Part 2: radius ≥ 10 m, shifts up to 6 m
- Part 3: radius ≥ 7 m, shifts up to 8 m
- Gate: every 20,000 steps, 100 test rides of 30 s; advance when ≥ 80 % stay within the error limit

**Say:** "Learning the full task from scratch is too hard, so we use a curriculum, as the paper does. The agent starts at one fixed speed on gentle curves. Each time it passes a test, we widen the speed range, tighten the curves, and enlarge the lane changes. The final stage covers 2 to 7 metres per second and curves down to 7 metres radius.

The test is 100 deterministic rides of 30 seconds. The agent moves on when at least 80 of them stay close to the path. Our test is larger than the paper's, which used 10 rides, so our pass rates are less noisy. This is one of several documented deviations from the paper; the backup slide lists all of them."

**Note for you:** the backup slide B1 (Part 6 below) holds the deviations table. Keep it out of the main flow unless the audience is the original authors.

## Part 3 — Learning results (slides 7–10)

### Slide 7 — Reproduction result

**On the slide:**

- Paper SAC reproduced: best policy passes 81/100 test rides in the hardest part
- Mean lateral error ≈ 0.22 m over 2–7 m/s
- Our task is harder than the paper's: start-up disturbances up to 10°, curves down to 7 m (paper: 8 m)
- But slow and fragile: 1 of 3 seeds finished the curriculum (1.36 M steps); 2 stalled in Part 2 at 2 M

**Video:** `paper_faithful_n4_seed19260817_lr1e4/videos/5mps/slalom.mp4` (play about 10 s, muted).

**Say:** "Here is the reproduced agent riding a slalom at 5 metres per second. It balances and follows the path using steering alone, as in the paper. In the hardest curriculum stage, our best SAC agent passes 81 of 100 test rides, with an average lateral error of about 22 centimetres.

Note that our version of the task is harder than the original. The bicycle starts with disturbances of up to 10 degrees, and the tightest curves have a 7-metre radius instead of 8.

The reproduction also showed a weakness. SAC is slow and unreliable here. Out of three random seeds, only one finished the curriculum, after 1.36 million steps. The other two were still stuck in the third stage after 2 million steps. That motivated the next question: can a better learning algorithm do this faster and more reliably?"

### Slide 8 — Better critics: TQC and CrossQ

**On the slide:**

- Same task, same curriculum, same test — only the algorithm changes
- TQC: an ensemble of distributional critics; drops the most optimistic value estimates
- CrossQ: batch-normalised critics, no target network; known for sample efficiency
- Steps to finish the curriculum: TQC 0.82 M, CrossQ 0.80 M, SAC 1.36 M
- Best final-stage pass rate: TQC 98/100, CrossQ 91/100, SAC 81/100

**Say:** "We kept everything else fixed and swapped only the learning algorithm. Both alternatives change the critic, the part of the agent that estimates how good an action is. TQC uses several critics that predict a distribution of returns and throws away the most optimistic estimates. This counters SAC's tendency to overestimate. CrossQ adds batch normalisation to the critic and removes the target network.

The effect is large. Both finished the curriculum in about 800,000 steps, compared with 1.36 million for the best SAC seed. TQC passes 98 of 100 test rides in the hardest stage. The cost is compute: on our shared server they ran at roughly half the steps per second of SAC, so in wall-clock time the gap is smaller than the step counts suggest. And this is one seed per method so far; the multi-seed runs come later."

### Slide 9 — Classical controller and residual RL

**On the slide:**

- Classical baseline: lean-steer controller — pure pursuit to a preview point → target lean → "steer into the fall"
- Gains tuned by an evolution strategy (30 iterations); reads the same 16 observations as the agent
- Alone: passes 100 / 98 / 71 / 59 % of rides in Parts 0–3
- Residual RL: steering = controller + learned correction (up to ±35°)
- Residual SAC: 96/100 in Part 3 (vs. 81 for plain SAC); residual CrossQ still in Part 2

**Say:** "The paper compares RL agents only with each other. We wanted a classical reference. Our controller works like a textbook rider model. It picks a point ahead on the path, computes the lean angle needed to turn towards it, and steers into the fall to reach that lean. We tuned its gains automatically with an evolution strategy.

Without any learning, it passes 59 % of the hardest test rides. That is a respectable baseline, but well below the learned agents.

Then we combined the two. In residual RL, the controller does the basic job and the agent only learns a correction on top. For SAC, this helped a lot: 96 of 100 rides in the hardest stage, against 81 without the controller. For CrossQ it did not help; it is still in stage 3. So a good prior helps a weak learner more than a strong one, at least for this single seed."

**Figure for slides 8–9** (redraw it in your slide style, or screenshot it):

&#91;embedded content: runs/reports (compare\_runs, 25 Oct 2026) and the tuned lean-steer validation · residual CrossQ has not reached Part 3\]

Every learned method clears the 80 % gate; the classical controller alone does not.

### Slide 10 — Head-to-head: the difficulty ladder

**On the slide:** a single video area. Caption per clip: path, speed, RMS lateral error for TQC (red) and SAC (blue).

**Video:** from `videos/20261025_tqc_vs_sac_ladder/`, play in this order:

1. `01_P0_random_v3.0.mp4` — easy warm-up: TQC 0.07 m, SAC 0.14 m (play 6 s)
2. `05_u_turns_r5_v4.0.mp4` — two U-turns, 5 m radius: TQC 0.05 m, SAC 0.29 m
3. `06_spiral_in_v5.5.mp4` — spiral tightening from 14 to 5 m: TQC 0.07 m, SAC 0.66 m
4. `08_P3_random_v7.0.mp4` — hardest of 400 random paths at 7 m/s: TQC 0.39 m, SAC 1.08 m (peak 2.9 m)

**Say:** "Numbers are one thing; let us watch them ride. Red is TQC, blue is the paper's SAC. Both start at the same point on the same path. On the left you see a chase camera, on the right live telemetry.

&#91;Clip 1\] On an easy path both are fine, but red already stays closer to the line.

&#91;Clip 2\] Two U-turns with a 5-metre radius. That is tighter than anything in training. Blue swings wide out of each turn; red stays within about 5 centimetres.

&#91;Clip 3\] A spiral that keeps tightening. Watch blue drift outwards as the curve closes. Its error is ten times larger than red's.

&#91;Clip 4\] The hardest case: the most twisting of 400 random paths, at top speed. Blue comes within 60 centimetres of the 3.5-metre limit where the episode would end. Red is about three times more accurate.

Neither agent fell in any clip. The gap between them grows with curvature and speed: from two times on easy paths to nine times on the tightest ones."

## Part 4 — Robustness and visualisation (slides 11–14)

### Slide 11 — A more realistic plant: measured tyres

**On the slide:**

- Second simulator: MuJoCo, built from the same benchmark parameters
- Real tyres instead of knife-edge wheels: Magic-Formula city tyre (37-622, 4 bar), measured by Bulsink (2016)
- Adds slip, camber thrust, twisting torque, relaxation length, rolling resistance
- Effect: self-stable window moves from 4.3–6.0 m/s to above 7.4 m/s — the bicycle is unstable everywhere in 2–7 m/s
- Visual: the self-stability plot (`runs/physics_gates_v2/self_stability.png`)

**Say:** "The Whipple model assumes knife-edge wheels that never slip. Real tyres deform and slip a little, and they create extra forces and torques. To test whether our agents depend on the ideal model, we built a second simulator in MuJoCo from exactly the same bicycle parameters.

For the tyres we used measured data: a city-bike tyre characterised on a test rig by Bulsink in 2016, at loads that match our bicycle. We first used rough tyre coefficients, found in an audit that they were guesses, and replaced them with these measured values.

The tyres change the physics a lot. This plot shows where the bicycle balances by itself. With ideal wheels, that window is 4.3 to 6 metres per second. With real tyres, it moves above 7.4 metres per second. So across our whole speed range of 2 to 7 metres per second, the uncontrolled bicycle now falls over. This shift agrees with Bulsink's own findings."

### Slide 12 — Sim-to-sim transfer

**On the slide:** the table below. Headline: "Trained on ideal wheels, tested on real tyres — no retraining."

| Pass rate on the tyre plant | Part 0 | Part 1 | Part 2 | Part 3 | Part 3 on training plant |
| --- | --- | --- | --- | --- | --- |
| Paper SAC | 100 % | 94 % | 65 % | 61 % | 81 % |
| TQC | 100 % | 100 % | 98 % | 95 % | 98 % |

**Say:** "Now the key test. We take agents trained only on the ideal model and drop them, unchanged, onto the tyre model. This is zero-shot transfer.

Both agents still balance; falls stay at 5 % or less. But their accuracy differs. The paper's SAC drops from 81 to 61 % in the hardest stage. TQC barely notices: 98 % on the ideal plant, 95 % on real tyres. The more accurate policy is also the more robust one.

We are now training directly on the tyre model, with randomised tyre properties, friction and mass. A SAC run is in progress, and a TQC run starts tonight."

### Slide 13 — CARLA as a renderer

**On the slide:**

- CARLA 0.9.16 used only for rendering; all physics stays in our simulator
- Bicycle actor with physics disabled, posed from our state at 20 Hz in synchronous mode
- Chase camera + live telemetry, several agents on one path, mp4 recording or browser stream

**Video:** `videos/20261025_tqc_vs_sac_ladder/03_hairpin_r7_v4.5.mp4` (S double hairpin, 7 m radius, 4.5 m/s).

**Say:** "The videos you saw come from CARLA, an open-source driving simulator. We use it only as a renderer. CARLA's own vehicle physics is built for cars, so we switch it off. At every control step we compute the bicycle's motion in our own simulator and place the CARLA bicycle exactly there.

That lets us put several agents on the same path, film them from a chase camera, and overlay live telemetry. Here is a double hairpin at 4.5 metres per second. TQC tracks it within 4 centimetres; SAC within 14."

### Slide 14 — A limitation we found

**On the slide:**

- The agent locates itself by the globally nearest point on the path
- Paths that come back on themselves (figure eight, repeated loops) confuse it
- Example: the best SAC leaves a figure eight 6 m into the first loop
- Fix: track progress along the path and search only nearby; needs retraining, so deferred

**Say:** "We also found a limitation, inherited from the problem design. The agent measures its position relative to the nearest point on the whole path. On a figure eight, near the crossing, the nearest point can suddenly belong to the other loop. The preview points then jump, and the agent follows the wrong branch.

The fix is simple: remember how far along the path you are and only search nearby. But it changes what the agent sees, so all agents would need retraining. We postponed it so that every result in this talk stays comparable."

## Part 5 — Expected outcomes (slide 15)

### Slide 15 — Expected outcomes

**On the slide:**

1. **Statistics:** ≥ 3 seeds per method, confidence intervals on pass rates and steps-to-finish
2. **Paper protocol:** tracking error T(v) at 21 speeds from 2 to 7 m/s, on random and benchmark paths
3. **Tyre training:** SAC and TQC trained on the tyre plant with randomised tyres, friction and mass
4. **Path projection fix:** local, progress-based projection; retrain and re-test figure eights
5. **Publication:** reproduction + extension paper; candidate venues Multibody System Dynamics, Vehicle System Dynamics, IEEE RA-L

**Say:** "Where is this going? Everything I showed today is one seed per method. The first step is statistics: at least three seeds each, with confidence intervals. Only then will we claim that TQC or CrossQ is better than SAC.

Second, we will run the paper's full evaluation protocol: the tracking error at 21 speeds between 2 and 7 metres per second. That gives a direct comparison with the original figures. The tool is written; the full runs are pending.

Third, training on the realistic tyre model. A SAC run is in progress, and a TQC run starts automatically tonight when the current runs finish. We expect agents trained with randomised tyres to close most of the transfer gap we saw for SAC.

Fourth, the path-projection fix, followed by retraining. We expect it to make figure eights and repeated loops rideable.

The goal is a paper that reproduces the original work, documents every deviation, and answers five questions: reproducibility, better algorithms, classical versus learned control, robustness to realistic tyres, and the limits of the problem design."

**Note for you:** as of 25 Oct 2026, the tyre-plant SAC run is at 0.52 M steps and has passed Part 0; the TQC tyre run is queued to start at about 22:15. Update these before the talk with `bash scripts/status.sh`.

## Part 6 — Closing, Q&A and backup

### Slide 16 — Summary

**On the slide:**

- Reproduced: SAC + curriculum rides and balances the benchmark bicycle at 2–7 m/s
- Better critics: TQC and CrossQ finish the curriculum \~40 % sooner; TQC passes 98/100 hardest rides
- Classical + learned: residual SAC 96/100 vs. 59 % for the controller alone
- Robust: TQC keeps 95 % on measured tyres without retraining
- Next: multi-seed statistics, full paper protocol, tyre-plant training

**Say:** "To summarise. We reproduced the original result independently. Swapping SAC for a modern critic made learning about 40 % faster in steps and clearly more accurate. A classical controller is a useful baseline and a useful prior for residual learning. And the best policy kept almost all of its performance on a much more realistic tyre model, without retraining. The next weeks are about turning these single-seed results into solid statistics. Thank you; I am happy to take questions."

### Likely questions

- **"Is TQC really better, with one seed?"** — Not yet proven. One seed per method so far; we will only claim it with ≥ 3 seeds and confidence intervals.
- **"Can you compare your step counts with the paper's?"** — Not directly. We use 4 parallel environments and a 100-ride / 80 % gate; the paper used 1 environment and 10 rides.
- **"Why was the best SAC model trained with a lower learning rate?"** — It came from a run resumed at 1e-4 after stalling. We flag this; the clean seeds use the constant 3e-4.
- **"Does the agent know its speed?"** — Only through the front-wheel rotation rate, as in the paper. There is no separate speed input.
- **"Will this work on a real bicycle?"** — Not shown. Simulation only. The tyre plant is a step towards reality, but the tyre data come from a matching city tyre, not measured on this bicycle.
- **"Why not use a large vision-language-action model?"** — Inference runs at a few hertz; stabilising a bicycle needs 20 Hz feedback.
- **"Why CARLA if it does no physics?"** — Realistic rendering and cameras for free; car physics would be wrong for a bicycle.

### Backup slide B1 — Deviations from the paper

| Aspect | Paper | Ours |
| --- | --- | --- |
| Parallel environments | 1 | 4 |
| Speed actuation | prescribed speed | bounded PI rear-wheel drive |
| Initial disturbances | ±0.01 rad fixed | 1° → 10° over the curriculum |
| Arc radii | 14 → 8–14 m | 20–28 → 7–28 m |
| Validation | 10 rides, all pass | 100 rides, ≥ 80 % pass |
| Error limit per part | 0.057 in every part | 0.20, 0.10, 0.057, 0.057 |
| Replay buffer at part change | kept | oldest 50 % dropped |

Full table with reasons: `docs/paper/deviations.md`.

### Backup slide B2 — Ladder clips, all numbers

All eight clips and their RMS errors are listed in `runs/videos/20261025_tqc_vs_sac_ladder/README.md`. Use clips 2, 4 and 7 (Part 1 at 4 m/s, Part 2 at 5 m/s, slalom at 5 m/s) if you need extra footage.

### Sources

- Weyrer, Manzl, Schwab, Gerstmayr (2024), [Path Following and Stabilisation of a Bicycle Model using a Reinforcement Learning Approach](https://arxiv.org/abs/2407.17156), arXiv:2407.17156
- Meijaard, Papadopoulos, Ruina, Schwab (2007), benchmark bicycle parameters (Proc. R. Soc. A)
- Bulsink (2016), measured bicycle-tyre data used for tyre model v2
- Project results: `runs/reports/`, `runs/transfer/`, `runs/physics_gates_v2/`, `docs/ROADMAP.md`
