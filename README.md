# Trader-segmentation-performance-analysis
This project showcases a trader performance analysis built using SQL and Excel, with structured data aggregation, behavioral segmentation, and analytical insights into profitability, efficiency, and win rate.
## Project Overview

**Project Title**: Trader-segmentation-performance-analysis  
**Database**: `project`

This project demonstrates the analysis of trader performance using SQL and Excel, where key performance metrics are derived using SQL queries and traders are segmented based on activity levels. The goal is to showcase skills in metric formulation, performance evaluation, and deriving actionable insights.
![project](https://github.com/suryansh-johri/Trader-segmentation-performance-analysis/blob/main/image%20(2).png)

## Objectives

1. **Prepare trading and sentiment datasets**: Load, clean, and align historical trading data with the Fear & Greed Index CSV file at a daily level to ensure data consistency and enable sentiment-based analysis.

2. **Formulate performance metrics using SQL**: Derive key performance and behavioral metrics such as PnL, win rate, trade frequency, and profit per trade using structured SQL queries.

3. **Trader segmentation and behavioral analysis**: Segment traders based on activity and performance characteristics (e.g., frequent vs infrequent traders) and analyze how these segments behave under different market sentiment regimes (Fear vs Greed).

4. **Insight generation and visualization**: Use Excel pivot tables and clustered column charts to compare trader performance across segments and sentiment phases, producing clear, data-backed insights.

5. **Actionable strategy formulation**: Propose practical trading rules or strategies based on observed relationships between trader behavior, performance metrics, and market sentiment.

## Project Structure

### 1. Database Setup

```sql
CREATE DATABASE project;
USE project;
CREATE TABLE historical_data 
(Account VARCHAR(80),
Coin VARCHAR(10),
Execution_Price FLOAT,
Size_Tokens FLOAT,
Size_USD FLOAT,
Side VARCHAR(10),
Date date,
Time Time,
Start_Position float,
Direction varchar(80),
Closed_PnL FLOAT,
Transaction_Hash VARCHAR(80),
Order_ID BIGINT,
Crossed VARCHAR(10),
Fee FLOAT,
Trade_ID BIGINT,
Timestamp BIGINT
);
CREATE TABLE fear_greed_index (
   timestamp BIGINT,
   value INT,
   classification VARCHAR(20),
   date DATE
);
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/fear_greed_index.csv'
INTO TABLE fear_greed_index
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS;
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/historical_data.csv'
INTO TABLE historical_data
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS;
```

**Number of rows/columns**

**fear_greed_index**

Dataset contains 2644 records excluding the header row and 4 columns.

**historical_data**

Dataset contains 2,11,224 records excluding the header row and 16 columns.

**missing values / duplicates** 

**for missing values**
```sql
SELECT
    SUM(Account IS NULL) AS Account_nulls,
    SUM(Coin IS NULL) AS Coin_nulls,
    SUM(Execution_Price IS NULL) AS Execution_Price_nulls,
    SUM(Size_Tokens IS NULL) AS Size_Tokens_nulls,
    SUM(Size_USD IS NULL) AS Size_USD_nulls,
    SUM(Side IS NULL) AS Side_nulls,
    SUM(Date IS NULL) AS Date_nulls,
    SUM(Time IS NULL) AS Time_nulls,
    SUM(Start_Position IS NULL) AS Start_Position_nulls,
    SUM(Direction IS NULL) AS Direction_nulls,
    SUM(Closed_PnL IS NULL) AS Closed_PnL_nulls,
    SUM(Transaction_Hash IS NULL) AS Transaction_Hash_nulls,
    SUM(Order_ID IS NULL) AS Order_ID_nulls,
    SUM(Crossed IS NULL) AS Crossed_nulls,
    SUM(Fee IS NULL) AS Fee_nulls,
    SUM(Trade_ID IS NULL) AS Trade_ID_nulls,
    SUM(Timestamp IS NULL) AS Timestamp_nulls
FROM historical_data;

SELECT
    SUM(timestamp IS NULL) AS timestamp_nulls,
    SUM(value IS NULL) AS value_nulls,
    SUM(classification IS NULL) AS classification_nulls,
    SUM(date IS NULL) AS date_nulls
FROM fear_greed_index;
```
**output**
 historical_data,fear_greed_index does not contain any missing values

**for duplicate data**
```sql
SELECT
  Account, Coin,
 Execution_Price,
 Size_Tokens,
 Size_USD,
 Side,
 Date,
 Time,
 Start_Position,
 Direction,
 Closed_PnL,
 Transaction_Hash,
 Order_ID,
 Crossed,
 Fee,
 Trade_ID,
 Timestamp,
  COUNT(*) AS cnt
FROM historical_data
GROUP BY
    Account, Coin,
 Execution_Price,Size_Tokens,
 Size_USD,Side,
 Date,Time,
 Start_Position,Direction,
 Closed_PnL,
 Transaction_Hash,Order_ID,
 Crossed,Fee,Trade_ID,Timestamp
HAVING COUNT(*) > 1;

SELECT timestamp, value, classification, date, COUNT(*) AS cnt
FROM fear_greed_index
GROUP BY timestamp, value, classification, date
HAVING COUNT(*) > 1;
```
**output**

No duplicate data was found in either table.


### 2. Creating the key metrics

```sql
-- daily PnL per trader (or per account)

SELECT
  account,
  date,
  SUM(closed_pnl) AS daily_pnl
FROM historical_data
GROUP BY
  account,
  date
ORDER BY
  account,
  date;

-- win rate, average trade size
select * from historical_data;

SELECT
  account,
  COUNT(*) AS total_trades,
  SUM(CASE WHEN closed_pnl > 0 THEN 1 ELSE 0 END) AS wins,
  SUM(CASE WHEN closed_pnl < 0 THEN 1 ELSE 0 END) AS losses,
  SUM(CASE WHEN closed_pnl = 0 THEN 1 ELSE 0 END) AS breakeven,
  ROUND(
    100.0 * SUM(CASE WHEN closed_pnl > 0 THEN 1 ELSE 0 END) 
    / NULLIF(COUNT(*), 0), 
    2
  ) AS win_rate_percent,
  AVG(size_tokens) AS avg_trade_size_tokens,
  AVG(size_usd) AS avg_trade_size_usd
FROM historical_data
WHERE closed_pnl IS NOT NULL
GROUP BY account;

-- Leverage Distribution
SELECT
  CASE
    WHEN Direction IN ('Buy','Sell','Spot Dust Conversion')
      THEN 'Non-Leveraged'
    ELSE 'Leveraged '
  END AS leverage_type,
  COUNT(*) AS trades
FROM historical_data
GROUP BY leverage_type;

-- number of trades per day
SELECT
  date,
  COUNT(*) AS trades_per_day
FROM historical_data
GROUP BY date
ORDER BY date;

--  long/short ratio
SELECT
  account,
  COUNT(*) AS total_trades_all,
  
  SUM(CASE WHEN Direction IN ('Open Long','Close Long') THEN 1 ELSE 0 END) AS long_trades,
  SUM(CASE WHEN Direction IN ('Open Short','Close Short') THEN 1 ELSE 0 END) AS short_trades,
  ROUND(
    SUM(CASE WHEN Direction IN ('Open Long','Close Long') THEN 1 ELSE 0 END) /
    NULLIF(SUM(CASE WHEN Direction IN ('Open Short','Close Short') THEN 1 ELSE 0 END), 0),
    2
  ) AS long_short_ratio
FROM historical_data
GROUP BY account;
```

### 3. Trader segmentation and behavioral analysis

**Does performance differ between Fear vs Greed days?**

```sql
-- Prepare daily trader performance
CREATE VIEW daily_trader_perf AS
SELECT
  account,
  date,
  SUM(Closed_PnL) AS daily_pnl,
  COUNT(*) AS trades,
  ROUND(
    100.0 * SUM(CASE WHEN Closed_PnL > 0 THEN 1 ELSE 0 END) /
    NULLIF(COUNT(*), 0),
    2
  ) AS win_rate
FROM historical_data
WHERE Closed_PnL IS NOT NULL
GROUP BY account, date;

-- Join with sentiment (Fear / Greed)
CREATE VIEW daily_perf_with_sentiment AS
SELECT
  d.account,
  d.date,
  d.daily_pnl,
  d.trades,
  d.win_rate,
  f.classification AS sentiment
FROM daily_trader_perf d
JOIN fear_greed_index f
  ON d.date = f.date;

-- discriptive analysis on daily_pnl 
SELECT
  date,
  sentiment,
  daily_pnl
FROM daily_perf_with_sentiment
WHERE sentiment IN ('Fear', 'Greed');

-- discriptive analysis on win_rate
SELECT
  date,
  sentiment,
  win_rate
FROM daily_perf_with_sentiment
WHERE sentiment IN ('Fear','Greed');

-- discriptive analysis on drawdown proxy
SELECT
  sentiment,
  AVG(CASE WHEN daily_pnl < 0 THEN 1 ELSE 0 END) AS losing_day_ratio
FROM daily_perf_with_sentiment
GROUP BY sentiment;

-- Prepare data for statistical test
SELECT
  date,
  sentiment,
  CASE WHEN daily_pnl < 0 THEN 1 ELSE 0 END AS losing_day
FROM daily_perf_with_sentiment
WHERE sentiment IN ('Fear','Greed');
```
**now exporting these result and making excel file and then applying t-test to answer this que**
**features**
| Column Name | Description                                 | Type    | Example Values         |
| ----------- | ------------------------------------------- | ------- | ---------------------- |
| `date`      | Trading date / record date                  | DATE    | 2023-05-01, 2023-12-14 |
| `sentiment` | Market sentiment classification for the day | VARCHAR | Greed, Fear   |
| `daily_pnl` | Daily profit or loss value                  | FLOAT   | -205.43, 0.00, 304.98  |
 **Download**: [performance.csv](https://www.kaggle.com/datasets/nabihazahid/spotify-dataset-for-churn-analysis/data) (included in repo).





