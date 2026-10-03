# Custom Base Camera Mount

This folder contains my custom CAD design for the base-mounted camera used in the SO-101 dual-camera setup.

## Purpose

The imitation-learning setup uses two camera viewpoints:

- a **base camera** for a stable external view of the workspace;
- a **wrist camera** for a close-up view near the gripper.

I designed the base-camera mount in **Onshape** and 3D-printed it for the physical robot. The mount keeps the external camera fixed relative to the follower arm/workspace during data collection and autonomous rollout.

## Files

- **Base Camera Mount.stl** — printable mesh
- **base_camera_mount.step** — editable CAD exchange format

## Scope

The SO-101 arm itself is existing open-source hardware. This camera mount is my own mechanical addition to the system.
