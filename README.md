# COPE Monthly Report Automation

Automated monthly impact reporting pipeline for the COPE: Counseling, Outreach, and Protection for Emergencies program that earthquake response initiative providing psychosocial support and disability inclusion services.

---

## Overview

This notebook replaces a manual monthly reporting process (SQL export, Excel cleaning, pivot table construction) with a reproducible, parameterized pipeline that automatically updating a Power BI dashboard in a single run.


---

## Output

A dated Excel workbook, source of PowerBI, containing five pivot tables and four raw data sheets:

| Sheet | Contents |
|---|---|
| Activity Counts | Session counts by PSS type for the reporting month |
| Activity by Location | Session distribution across the two container city sites |
| Beneficiaries by Activity | Unique new beneficiaries per service type, disaggregated by nationality and gender |
| Beneficiaries by Location | Same breakdown by delivery site |
| ECA Summary | Emergency cash assistance distribution by direct/indirect receipt, nationality, and gender |
| Activity Raw Data | Full attendance records |
| Member Data | Beneficiary profiles with disability coding |
| ECA Raw Data | Full ECA distribution records |

---

## Tech Stack

| Component | Tool |
|---|---|
| Data extraction | SQL Server via `pyodbc` — parameterized by reporting period |
| Data transformation | `pandas`, `NumPy` |
| Disability coding | Custom classification function (`classify_disability_code`) |
| Cumulative tracking | Anti-join against `cumulative_IDs.xlsx` |
| Report export | `openpyxl`, `pandas.ExcelWriter` |
| Interface | Jupyter Notebook with user-prompted date inputs |


---

## How to Run

```bash
pip install -r requirements.txt
jupyter notebook COPE_monthly_report.ipynb
```

When prompted, enter the first and last day of the reporting month in `YYYY-MM-DD` format. The Excel report is saved to the working directory as `COPE_Report_{Year}_{Month}.xlsx`.

---

## Analytical Design

### Disability Coding

The `classify_disability_code()` function parses the free-text `AdditionalInfo` field using a priority hierarchy of shortcodes to classify each beneficiary into one of four disability status categories:

| Code | Category |
|---|---|
| `ENGLBKMVRN` | Caregiver of a person with disability |
| `ENGLYKN` | Family member of a person with disability |
| `BKMVRN` | Caregiver (no disability flag on the beneficiary) |
| `ENGL` | Person with disability |
| *(none)* | No Code |

### Cumulative Deduplication

`cumulative_IDs.xlsx` is read at the start of each run and appended at the end. Each row records a TrID, demographic fields, service type, and reporting month. The `filter_new_beneficiaries()` function removes any TrID already present in the cumulative file for a given service type, preventing double-counting across reporting periods.

### Cross-Service Deduplication Note

Beneficiaries attending more than one service type are counted separately within each type. This is intentional: the indicator targets are defined per service type, not per individual across the full program portfolio.

---
