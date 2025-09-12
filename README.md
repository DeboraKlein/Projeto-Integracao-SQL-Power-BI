# 📊 Projeto de Integração SQL Server + Power BI

## 1. Apresentação

Este projeto tem como objetivo integrar dados do banco AdventureWorks 2014 com o Power BI, criando dashboards interativos e estratégicos para análise de vendas, clientes e desempenho por categoria e país.

---

## 2. Fonte de Dados

- **Banco:** AdventureWorks 2014  
- **Download:** [Microsoft Docs](https://docs.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)

---

## 3. Indicadores Definidos

### 🔹 Aba Geral
- Receita Total  
- Quantidade Vendida  
- Total de Categorias de Produtos  
- Quantidade de Clientes  
- Receita Total e Lucro Total por Mês  
- Margem de Lucro  
- Quantidade Vendida por Mês  
- Lucro por País  

### 🔹 Aba Clientes
- Vendas por País  
- Clientes por País  
- Vendas por Gênero  
- Vendas por Categoria  

### 🔹 KPIs Adicionais
- Ticket Médio  
- Custo Médio por Pedido  
- Lucro Médio por Pedido  
- Variação de Receita Mês a Mês  
- Receita Acumulada (YTD)  
- Top 5 Categorias mais Vendidas  
- Clientes Novos vs Recorrentes  

---

## 4. Tabelas Utilizadas

- `FactInternetSales`  
- `DimCustomer`  
- `DimGeography`  
- `DimProduct`  
- `DimProductSubcategory`  
- `DimProductCategory`  
- `DimSalesTerritory`

---

## 5. Views Criadas

### 🔸 View Principal: `vw_FatoVendas`

```sql
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

---
````
### 🔸 View de Dimensão: vw_DimClientes

```
CREATE OR ALTER VIEW vw_DimClientes AS
SELECT
    dc.CustomerKey AS [ID Cliente],
    dc.FirstName + ' ' + dc.LastName AS [Nome Cliente],
    REPLACE(REPLACE(dc.Gender, 'M', 'Masculino'), 'F', 'Feminino') AS [Sexo],
    dg.EnglishCountryRegionName AS [País]
FROM DimCustomer dc
INNER JOIN DimGeography dg ON dc.GeographyKey = dg.GeographyKey
---
```
## 6. Medidas DAX no Power BI
```
ReceitaTotal = SUM(vw_FatoVendas[Receita Venda])
LucroTotal = SUM(vw_FatoVendas[Lucro Venda])
MargemLucro = DIVIDE([LucroTotal], [ReceitaTotal])
TicketMedio = AVERAGE(vw_FatoVendas[Ticket Médio])
Pedidos = DISTINCTCOUNT(vw_FatoVendas[Nº Pedido])
---
```
## 7. Considerações Finais
Todas as colunas necessárias para análise estão integradas nas views.

O projeto está preparado para segmentações por país, gênero, categoria e tempo.

Evitamos múltiplas fontes desconectadas para garantir performance e consistência no Power BI.

Projeto desenvolvido por Debora Klein 💡 Integrando dados com propósito e inteligência.
