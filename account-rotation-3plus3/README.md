# Account Rotation (3+3 Model) – Customer Engagement Analysis

This project demonstrates a 3+3 account rotation model used for customer engagement tracking and risk identification.  
All data in this project is fully **anonymized** while keeping the business logic realistic.

---

## 🔍 Purpose of the Model

The goal of this model is to understand:

- Which customers are **Healthy**
- Which customers are **Rotatable**
- Which customers are **At Risk**
- Which customers have been **Reassigned**
- Which customers require **Immediate Trader Action**

This helps commercial teams prioritize efforts, reduce churn, and maintain strong customer relationships.

---

## 📊 Key Features of the Report

### **1. Customer Classification**

Customers are categorized based on recent activity:

- **Healthy** — Active in the last 90 days  
- **Rotatable** — No activity in the last 90 days  
- **At Risk** — Declining engagement signals  
- **New Accounts** — Newly added customers  
- **Reassigned Accounts** — Trader changed  

---

### **2. Engagement Metrics**

For each customer, the report provides:

- Enquiry Count  
- Deal Count  
- Strike Rate  
- Volume (and trend)  
- Last Enquiry Date  
- Last Deal Date  
- Activity within rotation window  
- Days until rotation  
- Recommended action  

---

### **3. Trader Workload View**

Shows:

- How many accounts each trader must follow up with  
- How many customers are **At Risk**  
- Reassigned customers  
- Priority levels  

---

### **4. Rotation Timeline**

Visualizes customer distribution across:

- **0–30 days**  
- **31–60 days**  
- **61–90 days**  
- **90+ days (overdue)**  

This helps managers understand the urgency and volume of pending actions.

---

## 🧠 3+3 Rotation Logic (Summary)

The customer is evaluated across two consecutive 3-month engagement windows:

### **What is the 3+3 Model?**

The **3+3 rotation model** evaluates customer engagement using two activity periods:

#### **1. First 3 months — Enquiry Window**
This period checks whether the customer has made **any enquiries** in the last 90 days.

#### **2. Next 3 months — Deal Window**
If a customer had enquiries, this second window checks whether any **deals** were concluded in the following 90 days.

---

### **How Customers Are Classified**

A customer is considered **Healthy** if they show engagement in either window:

- An enquiry in the last 3 months, **or**  
- A deal in the following 3 months.

If a customer shows **no activity for a full 6-month cycle**, they become:

- **Rotatable**, or  
- **At Risk**, depending on secondary indicators (volume trend, engagement level, priority, etc.)

This model ensures that **no account becomes neglected**, helping traders maintain consistent follow-ups and relationship quality.

---

## 🛠 Tools & Technologies

- **Power BI Desktop**  
- **Power Query (M)**  
- **DAX Measures**  
- **Synthetic, Anonymized CSV Datasets**

---

## 📂 Files Included

- `Account-Rotation-3plus3.pbix` — Full Power BI report  
- `datasets/Customers.csv`  
- `datasets/Deals.csv`

---

## ✔ Notes

This project is intended for **portfolio demonstration** only.  
All customer names, trader names, and sensitive information have been replaced with **synthetic anonymized values**.
