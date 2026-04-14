# Customer Segmentation Analysis (RFM) using SQL Server

# 📌 Overview
This project performs customer segmentation using transaction data to help businesses better understand customer behavior and improve marketing strategies.

Using RFM (Recency, Frequency, Monetary) analysis, customers are grouped into meaningful segments such as Champions, Loyal Customers, and At Risk.

# 📚 Data Source
Monthly Sales from 01.2025 to 12.2025

# 🛠️ Tools & Technologies
- Microsoft SQL Server
- SQL Server Management Studio

# 🔄 Workflow
## 1. APPEND ALL FILES
- Load monthly sales data into SQL Server Management Studio
- Using query to combine into 1 table
```
SELECT * FROM [202501]
UNION ALL SELECT * FROM [202502]
UNION ALL SELECT * FROM [202503]
UNION ALL SELECT * FROM [202504]
UNION ALL SELECT * FROM [202505]
UNION ALL SELECT * FROM [202506]
UNION ALL SELECT * FROM [202507]
UNION ALL SELECT * FROM [202508]
UNION ALL SELECT * FROM [202509]
UNION ALL SELECT * FROM [202510]
UNION ALL SELECT * FROM [202511]
UNION ALL SELECT * FROM [202512]
```
## 2. Calculate R, F, M values
Using SQL to compute days since last order (Recency), number of orders (Frequency) and total spend (Monetary) per customer
```
WITH RFM_RANK AS (
	SELECT 
		CustomerID,
		--CURRENT_DATE as today
		DATEDIFF(DAY, MAX(orderdate), (SELECT MAX(orderdate) FROM dbo.Sale_2025)) AS Recency,
		COUNT(*) AS Frequency,
		SUM(OrderValue) AS Monetary
	FROM dbo.Sale_2025
	GROUP BY CustomerID
)
SELECT 
	*,
	ROW_NUMBER() OVER(ORDER BY Recency ASC) AS Rank_recency,
	ROW_NUMBER() OVER(ORDER BY Frequency DESC) AS Rank_frequency,
	ROW_NUMBER() OVER(ORDER BY Monetary DESC) AS Rank_monetary
FROM RFM_RANK
ORDER BY Rank_recency DESC;
```
## 3. Assign decile scores and compute aggregate RFM Score
Score each dimension on 1-10 scale using quantiles - 10=best; 1=worst
```
WITH RFM_SCORE AS (
	SELECT *,
		NTILE(10) OVER(ORDER BY Rank_recency DESC) AS R_score,
		NTILE(10) OVER(ORDER BY Rank_frequency DESC) AS F_score,
		NTILE(10) OVER(ORDER BY Rank_monetary DESC) AS M_score
	FROM dbo.RFM_RANK
)
SELECT *,
	(R_score + F_score + M_score) AS Rfm_score
FROM RFM_SCORE
ORDER BY Rfm_score DESC
```
## 4. Define RFM segments
Map composite scores into named segments: Champions, Loyal VIP, Potential Loyalists, At risk and more
```
SELECT *,
	CASE
		WHEN Rfm_score >= 28 THEN 'Champions'
		WHEN Rfm_score >= 24 THEN 'Loyal VIP'
		WHEN Rfm_score >= 20 THEN 'Potential Loyalists'
		WHEN Rfm_score >= 16 THEN 'Promising'
		WHEN Rfm_score >= 12 THEN 'Engage'
		WHEN Rfm_score >= 8 THEN 'Require Attention'
		WHEN Rfm_score >= 4 THEN 'At risk'
		ELSE 'Lost/Inactive' 
		END AS Rfm_segment
FROM dbo.RFM_SCORE
ORDER BY Rfm_score DESC;
```
# 📈 Key Findings
- A small percentage of customers (Champions) contribute a large portion of revenue
- At-risk customers show low engagement and require retention strategies
- Loyal customers have consistent purchase behavior and high potential for upselling

# 💡 Business Recommendations
- Focus on Champions with loyalty programs and exclusive offers
- Re-engage At Risk customers with targeted campaigns
- Increase revenue from Loyal Customers through upselling and cross-selling
