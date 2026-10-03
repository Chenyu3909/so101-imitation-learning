# SO-101 Vision-Based Imitation Learning

A low-cost robotics project exploring whether a dual-camera SO-101 arm can learn an autonomous pick-and-place task from human demonstrations using ACT imitation learning.

**Current status:** Real-robot baseline is working. A 50-demonstration model successfully learned to pick up a white bottle and place it in a white box with substantially smoother and more accurate behavior than the initial 25-demonstration model.

> Demo video: add a short GIF or video here.

## Why I Built This

My high school did not have an active robotics team while I was developing this project, so I wanted to build a real robotic manipulation system independently and learn the full pipeline: hardware assembly, cameras, data collection, model training, debugging, and physical evaluation.

The goal was not to design a new robot arm or invent a new imitation-learning algorithm. Instead, I wanted to understand how an existing open-source robot-learning stack behaves on real hardware, identify its failure modes, and improve performance through controlled iteration.

## What Was Existing vs. What I Did

### Existing open-source work

This project builds on:

- **SO-101 hardware design** — an open-source leader/follower robot arm developed by The Robot Studio in collaboration with Hugging Face.
- **LeRobot** — Hugging Face's open-source robotics framework used for motor setup, calibration, teleoperation, dataset recording, training, and policy rollout.
- **ACT (Action Chunking with Transformers)** — an existing imitation-learning policy implemented in LeRobot.
- **PyTorch** — used by LeRobot for neural-network training and CUDA acceleration.
- **OpenCV / FFmpeg / Hugging Face libraries** — used for camera capture, video encoding, dataset handling, and model tooling.
- **Feetech STS3215 servo stack** — existing motor hardware and communication support used by the SO-101.

I did **not** design the original SO-101 mechanical system, invent ACT, or write the LeRobot framework.

### My contributions

I:

- sourced, assembled, configured, and calibrated the SO-101 leader and follower arms;
- assigned and debugged the six servo IDs for each arm;
- integrated **two simultaneous cameras**: one base-mounted camera and one wrist-mounted camera;
- designed and 3D-printed a custom base-camera housing/mount;
- configured dual-camera data collection at 640×480 / 30 FPS;
- debugged USB bandwidth/contention, COM-port issues, servo communication failures, video encoding problems, and local dataset handling;
- diagnosed that my initial PyTorch installation was CPU-only and reconfigured the environment so ACT trained on an RTX 4050 Laptop GPU;
- collected my own teleoperated demonstrations for the pick-and-place task;
- designed controlled tests with varied bottle and box positions;
- trained and compared ACT policies using different dataset sizes and training durations;
- identified a systematic rightward grasp bias in the first model;
- tested the hypothesis that the model was simply undertrained by training the same data longer;
- observed that additional training on the same limited dataset made the bias worse;
- collected 25 additional, more varied demonstrations and retrained a fresh model;
- obtained smoother, more accurate autonomous pick-and-place behavior;
- measured success rates across repeated physical trials.

## System

**Robot:** SO-101 leader/follower arms  
**Actuators:** Feetech STS3215 smart servos  
**Vision:** base camera + wrist camera  
**Training GPU:** NVIDIA GeForce RTX 4050 Laptop GPU  
**Framework:** LeRobot  
**Policy:** ACT imitation learning  
**Task:** “Pick up the white bottle and place it in the white box.”

### Pipeline

Human teleoperation → dual-camera + joint-action recording → LeRobot dataset → ACT training on GPU → autonomous rollout on the follower arm → physical evaluation

## Real-Robot Experiment

### Phase 1 — 25 demonstrations

I first collected 25 demonstrations with small variations in bottle and box position and trained ACT for 10,000 steps.

**Observed behavior:**
- the arm generally moved toward the bottle;
- it consistently missed to the right;
- motion was noticeably shaky.

I then loaded the trained 10k model and performed another 10k training updates on the same 25 demonstrations.

**Result:** the rightward bias became worse and the arm remained shaky.

This suggested that the main limitation was not simply insufficient optimization. Training longer was reinforcing behavior learned from an under-diverse dataset.

### Phase 2 — 50 demonstrations

I added 25 new demonstrations, increasing variation while keeping the task, cameras, and workspace consistent. I then trained a new ACT model from scratch on all 50 demonstrations.

The 50-demo / 10k model successfully completed the task and was noticeably smoother and more accurate.

I then loaded that model and performed another 10k training updates to test whether additional fitting improved reliability.

> Note: the “+10k” runs were started from the previous trained model weights with a fresh optimizer, rather than a true optimizer-state resume. I therefore describe them as additional fine-tuning rather than a single continuous 20k-step run.

## Results

Each reported category below used 5 physical trials. The same task and hardware were used for both checkpoints.

| Test position | 50 demos / 10k | 50 demos / 10k + 10k fine-tuning |
| --- | ---: | ---: |
| Center | 5/5 | 5/5 |
| Close Right | 0/5 | 3/5 |
| Slight Left | 5/5 | 4/5 |
| Big Swing | 4/5 | 4/5 |
| **Overall** | **14/20 (70%)** | **16/20 (80%)** |

These are small-sample physical tests, so I treat them as engineering evaluation results rather than a statistically conclusive benchmark.

## What I Learned

The biggest lesson was that **more training is not automatically better**.

With only 25 demonstrations, additional training reinforced a systematic grasp bias. Increasing the diversity of the dataset from 25 to 50 demonstrations produced a much larger improvement than simply continuing to optimize the original small dataset.

The project also showed how tightly coupled a real robot-learning system is: mechanical calibration, camera placement, USB reliability, dataset quality, GPU configuration, control-loop timing, and policy training can all affect the final behavior.

## Failure Modes I Debugged

See `EXPERIMENTS.md` for a longer log. Major issues included:

- follower servo status-packet failures;
- Windows COM-port locks;
- two-camera USB contention;
- video encoder incompatibilities;
- Hugging Face Hub uploads failing while local datasets remained intact;
- CPU-only PyTorch causing multi-day training estimates;
- occasional control-loop timing spikes during rollout;
- systematic learned grasp bias;
- post-success motion because the rollout continued after task completion.

## Repository Structure

```text
so101-imitation-learning/
├── README.md
├── ATTRIBUTION.md
├── EXPERIMENTS.md
├── results.csv
├── .gitignore
├── hardware/
│   └── README.md
├── real_robot/
│   └── README.md
├── isaac_lab/
│   └── README.md
└── media/
    └── README.md
```

Large raw datasets and model checkpoints are intentionally not committed to normal Git history.

## Next Steps

The next phase is to reproduce the manipulation task in **NVIDIA Isaac Lab** and investigate a sim-to-real question:

> Can simulation pretraining reduce the amount of real-world demonstration data required for a low-cost SO-101 arm?

Planned comparisons include real-only policies and simulation-assisted policies evaluated on the same physical task.

## Project Status

- [x] Assemble and calibrate leader/follower SO-101 arms
- [x] Integrate base and wrist cameras
- [x] Record 25-demo dataset
- [x] Train and evaluate first ACT policy
- [x] Diagnose systematic grasp bias
- [x] Expand to 50 demonstrations
- [x] Train working autonomous pick-and-place policy
- [x] Run repeated physical evaluation trials
- [ ] Add polished demo video and system diagram
- [ ] Reproduce task in Isaac Lab
- [ ] Run sim-to-real experiments

## Acknowledgements

This project depends on the open-source work of The Robot Studio, Hugging Face LeRobot contributors, the ACT research/codebase incorporated into LeRobot, PyTorch, OpenCV, FFmpeg, and the broader open-source robotics community.

I am documenting the distinction between the existing platform/framework and my own integration, experimentation, debugging, data collection, and analysis so that the scope of my work is clear.
