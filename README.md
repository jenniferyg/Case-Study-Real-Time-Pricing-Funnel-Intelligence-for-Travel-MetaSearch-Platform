#  Case Study: Real-Time Pricing Intelligence for Travel MetaSearch (Dubai → Turkey, Ramadan Peak Season)

### **Author:** Samantha Yoong  
📍 Kuala Lumpur, Malaysia  
🔗 [LinkedIn](#) | [GitHub Portfolio](#) | [Tableau Dashboard](#)  

---

## 1. Context & Problem Statement

Wego aggregates flight prices from multiple providers, but during **peak travel seasons** like Ramadan, the platform faces:

- **High price volatility** on popular routes (e.g., Dubai → Turkey)  
- **Delayed data updates**, causing price mismatches and **conversion drops**  
- Difficulty for stakeholders to **monitor price fluctuations in real-time**  

**Objective:** Build a **real-time pricing dashboard** to track price volatility and anomalies for the Dubai → Turkey route throughout the **30-day Ramadan period**, enabling timely insights for product and operations teams.

---

## 2. Business Goal

1. **Monitor price volatility per provider** on a high-demand route.  
2. **Detect price anomalies** (stale prices, FX drift, or misfeeds).  
3. **Enable real-time alerts** for operational or pricing adjustments.  
4. **Improve user trust and conversion rate** by ensuring displayed prices are accurate and timely.

---

## 3. Data Sources & Architecture

| Source              | Type        | Key Fields                               | Frequency |
| ------------------- | ----------- | ---------------------------------------- | -------- |
| Flight API Logs     | External    | provider_id, route, price, timestamp     | Real-time |
| Booking Clickstream | Event Data  | user_id, session_id, event_type, timestamp | Continuous |
| FX Rates API        | External    | currency_pair, rate, timestamp           | Hourly    |
| Provider Metadata   | Static      | provider_id, region, reliability_score   | Monthly   |

**Cloud Environment:** BigQuery (GCP)  
**ETL / Scheduling:** Airflow (simulated)  
**Visualization:** Tableau / Looker Studio  

---

## 4. Methodology

### Step 1: Data Ingestion & Cleaning
- Pull flight pricing logs and FX data every **5 minutes**.  
- Remove stale or duplicate prices (> 3 hours old).  
- Normalize currencies using FX rates to a common base (USD).

```sql
WITH clean_prices AS (
  SELECT
    provider_id,
    route,
    price,
    timestamp,
    ROW_NUMBER() OVER (PARTITION BY provider_id, route ORDER BY timestamp DESC) AS rn
  FROM raw_flight_logs
  WHERE price IS NOT NULL
)
SELECT * FROM clean_prices WHERE rn = 1;
```
---

## Step 2: Price Volatility & Accuracy Computation

**Metrics computed per provider and timeslot (5-min interval):**

| Metric                  | Formula                                                |
| ----------------------- | ------------------------------------------------------ |
| **Price Volatility Index** | `σ(price) / μ(price) × 100`                           |
| **FX Drift %**           | `(FX_rate_feed - FX_rate_benchmark) / FX_rate_benchmark × 100` |
| **Stale Price %**        | `(Stale Listings / Total Listings) × 100`            |
| **Price Accuracy %**     | `(Accurate Prices / Total Prices Displayed) × 100`    |

> Aggregated metrics are stored in **BigQuery aggregated tables**.

---

## Step 3: Time Series Analysis

- Plot **Price Volatility Index** per provider along the 24h timeline for each Ramadan day.  
- Overlay **FX Drift %** and **Stale Price %** to identify correlation with volatility spikes.  
_Are volatility spikes partly due to stale listings being refreshed suddenly ("price jump"), rather than real market price changes?_
- Annotate **key hours** with unusual volatility or FX drift.

---

## Step 4: Real-Time Dashboard Design

**Dashboard Features:**

### Provider Heatmap
- **X-axis:** Time (30-minute intervals)  
- **Y-axis:** Provider  
- **Color:** Price Volatility Index  
- **Tooltip:** FX Drift %, Stale Price %  

### Time Series Line Chart
- **X-axis:** Time (hourly)  
- **Y-axis:** Price Volatility (%)  
- **Lines:** Multiple providers  
- **Annotations:** Highlight volatility spikes and FX drift anomalies  

### Alert Layer
- Trigger alerts if **Price Accuracy < 95%** or **FX Drift > 2%**  
- Display **top 3 providers with highest volatility**

---

## Step 5: Sensitivity Analysis

- Identify how **FX Drift impacts CTR / Search-to-Booking conversion** on this route.  
- Perform **lag analysis** to check if volatility / FX drift occurs **before CTR drop**.  
- Visualize using **dual-axis time series** or separate **table chart** for clarity.

---

## 5. Key Findings (Simulated)

| Finding                        | Insight                                   | Recommendation                                      |
| ------------------------------- | ---------------------------------------- | -------------------------------------------------- |
| Provider A volatility spiked 15:00–17:00 | FX drift + stale prices caused temporary CTR drop | Prioritize API refresh and FX update at peak hours |
| Provider B showed 8% stale prices | Pricing misfeed detected                  | Introduce automated stale price alert              |
| Peak evening hours 20:00–22:00 | Price volatility highest                  | Consider provider reliability scoring and predictive alerting |

---

## 6. Tools & Skills Demonstrated

| Category                | Tools / Skills                                           |
| ----------------------- | -------------------------------------------------------- |
| Data Management         | SQL, BigQuery, Airflow                                   |
| Visualization           | Tableau / Looker Studio                                   |
| Metrics & Analytics     | Price Volatility Index, FX Drift %, Stale Price %, Price Accuracy % |
| Pipeline Simulation     | ETL scheduling, real-time aggregation                   |
| Predictive Analysis     | Lag analysis, correlation with CTR, anomaly detection    |

---

## 7. Strategic Takeaway

- Real-time monitoring of **price volatility** and **FX drift** allows Wego to **maintain accurate pricing**, especially during peak travel seasons like Ramadan.  
- Key insights help stakeholders make **fast decisions**, **reduce CTR drops**, and **boost booking conversions**.  
- Combining **time series analytics**, **heatmaps**, and **alert thresholds** turns raw flight data into actionable intelligence.

---

### 🔑 Keywords

`Travel Analytics`, `Real-Time Dashboard`, `Price Volatility`, `FX Drift`, `Stale Prices`, `ETL Pipeline`, `Airflow`, `BigQuery`, `Tableau`, `Conversion Analysis`, `Lag Analysis`

