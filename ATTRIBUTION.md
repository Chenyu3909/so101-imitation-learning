# Attribution and Scope

This project deliberately distinguishes the existing open-source platform from my own engineering work.

## Existing / third-party components

- SO-101 mechanical and electrical platform
- SO-101 leader/follower architecture
- Hugging Face LeRobot framework and command-line tools
- ACT imitation-learning algorithm and LeRobot implementation
- PyTorch
- OpenCV
- FFmpeg
- Hugging Face dataset/model tooling
- Feetech STS3215 servo hardware and low-level support

## My work

- 3D printing, physical assembly, wiring, and setup of the leader/follower system
- servo ID assignment, calibration, and joint-range configuration
- leader-to-follower teleoperation setup
- integration of base and wrist cameras
- design and fabrication of a custom base-camera mount
- configuration of recording, training, and rollout workflows
- collection of the real-world demonstration datasets
- debugging of USB, COM-port, servo, video, dataset, and CUDA issues
- experiment design comparing dataset size/diversity and additional training
- physical evaluation and success-rate measurement
- analysis of the systematic rightward grasp bias
- documentation of the full build and experiment process

## How I describe the project

I describe this work as **building, integrating, and experimentally evaluating a robot-learning system using SO-101, LeRobot, and ACT**.

I do not claim to have designed the original SO-101 platform, invented ACT, or written LeRobot.
