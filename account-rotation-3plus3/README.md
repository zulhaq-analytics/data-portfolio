📘 Account Rotation 3+3 Model — Customer Engagement Analysis

This project demonstrates a 3+3 account rotation model used to track customer engagement and identify accounts requiring trader follow-up.
All data in this project is fully anonymized while keeping the business logic realistic.

🔍 Purpose of the Model

The goal of the 3+3 model is to understand:

Which customers are Healthy

Which customers are Rotatable

Which customers are At Risk

Which customers have been Reassigned

Which customers require Immediate Trader Action

This helps commercial teams prioritize efforts, reduce churn, and maintain strong customer relationships.

📊 Key Features of the Report
1. Customer Classification

Customers are categorized based on recent activity:

Healthy → Active in the last 90 days

Rotatable → No activity in the last 90 days

At Risk → Declining engagement signals

New Accounts → Recently added

Reassigned Accounts → Trader changed

2. Engagement Metrics

For each customer, the report provides:

Enquiry Count

Deal Count

Strike Rate

Volume (and trend)

Last Enquiry Date

Last Deal Date

Activity within rotation window

Days left until rotation

Recommended action

3. Trader Workload View

Shows:

How many accounts each trader must follow up with

How many customers are at risk

Reassigned customers

Priority levels

4. Rotation Timeline

Visualizes customer distribution across:

0–30 days

31–60 days

61–90 days

90+ days (overdue)

This helps managers understand the urgency and volume of pending actions.

🧠 3+3 Rotation Logic (Summary)

A customer is evaluated over:

3 months of recent enquiries

3 months of recent deals

Engagement is assessed to determine the account’s current health status and rotation readiness.

🛠 Tools & Technologies

Power BI Desktop

Power Query (M)

DAX Measures

Synthetic, Anonymized CSV datasets

📂 Files Included

Account-Rotation-3plus3.pbix — Full Power BI report

datasets/Customers_Anonymized.csv

datasets/Deals_Anonymized.csv

✔ Notes

This project is intended for portfolio demonstration only.
All customer names, trader names, and sensitive information have been replaced with synthetic anonymized values.
