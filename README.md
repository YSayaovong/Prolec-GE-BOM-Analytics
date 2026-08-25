# Manufacturing Operations BOM & Engineering Analytics

## 1. Project Background

This independent portfolio project simulates a manufacturing analytics assignment focused on identifying BOM errors, measuring rework impact, monitoring resolution performance, and improving engineering release integrity.

Engineering and manufacturing teams generate significant operational data across BOM revisions, error logs, rework events, and resolution workflows. This data had previously been tracked in siloed spreadsheets with no unified view. This project consolidates, models, and analyzes that data to answer three critical business questions:

- **Where are errors happening, and how often?**
- **What is the measurable cost impact of those errors?**
- **How quickly is the organization resolving them — and is it improving?**

Insights and recommendations are structured around four key areas:

- **Error Trend Analysis:** Evaluation of BOM error frequency by type, department, and time period
- **Rework Cost Impact:** Quantifying the financial burden of manufacturing errors on operational margins
- **Resolution Efficiency:** Measuring average time-to-resolve across departments and error categories
- **Revision Activity:** Tracking engineering change order (ECO) volume and its relationship to downstream errors

---

## 2. Data Structure & Initial Checks

The dataset is modeled across two core operational tables — 120 BOM error records and 100 revision change log entries — capturing the full H1 2025 period.

```
BOM_Error_Data                        Revision_Change_Log
──────────────────────────            ──────────────────────────
Date             (Date)               Date             (Date)
Unit_ID          (String) ────────►   Unit_ID          (String)
Error_Type       (String)             Part_Number      (String)
Stage_Detected   (String)             Old_Revision     (String)
Department_Origin(String)             New_Revision     (String)
Time_to_Resolve_Hours (Float)         Change_Reason    (String)
Rework_Required  (Yes/No)             Rework_Required  (Yes/No)
Estimated_Impact_Cost_USD (Float)
```

| Table | Key Fields | Grain | Row Count |
|---|---|---|---|
| `BOM_Error_Data` | Unit_ID, Error_Type, Department_Origin | One row per BOM error event | 120 |
| `Revision_Change_Log` | Unit_ID, Part_Number, Old/New Revision | One row per ECO revision | 100 |

**Initial Checks Performed:**
- Verified records are event-level — Unit_IDs repeat across dates as the same unit can accumulate multiple errors
- Confirmed all dates fall within Jan–Jun 2025 with no future-dated records
- Validated all `Department_Origin` values fall within three expected categories: Engineering, Manufacturing, Procurement
- Confirmed all `Stage_Detected` values map to three known stages: Pre-Release Review, SAP Load, Manufacturing Review
- Checked `Estimated_Impact_Cost_USD` — $0 cost records are intentional (Rework_Required = No means no cost impact logged)

![BOM Error Data Table](https://github.com/YSayaovong/Manufacturing-Operations-ERP-BOM-Analytics/blob/main/assets/bom_error.png?raw=true)

![Revision Change Log Table](https://github.com/YSayaovong/Manufacturing-Operations-ERP-BOM-Analytics/blob/main/assets/revision_change_log.png?raw=true)

![Calculations Reference](https://github.com/YSayaovong/Manufacturing-Operations-ERP-BOM-Analytics/blob/main/assets/calculations.png?raw=true)

---

## 3. Executive Summary

BOM errors represent a measurable and addressable drag on manufacturing efficiency. Across H1 2025, the organization logged **120 total errors**, with **51.7% requiring rework**, incurring **$164,235 in estimated rework costs**, and averaging **3.09 hours to resolve** per incident.


| KPI | Value |
|---|---|
| Total BOM Errors | 120 |
| Rework Rate | 51.7% |
| Avg Time to Resolve | 3.09 hrs |
| Total Rework Cost Impact | $164,235 |

> **The core finding:** Error volume is spread across five error types, but Wrong Revision and Hierarchy Issue together account for **44.2% of all errors** and **40.9% of total cost ($67,217)**. Addressing the ECO handoff and revision control process is the single highest-leverage intervention available.

![Dashboard Overview](https://github.com/YSayaovong/Manufacturing-Operations-ERP-BOM-Analytics/blob/main/assets/dashboard.png?raw=true)

---

## 4. Insights Deep Dive

### 4a. Wrong Revision Is the Most Frequent Error — But Incorrect Metadata Costs the Most

**Metric:** BOM Error Count and Cost Impact by Error Type

**Finding:** Wrong Revision leads in frequency at 30 incidents (25.0% of all errors), but Incorrect Metadata carries the highest total cost at $38,725 despite occurring only 25 times (20.8%).

| Error Type | Count | % of Total | Rework Events | Total Cost |
|---|---|---|---|---|
| Wrong Revision | 30 | 25.0% | 14 | $34,114 |
| Incorrect Metadata | 25 | 20.8% | 14 | $38,725 |
| Hierarchy Issue | 23 | 19.2% | 11 | $33,102 |
| Duplicate Part | 23 | 19.2% | 14 | $32,591 |
| Missing Part | 19 | 15.8% | 9 | $25,703 |

Incorrect Metadata errors average **$2,766 per rework event** — the highest of any category. This suggests they tend to be discovered late, when correction is most expensive. Wrong Revision errors, while most frequent, are caught at lower average cost, indicating earlier detection in the process.

---

### 4b. 27.5% of Errors Still Reach the Manufacturing Floor

**Metric:** Error Count by Stage Detected

**Finding:** 39.2% of errors are caught at Pre-Release Review and 33.3% at SAP Load, meaning nearly three-quarters are intercepted before production. However, **33 errors — 27.5% — reach Manufacturing Review**, where correction disrupts active build schedules and carries the highest resolution cost.

| Stage Detected | Error Count | % of Total |
|---|---|---|
| Pre-Release Review | 47 | 39.2% |
| SAP Load | 40 | 33.3% |
| Manufacturing Review | 33 | 27.5% |

The goal should not simply be to catch more errors — it should be to catch them earlier. Each downstream stage shift multiplies both the cost and the complexity of resolution.

---

### 4c. March Spiked 154% Month-Over-Month — and the Pattern Hasn't Stabilized

**Metric:** Monthly BOM Error Count, Jan–Jun 2025

**Finding:** March 2025 recorded 33 errors — a **153.8% increase** over February's 13. The spike resolved in April (12 errors, -63.6%) but rebounded in May (23 errors, +91.7%), matching the January baseline. This oscillating pattern — not a clean recovery — suggests the underlying process is not yet stable.

| Month | Error Count | MoM Change |
|---|---|---|
| Jan 2025 | 23 | — |
| Feb 2025 | 13 | -43.5% |
| Mar 2025 | 33 | +153.8% |
| Apr 2025 | 12 | -63.6% |
| May 2025 | 23 | +91.7% |
| Jun 2025 | 16 | -30.4% |

The March spike's discrete, contained shape is characteristic of a specific event — a major ECO batch release, a new product introduction, or a process change — rather than gradual systemic deterioration. Identifying its root cause is a priority.

---

### 4d. Engineering Generates 28% More Cost Than Manufacturing Despite Similar Error Volume

**Metric:** Error Count and Cost Impact by Department of Origin

**Finding:** Manufacturing originates the most errors at 43 (35.8%), closely followed by Engineering at 42 (35.0%) and Procurement at 35 (29.2%). However, Engineering's errors carry a disproportionate cost at $62,142 — **28.4% more than Manufacturing** ($48,425) on nearly identical error counts.

| Department | Error Count | % of Total | Rework Events | Total Cost |
|---|---|---|---|---|
| Engineering | 42 | 35.0% | 24 | $62,142 |
| Manufacturing | 43 | 35.8% | 20 | $48,425 |
| Procurement | 35 | 29.2% | 18 | $53,668 |

Engineering errors average **$1,479 per incident** versus $1,126 for Manufacturing. This indicates Engineering-originated errors tend to be detected later in the process or require more complex correction — both of which compound cost.

---

### 4e. Revision Changes Carry a 53% Rework Rate — Slightly Higher Than BOM Errors Themselves

**Metric:** Revision Change Log — Rework Rate and Change Reason Distribution

**Finding:** Of 100 revision changes logged in H1 2025, **53 required rework (53.0%)** — slightly higher than the 51.7% BOM error rework rate. This is a critical signal: revision events are intended to improve the BOM, yet more than half generate downstream rework.

| Change Reason | Count | % of Revisions |
|---|---|---|
| Design Update | 28 | 28.0% |
| Error Correction | 28 | 28.0% |
| Vendor Change | 24 | 24.0% |
| Standardization | 20 | 20.0% |

Notably, **Error Correction revisions** — changes made specifically to fix existing problems — produce rework at a rate consistent with all other revision types. A correction that introduces rework is not a complete fix.

---

## 5. Recommendations

Based on the above findings, the following actions are recommended:

- **Implement a 24-hour ECO hold with mandatory cross-functional sign-off** before any revision goes active in SAP. Wrong Revision errors represent 25% of all BOM errors and historically spike after ECO releases. A structured handoff window between Engineering and Manufacturing would directly reduce the leading error category.

- **Investigate the March 2025 spike as a formal root cause analysis.** The 153.8% month-over-month increase was discrete and severe. Identifying the triggering event — a major ECO batch, new product introduction, or process change — and documenting learnings will help prevent recurrence, particularly given May's secondary rebound to 23 errors.

- **Target Incorrect Metadata errors in Engineering first.** This category carries the highest average cost per rework event ($2,766) and Engineering already accounts for 35% of all errors and 37.8% of total cost ($62,142). A BOM metadata validation checklist at the Engineering release gate would address the highest-cost error at its source.

- **Add automated validation rules at the SAP Load stage.** 27.5% of errors reach Manufacturing Review — the most disruptive and costly detection point. Automated BOM structure checks before records release to the floor could shift a meaningful portion of these late detections earlier in the pipeline at low incremental cost.

- **Require a secondary validation gate specifically for Error Correction revisions.** With 28 Error Correction change events logged and a 53% revision-wide rework rate, revisions made to fix existing problems must be validated before closure. The current process treats them no differently than routine design updates.

- **Formally join `BOM_Error_Data` and `Revision_Change_Log` via Unit_ID and date proximity** in the next data model iteration. This linkage would enable direct root-cause attribution — connecting which revision events preceded which errors — and significantly deepen the analytical value of both datasets going forward.
  
---

## Disclaimer
This project was originally developed using production data. To protect confidential and proprietary information, all datasets, identifiers, and business-specific details have been replaced with representative simulated data. The analytical approach, KPIs, workflow, and methodology accurately reflect the engineering work performed.
