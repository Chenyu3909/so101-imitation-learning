# Custom Base Camera Mount

This folder contains my custom CAD design for the base-mounted camera used in the SO-101 dual-camera setup.

## Purpose

The imitation-learning setup uses two viewpoints:

- a **base camera** for a stable external view of the workspace;
- a **wrist camera** for close-up information near the gripper.

I designed the base-camera mount in **Onshape** and 3D-printed it for the physical robot. It keeps the external camera fixed relative to the follower arm/workspace during demonstration recording and autonomous rollout.

## Files

- [base_camera_mount.stl](base_camera_mount.stl) — printable mesh
- [base_camera_mount.step](base_camera_mount.step) — editable CAD exchange format

<p align="center">
  <img src="../../media/base_camera_mount_installed.jpg" width="520" alt="Custom base camera mount installed on the SO-101">
</p>

## Scope

The SO-101 arm itself is existing open-source hardware. This camera mount is my own mechanical addition to the system.
