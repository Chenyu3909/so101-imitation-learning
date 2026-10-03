# Isaac Lab — Planned Sim-to-Real Extension

This folder is reserved for the next phase of the project.

## Research question

> **Can simulation pretraining reduce the amount of real-world demonstration data required for a low-cost SO-101 arm?**

## Planned comparison

The goal is to reproduce the same bottle-to-box task in NVIDIA Isaac Lab and compare:

- real-only policies trained from physical demonstrations;
- simulation-assisted policies pretrained in simulation and then fine-tuned with real demonstrations.

Both would be evaluated using the same physical placement protocol used for the current real-robot baseline.

## Planned work

- reproduce/import the SO-101 in Isaac Lab;
- match the physical pick-and-place workspace;
- create the bottle-to-box manipulation task;
- introduce controlled simulation variation/domain randomization;
- pretrain in simulation;
- fine-tune with different amounts of real demonstration data;
- compare real-world success rates.

No simulation results are claimed in the repository yet.
