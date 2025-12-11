## 📊 Dashboard Preview

<p align="center">
  <img src="Account-Rotation-Screenshot.png" width="1200">
</p>
<p align="center"><i>Main Power BI dashboard showing customer engagement, rotation timing, trader workload distribution, and account health insights.</i></p>

---

# Account Rotation (X+X Model) – Customer Engagement Analysis

This project demonstrates a X+X account rotation model used to assess customer engagement and identify accounts requiring trader follow-up.  
All data is fully **anonymized** while preserving realistic business logic.

---

## 🎯 Purpose of the Model

This model helps commercial teams understand:

- Which customers are **Healthy**
- Which customers are **Rotatable**
- Which customers are **At Risk**
- Which accounts require **immediate trader action**
- Which customers have been **Reassigned**
- Which vessels belonging to their customer list are **up for upcoming bunkering** (via alerts)

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

### 4️⃣ Vessel Alerts – Upcoming Bunkering Opportunities

This feature alerts traders when:

- One of **their customers' vessels** is expected to require bunkers soon  
- A vessel is approaching a port where the trader is active  
- A vessel has exceeded its **typical bunker interval**  
- A vessel has performed **recent STS activity**, suggesting increased consumption  
- A vessel has **no recent bunker record** and is within a risk threshold

Alerts appear as:

- **High Priority** (Immediate action recommended)  
- **Medium Priority** (Monitor closely)  
- **Low Priority** (Optional follow-up)

This allows traders to proactively contact customers **before** the vessel submits an enquiry, improving conversion rates.

---

### 5️⃣ Row-Level Security (RLS)

To ensure data confidentiality and personalized insights:

- **Traders only see their own customers + unassigned accounts**
- **Managers see their entire team’s accounts**
- **Leadership sees all customers**

RLS is implemented using:

- Trader table mapping  
- Role-to-customer relationship filters  
- Dynamic DAX security logic  

This ensures a secure, personalized experience that mirrors real-world operational access levels.

---

### 6️⃣ Rotation Timeline Distribution

Visual breakdown of customers by:

- **0–30 days**  
- **31–60 days**  
- **61–90 days**  
- **90+ days (Overdue)**  

This helps prioritize immediate actions and upcoming deadlines.

---

# 🧠 X+X Rotation Logic Explained

The customer is evaluated across two sequential 3-month windows.

### 1️⃣ First X Months — Enquiry Window

Checks whether the customer submitted **any enquiries** in the last 90 days.

### 2️⃣ Next X Months — Deal Window

If enquiries were made, this period checks whether any **deals** were completed in the following 90 days.

---

### 📌 Classification Logic

A customer is considered **Healthy** if there is activity in *either* window:

- Enquiry in the past X months, or  
- Deal in the subsequent X months  

If **no activity** occurs for a full 6-month period:

- The customer becomes **Rotatable**, or  
- **At Risk**, based on declining KPIs (volume trend, strike rate, low engagement)

---

## 🛠 Tools & Technologies

- **Power BI Desktop**  
- **DAX** (customer health logic, KPIs, time intelligence, alert logic)  
- **Power Query (M)** (data cleaning and transformation)  
- **Google Sheets** as a cloud data source  
- **Dynamic RLS** for secure, user-specific filtering  
- **Anonymized CSV datasets**

---

## 📁 Files Included

- `Account-Rotation-3plus3.pbix` – full Power BI report  
- `datasets/Customers_Anonymized.csv`  
- `datasets/Deals_Anonymized.csv`  
- `datasets/TraderLookup.csv` (if used)

---

## ✔ Notes

- This project is for **portfolio demonstration only**.  
- All customer names, trader names, vessel data, volumes, dates and identifiers have been replaced with **synthetic anonymized values**.  
- No company information, internal systems, or original file paths are included.  
