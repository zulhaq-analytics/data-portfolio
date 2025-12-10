## 📊 Dashboard Preview

<p align="center">
  <img src="Account-Rotation-Screenshot.png" width="1200">
</p>
<p align="center"><i>Main Power BI dashboard showing customer engagement, rotation timing, trader workload distribution, and account health insights.</i></p>

---

# Account Rotation (3+3 Model) – Customer Engagement Analysis

This project demonstrates a 3+3 account rotation model used to assess customer engagement and identify accounts requiring trader follow-up.  
All data is fully **anonymized** while preserving realistic business logic.

---

## 🎯 Purpose of the Model

This model helps commercial teams understand:

- Which customers are **Healthy**
- Which customers are **Rotatable**
- Which customers are **At Risk**
- Which accounts require **immediate trader action**
- Which customers have been **Reassigned**

The goal is to improve follow-up efficiency, prevent customer inactivity, and reduce churn.

---

## 📈 Key Features of the Report

### 1️⃣ Customer Classification

Customers are categorized into:

- **Healthy** — active in the last 90 days  
- **Rotatable** — no activity in the last 90 days  
- **At Risk** — declining or inconsistent engagement  
- **New Accounts** — recently onboarded  
- **Reassigned** — moved to another trader  

---

### 2️⃣ Engagement Metrics Tracked

For each customer, the model calculates:

- Enquiry count  
- Deal count  
- Strike rate  
- Total volume and trend  
- Last enquiry date  
- Last deal date  
- Days until rotation  
- Recommended action  

These KPIs give traders a complete view of account activity.

---

### 3️⃣ Trader Workload View

Shows:

- Accounts requiring follow-up  
- At-risk customers per trader  
- Reassigned accounts  
- Engagement priority (High, Medium, Low)

This helps managers balance workload and monitor trader performance.

---

### 4️⃣ Rotation Timeline Distribution

Visual breakdown of customers by:

- **0–30 days**  
- **31–60 days**  
- **61–90 days**  
- **90+ days (Overdue)**  

This helps prioritize immediate actions and upcoming deadlines.

---

# 🧠 3+3 Rotation Logic Explained

The customer is evaluated across two sequential 3-month windows.

### 1️⃣ First 3 Months — Enquiry Window

Checks whether the customer submitted **any enquiries** in the last 90 days.

### 2️⃣ Next 3 Months — Deal Window

If enquiries were made, this period checks whether any **deals** were completed in the following 90 days.

---

### 📌 Classification Logic

A customer is considered **Healthy** if there is activity in *either* window:

- Enquiry in the past 3 months, or  
- Deal in the subsequent 3 months  

If **no activity** occurs for a full 6-month period:

- The customer becomes **Rotatable**, or  
- **At Risk**, based on declining KPIs (volume trend, strike rate, low engagement)

This system ensures **no customer becomes inactive without attention**, improving relationship management.

---

## 🛠 Tools & Technologies

- **Power BI Desktop**
- **DAX** (customer health logic, KPIs, time intelligence)
- **Power Query (M)** (data cleaning and transformation)
- **Anonymized CSV datasets**
- **Google Sheets as a cloud data source** (safe for portfolio use)

---

## 📁 Files Included

- `Account-Rotation-3plus3.pbix` – full Power BI report  
- `datasets/Customers_Anonymized.csv`  
- `datasets/Deals_Anonymized.csv`  
- `datasets/TraderLookup.csv` (if used)

---

## ✔ Notes

- This project is for **portfolio demonstration only**.  
- All customer names, trader names, volumes, dates and identifiers have been replaced with **synthetic anonymized values**.  
- No company information, internal systems, or original file paths are included.
