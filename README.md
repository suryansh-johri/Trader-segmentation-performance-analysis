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

