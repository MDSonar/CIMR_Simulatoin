# How the Process Simulation Works

## 1. Introduction
This simulation represents a **continuous industrial mixing/reactor system** operating in real time (default: 1 sample per second). It behaves like a simplified **online model** or **digital-twin surrogate**, patterned after accepted practices in chemical-engineering simulation, soft-sensor design, and process monitoring.

The simulator emits realistic plant-like data streams, computes internally derived variables (KPIs & alarms), and supports downstream analytics and PoC scenarios such as drift detection, fault identification, and optimization trials.

---

## 2. What the Simulator Emulates
The simulated equipment approximates a **continuous stirred reactor** (CSTR-style), a common design in chemical, pharmaceutical, food, and specialty-materials industries.

The simulator replicates:
- Dynamic operating conditions (flow, temp, pressure, level)
- Physical relationships (mass balance → residence time)
- Quality effects (density influence, surrogate quality KPI)
- Mechanical signatures (vibration)
- Fault behaviors (runaway heating, flow drop, density drift)
- Normal plant noise + natural variability

This creates a realistic environment for testing analytics, alarms, dashboards, digital-twin logic, ML pipelines, and operator workflows.

---

## 3. Inputs (Measured Process Signals)

| Variable | Meaning | Typical Range | Purpose |
|---------|---------|----------------|---------|
| **temperature (°C)** | Internal process temperature | 20–200 | Drives reaction + pressure behavior |
| **pressure (bar)** | Reactor internal pressure | 0–10 | Safety, reaction progression |
| **flow_in (L/min)** | Inlet feed flow | 0–500 | Throughput + residence-time control |
| **flow_out (L/min)** | Outlet flow | 0–500 | Determines residence time and stability |
| **density (g/cm³)** | Product density (quality indicator) | 0.5–1.5 | Surrogate for composition/purity |
| **level (%)** | Reactor liquid volume % | 0–100 | Volume management |
| **vibration (g)** | Pump/agitator vibration | 0–5 | Mechanical health indicator |
| **timestamp** | ISO8601 | — | Temporal reference |

---

## 4. Core Simulation Logic

### 4.1 Mass Balance & Residence Time
**Reactor Volume:**  
`V = 1000 L`

**Residence Time (τ):**
```
τ (minutes) = V / flow_out
```

### 4.2 Temperature–Pressure Interaction
- Temperature influences pressure through simplified thermodynamic behavior.
- Runaway temperature elevates pressure → triggers alarms.

### 4.3 Density & Composition Effects
Density serves as a **soft indicator of product quality**, reacting to slow drifts or sudden changes.

---

## 5. Quality Index (Q) — Soft Sensor

### Formula
Setpoints:
- T_set = 120 °C  
- P_set = 4.5 bar  
- F_set = 200 L/min  
- ρ_set = 0.8600  

```
Q = 100
    - wT * |T - T_set|
    - wP * |P - P_set|
    - wF * |flow_out - F_set|
    - wρ * |density - ρ_set|
```

Q is clamped between **0 and 100**.

---

## 6. Signal Processing & Anomaly Detection

### 6.1 Moving Average & Standard Deviation
Window: **300 samples** (~5 minutes)

### 6.2 Z-Scores
```
z = (current_value - MA) / STD
```

### 6.3 Rate of Change (ROC)
```
ROC = (current - value_10s_ago) / 10
```

---

## 7. Alarm Engine

| Alarm | Trigger Condition | Severity |
|-------|------------------|----------|
| Heater Runaway | temperature > 150 °C OR z_temp > 4 | Critical |
| Overpressure | pressure > 6 bar | Critical |
| Low Flow | flow_out < 150 L/min | Major |
| High Vibration | vibration > 1.0 g | Major |
| Quality Drop | Q < 80 (warn), Q < 60 (critical) | Warning/Critical |

---

## 8. Output Structure

Example JSON:
```json
{
  "timestamp": "2025-12-05T05:00:12Z",
  "temperature": 120.3,
  "pressure": 4.6,
  "flow_in": 200.0,
  "flow_out": 199.0,
  "density": 0.8598,
  "vibration": 0.05,
  "level": 60.4,

  "residence_time_min": 5.025,
  "quality_index": 99.51,

  "moving_avg": {
      "temperature": 120.12,
      "density": 0.86002
  },
  "stddev": {
      "temperature": 0.45,
      "density": 0.0008
  },
  "z_scores": {
      "temperature": 0.42,
      "density": -0.275
  },
  "alarms": []
}
```

---

## 9. Built-In Scenario Behavior

### ✔ Normal Steady State  
- Stable operation  
- Q near 100  

### ✔ Gradual Drift Scenario  
- Density or flow changes slowly  
- Z-scores rise, Q decreases  

### ✔ Fault Scenario  
- Sudden temperature spike  
- Pressure increase  
- Flow collapse  
- Multiple alarms trigger  

---

## 10. Summary
This simulation behaves like a lightweight digital twin useful for:
- Edge analytics  
- Fault detection  
- Process monitoring  
- Soft-sensor modeling  
- Operator training  
- PoC demonstrations  

