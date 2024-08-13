
**Suggestions and Considerations:**

**Use of Aliases:**  
I consider using shorter or more descriptive aliases for tables and columns. This approach makes my queries easier to read and manage, especially when I’m dealing with complex queries.

**Indexing and Performance:**  
I make sure that the columns frequently used in WHERE, GROUP BY, and ORDER BY clauses (like `invoice_date`, `Region`, etc.) are indexed. This improves the performance of my queries, particularly when working with large datasets.

**Partitioning Large Queries:**  
For Common Table Expressions (CTEs), I might consider partitioning the data by year and region. This can help with performance, especially when the dataset is large.

**Handling NULL Values:**  
I ensure that my data doesn’t contain NULL values in key columns like `Total_Sales`, `Units_Sold`, or `Operating_Profit`. If there’s a possibility, I use COALESCE to handle such cases.

**Error Handling:**  
Depending on my SQL environment, I consider adding error-handling mechanisms or checks to ensure that my queries run smoothly without unexpected interruptions.

**Additional Insights:**

**Using CTEs:**  
In this example, I use a CTE to calculate the total and average sales by region and year. This makes the query cleaner and potentially improves readability.

**Combined Metrics:**  
For scenarios where I calculate multiple metrics (like `TotalSales` and `AverageSales`) for the same grouping, I use a CTE to encapsulate the logic and reduce redundancy.

The queries I’ve prepared cover a broad range of analyses, from simple aggregations to more complex rankings and comparisons. The structure and explanations I’ve added make the SQL highly readable and easy to understand, which is essential for maintaining and sharing my work.


```USE [YEMIFATODU2db];

-- Retrieve all columns from the ADIDDASSALE table
SELECT *
FROM [dbo].[ADIDDASSALE];```


-- Calculate the overall total sales for both years combined
```SELECT
    SUM(Total_Sales) AS OverallTotalSales
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021);```

-- Calculate the total sales for each region for the years 2020 and 2021
``SELECT
    YEAR(invoice_date) AS Year,      
    Region,                           
    SUM(Total_Sales) AS TotalSales   
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
    YEAR(invoice_date),              
    Region                           
ORDER BY
    Year,                           
    Region;```                           


-- Average Sales Per Region for 2020 and 2021
``SELECT
    Region,
    AVG(Total_Sales) AS AverageSales
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    Region;```

-- Retrieve the total sales for each region, ordered by highest sales per year
```SELECT
    YEAR(invoice_date) AS Year,
    Region,
    SUM(Total_Sales) AS TotalSales
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date),
    Region
ORDER BY
    Year,
    TotalSales DESC;

-- Find the highest sales region for each year
SELECT
    rs.Year,
    rs.Region,
    rs.TotalSales
FROM
    (
        SELECT
            YEAR(invoice_date) AS Year,
            Region,
            SUM(Total_Sales) AS TotalSales
        FROM
            [dbo].[ADIDDASSALE]
        WHERE
            YEAR(invoice_date) IN (2020, 2021)
        GROUP BY
            YEAR(invoice_date),
            Region
    ) rs
JOIN (
    SELECT
        Year,
        MAX(TotalSales) AS MaxSales
    FROM
        (
            SELECT
                YEAR(invoice_date) AS Year,
                Region,
                SUM(Total_Sales) AS TotalSales
            FROM
                [dbo].[ADIDDASSALE]
            WHERE
                YEAR(invoice_date) IN (2020, 2021)
            GROUP BY
                YEAR(invoice_date),
                Region
        ) subquery
    GROUP BY
        Year
) max_sales
ON rs.Year = max_sales.Year
AND rs.TotalSales = max_sales.MaxSales
ORDER BY
    rs.Year;

-- Find the minimum sales in each region per year
;WITH RegionalSales AS (
    SELECT
        YEAR(invoice_date) AS Year,
        Region,
        SUM(Total_Sales) AS TotalSales
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        Region
),
MinSales AS (
    SELECT
        Year,
        MIN(TotalSales) AS MinSales
    FROM
        RegionalSales
    GROUP BY
        Year
)
SELECT
    rs.Year,
    rs.Region,
    rs.TotalSales
FROM
    RegionalSales rs
JOIN
    MinSales ms
ON
    rs.Year = ms.Year
    AND rs.TotalSales = ms.MinSales
ORDER BY
    rs.Year;```

-- Average Sale in Each Region Per Year
```SELECT
    YEAR(invoice_date) AS Year,
    Region,
    AVG(Total_Sales) AS AverageSales
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date),
    Region
ORDER BY
    Year,
    Region;```


-- Calculate Total and Average Sales for Each Region and Year
```SELECT
    YEAR(invoice_date) AS Year,
    Region,
    SUM(Total_Sales) AS TotalSales,
    AVG(Total_Sales) AS AverageSales
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date),
    Region
ORDER BY
    Year,
    Region;```

-- Total Units Sold for Each Year
```SELECT
    YEAR(invoice_date) AS Year,
    SUM(Units_Sold) AS TotalUnitsSold
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date)
ORDER BY
    Year;```

-- Total Units Sold for Each Region and Year
```SELECT
    YEAR(invoice_date) AS Year,
    Region,
    SUM(Units_Sold) AS TotalUnitsSold
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date),
    Region
ORDER BY
    Year,
    Region;```

-- Calculate the Average Units Sold Per Region for Each Year
```SELECT
    YEAR(invoice_date) AS Year,
    Region,
    AVG(Units_Sold) AS AverageUnitsSold
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date),
    Region
ORDER BY
    Year,
    Region;```

-- The Region with the Highest Total Units Sold for Each Year (2020 and 2021)
```;WITH RegionalUnits AS (
    SELECT
        YEAR(invoice_date) AS Year,
        Region,
        SUM(Units_Sold) AS TotalUnitsSold
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        Region
),
RankedUnits AS (
    SELECT
        Year,
        Region,
        TotalUnitsSold,
        ROW_NUMBER() OVER (PARTITION BY Year ORDER BY TotalUnitsSold DESC) AS Rank
    FROM
        RegionalUnits
)
SELECT
    Year,
    Region,
    TotalUnitsSold
FROM
    RankedUnits
WHERE
    Rank = 1
ORDER BY
    Year,
    Region;```

-- The Region with the Lowest Total Units Sold for Each Year (2020 and 2021)
```;WITH RegionalUnits AS (
    SELECT
        YEAR(invoice_date) AS Year,
        Region,
        SUM(Units_Sold) AS TotalUnitsSold
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        Region
),
RankedUnits AS (
    SELECT
        Year,
        Region,
        TotalUnitsSold,
        ROW_NUMBER() OVER (PARTITION BY Year ORDER BY TotalUnitsSold ASC) AS Rank
    FROM
        RegionalUnits
)
SELECT
    Year,
    Region,
    TotalUnitsSold
FROM
    RankedUnits
WHERE
    Rank = 1
ORDER BY
    Year;```

-- The Region with the Lowest Total Operating Profit for Each Year (2020 and 2021)
```;WITH RegionalProfit AS (
    SELECT
        YEAR(invoice_date) AS Year,
        Region,
        SUM(Operating_Profit) AS TotalOperatingProfit
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        Region
),
RankedProfit AS (
    SELECT
        Year,
        Region,
        TotalOperatingProfit,
        ROW_NUMBER() OVER (PARTITION BY Year ORDER BY TotalOperatingProfit ASC) AS Rank
    FROM
        RegionalProfit
)
SELECT
    Year,
    Region,
    TotalOperatingProfit
FROM
    RankedProfit
WHERE
    Rank = 1
ORDER BY
    Year;```

-- Calculate the Total Operating Profit for Each Year
```SELECT
    YEAR(invoice_date) AS Year,
    SUM(Operating_Profit) AS TotalOperatingProfit
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date)
ORDER BY
    Year;```

-- Calculate the Average Operating Profit Per Region for Each Year
```SELECT
    YEAR(invoice_date) AS Year,
    Region,
    AVG(Operating_Profit) AS AverageOperatingProfit
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date),
    Region
ORDER BY
    Year,
    Region;```

-- Regional Sales Breakdown for Each Year
```;WITH RegionalData AS (
    SELECT
        YEAR(invoice_date) AS Year,
        Region,
        SUM(Total_Sales) AS TotalSales,
        SUM(Units_Sold) AS TotalUnitsSold,
        SUM(Operating_Profit) AS TotalOperatingProfit
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        Region IN ('South', 'West', 'Northeast', 'Southeast', 'Midwest')
        AND YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        Region
),
OverallTotals AS (
    SELECT
        Year,
        SUM(TotalSales) AS OverallTotalSales
    FROM
        RegionalData
    GROUP BY
        Year
)
SELECT
    r.Year,
    r.Region,
    r.TotalSales,
    r.TotalUnitsSold,
    r.TotalOperatingProfit,
    CAST(r.TotalSales AS FLOAT) / o.OverallTotalSales * 100 AS SalesPercentage
FROM
    RegionalData r
JOIN
    OverallTotals o
ON
    r.Year = o.Year
ORDER BY
    r.Year,
    r.Region;```


-- Regional Breakdown for Each Year (Example for South Region)
```;WITH RegionalData AS (
    SELECT
        YEAR(invoice_date) AS Year,
        Region,
        SUM(Total_Sales) AS TotalSales,
        SUM(Units_Sold) AS TotalUnitsSold,
        SUM(Operating_Profit) AS TotalOperatingProfit
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        Region IN ('South')
        AND YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        Region```
)


-- Month-by-Month Sales Analysis for South Region
```;WITH MonthlySales AS (
    SELECT
        YEAR(invoice_date) AS Year,
        MONTH(invoice_date) AS Month,
        SUM(Total_Sales) AS TotalSales
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        Region = 'South'
        AND YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        MONTH(invoice_date)
),
HighestLowestSales AS (
    SELECT
        Year,
        MAX(TotalSales) AS MaxSales,
        MIN(TotalSales) AS MinSales
    FROM
        MonthlySales
    GROUP BY
        Year
)
SELECT
    m.Year,
    m.Month,
    m.TotalSales,
    CASE
        WHEN m.TotalSales = h.MaxSales THEN 'Highest'
        WHEN m.TotalSales = h.MinSales THEN 'Lowest'
        ELSE 'In-between'
    END AS SalesTrend
FROM
    MonthlySales m
JOIN
    HighestLowestSales h
ON
    m.Year = h.Year
ORDER BY
    m.Year, m.Month ASC;```

-- Highest and Lowest Quarterly Sales for South Region
```;WITH QuarterlySales AS (
    SELECT
        YEAR(invoice_date) AS Year,
        DATEPART(QUARTER, invoice_date) AS Quarter,
        SUM(Total_Sales) AS TotalSales
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        Region = 'South'
        AND YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        DATEPART(QUARTER, invoice_date)
)
SELECT
    Year,
    Quarter,
    MAX(TotalSales) AS HighestSales,
    MIN(TotalSales) AS LowestSales
FROM
    QuarterlySales
GROUP BY
    Year,
    Quarter
ORDER BY
    Year, Quarter;```

-- Highest and Lowest Quarterly Sales for West Region
```WITH QuarterlySales AS (
    SELECT
        YEAR(invoice_date) AS Year,
        DATEPART(QUARTER, invoice_date) AS Quarter,
        SUM(Total_Sales) AS TotalSales
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        Region = 'West'
        AND YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        DATEPART(QUARTER, invoice_date)
)
SELECT
    Year,
    Quarter,
    MAX(TotalSales) AS HighestSales,
    MIN(TotalSales) AS LowestSales
FROM
    QuarterlySales
GROUP BY
    Year,
    Quarter
ORDER BY
    Year, Quarter;```

-- Highest and Lowest Quarterly Sales for Northeast Region
```WITH QuarterlySales AS (
    SELECT
        YEAR(invoice_date) AS Year,
        DATEPART(QUARTER, invoice_date) AS Quarter,
        SUM(Total_Sales) AS TotalSales
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        Region = 'Northeast'
        AND YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        DATEPART(QUARTER, invoice_date)
)
SELECT
    Year,
    Quarter,
    MAX(TotalSales) AS HighestSales,
    MIN(TotalSales) AS LowestSales
FROM
    QuarterlySales
GROUP BY
    Year,
    Quarter
ORDER BY
    Year, Quarter;``

-- Highest and Lowest Quarterly Sales for Southeast Region
```WITH QuarterlySales AS (
    SELECT
        YEAR(invoice_date) AS Year,
        DATEPART(QUARTER, invoice_date) AS Quarter,
        SUM(Total_Sales) AS TotalSales
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        Region = 'Southeast'
        AND YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        DATEPART(QUARTER, invoice_date)
)
SELECT
    Year,
    Quarter,
    MAX(TotalSales) AS HighestSales,
    MIN(TotalSales) AS LowestSales
FROM
    QuarterlySales
GROUP BY
    Year,
    Quarter
ORDER BY
    Year, Quarter;```

-- Highest and Lowest Quarterly Sales for Midwest Region
```WITH QuarterlySales AS (
    SELECT
        YEAR(invoice_date) AS Year,
        DATEPART(QUARTER, invoice_date) AS Quarter,
        SUM(Total_Sales) AS TotalSales
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        Region = 'Midwest'
        AND YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        DATEPART(QUARTER, invoice_date)
)
SELECT
    Year,
    Quarter,
    MAX(TotalSales) AS HighestSales,
    MIN(TotalSales) AS LowestSales
FROM
    QuarterlySales
GROUP BY
    Year,
    Quarter
ORDER BY
    Year, Quarter;```

-- Average Price per Unit of Products Sold for Each Year
```SELECT
    YEAR(invoice_date) AS Year,
    AVG(Price_per_Unit) AS AveragePrice
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date);```

-- Minimum and Maximum Price per Unit of Products Sold for Each Year
```SELECT
    YEAR(invoice_date) AS Year,
    MIN(Price_per_Unit) AS MinPrice,
    MAX(Price_per_Unit) AS MaxPrice
FROM
    [dbo].[ADIDDASSALE]
WHERE
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY
    YEAR(invoice_date);```

-- Correlation Between Price per Unit and Units Sold for Each Year
```WITH Stats AS (
    SELECT
        YEAR(invoice_date) AS Year,
        [Price_per_Unit] AS PricePerUnit,
        Units_Sold AS UnitsSold,
        AVG([Price_per_Unit]) OVER (PARTITION BY YEAR(invoice_date)) AS AvgPrice,
        AVG(Units_Sold) OVER (PARTITION BY YEAR(invoice_date)) AS AvgUnitsSold
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) IN (2020, 2021)
),
Covariance AS (
    SELECT
        Year,
        SUM((PricePerUnit - AvgPrice) * (UnitsSold - AvgUnitsSold)) AS CovXY,
        SQRT(SUM((PricePerUnit - AvgPrice) * (PricePerUnit - AvgPrice))) AS StdDevX,
        SQRT(SUM((UnitsSold - AvgUnitsSold) * (UnitsSold - AvgUnitsSold))) AS StdDevY
    FROM
        Stats
    GROUP BY
        Year
)
SELECT
    Year,
    CovXY / (StdDevX * StdDevY) AS CorrelationCoefficient
FROM
    Covariance
ORDER BY
    Year;```

-- Correlation Between Price per Unit and Units Sold
```SELECT 
    YEAR(invoice_date) AS Year,
    CORR(Price_per_Unit, Units_Sold) AS CorrelationCoefficient
FROM 
    [dbo].[ADIDDASSALE]
WHERE 
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY 
    YEAR(invoice_date);

-- Top Product Categories by Sales for Each Year
WITH CategorySales AS (
    SELECT
        YEAR(invoice_date) AS Year,
        Product AS Category,
        SUM(Total_Sales) AS SalesValue
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        YEAR(invoice_date),
        Product
)
SELECT
    Year,
    Category,
    SalesValue
FROM
    (
        SELECT
            Year,
            Category,
            SalesValue,
            ROW_NUMBER() OVER (PARTITION BY Year ORDER BY SalesValue DESC) AS RowNum
        FROM
            CategorySales
    ) AS RankedCategories
WHERE
    RowNum <= 3
ORDER BY
    Year,
    RowNum;```

-- Top 3 Product Categories by Sales for the South Region
```SELECT TOP 3
    Product AS Category,
    SUM(Total_Sales) AS SalesValue
FROM
    [dbo].[ADIDDASSALE]
WHERE
    Region = 'South'
GROUP BY
    Product
ORDER BY
    SalesValue DESC;```

-- Top 3 Product Categories by Sales for the West Region
```SELECT TOP 3
    Product AS Category,
    SUM(Total_Sales) AS SalesValue
FROM
    [dbo].[ADIDDASSALE]
WHERE
    Region = 'West'
GROUP BY
    Product
ORDER BY
    SalesValue DESC;```

-- Top 3 Product Categories by Sales for the Northeast Region
```SELECT TOP 3
    Product AS Category,
    SUM(Total_Sales) AS SalesValue
FROM
    [dbo].[ADIDDASSALE]
WHERE
    Region = 'Northeast'
GROUP BY
    Product
ORDER BY
    SalesValue DESC;```


-- Top 3 Product Categories by Sales for the Southeast Region
```SELECT TOP 3
    Product AS Category,
    SUM(Total_Sales) AS SalesValue
FROM
    [dbo].[ADIDDASSALE]
WHERE
    Region = 'Southeast'
GROUP BY
    Product
ORDER BY
    SalesValue DESC;```


-- Top 3 Product Categories by Sales for the Midwest Region
```SELECT TOP 3
    Product AS Category,
    SUM(Total_Sales) AS SalesValue
FROM
    [dbo].[ADIDDASSALE]
WHERE
    Region = 'Midwest'
GROUP BY
    Product
ORDER BY
    SalesValue DESC;```


-- Top Product Categories by Profit for the West Region in 2020
```SELECT TOP 3
    Product,
    SUM(Operating_Profit) AS TotalProfit
FROM
    [dbo].[ADIDDASSALE]
WHERE
    Region = 'West'
    AND YEAR(invoice_date) = 2020
GROUP BY
    Product
ORDER BY
    TotalProfit DESC;```


-- Top Product Categories by Profit for the West Region in 2021
```SELECT TOP 3
    Product,
    SUM(Operating_Profit) AS TotalProfit
FROM
    [dbo].[ADIDDASSALE]
WHERE
    Region = 'West'
    AND YEAR(invoice_date) = 2021
GROUP BY
    Product
ORDER BY
    TotalProfit DESC;```


-- Top 3 Product Categories by Sales in South Region for 2020
```WITH RankedCategories AS (
    SELECT
        Product,
        SUM(Total_Sales) AS TotalSales,
        ROW_NUMBER() OVER (ORDER BY SUM(Total_Sales) DESC) AS Rank
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) = 2020
        AND Region = 'South'
    GROUP BY
        Product
)
SELECT
    Product,
    TotalSales
FROM
    RankedCategories
WHERE
    Rank <= 3
ORDER BY
    Rank;```


-- Top 3 Product Categories by Sales in South Region for 2021
```WITH RankedCategories AS (
    SELECT
        Product,
        SUM(Total_Sales) AS TotalSales,
        ROW_NUMBER() OVER (ORDER BY SUM(Total_Sales) DESC) AS Rank
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) = 2021
        AND Region = 'South'
    GROUP BY
        Product
)
SELECT
    Product,
    TotalSales
FROM
    RankedCategories
WHERE
    Rank <= 3
ORDER BY
    Rank;```


-- Units Sold for Top 3 Product Categories in South Region for 2020
```WITH TopCategories AS (
    SELECT
        Product,
        SUM(Total_Sales) AS TotalSales,
        ROW_NUMBER() OVER (ORDER BY SUM(Total_Sales) DESC) AS Rank
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) = 2020
        AND Region = 'South'
    GROUP BY
        Product
),
CategoryUnits AS (
    SELECT
        Product,
        SUM(Units_Sold) AS UnitsSold
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) = 2020
        AND Region = 'South'
    GROUP BY
        Product
)
SELECT
    t.Product AS [Category],
    c.UnitsSold AS [Units Sold]
FROM
    TopCategories t
JOIN
    CategoryUnits c
ON
    t.Product = c.Product
WHERE
    t.Rank <= 3
ORDER BY
    t.Rank;```


-- Units Sold for Top 3 Product Categories in South Region for 2021
```WITH TopCategories AS (
    SELECT
        Product,
        SUM(Total_Sales) AS TotalSales,
        ROW_NUMBER() OVER (ORDER BY SUM(Total_Sales) DESC) AS Rank
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) = 2021
        AND Region = 'South'
    GROUP BY
        Product
),
CategoryUnits AS (
    SELECT
        Product,
        SUM(Units_Sold) AS UnitsSold
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) = 2021
        AND Region = 'South'
    GROUP BY
        Product
)
SELECT
    t.Product AS [Category],
    c.UnitsSold AS [Units Sold]
FROM
    TopCategories t
JOIN
    CategoryUnits c
ON
    t.Product = c.Product
WHERE
    t.Rank <= 3
ORDER BY
    t.Rank;```


-- Total Sales by Retailer for 2020 and 2021
```SELECT 
    Retailer, 
    YEAR(invoice_date) AS Year, 
    SUM(Total_Sales) AS TotalSales
FROM 
    [dbo].[ADIDDASSALE]
WHERE 
    YEAR(invoice_date) IN (2020, 2021)
GROUP BY 
    Retailer, 
    YEAR(invoice_date)
ORDER BY 
    Retailer, 
    Year;```


-- Sales Comparison for the Top 5 Retailers Between 2020 and 2021
```WITH RetailerSales AS (
    SELECT
        Retailer,
        YEAR(invoice_date) AS Year,
        SUM(Total_Sales) AS TotalSales
    FROM
        [dbo].[ADIDDASSALE]
    WHERE
        YEAR(invoice_date) IN (2020, 2021)
    GROUP BY
        Retailer,
        YEAR(invoice_date)
),
TopRetailers AS (
    SELECT
        Retailer,
        SUM(TotalSales) AS TotalSales
    FROM
        RetailerSales
    GROUP BY
        Retailer
    ORDER BY
        TotalSales DESC
    OFFSET 0 ROWS
    FETCH NEXT 5 ROWS ONLY
)
SELECT
    r.Retailer,
    r.Year,
    r.TotalSales
FROM
    RetailerSales r
JOIN
    TopRetailers t
ON
    r.Retailer = t.Retailer
ORDER BY
    r.Retailer,
    r.Year;```

