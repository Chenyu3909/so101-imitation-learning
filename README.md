# SO-101 Vision-Based Imitation Learning

A low-cost real-world robotics project using a dual-camera SO-101 arm, LeRobot, and ACT imitation learning to study autonomous pick-and-place from human demonstrations.

**Current status:** the real-robot baseline works. Increasing the dataset from 25 to 50 demonstrations produced much smoother and more reliable behavior than simply training the original 25 demonstrations longer.

## Project Scope

This project builds on existing open-source hardware and software. I did **not** design the SO-101 platform, invent ACT, or write LeRobot.

### Existing work used

- SO-101 leader/follower robot-arm platform
- Hugging Face LeRobot
- ACT (Action Chunking with Transformers) implementation in LeRobot
- PyTorch
- OpenCV
- FFmpeg
- Feetech STS3215 smart servos and existing low-level support

### My contributions

I:

- 3D-printed, assembled, wired, configured, and calibrated the leader and follower arms;
- assigned servo IDs and calibrated joint ranges;
- established leader-to-follower teleoperation;
- integrated two simultaneous cameras: a base-mounted camera and wrist-mounted camera;
- designed and 3D-printed a custom base-camera mount in Onshape;
- debugged USB hub, camera, COM-port, servo, encoding, dataset, and CUDA issues;
- collected all real-world demonstrations used in these experiments;
- trained and compared ACT policies across dataset sizes and training durations;
- diagnosed a systematic rightward grasp bias in the 25-demonstration model;
- tested whether additional training alone would correct that bias;
- collected 25 additional, more varied demonstrations after it did not;
- evaluated the 50-demonstration models across repeated physical trials.

See [ATTRIBUTION.md](ATTRIBUTION.md) for a more explicit scope breakdown.

## System

| Component | Configuration |
| --- | --- |
| Robot | SO-101 leader + follower |
| Actuators | Feetech STS3215 smart servos |
| Vision | Base camera + wrist camera |
| Camera capture | 640×480 at 30 FPS |
| Framework | LeRobot |
| Policy | ACT imitation learning |
| Training GPU | NVIDIA GeForce RTX 4050 Laptop GPU |
| Task | Pick up the white bottle and place it in the white box |

### Pipeline

Human teleoperation → dual-camera + joint-action recording → LeRobot dataset → ACT training on GPU → autonomous rollout → physical evaluation

## Experiment Progression

### 1. 25 demonstrations / 10k updates

The first ACT policy generally moved toward the bottle but was shaky and showed a systematic rightward grasp miss.

### 2. Same 25 demonstrations / +10k fine-tuning

I loaded the 10k model weights and performed another 10k training updates using the **same dataset**. The rightward bias became worse rather than disappearing.

> The second 10k run was initialized from the first model's weights with a fresh optimizer. It was not a continuous optimizer-state resume, so this repository describes it as **10k + 10k fine-tuning**.

### 3. 50 demonstrations / 10k updates

I added 25 new demonstrations with more spatial variation and smoother, more deliberate grasp trajectories, then trained a fresh ACT policy on all 50 demonstrations.

The resulting policy successfully completed the task and was visibly smoother and more accurate.

### 4. 50 demonstrations / +10k fine-tuning

I loaded the 50-demo / 10k model weights and trained for another 10k updates on the same 50 demonstrations.

## Physical Evaluation

Each placement category below contains 5 trials per checkpoint.

| Test position | 50 demos / 10k | 50 demos / 10k + 10k fine-tuning |
| --- | ---: | ---: |
| Center | 5/5 | 5/5 |
| Close Right | 0/5 | 3/5 |
| Slight Left | 5/5 | 4/5 |
| Big Swing | 4/5 | 4/5 |
| **Overall** | **14/20 (70%)** | **16/20 (80%)** |

These are small-sample engineering tests, not a statistically conclusive benchmark. The main qualitative result is that increasing dataset diversity from 25 to 50 demonstrations improved behavior substantially, while additional training on the original 25-demo dataset did not correct its systematic bias.

Raw results are in [results.csv](results.csv).

## Development Record

A dated build journal is available in [PROJECT_LOG.md](PROJECT_LOG.md).

The actual LeRobot commands used for teleoperation, recording, replay, training, fine-tuning, and rollout are documented in [real_robot/COMMANDS.md](real_robot/COMMANDS.md). Authentication credentials are intentionally excluded.

## Custom CAD

My custom base-camera mount is documented under [hardware/camera_mount/](hardware/camera_mount/).

## Next Research Step

The planned next phase is to reproduce the same manipulation task in NVIDIA Isaac Lab and investigate:

> **Can simulation pretraining reduce the amount of real-world demonstration data required for a low-cost SO-101 arm?**

## Project Status

- [x] Print, assemble, wire, and calibrate leader/follower SO-101 arms
- [x] Establish teleoperation
- [x] Integrate base and wrist cameras
- [x] Design and print custom base-camera mount
- [x] Record 25-demo pilot dataset
- [x] Train and evaluate pilot ACT policy
- [x] Diagnose systematic grasp bias
- [x] Expand dataset to 50 demonstrations
- [x] Train successful autonomous pick-and-place policy
- [x] Run repeated physical evaluation trials
- [ ] Add selected build photos and comparison videos
- [ ] Reproduce the task in Isaac Lab
- [ ] Run sim-to-real experiments

## Acknowledgements

This project depends on the open-source work of The Robot Studio, Hugging Face LeRobot contributors, the ACT research/codebase incorporated into LeRobot, PyTorch, OpenCV, FFmpeg, and the broader open-source robotics community.
