# SQL Server + Power BI Integration Project
This project uses data from the AdventureWorks DW 2014 database to build an interactive dashboard in Power BI, focusing on sales analysis, customer insights, and profit targets by country and year. 
Integration was done via custom SQL views, optimized DAX measures, and strategic visuals.

#### Download: Microsoft Docs 
[Microsoft Docs](https://docs.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)


#### Dashboard Link
[Access the Power BI Dashboard](https://app.powerbi.com/view?r=eyJrIjoiNGFiNzA1YjEtODI1ZS00MmIxLWJhYTItYWUzYzQ2YmYwZjFlIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)


## 1. Project Structure
**SQL Views:** vw_FatoVendas and vw_DimClientes with joins and calculations for profit, margin, and average ticket.

**DAX Measures:** Total Profit, Total Revenue, Target by Country and Year, among others.

**Target Table:** Dynamically calculated based on 10% growth over the previous year's profit.

### Visuals:

Bullet Chart with targets by country

Customer map by country

Sankey diagram by category, country, and customer

KPI cards

Smart Narrative for automated insights

## 2. Screenshots
Dashboard Cover
![Dashboard Visão Geral](https://github.com/user-attachments/assets/7359f55e-53f7-4a3c-92cc-6036491f303e)
Overview
![Bullet Chart](https://github.com/user-attachments/assets/88cb4f99-8984-47d1-adb5-2b57eed0658e)
Smart Narrative with Insights

## 3. Defined Indicators
General Tab
Total Revenue

Quantity Sold

Total Product Categories

Number of Customers

Monthly Revenue and Profit

Profit Margin

Monthly Quantity Sold

Profit by Country

Customers Tab
Sales by Country

Customers by Country

Sales by Gender

Sales by Category

Additional KPIs
Average Ticket

Average Cost per Order

Average Profit per Order

Month-over-Month Revenue Variation

Year-to-Date Revenue

Top 5 Best-Selling Categories

New vs. Returning Customers

## 4. Tables Used
FactInternetSales

DimCustomer

DimGeography

DimProduct

DimProductSubcategory

DimProductCategory

DimSalesTerritory

## 5. Created Views
Main View: vw_FatoVendas
sql
CREATE OR ALTER VIEW vw_FatoVendas AS
SELECT
    fis.SalesOrderNumber AS [Order No],
    fis.OrderDate AS [Order Date],
    FORMAT(fis.OrderDate, 'yyyy-MM') AS [Year-Month],
    YEAR(fis.OrderDate) AS [Year],
    MONTH(fis.OrderDate) AS [Month],
    dpc.EnglishProductCategoryName AS [Product Category],
    dc.CustomerKey AS [Customer ID],
    dc.FirstName + ' ' + dc.LastName AS [Customer Name],
    REPLACE(REPLACE(dc.Gender, 'M', 'Male'), 'F', 'Female') AS [Gender],
    dg.EnglishCountryRegionName AS [Country],
    fis.OrderQuantity AS [Quantity Sold],
    fis.SalesAmount AS [Sales Revenue],
    fis.TotalProductCost AS [Sales Cost],
    fis.SalesAmount - fis.TotalProductCost AS [Sales Profit],
    CASE 
        WHEN fis.SalesAmount = 0 THEN 0
        ELSE (fis.SalesAmount - fis.TotalProductCost) / fis.SalesAmount
    END AS [Profit Margin %],
    fis.SalesAmount / NULLIF(fis.OrderQuantity, 0) AS [Average Ticket]
FROM FactInternetSales fis
INNER JOIN DimProduct dp ON fis.ProductKey = dp.ProductKey
    INNER JOIN DimProductSubcategory dps ON dp.ProductSubcategoryKey = dps.ProductSubcategoryKey
        INNER JOIN DimProductCategory dpc ON dps.ProductCategoryKey = dpc.ProductCategoryKey
INNER JOIN DimCustomer dc ON fis.CustomerKey = dc.CustomerKey
INNER JOIN DimGeography dg ON dc.GeographyKey = dg.GeographyKey
Dimension View: vw_DimClientes
sql
CREATE OR ALTER VIEW vw_DimClientes AS
SELECT
    dc.CustomerKey AS [Customer ID],
    dc.FirstName + ' ' + dc.LastName AS [Customer Name],
    REPLACE(REPLACE(dc.Gender, 'M', 'Male'), 'F', 'Female') AS [Gender],
    dg.EnglishCountryRegionName AS [Country]
FROM DimCustomer dc
INNER JOIN DimGeography dg ON dc.GeographyKey = dg.GeographyKey

## 6. DAX Measures in Power BI
Total Revenue

DAX
Receita Total = SUM(vw_FatoVendas[Sales Revenue])
Total Profit

DAX
Lucro Total = SUM(vw_FatoVendas[Sales Profit])
Profit Margin

DAX
Margem Lucro = DIVIDE([LucroTotal], [ReceitaTotal])
Average Ticket

DAX
Ticket Medio = AVERAGE(vw_FatoVendas[Average Ticket])
Order Count

DAX
QTDPedidos = DISTINCTCOUNT(vw_FatoVendas[Order No])
Total Cost

DAX
Custo Total = SUM(vw_FatoVendas[Sales Cost])
Customer Count

DAX
QTD Clientes = COUNT(vw_DimClientes[Customer ID])
Quantity Sold

DAX
QTD Vendida = SUM(vw_FatoVendas[Quantity Sold])
Profit Target by Country and Year

DAX
MetaLucroPorPaisAno = 
VAR pais = SELECTEDVALUE(vw_FatoVendas[Country])
VAR ano = SELECTEDVALUE(vw_FatoVendas[Year])
VAR lucroAnterior = 
 CALCULATE(
 SUM(vw_FatoVendas[Sales Profit]),
 FILTER(
 ALL(vw_FatoVendas),
 vw_FatoVendas[Country] = pais &&
 vw_FatoVendas[Year] = ano - 1
 )
 )
RETURN
 COALESCE(lucroAnterior * 1.1, BLANK())
## 7. DAX Tables in Power BI
Profit by Country and Year

DAX
LucroPorPaisAno = 
SUMMARIZE(
    vw_FatoVendas,
    vw_FatoVendas[Country],
    vw_FatoVendas[Year],
    "RealProfit", SUM(vw_FatoVendas[Sales Profit])
)
Profit by Country and Year with Target

DAX
LucroPorPaisAnoComMeta = 
ADDCOLUMNS(
    LucroPorPaisAno,
    "ProfitTarget",
    VAR pais = [Country]
    VAR ano = [Year]
    VAR lucroAnterior = 
        CALCULATE(
            SUM(vw_FatoVendas[Sales Profit]),
            FILTER(
                vw_FatoVendas,
                vw_FatoVendas[Country] = pais &&
                vw_FatoVendas[Year] = ano - 1
            )
        )
    RETURN
        IF(ISBLANK(lucroAnterior), BLANK(), lucroAnterior * 1.1)
)
## 8. Final Considerations
All columns required for analysis are integrated into the views.
