# Development and Experiment Log

## September 25, 2026 — Printing

- Downloaded the SO-101 follower-arm 3D parts from GitHub.
- Sliced the parts in Cura and transferred them to the printer.
- Calibrated the printer using auxiliary and automatic calibration.
- Initial printing repeatedly failed.
- Fixed the issue by removing the extra build-plate adhesion.
- Restarted a print lasting approximately **15 hours 14 minutes**.

## September 29–30, 2026 — Assembly and teleoperation

Before software setup, I completed a **4.5-hour Python refresher course** over two days.

After the final motors arrived:

- assembled the leader and follower arms;
- spent approximately **3.5 hours** on assembly;
- connected the servo wiring;
- assigned servo IDs to the corresponding joints;
- calibrated minimum and maximum joint ranges to prevent overextension;
- configured the local arm IDs;
- established successful leader-to-follower teleoperation.

Approximate servo/software setup time: **3 hours**.

Local Windows port assignment during this build:

- Leader arm: COM4
- Follower arm: COM3

These ports are machine-specific.

## October 1, 2026 — Dual-camera integration

- Installed the custom CAD base-camera mount.
- Added one base-mounted camera and one wrist-mounted camera.
- Encountered USB hub and connection conflicts.
- Debugged the system until both cameras could operate simultaneously.
- Final production capture used both cameras at **640×480, 30 FPS**.
- Approximate camera/debugging time: **4 hours**.

I then recorded a small 5-episode test dataset, replayed a recorded episode, collected the 25-demo pilot dataset, and trained the first ACT model.

---

# Real-Robot Experiments

## Task

**Pick up the white bottle and place it in the white box.**

## Hardware and sensing

- SO-101 follower arm
- SO-101 leader arm for teleoperation
- Base-mounted camera
- Wrist-mounted camera
- 640×480 at 30 FPS for both camera streams

## Experiment 1 — 25 demonstrations / 10k

The first dataset contained 25 teleoperated demonstrations.

Observed behavior:

- the arm moved generally toward the bottle;
- motion was visibly shaky;
- the grasp consistently missed to the right.

The model had learned the overall direction of the task but showed a systematic positional bias.

## Experiment 2 — same 25 demonstrations / +10k fine-tuning

I initialized from the 25-demo / 10k policy weights and performed another 10,000 updates using the same 25-demo dataset.

Observed behavior:

- the rightward miss became worse;
- shakiness remained.

The failure was not corrected by simply optimizing the same limited dataset longer.

This run used the previous policy weights but a fresh optimizer, so it is described as **10k + 10k fine-tuning**, not a true continuous 20k optimizer-state resume.

## Experiment 3 — 50 demonstrations / 10k

I added 25 new demonstrations with greater spatial variation and careful, smooth grasp trajectories, bringing the dataset to 50 episodes.

I then trained a fresh ACT model for 10,000 updates.

Observed behavior:

- successful autonomous pick-and-place;
- less shaky motion;
- more accurate grasping.

### Physical evaluation

| Position | Success |
| --- | ---: |
| Center | 5/5 |
| Close Right | 0/5 |
| Slight Left | 5/5 |
| Big Swing | 4/5 |
| **Overall** | **14/20 (70%)** |

## Experiment 4 — same 50 demonstrations / +10k fine-tuning

I initialized from the 50-demo / 10k policy weights and performed another 10,000 updates on the same dataset.

### Physical evaluation

| Position | Success |
| --- | ---: |
| Center | 5/5 |
| Close Right | 3/5 |
| Slight Left | 4/5 |
| Big Swing | 4/5 |
| **Overall** | **16/20 (80%)** |

Again, this was initialized from pretrained weights with a fresh optimizer rather than a true optimizer-state resume.

## Rollout observations

- Target control-loop rate: 30 Hz.
- One observed rollout averaged roughly 27.9 Hz with occasional timing spikes.
- After successful placement, the policy could continue making movements until the rollout was manually stopped because no explicit task-success termination condition was implemented.

## Current engineering conclusion

Increasing the diversity of the real demonstration dataset from 25 to 50 episodes produced a much larger improvement than continuing to optimize the original 25-demo dataset.

The current experiments do **not** isolate demonstration count from demonstration quality/diversity, because the added 25 episodes were also intentionally more varied and carefully executed.

## Next experiment

Reproduce the task in NVIDIA Isaac Lab and compare real-only versus simulation-assisted learning under the same real-world evaluation protocol.
