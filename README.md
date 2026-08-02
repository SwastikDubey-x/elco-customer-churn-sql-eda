# 📊 Telco Customer Churn & Revenue Loss Analysis (SQL & Python EDA)

## 📌 Executive Summary
Customer churn is one of the most critical metrics for subscription-based businesses. This project delivers an end-to-end data analytics workflow using **SQL (SQLite)** and **Python (Pandas, Seaborn)** to inspect, clean, and analyze customer retention patterns across **7,000+ subscriber records**.

The primary objective was to uncover structural churn drivers and quantify monthly revenue leaks for executive decision-making.

---

## 🛠️ Tech Stack & Key Concepts
* **Database & Querying:** SQLite, Advanced SQL (`CASE WHEN`, CTEs, Window Functions: `DENSE_RANK`, `AVG OVER`)
* **Data Processing & EDA:** Python, Pandas, NumPy
* **Data Visualization:** Seaborn, Matplotlib
* **Environment:** Google Colab / Jupyter Notebooks

---

## 💡 Key Business Insights

1. **Contract Type Drives Retention:** 
   * **Month-to-month contracts** suffer from a staggering **42.71% churn rate**, accounting for **$120,847.10** in lost monthly recurring revenue (MRR).
   * By contrast, **Two-year contracts** demonstrate exceptional stability with only a **2.83% churn rate**.
2. **Financial Revenue Impact:** Month-to-month cancellations account for over **89% of total lost monthly revenue**, indicating that converting users to long-term plans is the single highest-leverage retention strategy.
3. **High-Value Churn Behavior:** Using SQL window functions (`DENSE_RANK()`), we identified specific high-paying accounts in each tier who churned, revealing patterns in payment methods and service downgrades prior to cancellation.

---

## 🧹 Data Cleaning & Pipeline Setup

Raw data imported from transactional sources often contains incorrect data types and hidden blank strings. 

### Schema Sanitization & Handling Dirty Values
The `TotalCharges` field was originally imported as `TEXT` due to blank space characters (`' '`) for zero-tenure customers. A clean dynamic **SQL View** was established to cast data types safely without mutating raw tables:

```sql
CREATE VIEW clean_customer_churn AS
SELECT 
    customerID,
    Contract,
    PaymentMethod,
    MonthlyCharges,
    CAST(
        CASE 
            WHEN TRIM(TotalCharges) = '' THEN '0.0'
            ELSE TotalCharges 
        END AS REAL
    ) AS TotalCharges,
    Churn
FROM customer_churn;
```
## 🔍 Advanced SQL Queries & Analysis
1. **Aggregating Revenue Loss by Contract Tier**
Used conditional aggregation (SUM(CASE WHEN...)) to evaluate churn percentage and MRR impact simultaneously across contract types:
```
SELECT 
    Contract,
    COUNT(customerID) as Total_Customers,
    SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) as Churned_Customers,
    ROUND(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(customerID), 2) as Churn_Rate_Percentage,
    ROUND(SUM(CASE WHEN Churn = 'Yes' THEN MonthlyCharges ELSE 0 END), 2) as Lost_Monthly_Revenue
FROM clean_customer_churn
GROUP BY Contract
ORDER BY Churn_Rate_Percentage DESC;
```
2. **Window Functions: Ranking High-Value Churners**
Applied PARTITION BY and DENSE_RANK() within a Common Table Expression (CTE) to benchmark churned customer billing against tier averages:
```
WITH RankedChurners AS (
    SELECT 
        customerID,
        Contract,
        MonthlyCharges,
        ROUND(AVG(MonthlyCharges) OVER(PARTITION BY Contract), 2) as Avg_Contract_MonthlyCharge,
        ROUND(MonthlyCharges - AVG(MonthlyCharges) OVER(PARTITION BY Contract), 2) as Diff_From_Avg,
        DENSE_RANK() OVER(PARTITION BY Contract ORDER BY MonthlyCharges DESC) as Charge_Rank
    FROM clean_customer_churn
    WHERE Churn = 'Yes'
)
SELECT * 
FROM RankedChurners
WHERE Charge_Rank <= 3;
```
## 📈 Visual Exploratory Data Analysis
```
# Code snippet used to generate Seaborn bar plots comparing churn rates
sns.barplot(data=contract_data, x='Contract', y='Churn_Rate', palette='Reds_d')
```
## 🚀 Strategic Recommendations
+ Incentivise Plan Upgrades: Offer annual plan discounts to Month-to-Month subscribers within their first 60 days to lock in longer tenure.

+ Targeted Interventions: Set up automated billing alerts for high-value accounts whose monthly bills deviate significantly from tier averages.

