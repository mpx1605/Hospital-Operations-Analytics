# 🏥 Hospital Operations & Quality Analytics (CMS Data)



---
## Overview
This project builds an **end-to-end healthcare analytics and machine learning pipeline**
using publicly reported hospital quality data from the  
**Centers for Medicare & Medicaid Services (CMS)**.

The project demonstrates:
- Healthcare data engineering with real-world CMS data
- Advanced SQL analytics and database design
- Feature engineering for machine learning
- PyTorch-based modeling with proper evaluation for imbalanced data

This repository is designed as a **portfolio-quality healthcare analytics project**.

---

## Business Questions
- How do hospital quality metrics vary across hospitals and states?
- Which measures show the greatest variability in hospital performance?
- Can hospital timeliness and infection metrics predict CMS overall ratings?
- How should class imbalance be handled in healthcare ML problems?

---

## Data Source
**CMS Provider Data – Hospitals**

Datasets used:
- Hospital General Information
- Timely & Effective Care Measures
- Healthcare Associated Infections

Hospitals accepting Medicare or Medicaid are required to report these metrics quarterly.

---

## Data Scale
- **5,421 hospitals**
- **66 quality measures**
- **138,182 timeliness records**
- **172,476 infection records**
- **2,869 hospitals with labeled CMS overall ratings**

---

## Tech Stack

### Data Engineering
- PostgreSQL
- Python (Pandas, SQLAlchemy)

### Analytics
- Advanced SQL (CTEs, window functions, ranking & variance analysis)

### Machine Learning
- PyTorch
- scikit-learn (preprocessing & evaluation)

---

## Database Schema
- `dim_hospital`: hospital metadata and CMS comparison groups
- `dim_measures`: standardized quality measures
- `fact_quality_metrics`: timeliness & effective care scores
- `fact_infections`: healthcare-associated infection metrics
- `model_hospital_features`: engineered hospital-level ML features

---

## Feature Engineering
Hospital-level features include:
- Mean, standard deviation, and count of timeliness scores
- Mean, standard deviation, and count of infection scores
- Filtering applied to remove non-rate infection measures (e.g., patient days)

---

## Machine Learning Results

### Baseline Model
- Accuracy: ~0.33  
- Weighted F1: ~0.19  
- Issue: predictions collapsed to the majority class (rating = 3)

### Improved Model (Class-Weighted Loss)
- Accuracy: ~0.21  
- **Weighted F1: ~0.21**
- Predictions across **all rating classes (1–5)**
- Improved recall for minority classes (ratings 1 and 5)

**Key Insight:**  
Accuracy alone is misleading in imbalanced healthcare datasets.  
Class-weighted loss produced a more balanced and clinically meaningful model.

---

## Example SQL Analysis
```sql
WITH ranked AS (
  SELECT
    state,
    facility_name,
    hospital_overall_rating,
    RANK() OVER (PARTITION BY state ORDER BY hospital_overall_rating DESC) AS rnk
  FROM dim_hospital
)
SELECT *
FROM ranked
WHERE rnk <= 10;
```
Average Hospital Rating by State
```

SELECT
  state,
  ROUND(AVG(hospital_overall_rating), 2) AS avg_rating,
  COUNT(*) AS hospital_count
FROM dim_hospital
WHERE hospital_overall_rating IS NOT NULL
GROUP BY state
ORDER BY avg_rating DESC;
```
#Lowest-Ranked Hospital in Each State
```
WITH national_avg AS (
  SELECT AVG(hospital_overall_rating) AS avg_rating
  FROM dim_hospital
  WHERE hospital_overall_rating IS NOT NULL
)
SELECT
  h.state,
  h.facility_name,
  h.hospital_overall_rating
FROM dim_hospital h
CROSS JOIN national_avg n
WHERE h.hospital_overall_rating < n.avg_rating
ORDER BY h.hospital_overall_rating;

```

# Environment Setup
```python
-m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

# Create Database Schema
```
psql hospital_analytics -f sql/schema.sql
```
# Load CMS Data
```
python src/load_to_postgres.py
```
# Train Model
```
python src/train_rating_model.py
```
