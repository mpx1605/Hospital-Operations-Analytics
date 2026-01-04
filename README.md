# Hospital-Operations-Analytics
SELECT
service_line,
AVG(DATEDIFF(day, admit_date, discharge_date)) AS avg_length_of_stay
FROM admissions
WHERE discharge_date IS NOT NULL
GROUP BY service_line
ORDER BY avg_length_of_stay DESC;


WITH occupied_beds AS (
SELECT
CAST(assign_date AS DATE) AS occupancy_date,
COUNT(DISTINCT bed_id) AS occupied_beds
FROM bed_assignments
WHERE release_date IS NULL OR release_date >= assign_date
GROUP BY CAST(assign_date AS DATE)
),


total_beds AS (
SELECT COUNT(*) AS total_beds
FROM beds
WHERE is_active = 1
)
SELECT
o.occupancy_date,
o.occupied_beds,
t.total_beds,
CAST(o.occupied_beds AS FLOAT) / t.total_beds AS occupancy_rate
FROM occupied_beds o
CROSS JOIN total_beds t
ORDER BY o.occupancy_date;


WITH ranked_admits AS (
SELECT
patient_id,
admission_id,
admit_date,
LEAD(admit_date) OVER (
PARTITION BY patient_id
ORDER BY admit_date
) AS next_admit_date
FROM admissions
)
SELECT
COUNT(*) AS readmissions_30_day
FROM ranked_admits
WHERE DATEDIFF(day, admit_date, next_admit_date) BETWEEN 1 AND 30;


WITH patient_admits AS (
SELECT
patient_id,
service_line,
admit_date,
LEAD(admit_date) OVER (
PARTITION BY patient_id
ORDER BY admit_date
) AS next_admit
FROM admissions
)
SELECT
service_line,
COUNT(*) AS total_discharges,
SUM(
CASE
WHEN DATEDIFF(day, admit_date, next_admit) BETWEEN 1 AND 30
THEN 1 ELSE 0
END
) AS readmissions_30_day,
CAST(
SUM(CASE WHEN DATEDIFF(day, admit_date, next_admit) BETWEEN 1 AND 30 THEN 1 ELSE 0 END)
AS FLOAT
) / COUNT(*) AS readmission_rate
FROM patient_admits


SELECT
service_line,
COUNT(*) AS total_admissions
FROM admissions
GROUP BY service_line
ORDER BY total_admissions DESC;
GROUP BY service_line;


SELECT
COUNT(*) AS total_records,
SUM(CASE WHEN discharge_date IS NULL THEN 1 ELSE 0 END) AS missing_discharge_dates,
CAST(
SUM(CASE WHEN discharge_date IS NULL THEN 1 ELSE 0 END)
AS FLOAT
) / COUNT(*) AS missing_rate
FROM admissions;
