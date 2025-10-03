
# Projeto de Integração SQL Server + Power BI
Este projeto utiliza dados do banco AdventureWorks DW 2014 para construir um dashboard interativo no Power BI, com foco em análise de vendas, clientes e metas por país e ano. A integração foi feita via views SQL personalizadas, com medidas DAX otimizadas e visuais estratégicos.
---

## 1. Business Understanding

#### O objetivo é fornecer uma visão estratégica das vendas globais, com foco em:

- Lucro e receita por país e ano

- Performance por categoria de produto

- Análise de clientes e comportamento de compra

- Acompanhamento de metas com crescimento projetado
---

## 2. Data Understanding
Fonte: AdventureWorks DW 2014 **Download Database:** [Microsoft Docs](https://docs.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)

#### Tabelas utilizadas:

- FactInternetSales

- DimCustomer, DimGeography

- DimProduct, DimProductSubcategory, DimProductCategory

- DimSalesTerritory
---

#### Principais observações iniciais:

- Variações sazonais de receita

- Diferenças de margem entre países

- Distribuição desigual de clientes por região
---

## 3. Indicadores Definidos

### Aba Geral
- Receita Total  
- Quantidade Vendida  
- Total de Categorias de Produtos  
- Quantidade de Clientes  
- Receita Total e Lucro Total por Mês  
- Margem de Lucro  
- Quantidade Vendida por Mês  
- Lucro por País  

###  Aba Clientes
- Vendas por País  
- Clientes por País  
- Vendas por Gênero  
- Vendas por Categoria  

###  KPIs Adicionais
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

## 5.  Data Preparation
Criação de duas views SQL para facilitar a modelagem:

###  View Principal: `vw_FatoVendas`

```
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
###  View de Dimensão: vw_DimClientes

```
CREATE OR ALTER VIEW vw_DimClientes AS
SELECT
    dc.CustomerKey AS [ID Cliente],
    dc.FirstName + ' ' + dc.LastName AS [Nome Cliente],
    REPLACE(REPLACE(dc.Gender, 'M', 'Masculino'), 'F', 'Feminino') AS [Sexo],
    dg.EnglishCountryRegionName AS [País]
FROM DimCustomer dc
INNER JOIN DimGeography dg ON dc.GeographyKey = dg.GeographyKey

```
## 6. Modeling
Medidas DAX criadas no Power BI:

**Receita Total**, **Lucro Total**, **Margem Lucro**, **Ticket Médio**

**MetaLucroPorPaisAno:** crescimento de 10% sobre o lucro do ano anterior

**LucroPorPaisAnoComMeta:** comparação entre lucro real e meta

Tabelas DAX:

DAX
````
LucroPorPaisAno = 
SUMMARIZE(vw_FatoVendas, vw_FatoVendas[País]
````
## 7. Evaluation
Insights extraídos:

- Países com maior lucro: EUA, Canadá e Reino Unido

- Categorias com maior margem: Bikes e Components

- Ticket médio mais alto em países com menor volume de vendas

- Clientes recorrentes geram maior receita por pedido

- Metas de lucro são atingíveis com foco em países de alta margem
---
  
## 8. Capturas de Tela

###  Capa do Dashboard
![Dashboard Visão Geral](https://github.com/user-attachments/assets/7359f55e-53f7-4a3c-92cc-6036491f303e)

###  Visão Geral
![Bullet Chart](https://github.com/user-attachments/assets/88cb4f99-8984-47d1-adb5-2b57eed0658e)

###  Smart Narrative com Insights
![Smart Narrative](https://github.com/user-attachments/assets/5b522dca-2249-4e22-a538-0743a5c6de69)

---

## 9. Link do Dashboard

[ Acesse o dashboard no Power BI](https://app.powerbi.com/view?r=eyJrIjoiNGFiNzA1YjEtODI1ZS00MmIxLWJhYTItYWUzYzQ2YmYwZjFlIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)

--- 
## 10. Considerações Finais

Todas as colunas necessárias para análise estão integradas nas views.

O projeto está preparado para segmentações por país, gênero, categoria e tempo.

Evitamos múltiplas fontes desconectadas para garantir performance e consistência no Power BI.

Projeto desenvolvido por Debora Klein Integrando dados com propósito e inteligência.
