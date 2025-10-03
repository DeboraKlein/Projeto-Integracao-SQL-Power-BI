# Projeto de Integração SQL Server + Power BI

Este projeto utiliza dados do banco **AdventureWorks DW 2014** para construir um dashboard interativo no Power BI, com foco em **análise de vendas, clientes e metas por país e ano**. A integração foi feita via views SQL personalizadas, com medidas DAX otimizadas e visuais estratégicos.
- **Download:** [Microsoft Docs](https://docs.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)

## Link do Dashboard

[ Acesse o dashboard no Power BI](https://app.powerbi.com/view?r=eyJrIjoiNGFiNzA1YjEtODI1ZS00MmIxLWJhYTItYWUzYzQ2YmYwZjFlIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)

## 1. Estrutura do Projeto

- **Views SQL**: `vw_FatoVendas` e `vw_DimClientes` com joins e cálculos de lucro, margem e ticket médio.
- **Medidas DAX**: Lucro Total, Receita Total, Meta por País e Ano, entre outras.
- **Tabela de Metas**: Calculada dinamicamente com base em crescimento de 10% sobre o lucro do ano anterior.
- **Visuais**:
  - Bullet Chart com metas por país
  - Mapa de clientes por país
  - Sankey por categoria, país e cliente
  - Cartões de KPIs
  - Smart Narrative para insights automáticos

## 2. Capturas de Tela

###  Capa do Dashboard
![Dashboard Visão Geral](https://github.com/user-attachments/assets/7359f55e-53f7-4a3c-92cc-6036491f303e)

###  Visão Geral
![Bullet Chart](https://github.com/user-attachments/assets/88cb4f99-8984-47d1-adb5-2b57eed0658e)

###  Smart Narrative com Insights
![Smart Narrative](https://github.com/user-attachments/assets/5b522dca-2249-4e22-a538-0743a5c6de69)

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

## 5. Views Criadas

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
## 6. Medidas DAX no Power BI

**Receita Total**
```
Receita Total = SUM(vw_FatoVendas[Receita Venda])
```
**Lucro Total**
```
Lucro Total = SUM(vw_FatoVendas[Lucro Venda])
```
**Margem Lucro**
```
Margem Lucro = DIVIDE([LucroTotal], [ReceitaTotal])
```
**Ticket Medio**
```
Ticket Medio = AVERAGE(vw_FatoVendas[Ticket Médio])
```
**QTD Pedidos**
```
QTDPedidos = DISTINCTCOUNT(vw_FatoVendas[Nº Pedido])
```
**Custo Total**
```
Custo Total = SUM(vw_FatoVendas[Custo Venda])
```
**QTD Clientes**
```
QTD Clientes = COUNT(vw_DimClientes[ID Cliente])
```
**QTD Vendida**
```
QTD Vendida = SUM(vw_FatoVendas[Qtd Vendida])

```
**Meta Lucro Pais Ano**
```

MetaLucroPorPaisAno = 
VAR pais = SELECTEDVALUE(vw_FatoVendas[País])
VAR ano = SELECTEDVALUE(vw_FatoVendas[Ano])
VAR lucroAnterior = 
 CALCULATE(
 SUM(vw_FatoVendas[Lucro Venda]),
 FILTER(
 ALL(vw_FatoVendas),
 vw_FatoVendas[País] = pais &&
 vw_FatoVendas[Ano] = ano - 1
 )
 )
RETURN
 COALESCE(lucroAnterior * 1.1, BLANK())
```
## 7. Tabelas DAX no Power BI
```

```
**Lucro Por Pais Ano**
```
LucroPorPaisAno = 
SUMMARIZE(
    vw_FatoVendas,
    vw_FatoVendas[País],
    vw_FatoVendas[Ano],
    "LucroReal", SUM(vw_FatoVendas[Lucro Venda])
)

```
**Lucro Por Pais Ano Com Meta**
```
LucroPorPaisAnoComMeta = 
ADDCOLUMNS(
    LucroPorPaisAno,
    "MetaLucro",
    VAR pais = [País]
    VAR ano = [Ano]
    VAR lucroAnterior = 
        CALCULATE(
            SUM(vw_FatoVendas[Lucro Venda]),
            FILTER(
                vw_FatoVendas,
                vw_FatoVendas[País] = pais &&
                vw_FatoVendas[Ano] = ano - 1
            )
        )
    RETURN
        IF(ISBLANK(lucroAnterior), BLANK(), lucroAnterior * 1.1)
)



```
## 8. Considerações Finais
Todas as colunas necessárias para análise estão integradas nas views.

O projeto está preparado para segmentações por país, gênero, categoria e tempo.

Evitamos múltiplas fontes desconectadas para garantir performance e consistência no Power BI.

Projeto desenvolvido por Debora Klein Integrando dados com propósito e inteligência.
