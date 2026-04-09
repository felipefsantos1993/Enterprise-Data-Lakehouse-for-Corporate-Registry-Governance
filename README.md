# National Corporate Data Lakehouse & Governance Platform (CNPJ)

Plataforma de engenharia de dados baseada em arquitetura Lakehouse utilizando Delta Live Tables (DLT) para ingestão, transformação, governança e análise dos dados públicos da Receita Federal (CNPJ).

---

## Visão Geral

Este projeto implementa uma pipeline completa de dados utilizando o conceito de **Medallion Architecture (Bronze, Silver, Gold)** dentro do Databricks.

A solução é responsável por:

* Ingestão de dados públicos da Receita Federal
* Estruturação em camadas (raw → bronze → silver → gold)
* Limpeza e padronização dos dados
* Validação de qualidade com regras declarativas
* Geração de tabelas analíticas

---

## Arquitetura

```
Databricks Workflow
│
├── Task 1 → Ingestão (Python)
│
├── Task 2 → DLT Empresas
│
├── Task 3 → DLT Sócios
│
└── Task 4 → DLT Estabelecimentos
        ↓
Delta Tables (Unity Catalog)
        ↓
Consumo (SQL / BI / Analytics)
```

---

## Stack Tecnológica

* Databricks
* Delta Live Tables (DLT)
* Apache Spark (PySpark)
* Delta Lake
* DBFS / Cloud Storage (S3, ADLS ou GCS)

---

## Estrutura do Projeto

```
cnpj-dlt-pipeline/
│
├── ingestion/
│   └── ingestion_cnpj.py
│
├── dlt/
│   ├── empresas/
│   │   └── dlt_empresas.py
│   │
│   ├── socios/
│   │   └── dlt_socios.py
│   │
│   └── estabelecimentos/
│       └── dlt_estabelecimentos.py
│
├── config/
├── notebooks/
└── README.md
```

---

## Ingestão de Dados

Os dados são obtidos diretamente da Receita Federal (CNPJ aberto) e disponibilizados em formato ZIP.

### Etapas:

1. Download dos arquivos
2. Descompactação
3. Upload para o Data Lake (`/mnt/cnpj/raw/`)

---

## Pipelines DLT por Domínio

A arquitetura utiliza múltiplos pipelines DLT independentes, organizados por domínio de dados:

* Empresas
* Sócios
* Estabelecimentos

Cada pipeline implementa sua própria lógica de Bronze → Silver → Gold.

### Exemplo: Pipeline Empresas

```python
@dlt.table
def bronze_empresas():
    return (
        spark.read
        .format("csv")
        .option("sep", ";")
        .load("/mnt/cnpj/raw/empresas/")
    )

@dlt.table
@dlt.expect("cnpj_not_null", "cnpj_basico IS NOT NULL")
def silver_empresas():
    return dlt.read("bronze_empresas").dropDuplicates(["cnpj_basico"])
```

---

## Orquestração

A orquestração é realizada através de **Databricks Workflows**, que coordenam a execução dos pipelines:

```
Task 1 (Ingestão)
   ↓
Task 2 (Empresas)
Task 3 (Sócios)
Task 4 (Estabelecimentos)
```

* A ingestão é executada primeiro
* Os pipelines DLT são executados em paralelo
* Cada pipeline é independente

Essa abordagem permite maior escalabilidade, isolamento e reprocessamento seletivo.

---

## Configuração do Pipeline

Parâmetros principais:

* **Source**: notebook ou repositório Git
* **Target Schema**: `cnpj_dlt`
* **Storage Location**: `/mnt/cnpj/dlt/`
* **Mode**:

  * Triggered (batch)
  * Continuous (streaming)

---

## Consumo de Dados

Os dados podem ser consumidos via:

* Databricks SQL
* Power BI / Tableau
* Notebooks analíticos
* APIs (opcional)

---

## Possíveis Evoluções

* Integração com tabelas de Sócios e Estabelecimentos
* Enriquecimento com dados geográficos (IBGE)
* Dashboard analítico
* Ingestão incremental (Auto Loader)
* Testes de qualidade mais avançados
* Feature Store para Machine Learning

---

## Casos de Uso

* Análise de empresas por setor (CNAE)
* Distribuição geográfica de empresas
* Perfil de empresas por porte
* Evolução do número de empresas

---

## Objetivo do Projeto

Demonstrar habilidades em:

* Engenharia de Dados moderna (Lakehouse)
* Processamento distribuído com Spark
* Pipelines declarativos com DLT
* Data Quality e governança
* Modelagem analítica

---

## Status

🚧 Em desenvolvimento

---

## Contribuição

Sinta-se à vontade para abrir issues ou contribuir com melhorias.

---

## Licença

Este projeto utiliza dados públicos da Receita Federal do Brasil.



# National Corporate Data Lakehouse & Governance Platform (CNPJ)

A modern data engineering platform based on Lakehouse architecture using Delta Live Tables (DLT) for ingestion, transformation, governance, and analytics of Brazilian Federal Revenue public data (CNPJ).

---

## Overview

This project implements an end-to-end data pipeline following the **Medallion Architecture (Bronze, Silver, Gold)** using Databricks.

The solution is responsible for:

* Ingesting public data from the Brazilian Federal Revenue
* Structuring data into layered architecture (raw → bronze → silver → gold)
* Cleaning and standardizing datasets
* Applying declarative data quality rules
* Generating analytical datasets for consumption

---

## Architecture

```
Databricks Workflow
│
├── Task 1 → Ingestion (Python)
│
├── Task 2 → DLT Companies
│
├── Task 3 → DLT Shareholders
│
└── Task 4 → DLT Establishments
        ↓
Delta Tables (Unity Catalog)
        ↓
Consumption (SQL / BI / Analytics)
```

---

## Tech Stack

| Layer         | Technology                                 |
| ------------- | ------------------------------------------ |
| Processing    | PySpark                                    |
| Orchestration | Delta Live Tables + Workflows              |
| Storage       | Delta Lake                                 |
| Platform      | Databricks                                 |
| Data Source   | Brazilian Federal Revenue (CNPJ Open Data) |

---

## Project Structure

```
cnpj-dlt-pipeline/
│
├── ingestion/
│   └── ingestion_cnpj.py
│
├── dlt/
│   ├── companies/
│   │   └── dlt_companies.py
│   │
│   ├── shareholders/
│   │   └── dlt_shareholders.py
│   │
│   └── establishments/
│       └── dlt_establishments.py
│
├── config/
├── notebooks/
└── README.md
```

---

## Data Ingestion

Data is sourced from the Brazilian Federal Revenue (CNPJ open dataset) and provided as ZIP files.

### Steps:

1. Download datasets
2. Extract files
3. Upload to Data Lake (`/mnt/cnpj/raw/`)

---

## Domain-Oriented DLT Pipelines

The architecture uses multiple independent DLT pipelines organized by data domain:

* Companies
* Shareholders
* Establishments

Each pipeline implements its own **Bronze → Silver → Gold** transformations.

---

### Example: Companies Pipeline

```python
@dlt.table
def bronze_companies():
    return (
        spark.read
        .format("csv")
        .option("sep", ";")
        .load("/mnt/cnpj/raw/empresas/")
    )

@dlt.table
@dlt.expect("cnpj_not_null", "cnpj_basico IS NOT NULL")
def silver_companies():
    return dlt.read("bronze_companies").dropDuplicates(["cnpj_basico"])
```

---

## Orchestration

Orchestration is handled using **Databricks Workflows**, coordinating all pipelines:

```
Task 1 (Ingestion)
   ↓
Task 2 (Companies)
Task 3 (Shareholders)
Task 4 (Establishments)
```

* Ingestion runs first
* DLT pipelines run in parallel
* Pipelines are independent and decoupled

This design enables scalability, modularity, and selective reprocessing.

---

## Data Quality

DLT allows defining built-in data quality rules:

* `@dlt.expect` → validation
* `@dlt.expect_or_drop` → drop invalid records
* `@dlt.expect_or_fail` → fail pipeline

Example:

```python
@dlt.expect_or_drop("valid_capital", "capital_social >= 0")
```

---

## Pipeline Configuration

Key parameters:

* **Source**: notebook or Git repository
* **Target Schema**: `cnpj_dlt`
* **Storage Location**: `/mnt/cnpj/dlt/`
* **Mode**:

  * Triggered (batch)
  * Continuous (streaming)

---

## Data Consumption

Data can be consumed via:

* Databricks SQL
* BI tools (Power BI, Tableau)
* Analytical notebooks
* APIs (optional)

---

## Future Enhancements

* Join companies, shareholders, and establishments
* Enrich with geolocation data (IBGE)
* Build interactive dashboards
* Implement incremental ingestion (Auto Loader)
* Advanced data quality testing
* Feature Store for Machine Learning

---

## Use Cases

* Company distribution by sector (CNAE)
* Geographic distribution of companies
* Company profiling by size
* Business landscape analysis

---

## Project Goals

Demonstrate expertise in:

* Modern Data Engineering (Lakehouse architecture)
* Distributed processing with Spark
* Declarative pipelines with DLT
* Data quality and governance
* Analytical data modeling

---

## Status

In development

---

## Contributing

Feel free to open issues or contribute improvements.

---

## License

This project uses publicly available data from the Brazilian Federal Revenue.