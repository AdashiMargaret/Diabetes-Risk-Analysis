🩺 Diabetes Risk Analysis — SQL & Tableau Portfolio Project
A self-taught data analytics project exploring diabetes risk factors using SQL for data engineering and Tableau for interactive dashboards.

📌 Project Overview
Business Question: What demographic and health factors are most associated with diabetes diagnosis — and can we build a risk model to identify high risk patients?
Dataset: Pima Indians Diabetes Dataset (768 patients, 9 variables)
Tools Used: DuckDB (SQL), Tableau Public
Skills Demonstrated: Data cleaning, feature engineering, CTEs, window functions, subqueries, risk profiling, interactive dashboards

🗂️ Project Structure
Diabetes-Risk-Analysis/
│
├── SQL/
│   └── diabetes_analysis.sql        ← Full SQL pipeline
│
├── Data/
│   ├── diabetes.csv                 ← Raw dataset
│   └── Diabetes_analytics.xlsx      ← Cleaned export for Tableau
│
└── README.md

🛠️ SQL Pipeline
The SQL code builds a layered data pipeline from raw data to analytics-ready output:
diabetes_table (raw CSV import)
       ↓
diabetes_clean (nulls + zero values removed)
       ↓
diabetes_final (category columns added)
       ↓
diabetes_analytics (window functions + risk scoring)
Key SQL Techniques Used
1. Data Cleaning
sqlCREATE TABLE diabetes_clean AS 
SELECT * FROM diabetes_table 
WHERE Glucose > 0 AND Age > 0 AND BMI > 0 
  AND Outcome IS NOT NULL AND Outcome <= 1;
2. Feature Engineering with CASE Statements
sqlCASE WHEN BMI < 18.5 THEN 'Underweight'
     WHEN BMI < 25.0 THEN 'Normal'
     WHEN BMI < 30.0 THEN 'Overweight'
     ELSE 'Obese'
END AS bmi_group
3. CTE — Diagnosis Rate by BMI Group
sqlWITH BMI_SUMMARY AS (
    SELECT bmi_group,
           COUNT(*) AS total_patients,
           COUNT(CASE WHEN diagnosed = 'Y' THEN 1 END) AS diagnosed_count
    FROM diabetes_final
    GROUP BY bmi_group
)
SELECT *, ROUND((diagnosed_count * 100.0 / total_patients), 2) AS diagnosis_pct
FROM BMI_SUMMARY
ORDER BY diagnosis_pct DESC;
4. Window Functions — Glucose Ranking & Group Averages
sqlSELECT Age, age_group, Glucose, diagnosed,
    RANK() OVER (PARTITION BY age_group ORDER BY Glucose DESC) AS glucose_rank,
    ROUND(AVG(Glucose) OVER (PARTITION BY age_group), 2) AS glucose_avg,
    ROUND(Glucose - AVG(Glucose) OVER (PARTITION BY age_group), 2) AS glucose_avg_diff
FROM diabetes_final;
5. Correlated Subquery — Patients Above Group Average
sqlSELECT f.Age, f.age_group, f.Glucose, f.diagnosed
FROM diabetes_final f
WHERE f.Glucose > (
    SELECT AVG(Glucose) FROM diabetes_final
    WHERE age_group = f.age_group
);
6. Risk Profiling Logic
sqlCASE 
    WHEN glucose_category = 'Diabetic Range' 
     AND bmi_group = 'Obese' 
     AND age_group IN ('40-49', '50+')
     AND preg_group IN ('Medium (3-5)', 'High (6+)') THEN 'High Risk'
    WHEN glucose_category IN ('Diabetic Range', 'Pre-diabetic') 
     AND bmi_group IN ('Obese', 'Overweight') 
     AND preg_group IN ('Medium (3-5)', 'High (6+)') THEN 'Medium Risk'
    ELSE 'Low Risk'
END AS risk_level

📊 Tableau Dashboards

🔗 View Live Dashboards on Tableau Public ← https://public.tableau.com/app/profile/adashi.odama/viz/DiabetesRiskAnalysis-SQLTABLEAU/Dashboard1-Overview

Dashboard 1 — Overview & KPIs

Total patients, % diagnosed, average glucose, average BMI
Donut chart showing diagnosed vs not diagnosed split
Stacked bar chart of patient count by age group

Dashboard 2 — Risk Analysis

Stacked bar chart of risk level by BMI group
Heatmap crossing glucose category vs BMI group
Interactive filter by pregnancy group

Dashboard 3 — Glucose Deep Dive

Scatter plot of BMI vs Glucose colored by diagnosis
Side by side bar chart of average glucose by age group
Highlight table showing patients furthest above their group average (from window function)

Dashboard 4 — Pregnancy & Demographics

Bar chart of diagnosis rate by pregnancy group
Bubble chart showing pregnancy group size vs diagnosis rate
Line chart of average glucose across age groups split by pregnancy group


💡 Key Insights

Obese patients with Diabetic Range glucose have the highest diagnosis rate across all BMI and glucose combinations
High (6+) pregnancy patients have a 50% diagnosis rate — the highest of any pregnancy group
Diagnosed patients average 30 glucose points higher than non-diagnosed patients across every age group
50+ age group shows the steepest glucose increase with diagnosed patients averaging 153 vs 127 for non-diagnosed
Pre-diabetic + Obese combination accounts for 220 patients — the largest single risk concentration in the dataset


🚀 About This Project
This project was built as part of my self-taught data analytics journey to demonstrate practical SQL and Tableau skills. The goal was to go beyond basic queries and show a complete analytics workflow — from raw data cleaning through to interactive business dashboards.
Skills demonstrated:

SQL: CTEs, window functions, correlated subqueries, CASE logic, layered table architecture
Tableau: KPI cards, scatter plots, heatmaps, bubble charts, line charts, interactive filters
Analytics thinking: risk modelling, demographic segmentation, clinical insight


📁 Dataset Source
Pima Indians Diabetes Dataset — originally from the National Institute of Diabetes and Digestive and Kidney Diseases. Available on Kaggle.

Built by a self-taught data analyst passionate about turning raw data into actionable insights.
