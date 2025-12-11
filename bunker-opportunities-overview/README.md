## 📊 Dashboard Preview

<p align="center">
  <img src="possible-bunker-opportunities.png" width="1500">
</p>
<p align="center"><i>Dashboard preview with sensitive vessel and operator details blurred for confidentiality.</i></p>

---

# Bunker Opportunities Overview – Vessel Activity & Potential Demand Analysis

This dashboard identifies **potential bunker opportunities** by analyzing vessel activity, estimated arrival dates, last bunker history, and historical STS operations.  
All vessel names, IMOs, operators, and sensitive fields in this preview have been **blurred or anonymized**.

---

## 🎯 Purpose of the Model

The analytics model helps commercial and trading teams:

- Detect **vessels likely to require bunkers soon**
- Identify **recently bunkered vessels**
- Forecast **upcoming vessel arrivals**
- Prioritize high-potential commercial leads
- Analyze **historical STS operations**
- Provide a focused, high-quality **opportunity pipeline** for traders

The goal is to improve proactive outreach and increase bunker conversion rates.

---

## 📈 Key Metrics

- **Potential Bunker Opportunities**
- **Recently Bunkered Vessels**
- **Upcoming Arrivals – Next 30 Days**
- **Days Since Last Bunker**
- **High Potential Flags**
- **Confidence Level / Source**
- **Commodity & Grade Summary**

---

# 🧠 Opportunity Identification Logic (Simplified Overview)

The model determines bunker demand likelihood using several factors:

### **1️⃣ Last Bunker Date Logic**
- Compares the vessel’s last known bunker date with:
  - Typical bunker intervals
  - Vessel type consumption patterns
  - Historical cruising behavior

### **2️⃣ Destination-Based Prediction**
- If a vessel is headed toward a major bunker port with no recent bunker event → **increases potential**.

### **3️⃣ Vessel Type Consumption Profile**
Fuel consumption varies widely by vessel class:
- High burn rate → Container / Tanker / VLCC  
- Lower burn rate → Bulk Carrier / LPG  
Scoring adjusts accordingly.

### **4️⃣ STS Operation History**
Recent STS involvement indicates:
- Possible refueling offshore  
- Transfer operations that impact consumption  
Used as a weighting factor.

### **5️⃣ Recently Bunkered Filter**
Automatically excludes vessels that have bunkered very recently.

---

# ⭐ High Potential Engine – How It Works

The **High Potential Engine** is a custom scoring and classification model that ranks vessels by their likelihood to require bunkers soon. It combines multiple maritime signals into one unified output.

### **What the Engine Evaluates**

#### **1️⃣ Days Since Last Bunker**
- Measures time passed vs. vessel-type bunker cycle.
- Example:
  - Tanker → shorter interval → faster scoring
  - Bulk Carrier → longer interval → slower scoring

#### **2️⃣ Destination Port Analysis**
- Looks at next known port or ETA.
- If destination is a high-bunkering port (e.g., Fujairah / Singapore / Rotterdam) and last bunker was long ago → **High Potential**.

#### **3️⃣ ETA Forecast**
- Vessels arriving in the next **7–30 days** are prioritized.
- Helps traders prepare ahead of time.

#### **4️⃣ Consumption Model by Vessel Type**
Applies expected fuel burn rate:
- High consumption = climbs to High Potential faster  
- Low consumption = slower movement

#### **5️⃣ Compliance & Alert Flags**
- Operational alerts may modify scoring.
- E.g., compliance alerts, restricted operations, etc.

#### **6️⃣ STS Event Detection**
- Recent STS activity can mean:
  - A bunker event already happened → reduce score
  - Vessel preparing for a long voyage → increase score  
Engine adjusts automatically.

#### **7️⃣ Recently Bunkered Filter**
- Overrides all scoring and marks vessel as **Not a Lead**.

---

### **🏁 Engine Output Categories**

- **High Potential — Upcoming Bunker Need**
- **Medium Potential — Monitor**
- **Recently Bunkered — Excluded**
- **Not a Lead — Low Priority**

This system allows traders to focus immediately on vessels with the highest probability of needing bunkers.

---

# 🗂 Dashboard Components

### **1️⃣ Port-Level Opportunity Summary**
Summarizes:
- Total opportunities  
- Recently bunkered vessels  
- Opportunity distribution by port or region  

---

### **2️⃣ Upcoming Bunker Opportunities**
Detailed table including:
- Vessel characteristics  
- ETA and destination  
- Last bunker date and port  
- Days since last bunker  
- Opportunity classification  
- High Potential indicators  
- Compliance alerts  

---

### **3️⃣ Possible Bunker Locations**
Predicts which ports are likely to be considered by vessels soon.

---

### **4️⃣ Historic STS Operations**
Displays synthetic, safe STS data including:
- Operation dates  
- Vessel types  
- Cargo types  
- Operators  
Sensitive real-world identifiers are blurred.

---

# 🛠 Tools & Technologies

- **Power BI Desktop**
- **Power Query (M)** – data transformation
- **DAX** – scoring logic, date intervals, custom KPIs
- **Synthetic & blurred datasets** (for portfolio safety)
- **Maritime domain logic** (ETA, bunker cycles, STS activity)

---

# ✔ Notes

- All vessel names, IMOs, operators, and dates are **blurred** for confidentiality.  
- Underlying operational data is not included due to sensitivity and volume.  
- This dashboard is a **representation** of an analytical model used in maritime/bunkering workflows.  
- The scoring logic is simplified for portfolio clarity while demonstrating strong analytical capability.

