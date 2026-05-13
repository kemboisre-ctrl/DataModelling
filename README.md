# Plataforma de Dados E-Commerce | Arquitetura Medalhão

Pipeline de engenharia de dados production-grade implementando a Arquitetura
Medalhão (Bronze → Silver → Gold) no Databricks com Delta Lake.
Construído para demonstrar ingestão incremental, tratamento de qualidade de dados e modelagem dimensional para analytics.

![Architecture](https://img.shields.io/badge/Arquitetura-Medalhão-blue)
![Platform](https://img.shields.io/badge/Plataforma-Databricks-red)
![Lakehouse](https://img.shields.io/badge/Lakehouse-Delta%20Lake-success)

---

## Visão Geral da Arquitetura


---

## Stack Tecnológico

| Camada        | Tecnologia                             | Propósito |
|--------       |-----------                             |-----------|
| Computação    | Databricks (Spark SQL + PySpark)       | Processamento distribuído |
| Armazenamento | Delta Lake                             | Transações ACID, time travel |
| Ingestão      | Watermark Incremental (last_load_date) | Cargas delta eficientes |
| Modelagem     | Star Schema (Kimball)                  | Modelo dimensional pronto para analytics |
| Qualidade     | PySpark DataFrame API                  | Tratamento de dados sujos e type safety |



---

## Detalhamento por Camada

### 1. Camada Fonte (source.ipynb)
- Schema enforcement com tabelas Delta Lake
- Simulação e tratamento de dados sujos:
  - Datas inválidas (2024-13-45, 9999-12-31) convertidas para NULL
  - Múltiplos formatos de data (02-03-2024, 2024/02/03, 03-FEB-2024) padronizados para DATE
  - Conversão Float-to-Decimal para precisão financeira (unit_price, revenue)
- Carga incremental com modo append

### 2. Camada Bronze (Bronze.ipynb)
- Padrão de carga incremental usando watermark last_load_date
- Verificação dinâmica de existência de tabela com spark.catalog.tableExists()
- Processa apenas registros novos/alterados da fonte
- Preservação de dados brutos para lineage e replay

### 3. Camada Silver (Silver.ipynb)
- Injeção de colunas de auditoria: process_date para rastreabilidade
- Cargas idempotentes: CREATE TABLE IF NOT EXISTS + padrão de temp view
- Fundação consistente para consumo downstream

### 4. Camada Gold (Gold.ipynb)
Implementação do Star Schema:

| Tabela         | Tipo      | Chave de Negócio    | Chave Surrogate |
|--------        |------     |------------------   |-----------------|
| DimCustomers   | Dimensão  | order_customer_id   | DimCustomerKey |
| DimProducts    | Dimensão  | product_id          | DimProductKey |
| DimPayments    | Dimensão  | payment_type        | DimPaymentKey |
| FactSales      | Fato      | order_id            | DimSalesKey |

- Geração de surrogate keys via ROW_NUMBER()
- SCD Tipo 0 (dimensões estáticas) — pronto para extensão para SCD Tipo 2
- Fact table faz join com todas as dimensões para queries analíticas

---



## Como Executar

1. Pré-requisitos: Workspace Databricks com DBR 13.x+ e Unity Catalog habilitado
2. Setup:

```sql
CREATE CATALOG IF NOT EXISTS datamodeling;
CREATE SCHEMA IF NOT EXISTS datamodeling.default;
CREATE SCHEMA IF NOT EXISTS datamodeling.bronze;
CREATE SCHEMA IF NOT EXISTS datamodeling.silver;
CREATE SCHEMA IF NOT EXISTS datamodeling.gold;

Ordem de Execução:
1. Execute source.ipynb para inserir dados brutos
2. Execute Bronze.ipynb para carga incremental bronze
3. Execute Silver.ipynb para camada silver limpa
4. vExecute Gold.ipynb para modelo dimensional

Query de Exemplo

-- Receita total por categoria de produto
SELECT 
    p.product_category,
    SUM(f.quantity * f.unit_price) as total_revenue
FROM datamodeling.gold.FactSales f
JOIN datamodeling.gold.DimProducts p 
    ON f.dimProductKey = p.dimProductKey
GROUP BY p.product_category
ORDER BY total_revenue DESC;




---


#Estrutura do Projeto


##Próximos Passos / Evoluções
 ###Implementar SCD Tipo 2 para rastreamento histórico de clientes
 ###Adicionar Great Expectations / dbt tests para assertions de qualidade
 ###Migrar para Delta Live Tables (DLT) para pipelines declarativos
 ###Adicionar lineage do Unity Catalog e tags de governança


# Sobre Este Projeto
Construído como peça de portfólio para demonstrar skills end-to-end de engenharia de dados: desde padrões resilientes de ingestão até modelagem dimensional pronta para analytics na plataforma Databricks Lakehouse.
