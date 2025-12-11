## 📊 Dashboard Preview

<p align="center">
  <img src="possible-bunker-opportunities.png" width="1200">
</p>
<p align="center"><i>Dashboard preview with sensitive vessel and operator details blurred for confidentiality.</i></p>

---

# Bunker Opportunities Overview – Vessel Activity & Potential Demand Analysis

This dashboard identifies **potential bunker opportunities** by analyzing vessel activity, estimated arrival dates, last bunker history, and historic STS operations.  
All vessel names, IMOs, operators, and sensitive fields in this preview have been **blurred or anonymized**.

---

## 🎯 Purpose of the Model

This model enables commercial and trading teams to:

- Detect **vessels likely to require bunkers soon**  
- Identify **recently bunkered vessels**  
- Prioritize high-potential opportunities  
- Forecast **upcoming vessel arrivals** at key ports  
- Analyze **historical STS movements** to identify offshore bunker activity  
- Provide traders with a focused, actionable vessel list  

The goal is to support proactive outreach and improve conversion rates in bunker sales.

---

## 📈 Key Metrics

- **Potential Bunker Opportunities**
- **Recently Bunkered Vessels**
- **Upcoming Arrivals – Next 30 Days**
- **Days Since Last Bunker**
- **High Potential Flags**
- **Confidence Level / Source Type**
- **Commodity & Grade Summary**

---

## 🧠 Opportunity Identification Logic (Simplified)

### 1️⃣ Last Bunker Date Logic  
- Each vessel’s last bunker date is compared against:  
  - Typical bunker intervals by vessel type  
  - Known consumption patterns  
  - Historical bunker behavior  

### 2️⃣ Destination-Based Analysis  
- Vessels heading toward major bunker ports with no recent bunker event are flagged.

### 3️⃣ Consumption / Vessel Type Scoring  
Different vessel classes (Container, Tanker, Bulk, LPG, etc.) have different fuel burn rates.  
This affects opportunity scoring.

### 4️⃣ STS History  
Recent STS operations often indicate fuel movement or transshipment activities.  
This helps refine opportunity predictions.

### 5️⃣ Recently Bunkered Filter  
Quickly removes vessels that have already bunkered and should not be contacted.

---

## 🗂 Dashboard Sections

### **1. Port-Level Opportunity Summary**
Shows potential and recently bunkered totals by port.

### **2. Upcoming Bunker Opportunities Table**
Includes:
- Vessel characteristics  
- ETA to destination port  
- Last bunker port + date  
- Days since last bunker  
- Compliance alerts  
- High potential indicators  

### **3. Possible Bunker Locations**
Forecasts potential ports where vessels may seek bunkers soon.

### **4. Historic STS Operations**
Shows:
- Bunker dates  
- Transfer activity  
- Vessel operators  
- Commodity & cargo details  
(Sensitive data blurred in screenshot.)

---

## 🛠 Tools & Technologies

- **Power BI Desktop**
- **DAX** (date interval logic, scoring algorithms, custom flags)
- **Power Query (M)** (data cleanup)
- **Geographic & maritime datasets** (masked for privacy)

---

## ✔ Notes

- All identifying vessel and operator information has been blurred for confidentiality.  
- This dashboard is shown for **portfolio demonstration** and does not include the underlying dataset.  
- The structure reflects real-world maritime analytics models used in bunker trading.  
