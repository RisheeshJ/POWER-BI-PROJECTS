# Power BI Project: Analyzing Short-Term Trading vs. Long-Term Investment Strategies

## 📌 Overview
This project compares **short-term trading** (leveraging market volatility) and **long-term investing** (compounding-based strategies) using Power BI. It analyzes stock data from six Indian companies—**Reliance, Adani, Infosys, ITC, HDFC, and Tata**—across diverse sectors to identify patterns, risks, and returns. The dashboards provide actionable insights for investors to make data-driven decisions.

**Key Questions Addressed**:
- Which strategy (short-term vs. long-term) yields higher returns?
- How does volatility impact returns?
- Which companies/sectors perform best under different market conditions?

---

## 🛠️ Technical Stack
- **Tools**: Power BI, Excel, DAX
- **Data Sources**: Kaggle (historical stock data), Yahoo Finance API (sector volatility)
- **Key Skills**: Data modeling, DAX measures, API integration, dynamic dashboards

---

## 📂 Data Sources & Preparation
### 1. **Data Collection**
- **Stock Data**: Downloaded from Kaggle (2003–2022) for six companies. Ensured high usability scores for accuracy.
- **Sector Volatility**: Integrated via **Yahoo Finance API** for real-time sector risk analysis.

### 2. **Data Cleaning (Excel)**
- Removed duplicates, missing values, and inconsistencies.
- Added calculated columns:
  - **Daily Return**: `(Today’s Close - Yesterday’s Close) / Yesterday’s Close`
  - **Percentage Daily Return**: `Daily Return * 100`
  - **1 + Daily Return**: For compounding calculations.

### 3. **Power BI Data Model**
- Created a **centralized date table** (1 Feb 2003 – 31 Dec 2022) to standardize timelines.
- Linked all company tables to the date table using relationships.
- Built separate tables for **measures** (e.g., volatility, returns) and **Sharpe Ratios** to keep the model organized.

---

## 🔑 Key DAX Measures & Formulas
### 1. **Date Difference (Excluding Weekends)**
```dax
date_diffy = 
VAR StartDate = MIN('date_table'[Date])
VAR EndDate = MAX('date_table'[Date])
RETURN 
    SUMX(
        FILTER(
            CALENDAR(StartDate, EndDate),
            WEEKDAY([Date], 2) < 6
        ), 1
    )

    <!-- 2. Volatility (Exponentially Weighted Moving Average - EWMA)
Used to quantify risk for individual companies. EWMA assigns higher weight to recent data.
Formula for HDFC: -->

Volatility_HDFC = 
VAR TradingDays = [datediffy]  
VAR Lambda = EXP(-2 / TradingDays)  -- Decay factor
VAR ValidReturns = FILTER(HDFC_FINAL, HDFC_FINAL[product] > 0)  
VAR LogReturns = AVERAGEX(ValidReturns, LN(HDFC_FINAL[product]))  
VAR StdrDev = STDEVX.P(ValidReturns, LN(HDFC_FINAL[product]))  
VAR EWMA = SQRT((1 - Lambda) * SUMX(ValidReturns, POWER(LN(HDFC_FINAL[product]), 2)))  
RETURN IF(TradingDays > 0, (StdrDev * SQRT(TradingDays)) * EWMA, BLANK())

<!-- 3. Aggregate Volatility (Root Mean Square - RMS)
Measures combined volatility across all six companies. -->

Aggregate_Volatility_RMS = 
VAR CountStocks = 6  
VAR RMS_Volatility = SQRT( 
    (POWER([Volatility_Adani], 2) + 
    POWER([Volatility_HDFC], 2) + 
    POWER([Volatility_Infosys], 2) + 
    POWER([Volatility_ITC], 2) + 
    POWER([Volatility_Reliance], 2) + 
    POWER([Volatility_Tata], 2) ) / CountStocks 
)
RETURN RMS_Volatility


<!-- 4. Short-Term Returns (Linear Growth)
Example for Tata: -->

shortreturn_tata = 
VAR DailyReturnPercentage = AVERAGEX(TATA_FINAL, TATA_FINAL[TATA_%daily] / 100)  
VAR TradingDays = [datediffy]  
VAR TotalReturn = DailyReturnPercentage * TradingDays  
RETURN ('ENTER AMOUNT FOR ANALYSIS'[Value]) * (1 + TotalReturn)

<!-- 5. Long-Term Returns (Compounding)
Example for Adani: -->

longreturn_adani = 
VAR TradingDays = [datediffy] 
VAR tab = FILTER(ADANI_FINAL, ADANI_FINAL[Date] >= MIN(ADANI_FINAL[Date]))
VAR tab_filtered = TOPN(TradingDays, tab, ADANI_FINAL[Date], ASC)  
VAR compound = PRODUCTX(tab_filtered, ADANI_FINAL[product]) 
RETURN compound * 'ENTER AMOUNT FOR ANALYSIS'[Value]

<!-- 6. CAGR (Compound Annual Growth Rate) -->

CAGR_HDFC = 
VAR StartPrice = FIRSTNONBLANK(HDFC_FINAL[Close], 1)
VAR EndPrice = LASTNONBLANK(HDFC_FINAL[Close], 1)
VAR DaysCount = [datediffy] 
VAR YearsCount = DaysCount / 252 
RETURN IF(YearsCount > 0, ( (EndPrice / StartPrice) ^ (1 / YearsCount) ) - 1, BLANK())

<!-- 7. Sharpe Ratio (Risk-Adjusted Returns) -->

HDFC_Sharpe_Yearly = 
VAR RiskFreeRate = 0
VAR AvgReturn = CALCULATE(AVERAGE(HDFC_FINAL[HDFC_/daily]), ALLEXCEPT('date_table', 'date_table'[Date].[Year]))
VAR AnnualizedReturn = AvgReturn * 252
VAR AnnualizedVolatility = [Volatility_HDFC]
RETURN DIVIDE(AnnualizedReturn - RiskFreeRate, AnnualizedVolatility, BLANK())

<!-- 📊 Dashboards
1. Investment Strategy Comparison
Features:

Date Range & Investment Amount Sliders (₹5,000 – ₹500,000).

Cards: Total Returns (Short vs. Long), Percentage Gains, Risk Level (Low/Medium/High).

Line Chart: Yearly Returns Comparison (Short-Term vs. Long-Term).

Funnel Chart: Company-wise CAGR Rankings.

Key Insight: Long-term returns dominate due to compounding, despite short-term spikes.

2. Sector Volatility Analysis
Features:

Area Charts: Company vs. Sector Volatility (data from Yahoo Finance API).

Cards: Sector-wise Average Volatility.

Nifty Risk Level: Market-wide risk indicator.

Key Insight: High volatility sectors (e.g., Adani) offer higher short-term returns but higher risk.

3. Sharpe Ratio Analysis
Features:

Table: Year-wise Sharpe Ratios for All Companies.

Pie Chart: Contribution of Each Company’s Sharpe Ratio.

Multi-Row Card: Average Sharpe Ratio for Selected Period.

Key Insight: Companies with Sharpe Ratio > 1 are ideal for risk-averse investors.

4. Company-Specific Dashboards
Features:

Pie Chart: Short-Term vs. Long-Term Profit Split.

Area Charts: Long-Term/Short-Term Returns Over Years.

Scatter Plot: Volatility vs. Daily Return (Negative Correlation).

Multi-Row Cards: CAGR, Risk Level, Sharpe Ratio.

📈 Key Insights
Long-Term Dominance:

Example: Adani’s long-term return (2002–2022) = ₹685.65M vs. short-term = ₹7.33M.

Compounding drives exponential growth over time.

Volatility Impact:

Short-term returns fluctuate wildly in high-volatility sectors (e.g., Adani).

CAGR & Sharpe Ratio:

Companies with higher CAGR (e.g., Tata) and Sharpe Ratio > 1 (e.g., HDFC) are safer long-term bets.

Sector Trends:

IT (Infosys) showed stable growth, while conglomerates (Reliance) had higher volatility.

🚀 How to Use This Project
Prerequisites:

Power BI Desktop (latest version).

Basic understanding of stock metrics.

Steps:

Download the .pbix file and datasets from the Data folder.

Open the .pbix file in Power BI.

Use the sliders to adjust dates and investment amounts.

Explore dashboards using the navigation pane. -->

├── Data                   
│   ├── Company_Stock_Data     # Cleaned CSV/Excel files
│   └── Sector_Volatility      # Yahoo Finance API data
├── PowerBI_File             
│   └── Investment_Analysis.pbix  
├── Media                   
│   ├── Screenshots           # Dashboard images
│   └── Demo_Video.mp4        # Walkthrough video
└── README.md                


<!-- 🔮 Future Enhancements
Add real-time stock data via APIs.

Include macroeconomic indicators (GDP, inflation).

Machine Learning integration for predictive analytics. -->

🙌 Connect
For questions or collaborations, reach out on www.linkedin.com/in/risheesh-jain-95871a135 or risheeshj@gmail.com.