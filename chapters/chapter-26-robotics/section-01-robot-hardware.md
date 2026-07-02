# Section 26.1: Robot Hardware

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 26.1
> **Previous:** [Chapter 25 - Computer Vision](../chapter-25-computer-vision/README.md)
> **Next:** [Section 26.2 - Kinematics](./section-02-kinematics.md)
> **Prerequisites:** [Chapter 03 - Search; Chapter 14 - Time](../chapter-03-solving-problems-by-searching/README.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Physical Embodiment

A **robot** is a physical agent that senses, computes, and acts on the world. Russell and Norvig open Chapter 26 by emphasizing that robotics integrates **all AI subfields** - search, logic, probability, learning, and vision - in systems constrained by physics, time, and safety.

Hardware choices define what the robot can perceive and how it can affect the environment. Software cannot compensate for missing sensors or insufficient actuator authority.

> **Readable form:** Robot hardware is the body - sensors are eyes and ears, actuators are muscles, the computer is the brain connecting them.

---

## Robot Morphologies

| Type | Structure | Typical use |
|------|-----------|-------------|
| **Manipulator** | Serial chain of links | Assembly, surgery |
| **Mobile robot** | Wheeled/legged base | Warehouses, exploration |
| **Humanoid** | Bipedal + arms | Research, service |
| **Aerial (UAV)** | Flying platform | Mapping, inspection |
| **Underwater ROV** | Thrusters | Ocean inspection |

**Degrees of freedom (DoF):** independent ways a robot can move. A 6-DoF arm reaches any position and orientation in 3D workspace (within limits).

---

## Sensors

### Proprioceptive (Internal State)

| Sensor | Measures |
|--------|----------|
| Encoders | Joint angles |
| IMU | Acceleration, angular velocity |
| Force/torque | Contact forces at wrist |
| Motor current | Proxy for load |

### Exteroceptive (External World)

| Sensor | Measures | Strength | Weakness |
|--------|----------|----------|----------|
| Camera | Appearance | Rich semantics | No direct depth |
| LiDAR | Range points | Accurate 3D | Cost, weather |
| Radar | Range + velocity | Weather robust | Low resolution |
| Ultrasonic | Short range | Cheap | Narrow beam |
| GPS | Global position | Outdoor absolute | Multipath, drift |
| Tactile | Contact, pressure | Fine manipulation | Limited coverage |

> **Readable form:** Proprioceptive sensors tell the robot about itself; exteroceptive sensors tell it about the world outside.

---

## Actuators and Effectors

**Actuators** convert electrical signals to mechanical motion:

| Type | Characteristics |
|------|-----------------|
| DC/brushed motors | Simple, need gearing |
| Brushless motors | Efficient, precise |
| Servo motors | Integrated position control |
| Stepper motors | Open-loop positioning |
| Hydraulic | High force, heavy |
| Pneumatic | Fast, compliant |

**End effectors:** grippers (parallel jaw, vacuum, soft), tools (welder, screwdriver).

**Power and thermal:** sustained torque limited by battery and cooling - real constraint on duty cycle.

```python
# Simplified encoder reading → joint angle
def encoder_to_angle(ticks, ticks_per_rev=4096):
    return 2 * np.pi * ticks / ticks_per_rev
```

---

## Control Hardware

**Embedded controllers** run low-level loops at kHz rates:
- Motor drivers (PWM, current control)
- Safety PLCs (emergency stop, joint limits)
- Main computer (ROS, planning, perception)

**Real-time requirements:**
- Joint servo: 1 kHz
- Whole-body control: 500 Hz-1 kHz
- Perception/planning: 10-30 Hz

Latency from sensor to actuator bounds achievable control bandwidth.

---

## Wheeled Mobile Robot Anatomy

Common warehouse AMR (autonomous mobile robot):

1. **Differential drive** - two powered wheels + caster
2. **2D LiDAR** - 360° scan at 10-40 Hz
3. **Wheel encoders** - odometry
4. **IMU** - orientation
5. **Onboard PC** - ROS 2 stack
6. **Bumper + E-stop** - safety

**Holonomic vs. nonholonomic:** car-like robots cannot move sideways - constrains motion planning (Section 26.6).

---

## Manipulator Anatomy

6-DoF serial arm:
- Links connected by revolute or prismatic joints
- **Workspace** - reachable positions
- **Payload** - max mass at full extension
- **Repeatability** - precision of return to same pose (±0.1 mm industrial)

**Collaborative robots (cobots):** force-limited, safe near humans - different safety standards than industrial cages.

---

## Safety Systems

| Layer | Mechanism |
|-------|-----------|
| Mechanical | E-stop, brakes |
| Electrical | Safety relays, light curtains |
| Software | Velocity limits, workspace bounds |
| Functional safety | ISO 10218, ISO 13849 |

Russell and Norvig stress **fail-safe** design - loss of power or communication should not cause hazardous motion.

---

## AIMA Perspective

Chapter 26 frames robots as **rational agents** with PEAS specification:
- **Performance:** task completion, safety, efficiency
- **Environment:** physical, partially observable, dynamic
- **Actuators:** wheels, joints, grippers
- **Sensors:** cameras, LiDAR, encoders

Hardware determines the **perceptual and action space** - the agent designer's first constraints.

---

## Selection Guidelines

| Requirement | Favor |
|-------------|-------|
| Indoor navigation | 2D LiDAR + wheel encoders |
| Outdoor driving | Camera + LiDAR + radar fusion |
| Precision assembly | 6-DoF arm + force sensing |
| Human proximity | Cobot + capacitive skin |
| Long range | UAV + GPS + camera |

Cost, weight, power, and maintainability trade off against capability.

---

## Key Takeaways

1. Robot morphology (manipulator, mobile, aerial) determines task suitability and DoF.
2. Proprioceptive sensors measure internal state; exteroceptive sensors measure the external world.
3. Actuators and end effectors define how the robot physically interacts with objects.
4. Real-time control hardware separates fast motor loops from slower planning cycles.
5. Safety systems span mechanical, electrical, and software layers with industry standards.

## Additional Notes

This section's extra practice: sketch a robot for one task, list its sensors, effectors, compute budget, and safety envelope, then explain which hardware limitation will dominate the agent design. Robotics turns abstract agents into physical systems with latency, wear, calibration, and risk.

---


## Key Terms (Glossary)

- **sensor** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **actuator** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **degrees of freedom** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Specify PEAS for a warehouse package-sorting robot.
2. Compare differential drive vs. omnidirectional wheels for a hospital delivery robot.
3. List sensors needed for autonomous highway driving and justify each.
4. Explain why encoder odometry drifts over time on carpet vs. tile.
5. Research ISO 10218 and summarize two safety requirements for industrial robots.

---

**Previous:** [Chapter 25 - Computer Vision](../chapter-25-computer-vision/README.md) · **Next:** [Section 26.2 - Kinematics](./section-02-kinematics.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
