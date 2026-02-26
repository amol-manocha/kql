# KQL Queries — Microsoft Fabric Eventhouse

A collection of **Kusto Query Language (KQL)** scripts designed to run in **Microsoft Fabric Eventhouse**.  
Each script builds a complete analytics pipeline — table schemas, update policies, functions, materialized views, and validation queries — ready to paste into the Eventhouse query editor.

## Workspace / Eventhouse

| Setting | Value |
|---------|-------|
| Eventhouse | `jsonload` |
| Workspace  | `telematics json` |

## Available Pipelines

### [`MPH_commented.kql`](MPH_commented.kql) — GPS Speed (MPH)

Computes **vehicle speed in miles per hour** from raw GPS coordinate data collected by a smartphone mounted in a moving vehicle.

**Data flow:**
```
RawTelematicsData → GPS → MPH_detail → MPH_summary (materialized view)
```

**Key steps:**
1. Create `MPH_detail` table
2. Create the `ComputeMPH` function (geodesic distance + time delta)
3. Attach update policy to `GPS` → populates `MPH_detail` automatically
4. Create `MPH_summary` materialized view (per-trip min/avg/max speed)
5. Backfill historical data
6. Validation queries

**Speed calculation:** `geo_distance_2points()` (WGS-84 geodesic) ÷ elapsed seconds × 2.2369 (m/s → mph).  
Readings above 120 mph are nulled out as GPS noise.

---

### [`Quaternion_commented.kql`](Quaternion_commented.kql) — Driving Events Detection

Detects **dangerous driving events** (sharp turns, hard stops, aggressive acceleration) from smartphone IMU quaternion data.

**Data flow:**
```
RawTelematicsData → Quaternion → DrivingEvents_detail → DrivingEvents_summary (materialized view)
```

**Detected events:**

| Event | Condition |
|-------|-----------|
| Sharp Turn | \|yaw_rate\| > 20 °/s |
| Hard Stop (braking) | pitch_rate > 15 °/s |
| Aggressive Acceleration | pitch_rate < −15 °/s |

**Key steps:**
1. Create `DrivingEvents_detail` table
2. Create `ComputeQuaternionDelta` function (sign-normalised relative rotation → angular rates)
3. Attach update policy to `Quaternion` → populates `DrivingEvents_detail` automatically
4. Create `DrivingEvents_summary` materialized view (per-trip event counts)
5. Backfill historical data
6. Validation queries

---

## How to Use

1. Open your **Microsoft Fabric Eventhouse** in the [Fabric portal](https://app.fabric.microsoft.com).
2. Navigate to the **`jsonload`** Eventhouse inside the **`telematics json`** workspace.
3. Open the **Query editor**.
4. Open the desired `.kql` file and paste its contents into the editor.
5. Run each numbered section **one at a time, in order**, as described in the comments at the top of each file.
6. Run the **Validation queries** at the end of each file to verify correctness.

> **Tip:** Use `Ctrl+K Ctrl+C` to comment/uncomment blocks in the Fabric query editor.

## Prerequisites

- The source table **`RawTelematicsData`** must exist and be populated with raw JSON from the smartphone sensor app.
- The **`GPS`** and **`Quaternion`** tables (populated by their own update policies) must exist before running these scripts. Those update policies are managed separately.

## Repository Structure

```
.
├── MPH_commented.kql          # GPS speed pipeline
└── Quaternion_commented.kql   # Driving-events detection pipeline
```
