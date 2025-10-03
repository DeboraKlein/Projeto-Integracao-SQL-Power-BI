# SQL Server + Power BI Integration Project
#### This project uses data from the AdventureWorks DW 2014 database to build an interactive dashboard in Power BI, focusing on sales analysis, customer behavior, and profit targets by country and year. Integration was done via custom SQL views, optimized DAX measures, and strategic visuals.

## 1. Business Understanding
The goal is to deliver a strategic view of global sales performance, with emphasis on:

- Profit and revenue by country and year

- Performance by product category

- Customer behavior and segmentation

- Monitoring targets with projected growth

## 2. Data Understanding
#### Source: Download: Microsoft Docs 
[Microsoft Docs](https://docs.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)

### Tables used:

- `FactInternetSales`
- `DimCustomer`
- `DimGeography`
- `DimProduct`
- `DimProductSubcategory`
- `DimProductCategory`
- `DimSalesTerritory`

### Initial observations:

- Seasonal variations in revenue

- Margin differences across countries

- Uneven customer distribution by region

## 3. Defined Indicators
### General Tab
- Total Revenue

- Quantity Sold

- Total Product Categories

- Number of Customers

- Monthly Revenue and Profit

- Profit Margin

- Monthly Quantity Sold

- Profit by Country

### Customers Tab
- Sales by Country

- Customers by Country

- Sales by Gender

- Sales by Category

### Additional KPIs
- Average Ticket

- Average Cost per Order

- Average Profit per Order

- Month-over-Month Revenue Variation

- Year-to-Date Revenue (YTD)

- Top 5 Best-Selling Categories

- New vs. Returning Customers


## 4. Data Preparation
#### Two SQL views were created to streamline modeling:

### Main View: vw_FatoVendas
````
CREATE OR ALTER VIEW vw_FatoVendas AS
SELECT
    fis.SalesOrderNumber AS [Nº Pedido],
    fis.OrderDate AS [Data Pedido],
    FORMAT(fis.OrderDate, 'yyyy-MM') AS [Ano-Mês],
    YEAR(fis.OrderDate) AS [Ano],
    MONTH(fis.OrderDate) AS [Mês],
    dpc.EnglishProductCategoryName AS [Categoria Produto],
    dc.CustomerKey AS [ID Cliente],
    dc.FirstName + ' ' + dc.LastName AS [Nome Cliente],
    REPLACE(REPLACE(dc.Gender, 'M', 'Masculino'), 'F', 'Feminino') AS [Sexo],
    dg.EnglishCountryRegionName AS [País],
    fis.OrderQuantity AS [Qtd Vendida],
    fis.SalesAmount AS [Receita Venda],
    fis.TotalProductCost AS [Custo Venda],
    fis.SalesAmount - fis.TotalProductCost AS [Lucro Venda],
    CASE 
        WHEN fis.SalesAmount = 0 THEN 0
        ELSE (fis.SalesAmount - fis.TotalProductCost) / fis.SalesAmount
    END AS [Margem Lucro %],
    fis.SalesAmount / NULLIF(fis.OrderQuantity, 0) AS [Ticket Médio]
FROM FactInternetSales fis
INNER JOIN DimProduct dp ON fis.ProductKey = dp.ProductKey
    INNER JOIN DimProductSubcategory dps ON dp.ProductSubcategoryKey = dps.ProductSubcategoryKey
        INNER JOIN DimProductCategory dpc ON dps.ProductCategoryKey = dpc.ProductCategoryKey
INNER JOIN DimCustomer dc ON fis.CustomerKey = dc.CustomerKey
INNER JOIN DimGeography dg ON dc.GeographyKey = dg.GeographyKey
````
### Dimension View: vw_DimClientes
````
CREATE OR ALTER VIEW vw_DimClientes AS
SELECT
    dc.CustomerKey AS [ID Cliente],
    dc.FirstName + ' ' + dc.LastName AS [Nome Cliente],
    REPLACE(REPLACE(dc.Gender, 'M', 'Masculino'), 'F', 'Feminino') AS [Sexo],
    dg.EnglishCountryRegionName AS [País]
FROM DimCustomer dc
INNER JOIN DimGeography dg ON dc.GeographyKey = dg.GeographyKey
````

## 5. Modeling
### DAX measures created in Power BI:

- Total Revenue, Total Profit, Profit Margin, Average Ticket

- ProfitTargetByCountryYear: 10% growth over previous year's profit

- ProfitByCountryYearWithTarget: comparison between actual profit and target

#### Example DAX table:
````
ProfitByCountryYear = 
SUMMARIZE(
    vw_FatoVendas,
    vw_FatoVendas[Country],
    vw_FatoVendas[Year],
    "ActualProfit", SUM(vw_FatoVendas[Sales Profit])
)
````
## 6. Evaluation
### Extracted insights:

- Top profit countries: USA, Canada, and UK

- Highest margin categories: Bikes and Components

- Higher average ticket in countries with lower sales volume

- Returning customers generate more revenue per order

- Profit targets are achievable by focusing on high-margin countries

## 7. Screenshots

### Dashboard Cover
![Dashboard Visão Geral](https://github.com/user-attachments/assets/7359f55e-53f7-4a3c-92cc-6036491f303e)
### Overview
![Bullet Chart](https://github.com/user-attachments/assets/88cb4f99-8984-47d1-adb5-2b57eed0658e)
### Smart Narrative with Insights
![Smart Narrative](https://github.com/user-attachments/assets/5b522dca-2249-4e22-a538-0743a5c6de69)


## 8. Dashboard Link

[Access the Power BI Dashboard](https://app.powerbi.com/view?r=eyJrIjoiNGFiNzA1YjEtODI1ZS00MmIxLWJhYTItYWUzYzQ2YmYwZjFlIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)

## 9. Final Considerations
- All columns required for analysis are integrated into the views.
- The project is ready for segmentation by country, gender, category, and time. 
- Disconnected sources were avoided to ensure performance and consistency in Power BI.
- Project developed by Débora Klein — integrating data with purpose and intelligence.


