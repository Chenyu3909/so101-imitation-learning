# Attribution and Scope

This file is intentionally explicit about which parts of the project were pre-existing and which parts were my own work.

## Existing / third-party components

- SO-101 mechanical and electrical platform
- SO-101 leader/follower architecture
- LeRobot framework and command-line tools
- ACT imitation-learning algorithm and LeRobot implementation
- PyTorch
- OpenCV
- FFmpeg
- Hugging Face dataset/model tooling
- Feetech STS3215 servo hardware and low-level support

## My work

- physical assembly and setup of the leader/follower system;
- servo ID assignment, calibration, and hardware debugging;
- integration of two cameras (base + wrist);
- design and fabrication of a custom base-camera mount;
- configuration of recording, training, and rollout pipelines;
- dataset collection and controlled variation of task setups;
- debugging USB, COM, video encoding, dataset, and CUDA issues;
- experiment design comparing dataset size and training duration;
- physical evaluation and success-rate measurement;
- analysis of systematic rightward grasp bias;
- redesign of the demonstration dataset from 25 to 50 examples;
- documentation and future sim-to-real extension.

## Important wording

I describe the project as **building and integrating a robot-learning system using SO-101, LeRobot, and ACT**, not as inventing the SO-101 or ACT.

The originality of the project is in the system integration, experimentation, failure analysis, data collection, evaluation, and planned sim-to-real extension.
