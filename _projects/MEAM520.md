---
layout: page
title: Block Pick & Place with Franka Panda
description: Franka Panda - Block Pick and Place
img: assets/img/MEAM520/4stacked.png
importance: 2
category: etc
related_publications: false
---

This project implements an end-to-end **pick-and-place + stacking** pipeline on the **Franka Emika Panda**. The robot must pick up **static blocks** on the team platform and **dynamic blocks** rotating on a turntable, under a strict safety constraint. 

### System Overview
The pipeline closes the loop from perception to execution:

- **Perception:** AprilTag-based block detections from an end-effector-mounted camera 
- **Frame transforms:** Convert block poses from camera frame to base frame via FK and calibrated camera extrinsics:  
  \[
  {}^{0}T_b = {}^{0}T_e \cdot {}^{e}T_c \cdot {}^{c}T_b
  \]
- **Grasp generation:** Construct pre-grasp / grasp poses using an approach offset to account for gripper geometry and clearance. 
- **Planning & Control:** Solve numerical IK with joint-limit checks; execute collision-safe motions.

### Strategy

1. **Use a safe 3-step transport trajectory to avoid collisions and preserve stack stability.**  
   Each static pick-place follows a repeatable loop:
   - View from a fixed vantage pose (stable detections)
   - **Pick**
   - **Elevated transfer** to avoid hitting already-stacked blocks
   - **Place/stack**, then return to the vantage pose  
   This “up-and-over” transfer is explicitly chosen for collision avoidance and stability.

2. **Attempt dynamic blocks opportunistically with prediction + retries.**  
   After static stacking, we switch to dynamic blocks on the rotating turntable:
   - Move to a fixed **hover pose** near the turntable (keeps the EE safely above the workspace while enabling consistent detections).
   - Select a target using a simple heuristic (**leftmost block**) for repeatability.
   - Predict an intercept pose using a **constant angular velocity** model: rotate the detected pose forward by  
     \[
     \theta = \omega \Delta t
     \]
     then solve IK for the predicted grasp pose.  
   - Execute grasp; use gripper width feedback to detect success; on failure, return to hover, reopen gripper, and retry with a new detection. 

### Demos

<video controls muted playsinline width="100%" preload="metadata">
  <source src="/assets/videos/meam520/staticblock.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>


<video controls muted playsinline width="100%" preload="metadata">
  <source src="/assets/videos/meam520/dynamicblock.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>


### Results (Sim → Real)
- **Static blocks:** Highly reliable in simulation and partially transferable to hardware after tuning.
- **Dynamic blocks:** Sensitive to perception noise and timing; dominant hardware failure mode was **pose noise + timing**. 

### Key Lessons
This project highlighted the **sim-to-real gap** in manipulation: performance on hardware was strongly affected by detection noise, systematic EE offsets, and safety-driven motion constraints. 
