# BigQuery ML: Predicting Website Transactions using Google Analytics Data

## Overview
This guide demonstrates how to use **BigQuery ML (BQML)** to train and deploy a **Machine Learning model** that predicts whether a website visitor will make a transaction. The dataset comes from **Google Analytics public data** available in BigQuery.

## Prerequisites
- **Google Cloud Project** with BigQuery enabled.
- Basic knowledge of **SQL**.

---

## **Step 1: Open BigQuery Console**
1. Go to the **Google Cloud Console**: [https://console.cloud.google.com/](https://console.cloud.google.com/).
2. Navigate to **BigQuery** via the **Navigation Menu**.
3. Click **Done** on the welcome message.

---

## **Step 2: Create a Dataset**
1. In the **Explorer** panel, locate your project (e.g., `hien-elt-project`).
2. Click the **three-dot menu (⋮)** next to your project name.
3. Select **Create dataset**.
4. Set **Dataset ID** to `bqml_lab`.
5. Click **CREATE DATASET**.

![{A3E77AED-9101-4CEF-8785-7E48875036F9}](https://github.com/user-attachments/assets/95238eb4-4603-4875-8fa0-471acd3d3106)


---

## **Step 3: Explore the Data**
### **Query to Preview the Data**
```sql
#standardSQL
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews,
  totals.timeOnSite AS time_on_site,  -- New Feature
  trafficSource.source AS referral_source  -- New Feature
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170631'
LIMIT 10000;
```

### **Explanation**
- **`label`**: Target variable (1 = transaction, 0 = no transaction).
- **Features used for prediction**:
  - `os`: Operating system.
  - `is_mobile`: Whether the device is mobile.
  - `country`: Visitor’s country.
  - `pageviews`: Number of pages viewed.
  - `time_on_site`: Time spent on site (New feature).
  - `referral_source`: Traffic source (search, social, direct, etc.).
- **Time Range**: Data from **August 2016 - June 2017**.

### **Save as a View**
1. Click **Save** → **Save view**.
2. Set **Dataset** to `bqml_lab`, and name it `training_data`.
3. Click **Save**.

---

## **Step 4: Choosing the Right Machine Learning Model**
BigQuery ML provides different models for different use cases. Below are some options to choose from:

### **1️⃣ Logistic Regression (Fastest, but Simpler)**
```sql
CREATE OR REPLACE MODEL `bqml_lab.logistic_model`
OPTIONS(model_type='logistic_reg') AS
SELECT * FROM `bqml_lab.training_data`;
```
**Pros:** Fast, interpretable, good for binary classification.
**Cons:** May not capture complex relationships in data.

### **2️⃣ AutoML Classifier (Most Accurate, but Slowest)**
```sql
CREATE OR REPLACE MODEL `bqml_lab.automl_model`
OPTIONS(model_type='automl_classifier') AS
SELECT * FROM `bqml_lab.training_data`;
```
**Pros:** Automatically selects the best model.
**Cons:** Takes longer to train, requires more compute resources.

### **3️⃣ Deep Neural Network (DNN) Classifier (Balanced Performance)**
```sql
CREATE OR REPLACE MODEL `bqml_lab.dnn_model`
OPTIONS(model_type='dnn_classifier') AS
SELECT * FROM `bqml_lab.training_data`;
```
**Pros:** More powerful than logistic regression, faster than AutoML.
**Cons:** Requires more tuning than AutoML.

---

## **Step 5: Evaluating the Model**
### **Run the Evaluation Query**
```sql
#standardSQL
SELECT * FROM ml.EVALUATE(MODEL `bqml_lab.automl_model`);
```

### **Key Metrics in the Output**
- **Precision**: How accurate positive predictions are.
- **Recall**: How many actual transactions were correctly predicted.
- **F1-score**: Balance between precision and recall.
- **AUC (Area Under Curve)**: Measures the model’s ability to distinguish between classes.

---

## **Step 6: Use the Model for Predictions**

### **Prepare New Data (July 2017)**
```sql
#standardSQL
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews,
  fullVisitorId
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170701' AND '20170801';
```

### **Save as a View**
1. Click **Save** → **Save view**.
2. Set **Dataset** to `bqml_lab`, and name it `july_data`.
3. Click **Save**.

---

## **Step 7: Predict Purchases by Country**
### **Query to Predict Total Transactions by Country**
```sql
#standardSQL
SELECT
  country,
  SUM(predicted_label) AS total_predicted_purchases
FROM
  ml.PREDICT(MODEL `bqml_lab.automl_model`, (
    SELECT * FROM `bqml_lab.july_data`))
GROUP BY country
ORDER BY total_predicted_purchases DESC
LIMIT 10;
```

---

## **Step 8: Understanding Model Training Time**
### **Why is AutoML Training Slow?**
- **AutoML models** perform extensive hyperparameter tuning and feature selection.
- **Larger datasets take longer** to process.
- **Cloud quota limitations** can slow down performance.

### **Faster Alternatives**
If training time is too long, try:
1. **Using Logistic Regression (`logistic_reg`)** for quicker results.
2. **Training on a sample dataset:**
   ```sql
   CREATE OR REPLACE MODEL `bqml_lab.automl_sample`
   OPTIONS(model_type='automl_classifier') AS
   SELECT * FROM `bqml_lab.training_data`
   WHERE RAND() < 0.1;
   ```

---

## **Resources**
- [BigQuery ML Documentation](https://cloud.google.com/bigquery-ml/docs)
- [Google Cloud Public Datasets](https://cloud.google.com/public-datasets)

---

This enhanced guide helps users **choose the right model**, **understand training speed**, and **deploy machine learning efficiently in BigQuery ML**. 🚀
