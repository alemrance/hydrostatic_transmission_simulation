# Hydrostatic Transmission Simulation (Mobile Working Machines)

This project models and simulates the benchmark problem from **Section 3.6, "Hydrostatic transmission in mobile working machines"**, of Kroll & Schulte (2014) in MATLAB/Simulink. The model is parameterized and run through a MATLAB App Designer interface, and the vehicle motion is visualized with Simscape Multibody.

> A. Kroll, H. Schulte, *Benchmark problems for nonlinear system identification and control using Soft Computing methods: Need and overview*, Applied Soft Computing 25 (2014) 496–513. (`proje5_hydrostatic.pdf`)

## The problem in the paper (Section 3.6)

Hydrostatic transmissions are high-power-density gears widely used in mobile working machines and off-road vehicles such as wheel loaders, graders and tractors. They offer continuously variable transmission, maximum tractive force at low speeds, and reversing without changing gear.

**System:** A combustion engine drives a variable-displacement hydraulic pump. The pump feeds a variable-displacement hydraulic motor in a closed oil circuit. The motor drives the vehicle axle through a fixed-ratio gearbox. Two electro-hydraulic control elements change the displacement of the pump and the motor. The pump displacement can be increased, set to zero or reversed, which gives forward, neutral and reverse motion.

**State and input vectors:**

- `x = [x1, x2, x3, x4]ᵀ` = [pump angle α̃P, motor angle α̃M, pressure difference Δp, hydromotor speed ωM]
- `u = [u1, u2]ᵀ` = [pump control signal uP ∈ [−1, 1], motor control signal uM ∈ [0, 1]]

**Nonlinear model (Eq. 35):**

```
ẋ1 = −(1/τuP)·x1 + (kP/τuP)·u1
ẋ2 = −(1/τuM)·x2 + (kM/τuM)·u2
ẋ3 = (10/CH)·( ṼmaxP·x1·ωP − ṼmaxM·x2·x4 − kleak·x3 )
ẋ4 = ( ig²·ia²·ηg·ηmh·ṼmaxP·10⁻⁴·x2·x3 − d̃vc·ia²·x4 − TLw·ig·ia ) / Jv
```

**Control objective:** Decoupled control of speed and torque. A given control performance must be achieved in every driving situation, independent of slope and road conditions. Ramp, sinusoidal and trapezoidal profiles are used as reference signals.

**Assessment criteria:** Steady-state accuracy (J_IAE), RMS tracking error during acceleration/deceleration (J_RMSE,Ref), peak control signal (J_peak,u) and integral squared control effort. Controllers are compared on a driving profile with off-road and on-road scenarios (Fig. 6 in the paper).

## What this project implements

- **Simulink model (`hidrostatik_simulink1.slx`):** The four differential equations of Eq. 35 are written in a MATLAB Function block (`fcn`) and solved with four integrators. The external load torque `T_input` is read from the workspace as a time-dependent signal. Results (angle changes, pressure difference, flow, speed and acceleration) are exported with `To Workspace` blocks.
- **Multibody visualization:** The "Multibody Block" subsystem models the truck body (`Kamyon Govde1.STEP`), the axles and the ground in Simscape Multibody. The computed motor speed drives the vehicle motion.
- **User interface (`hidrostatik_transmisyon.mlapp`):** Built with App Designer. The interface sets:
  - Pump input angular speed ωP
  - Initial conditions x1–x4
  - Control inputs u1 (gear/pump) and u2 (pedal/motor)
  - Simulation stop time and step size
  - **Road-dependent load torque** `T_load(a)`: any road function (sinusoidal, exponential, trapezoidal, step, etc.) can be typed in as text.

  "Run Simulation" runs the model; speed, angle and pressure changes are plotted in tabbed graphs.

### Parameters used in the model

| Symbol | Description | Paper (Table 5) | Model |
|---|---|---|---|
| τuP | Pump time constant | 0.13 s | 1.3 |
| τuM | Motor time constant | 0.22 s | 2.2 |
| kP / kM | Pump / motor static gain | 241.67 / 283.33 | 241.67 / 283.33 |
| ṼmaxP / ṼmaxM | Pump / motor max. displacement | 145 / 170 cm³ | 1.45e-4 / 1.7e-4 m³ |
| CH | Hydraulic capacitance | 1840.8 mm⁵/N | 1.840 |
| d̃vc | Viscous damping | 0.33 Nms | 0.33 |
| ia / ig | Axle / transmission ratio | −23.3 / −6.12 | −23.3 / −6.12 |
| Jv | Vehicle moment of inertia | 16512 Nms² | 1651 |
| kleak | Leakage coefficient | 0.14 | 0.14 |
| ηg / ηmh | Gearbox / hydro-mechanical efficiency | 0.98 / 0.697 | 0.98 / 0.697 |

> Note: The time constants, CH and Jv are entered in the model with different scaling than in the paper.

## Files

| File | Contents |
|---|---|
| `hidrostatik_simulink1.slx` | Simulink + Simscape Multibody model |
| `hidrostatik_transmisyon.mlapp` | App Designer interface |
| `Kamyon Govde1.SLDPRT` / `.STEP` | Truck body CAD model (SolidWorks / STEP) |
| `Hydrostatic_Transmission.mp4` | Hydrostatic transmission animation played in the app |
| `out.mat` | Sample simulation output |
| `proje5_hydrostatic.pdf` | Source paper |

## How to run

1. Open the project folder in MATLAB. Simulink, Simscape and Simscape Multibody must be installed.
2. Open `hidrostatik_transmisyon.mlapp` in App Designer. To play the video, paste the full path of `Hydrostatic_Transmission.mp4` into the `videoFile` line in `SaveParametersButtonPushed` (around line 191).
3. In the Simulink model, point the File Solid block under *Multibody Block → gövde* to `Kamyon Govde1.STEP`.
4. Run the app. Enter the parameters and the load torque function, click **Save Parameters** to push them to the workspace, then click **Run Simulation**.
