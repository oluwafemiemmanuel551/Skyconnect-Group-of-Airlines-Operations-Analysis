# ✈️ SkyConnect Group of Airlines — Business Intelligence Dashboard

> A comprehensive Power BI dashboard analyzing flight operations, revenue performance, delay patterns, and passenger behaviour across a fast-growing regional airline group (2022–2024).

---

## 📊 Dashboard Previews

### 1. Overview Dashboard
![Overview Dashboard](Overview_Dashboard.png)

> High-level KPIs: **1,000 flights · 1,200 passengers · ₦590M revenue · ₦136M profit · 23% profit margin**

---

### 2. Delay & Flight Analysis
![Delay and Flight Analysis](Delay_and_Flight_Analysis.png)

> Operational metrics: **241 delayed flights · 91 cancellations · 67% on-time rate · 124.22 avg delay minutes**

---

### 3. Bookings & Passenger Analysis
![Bookings and Passenger Analysis](Passengers_and_Bookings_Analysis.png)

> Passenger insights: **1,200 bookings · avg age 44.21 · Revenue peaks in October (₦73M)**

---

## 🗂️ Project Structure

```
skyconnect-airlines-dashboard/
│
├── 📁 data/
│   ├── airports.csv          # Airport metadata (ID, city, country, IATA code)
│   ├── bookings.csv          # Booking transactions (ticket price, seat class, cost)
│   ├── delays.csv            # Delay records (minutes, reason)
│   ├── flights.csv           # Flight details (airline, route, status, fare)
│   └── passengers.csv        # Passenger profiles (age, gender, loyalty status)
│
├── 📁 dashboard/
│   └── Skyconnect_Group_of_Airlines_Dashboard.pbix   # Power BI report file
│
├── 📁 previews/
│   ├── Overview_Dashboard.png
│   ├── Delay_and_Flight_Analysis.png
│   └── Passengers_and_Bookings_Analysis.png
│
└── README.md
```

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Power BI Desktop** | Dashboard design & interactive visualizations |
| **SQL (SQLite / T-SQL)** | Data exploration and KPI queries |
| **Microsoft Excel / CSV** | Raw data storage and pre-processing |
| **DAX** | Calculated measures and KPIs in Power BI |

---

## 🔍 SQL Queries Used

<details>
<summary><strong>1. Revenue, Cost & Profit by Year</strong></summary>

```sql
SELECT 
    SUBSTR(Booking_Date, 7, 4)                                      AS Year,
    COUNT(DISTINCT Flight_ID)                                        AS Total_Flights,
    COUNT(Booking_ID)                                               AS Total_Bookings,
    ROUND(SUM(CAST(Ticket_Price AS REAL)) / 1000000, 2)            AS Revenue_M,
    ROUND(SUM(CAST(Cost_NGN AS REAL)) / 1000000, 2)                AS Cost_M,
    ROUND((SUM(CAST(Ticket_Price AS REAL)) - SUM(CAST(Cost_NGN AS REAL))) / 1000000, 2) AS Profit_M,
    ROUND(100.0 * (SUM(CAST(Ticket_Price AS REAL)) - SUM(CAST(Cost_NGN AS REAL)))
          / SUM(CAST(Ticket_Price AS REAL)), 1)                     AS Profit_Margin_Pct
FROM bookings
GROUP BY 1
ORDER BY 1;
```

**Result:**

| Year | Total_Flights | Total_Bookings | Revenue_M | Cost_M | Profit_M | Profit_Margin_Pct |
|------|--------------|----------------|-----------|--------|----------|-------------------|
| 2022 | 94 | 98 | 43.64 | 33.57 | 10.07 | 23.1% |
| 2023 | 290 | 330 | 164.04 | 126.18 | 37.85 | 23.1% |
| 2024 | 273 | 318 | 154.62 | 118.94 | 35.68 | 23.1% |

> ⚠️ **Key Insight:** Revenue dropped from ₦272M (2023) → ₦250M (2024), confirming the gross profit decline noted in project notes. Profit margin held steady at 23.1%, meaning the drop was volume-driven, not a cost efficiency issue.

</details>

<details>
<summary><strong>2. Revenue & Profit by Airline</strong></summary>

```sql
SELECT 
    f.Airline_Name,
    COUNT(b.Booking_ID)                                                          AS Bookings,
    ROUND(SUM(CAST(b.Ticket_Price AS REAL)) / 1000000, 2)                       AS Revenue_M,
    ROUND(SUM(CAST(b.Ticket_Price AS REAL) - CAST(b.Cost_NGN AS REAL)) / 1000000, 2) AS Profit_M
FROM bookings b
JOIN flights f ON b.Flight_ID = f.Flight_ID
GROUP BY 1
ORDER BY 3 DESC;
```

**Result:**

| Airline | Bookings | Revenue_M | Profit_M |
|---------|----------|-----------|----------|
| EgyptAir | 196 | 115.06 | 26.55 |
| Ethiopian Airlines | 201 | 114.59 | 26.44 |
| Arik Air | 233 | 100.54 | 23.20 |
| Air Peace | 200 | 90.79 | 20.95 |
| Dana Air | 216 | 88.74 | 20.48 |
| Kenya Airways | 154 | 80.54 | 18.59 |

</details>

<details>
<summary><strong>3. Flight Status Summary</strong></summary>

```sql
SELECT 
    Flight_Status,
    COUNT(*)                              AS Count,
    ROUND(100.0 * COUNT(*) / 1000, 1)    AS Pct
FROM flights
GROUP BY 1
ORDER BY 2 DESC;
```

**Result:**

| Flight_Status | Count | Pct |
|---------------|-------|-----|
| On-Time | 668 | 66.8% |
| Delayed | 241 | 24.1% |
| Cancelled | 91 | 9.1% |

</details>

<details>
<summary><strong>4. Average Delay by Reason</strong></summary>

```sql
SELECT 
    Delay_Reason,
    COUNT(*)                                         AS Count,
    ROUND(AVG(CAST(Delay_Minutes AS REAL)), 1)       AS Avg_Delay_Min
FROM delays
GROUP BY 1
ORDER BY 3 DESC;
```

**Result:**

| Delay_Reason | Count | Avg_Delay_Min |
|--------------|-------|--------------|
| Weather | 32 | 145.3 |
| Air Traffic Control | 30 | 137.5 |
| Late Arriving Aircraft | 30 | 127.9 |
| Technical Fault | 33 | 122.2 |
| Security Check | 39 | 116.2 |
| Fueling Delay | 28 | 116.0 |
| Crew Availability | 24 | 115.8 |
| Catering Delay | 25 | 109.4 |

</details>

<details>
<summary><strong>5. Delay Severity Breakdown</strong></summary>

```sql
SELECT 
    CASE 
        WHEN CAST(Delay_Minutes AS INT) >= 120 THEN 'High (≥120 min)'
        WHEN CAST(Delay_Minutes AS INT) >= 60  THEN 'Moderate (60–119 min)'
        ELSE 'Low (<60 min)'
    END AS Severity,
    COUNT(*)                                         AS Count,
    ROUND(AVG(CAST(Delay_Minutes AS REAL)), 1)       AS Avg_Min
FROM delays
GROUP BY 1
ORDER BY 2 DESC;
```

**Result:**

| Severity | Count | Avg_Min |
|----------|-------|---------|
| High (≥120 min) | 119 | 185.7 |
| Moderate (60–119 min) | 69 | 87.6 |
| Low (<60 min) | 53 | 33.8 |

</details>

<details>
<summary><strong>6. Top 5 Revenue Routes</strong></summary>

```sql
SELECT 
    f.Origin_City || ' → ' || f.Destination_City   AS Route,
    COUNT(b.Booking_ID)                            AS Bookings,
    ROUND(SUM(CAST(b.Ticket_Price AS REAL)) / 1000000, 2) AS Revenue_M
FROM bookings b
JOIN flights f ON b.Flight_ID = f.Flight_ID
GROUP BY 1
ORDER BY 3 DESC
LIMIT 5;
```

**Result:**

| Route | Bookings | Revenue_M |
|-------|----------|-----------|
| Lagos → Nairobi | 16 | 13.30 |
| Cairo → Port Harcourt | 17 | 10.50 |
| Port Harcourt → Dar es Salaam | 16 | 10.38 |
| Port Harcourt → Lusaka | 13 | 9.91 |
| Lagos → Dakar | 17 | 9.87 |

</details>

<details>
<summary><strong>7. Bookings & Revenue by Seat Class</strong></summary>

```sql
SELECT 
    Seat_Class,
    COUNT(*)                                                         AS Bookings,
    ROUND(SUM(CAST(Ticket_Price AS REAL)) / 1000000, 2)             AS Revenue_M,
    ROUND(100.0 * SUM(CAST(Ticket_Price AS REAL)) /
          (SELECT SUM(CAST(Ticket_Price AS REAL)) FROM bookings), 1) AS Revenue_Pct
FROM bookings
GROUP BY 1
ORDER BY 2 DESC;
```

**Result:**

| Seat_Class | Bookings | Revenue_M | Revenue_Pct |
|------------|----------|-----------|-------------|
| Economy | 786 | 281.34 | 47.7% |
| Business | 303 | 201.67 | 34.2% |
| First Class | 111 | 107.23 | 18.2% |

</details>

<details>
<summary><strong>8. Revenue by Passenger Loyalty Status</strong></summary>

```sql
SELECT 
    p.Loyalty_Status,
    COUNT(DISTINCT p.Passenger_ID)                               AS Passengers,
    ROUND(SUM(CAST(b.Ticket_Price AS REAL)) / 1000000, 2)        AS Revenue_M
FROM passengers p
JOIN bookings b ON p.Passenger_ID = b.Passenger_ID
GROUP BY 1
ORDER BY 3 DESC;
```

**Result:**

| Loyalty_Status | Passengers | Revenue_M |
|----------------|------------|-----------|
| Regular | 217 | 288.38 |
| Silver | 110 | 135.55 |
| Gold | 75 | 102.63 |
| Platinum | 50 | 63.68 |

</details>

<details>
<summary><strong>9. Average Delay Minutes by Airline</strong></summary>

```sql
SELECT 
    f.Airline_Name,
    COUNT(d.Delay_ID)                                        AS Delayed_Flights,
    ROUND(AVG(CAST(d.Delay_Minutes AS REAL)), 2)             AS Avg_Delay_Min
FROM delays d
JOIN flights f ON d.Flight_ID = f.Flight_ID
GROUP BY 1
ORDER BY 3 DESC;
```

**Result:**

| Airline | Delayed_Flights | Avg_Delay_Min |
|---------|----------------|--------------|
| EgyptAir | 35 | 136.86 |
| Dana Air | 51 | 133.69 |
| Ethiopian Airlines | 40 | 127.35 |
| Kenya Airways | 38 | 122.50 |
| Air Peace | 40 | 112.22 |
| Arik Air | 37 | 110.59 |

</details>

---

## 💡 Key Insights

| # | Insight |
|---|---------|
| 📉 | **Revenue declined in 2024** (₦272M → ₦250M) but profit margin remained stable at **23%**, indicating a volume drop rather than rising costs |
| ✈️ | **66.8% on-time rate** — nearly 1 in 3 flights was either delayed or cancelled |
| 🌦️ | **Weather is the costliest delay driver** at an average of 145 minutes, followed by Air Traffic Control (138 min) |
| 🛣️ | **Lagos → Nairobi** is the top-grossing route at ₦13.3M revenue |
| 💺 | **Economy class** dominates bookings (786 of 1,200) but **Business class** contributes 34% of revenue with far fewer seats sold |
| 👤 | **Regular-tier passengers** generate the most revenue (₦288M) — an opportunity to convert them to loyalty tiers |
| 🏢 | **EgyptAir** leads in both revenue (₦115M) and average delay (136.86 min) |

---

## 📂 Data Dictionary

| Table | Key Columns | Description |
|-------|------------|-------------|
| `flights` | Flight_ID, Airline_Name, Origin_City, Destination_City, Flight_Status | Core flight records |
| `bookings` | Booking_ID, Passenger_ID, Flight_ID, Ticket_Price, Cost_NGN, Seat_Class | Revenue & cost per booking |
| `delays` | Delay_ID, Flight_ID, Delay_Minutes, Delay_Reason | Delay details |
| `passengers` | Passenger_ID, Age, Gender, Nationality, Loyalty_Status | Passenger demographics |
| `airports` | Airport_ID, City, Country, IATA_Code, Airport_Type | Airport reference data |

---

## 🚀 How to Use

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/skyconnect-airlines-dashboard.git
   ```

2. **Open the Power BI file**
   - Open `Skyconnect_Group_of_Airlines_Dashboard.pbix` in **Power BI Desktop**
   - If prompted, update the data source path to your local `/data` folder

3. **Explore the SQL queries**
   - All queries in this README are compatible with **SQLite**, **SQL Server**, and **PostgreSQL** with minor adjustments

---

## 👤 Author

**Your Name**  
Data Analyst | Power BI Developer  
[LinkedIn](https://linkedin.com/in/your-profile) · [Portfolio](https://your-portfolio.com)

---

*Built with Power BI · SQL · DAX*
