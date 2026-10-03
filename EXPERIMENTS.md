# Development and Experiment Log

This log records the main build, debugging, training, and evaluation milestones for the real-robot SO-101 project.

## September 25, 2026 — Printing

- Downloaded the SO-101 follower-arm 3D parts from GitHub.
- Sliced the parts in Cura and transferred them to the printer.
- Calibrated the printer using auxiliary and automatic calibration.
- The initial print repeatedly failed.
- Traced the problem to the build-plate adhesion setup and removed the extra adhesion.
- Restarted the print successfully.
- Estimated print time: approximately **15 hours 14 minutes**.

## September 29–30, 2026 — Assembly and teleoperation

Before software setup, I completed a **4.5-hour Python refresher course** over two days so I could better understand the robotics software stack.

After the final motors arrived:

- assembled the leader and follower arms;
- spent approximately **3.5 hours** on assembly;
- connected the servo wiring;
- assigned servo IDs to the corresponding joints;
- configured local arm IDs;
- calibrated minimum and maximum joint ranges to prevent overextension;
- established successful leader-to-follower teleoperation.

Approximate servo/software setup time: **3 hours**.

Local Windows port assignment during this build:

- Leader arm: **COM4**
- Follower arm: **COM3**

These ports are machine-specific.

## October 1, 2026 — Dual-camera integration and first dataset

- Installed the custom CAD base-camera mount.
- Added one base-mounted camera and one wrist-mounted camera.
- Initially attempted to run the cameras through a USB hub.
- Encountered repeated camera/USB connection conflicts.
- Debugged the setup until both cameras could operate simultaneously.
- Final production capture used both cameras at **640×480, 30 FPS**.
- Approximate camera/debugging time: **4 hours**.

I then:

1. recorded a small 5-episode test dataset;
2. replayed a recorded episode to verify the pipeline;
3. collected the 25-demonstration pilot dataset;
4. trained the first ACT model for 10,000 updates.

## October 2, 2026 — Failure analysis, dataset expansion, and evaluation

This was the main iteration day for the project.

### 1. Evaluated the 25-demo / 10k policy

The arm generally moved toward the bottle, but the motion was shaky and the gripper repeatedly missed to the **right**, even when the bottle position changed.

That made the error look systematic rather than like a single bad rollout.

### 2. Tested the “undertrained” hypothesis

I initialized from the 25-demo / 10k policy weights and performed another 10,000 updates on the **same 25 demonstrations**.

The rightward bias became worse rather than improving, and the motion remained shaky.

That result pushed me away from the idea that the policy simply needed more optimization.

### 3. Expanded the dataset from 25 to 50 demonstrations

I collected 25 additional teleoperated episodes with:

- more variation in bottle and box placement;
- smoother trajectories;
- more deliberate grasp alignment.

The final local dataset contained 50 episodes even though the original dataset/repository name still referenced “pilot-25.”

### 4. Trained a fresh 50-demo model

I trained a new ACT policy from scratch on all 50 demonstrations for 10,000 updates.

This model successfully completed the autonomous bottle-to-box task and was noticeably smoother and more accurate than the 25-demo model.

### 5. Fine-tuned the 50-demo model for another 10k updates

I initialized from the 50-demo / 10k policy weights and performed another 10,000 updates on the same dataset.

As with the 25-demo fine-tuning run, this used the previous policy weights with a **fresh optimizer**, so it is documented as **10k + 10k fine-tuning**, not a true continuous 20k optimizer-state resume.

### 6. Ran standardized physical trials

I tested the 50-demo / 10k and 50-demo / 10k + 10k checkpoints under the same four placement categories.

| Position | 50 demos / 10k | 50 demos / 10k + 10k fine-tuning |
| --- | ---: | ---: |
| Center | 5/5 | 5/5 |
| Close Right | 0/5 | 3/5 |
| Slight Left | 5/5 | 4/5 |
| Big Swing | 4/5 | 4/5 |
| **Overall** | **14/20 (70%)** | **16/20 (80%)** |

### 7. Other debugging during the training/rollout cycle

During this stage I also diagnosed a major performance issue in the training environment: PyTorch had initially been installed as a CPU-only build. After switching to a CUDA-enabled build and confirming the RTX 4050 Laptop GPU was available, ACT training became practical for repeated experiments.

During rollout, I also observed that:

- the control loop could occasionally experience timing spikes;
- after a successful placement, the policy could continue moving because the rollout had no explicit task-completion termination condition.

### Video examples

- [25 demos / 10k + 10k fine-tuning — failure example](media/25_demo_10k_plus_10k_failure.mp4)
- [50 demos / 10k + 10k fine-tuning — successful example](media/50_demo_10k_plus_10k_success.mp4)

## Interpretation

The current results support a practical engineering conclusion: **increasing the coverage and diversity of the demonstration dataset mattered more than simply training the original 25-demo dataset longer.**

The results do **not** isolate dataset size from demonstration quality/diversity, because the added 25 episodes were also intentionally more varied and carefully executed.

Likewise, the 20-trial evaluations are useful engineering measurements but are too small to treat as a statistically conclusive benchmark.

## Next experiment

Reproduce the same task in NVIDIA Isaac Lab and compare real-only versus simulation-assisted learning under the same physical evaluation protocol.
