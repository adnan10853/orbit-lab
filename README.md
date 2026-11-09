# Orbit Lab — Robotics Fault Investigation

An interactive educational prototype for exploring how commanded motion, measured joint angles, and injected faults relate. Built with AI-assisted development; uses synthetic data only.

The project explores a basic engineering question: **did the system follow the command, and what evidence explains an unexpected response?**

## Features

- Three joint controls: base yaw, shoulder pitch, elbow pitch.
- Orbitable 3D projection of a two-link arm, with forward kinematics.
- Shoulder commanded-versus-measured telemetry over a rolling 30-second window.
- Stuck-joint, sensor-bias and signal-oscillation scenarios.
- Threshold/dwell alert, event timeline, investigation checklist and JSON report export.
- Browser-local investigation notes; no backend or real spacecraft telemetry.

## Run

Open `index.html` in a modern desktop browser. No build step or packages required. Google Fonts is optional; local fallback fonts are provided.

For model checks, install Node.js and run:

```sh
node test.cjs
```

## First investigation

1. Keep **Nominal** selected and click **Approach target**.
2. Wait for the shoulder to settle at 50 degrees.
3. Select **Stuck joint**, then command J2 to 70 degrees.
4. Compare the commanded and measured lines: the joint stays at 50 degrees while the command is 70 degrees.
5. Observe the tracking alert; record observations and alternative causes.
6. Select **Nominal** to remove the injected fault and observe recovery.

Target distance and tracking error are different: the arm can follow a joint command correctly while moving away from the target.

## Model

The arm has a base height of 0.45 and link lengths of 2 and 1.6 arbitrary model units. Let yaw be a, shoulder pitch be b, and elbow pitch be e (angles converted to radians):

```
r = 2*cos(b) + 1.6*cos(b+e)
x = r*cos(a)
y = r*sin(a)
z = 0.45 + 2*sin(b) + 1.6*sin(b+e)
```

Normal joint response uses a first-order lag, time constant 0.65 seconds. Commands are bounded by the UI: yaw ±180°, shoulder ±80°, elbow ±120°. Rendering uses the true simulated joint state; the plot uses the measured signal.

| Scenario | Implementation |
| --- | --- |
| Nominal | Measured position equals true simulated position. |
| Stuck joint | J2 true position is frozen. |
| Sensor bias | Add 8° to the J2 measurement. |
| Signal oscillation | Add a 7° amplitude, 0.8 Hz sinusoid to J2 measurement; true position is unaffected. |

An alert requires absolute J2 command-to-measurement error above 5° continuously for 1.5 seconds. This is a screening rule, not a diagnostic classifier. It can alert on large normal command changes and miss periodic oscillation because the error repeatedly returns below the threshold.

**Approach target** commands a known pose [65°, 50°, −70°]; no inverse kinematics or path planning is used. **Hold command** sets commands to the true simulated positions, an idealized operation that assumes perfect position knowledge. **Pause** freezes simulation time.

## Verification

`test.cjs` contains seven deterministic model checks:

1. Straight-arm forward kinematics.
2. Yaw rotation geometry.
3. Nominal convergence and cleared residual alert.
4. Stuck-joint persistence and alert activation.
5. Sensor bias affecting measurement only.
6. Bounded oscillation and the continuous-dwell detector's missed detection.
7. Pause preserving state.

These checks verify the simplified implementation; they do not validate a physical robot or a spacecraft system. Manual browser checks covered the interface, target command, fault selector and guide. Report export and cross-browser behaviour need broader validation.

## Limitations and development goals

This is a desktop 3D projection, not AR/VR. There are no orbital dynamics, inertia, collision/contact checks, torques, communication delays or realistic control loops. Oscillation is a signal anomaly, not structural vibration. Fault labels are visible for learning, not blind diagnosis. Simulation time may run slower under browser throttling. Reports include only the latest 30 seconds of telemetry and up to 100 events.

Future work: transient-aware alerts, oscillation detection, sensor noise, blind investigation scenarios and more extensive UI/export tests.

Independent personal learning project. No affiliation with MDA, CSA or NASA. Not suitable for flight operations.
