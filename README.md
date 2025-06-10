# sample-sales

## 📌 Overview
This project analyzes type of customer to lauch promotion campaigns.

## 📊 Tools Used
- SQL Server Management
## Code
CREATE DATABASE SalesSample

SELECT * From[dbo].[sales_sample]

-- Sales by countries
select 
    TBL.[COUNTRY],
    ROUND(SUM(TBL.[SALES]),2) as TotalSalesByCountry
from [dbo].[sales_sample] AS TBL
GROUP BY (TBL.[COUNTRY])
ORDER BY TotalSalesByCountry DESC

-- Sales by years
SELECT
    TBL.[YEAR_ID],
    TBL.[MONTH_ID],
    ROUND(SUM(TBL.[SALES]),2) AS TotalSalesByYear
FROM [dbo].[sales_sample] AS TBL
GROUP BY TBL.[YEAR_ID], TBL.[MONTH_ID]
ORDER BY TBL.[YEAR_ID] Desc

-- Total order by Status

Select 
    TBL.[STATUS],
    COUNT(TBL.[ORDERNUMBER]) AS TotalOrderByStatus
FROM [dbo].[sales_sample] AS TBL
GROUP BY TBL.[STATUS]
ORDER BY TotalOrderByStatus Desc

-- Sales by Productline

SELECT 
    TBL.[PRODUCTLINE],
    Round(SUM(TBL.[SALES]),2) AS TotalSales
FROM [dbo].[sales_sample] AS TBL
GROUP BY TBL.[PRODUCTLINE]
ORDER BY TotalSales Desc

-- Customers have the most cancelled orders

SELECT 
    TBL.[CUSTOMERNAME],
    COUNT(*) AS CancelledOrder
FROM [dbo].[sales_sample] as TBL    
WHERE TBL.[STATUS]='Cancelled'
GROUP BY [CUSTOMERNAME]
ORDER BY CancelledOrder DESC

-- Total products by productline

SELECT
    TBL.[PRODUCTLINE],
    SUM(QUANTITYORDERED) AS TotalQuantities
FROM [dbo].[sales_sample] AS TBL
GROUP BY TBL.[PRODUCTLINE]
ORDER BY TotalQuantities DESC

-- Customers with the highest sales 

Select 
    TBL.[CUSTOMERNAME],
    SUM(TBL.[QUANTITYORDERED]) AS TotalQuantities
FROM [dbo].[sales_sample] AS TBL 
GROUP BY TBL.[CUSTOMERNAME]
ORDER BY TotalQuantities DESC

-- Customers bought total products

Select 
    TBL.[CUSTOMERNAME],
    TBL.[PRODUCTLINE],
    SUM(TBL.[QUANTITYORDERED]) AS TotalQuantities
FROM [dbo].[sales_sample] AS TBL 
GROUP BY TBL.[CUSTOMERNAME], TBL.[PRODUCTLINE]
ORDER BY TotalQuantities DESC

-- Calculate RFM

WITH RFM_BASE AS
(
    SELECT
        TBL.[CUSTOMERNAME]
        ,MIN(TBL.[ORDERDATE]) AS FirstOrder
        ,MAX(TBL.[ORDERDATE]) AS LastOrder
        ,DATEDIFF(DAY, MAX(TBL.[ORDERDATE]), '2005-06-01') AS Recency
        ,COUNT(DISTINCT TBL.[ORDERNUMBER]) AS   Frequency
        ,ROUND(SUM(TBL.[SALES]),2) AS Monetary
    From[dbo].[sales_sample] AS TBL
    GROUP BY TBL.[CUSTOMERNAME]
)

,RFM_SCORE AS
(
    SELECT *
        ,CASE 
            WHEN PERCENT_RANK() OVER (ORDER BY Recency) < 0.2 THEN 5
            WHEN PERCENT_RANK() OVER (ORDER BY Recency) < 0.4 THEN 4
            WHEN PERCENT_RANK() OVER (ORDER BY Recency) < 0.6 THEN 3
            WHEN PERCENT_RANK() OVER (ORDER BY Recency) < 0.8 THEN 2
        ELSE 1
        END AS R

        ,CASE 
            WHEN PERCENT_RANK() OVER (ORDER BY Frequency) < 0.2 THEN 1
            WHEN PERCENT_RANK() OVER (ORDER BY Frequency) < 0.4 THEN 2
            WHEN PERCENT_RANK() OVER (ORDER BY Frequency) < 0.6 THEN 3
            WHEN PERCENT_RANK() OVER (ORDER BY Frequency) < 0.8 THEN 4
        ELSE 5
        END AS F

        ,CASE 
            WHEN PERCENT_RANK() OVER (ORDER BY Monetary) < 0.2 THEN 1
            WHEN PERCENT_RANK() OVER (ORDER BY Monetary) < 0.4 THEN 2
            WHEN PERCENT_RANK() OVER (ORDER BY Monetary) < 0.6 THEN 3
            WHEN PERCENT_RANK() OVER (ORDER BY Monetary) < 0.8 THEN 4
        ELSE 5
        END AS M
            
            FROM RFM_BASE
)

,FINAL AS 
(   
    SELECT 
    *
    ,CONCAT(R, F, M) AS RFM_Code
    FROM RFM_SCORE
)

, MERGEALL AS
(
    SELECT *
    FROM FINAL
    INNER JOIN [dbo].[ranking]
    ON FINAL.[RFM_Code]= [Code]

)

SELECT 
    MERGEALL.[Type ]
    ,COUNT(MERGEALL.[Type]) AS TotalCus
FROM MERGEALL
GROUP BY MERGEALL.[Type]
ORDER BY TotalCus DESC




## 📷 Screenshot


