# Revenue-Dashboard
This project analyzes sales data using SQL and visualizes key insights using Power Bi
## Problem Statement
This project analyzes revenue performance across different dimensions to gain insights into business performance. The key focus areas include:
    • Revenue distribution by region
    • Revenue contribution by department
    • Identifying the top revenue-generating clients
## Key Performance Indicators (KPIs)
To measure and evaluate revenue performance, the following KPIs are used:
Total Revenue – Overall revenue generated during the period.
Total Hours – Total hours logged contributing to revenue.
Revenue Per Region – Percentage contribution of each region to the overall revenue.
Month-on-Month Revenue Growth – Tracking revenue increase over months.
## Data Analysis Approach
SQL for Data Extraction & Transformation
        ◦ SQL queries were used to fetch, clean, and aggregate revenue data.
        ◦ Queries included calculations for regional revenue, department-wise revenue, and client ranking.
        
    
    1. Revenue by Region:

SELECT b.Region, SUM(s.total_revenue) AS TotalRevenue
FROM services_data s
JOIN branch_data b ON s.branch_id = b.Branch_ID
GROUP BY b.Region
ORDER BY TotalRevenue DESC;

    2. Revenue by Department:

SELECT department, SUM(total_revenue) AS TotalRevenue
FROM services_data
GROUP BY department
ORDER BY TotalRevenue DESC;


    3. Revenue by Customer

SELECT client_name, SUM(total_revenue) AS TotalRevenue
FROM services_data
GROUP BY client_name
ORDER BY TotalRevenue DESC



KPIs Queries
    1. Total Revenue:
SELECT SUM(total_revenue) AS TotalRevenue
FROM services_data;

    2. Total Hours:
SELECT SUM(hours) AS TotalHours
FROM services_data;

    3. Revenue by Region over Overall Revenue:
SELECT 
    department, 
    SUM(total_revenue) AS DepartmentRevenue,
    (SUM(total_revenue) / (SELECT SUM(total_revenue) FROM services_data)) * 100 AS RevenuePercentage
FROM 
    services_data
GROUP BY 
    department;
    4. Month on Month Percentage Increase:

WITH MonthlyRevenue AS (
    SELECT 
        FORMAT(service_date, 'yyyy-MM') AS Month,
        SUM(total_revenue) AS Revenue
    FROM 
        services_data
    GROUP BY 
        FORMAT(service_date, 'yyyy-MM')
),
RevenueComparison AS (
    SELECT 
        Month,
        Revenue,
        LAG(Revenue) OVER (ORDER BY Month) AS PreviousMonthRevenue
    FROM 
        MonthlyRevenue
)
SELECT 
    Month,
    Revenue,
    PreviousMonthRevenue,
    CASE WHEN PreviousMonthRevenue > 0 THEN ((Revenue - PreviousMonthRevenue) / PreviousMonthRevenue) * 100 ELSE NULL END AS RevenuePercentageIncrease
FROM 
    RevenueComparison
WHERE 
    PreviousMonthRevenue IS NOT NULL;


Power BI for Visualization
        ◦ Created an interactive dashboard displaying revenue trends, top clients, and KPIs.
        ◦ Used DAX measures to calculate MoM growth and revenue share by region.
        
    Power BI DAX Measures for Our KPIs
    1. Total Revenue:
Total Revenue = SUM(services_data[total_revenue])
    2. Total Hours:
Total Hours = SUM(services_data[hours])
    3. Revenue % of Overall by Region:
Revenue % of Overall by Region = 
DIVIDE(
    SUM(services_data[total_revenue]),
    CALCULATE([Total Revenue], ALL(services_data))
) * 100
    4. Month on Month Revenue % Increase:
Month on Month Revenue % Increase = 
VAR CurrentMonthRevenue = SUM(services_data[total_revenue])
VAR PreviousMonthRevenue = CALCULATE(
    SUM(services_data[total_revenue]),
    DATEADD(services_data[date_column], -1, MONTH)
)
VAR RevenueChange = IF(
    PreviousMonthRevenue = 0, 
    BLANK(), 
    (CurrentMonthRevenue / PreviousMonthRevenue) - 1
)
RETURN
    RevenueChange * 100

DAX EXPRESSION FOR CREATING A NEW TABLE & COLUMNS

    1. Date Table:

DateTable = CALENDAR(MIN(services_data[Service_Date]), MAX(services_data[Service_Date]))

    2. Year Column: 

Year = YEAR(DateTable[Date])

    3. Quarter Column: 

Quarter = "Qtr " & QUARTER(DateTable[Date])

    4. Month Column: 

Month = FORMAT('DateTable'[Date], "MMMM")

