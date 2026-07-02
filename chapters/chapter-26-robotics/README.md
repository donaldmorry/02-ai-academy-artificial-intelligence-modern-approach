# Chapter 26: Robotics

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 26
> **Part:** Part VI: Communicating, Perceiving & Acting
> **Estimated time:** 12–14 hours
> **Prerequisites:** Chapter 03: Solving Problems by Searching; Chapter 14: Probabilistic Reasoning over Time

---

## Chapter Overview

Integrate AI techniques in physical agents: robot hardware, kinematics, perception, localization and mapping (SLAM), motion planning, control, and human-robot interaction. Synthesize search, planning, probability, learning, and vision for embodied intelligence.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Describe robot components: sensors, actuators, effectors, controllers
2. Explain kinematics for manipulators and mobile robots
3. Apply probabilistic localization (Kalman, particle filters) on robots
4. Outline SLAM problem and solution families
5. Plan motions with configuration-space and sampling-based planners (RRT)
6. Distinguish motion planning, path planning, and task planning
7. Discuss HRI, safety standards, and ethical deployment

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Robot Hardware](./section-01-robot-hardware.md) | Sensors, actuators, degrees of freedom |
| 2 | [Kinematics](./section-02-kinematics.md) | Forward/inverse kinematics, Jacobians overview |
| 3 | [Perception for Robots](./section-03-perception-for-robots.md) | Vision, lidar, sensor fusion |
| 4 | [Localization](./section-04-localization.md) | Odometry, Kalman localization, Markov localization |
| 5 | [Mapping and SLAM](./section-05-mapping-and-slam.md) | Occupancy grids, FastSLAM concept |
| 6 | [Motion Planning](./section-06-motion-planning.md) | C-space, visibility graphs, RRT, PRM |
| 7 | [Control](./section-07-control.md) | PID, trajectory tracking, model predictive control preview |
| 8 | [Human-Robot Interaction](./section-08-human-robot-interaction.md) | Shared autonomy, safety, social robots |

---

## Lab / Project

Simulate a mobile robot in a 2D grid: implement Monte Carlo localization and plan paths with A* in configuration space.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Production ML systems | Course 1 Chapter 07: deploying perception models |
| Deep RL for control | Course 3 Chapter 12: continuous control |
| Sim-to-real | Course 4 Chapter 10: synthetic data for robot perception |

---

## Prerequisites

- Chapter 03: Solving Problems by Searching
- Chapter 14: Probabilistic Reasoning over Time
- Chapter 25: Computer Vision

---

## Self-Assessment

1. What is configuration space and why is planning done there?
2. How does Monte Carlo localization represent belief?
3. Distinguish SLAM from localization alone.
4. When are sampling-based planners preferred over grid search?
5. What sensors are common for indoor mobile robots?
6. Name two HRI safety considerations.

---

**Previous:** [Chapter 25 — Computer Vision](../chapter-25-computer-vision/README.md)
**Next:** [Chapter 27 — Philosophy, Ethics, and Safety](../chapter-27-philosophy-ethics-safety/README.md)
