# Enterprise Data Lakehouse for Corporate Registry & Governance

Pipeline de engenharia de dados baseado em arquitetura Lakehouse utilizando Delta Live Tables (DLT) para ingestão, transformação e análise dos dados públicos da Receita Federal (CNPJ).

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
Receita Federal (ZIP)
        ↓
Ingestão (Python Notebook)
        ↓
Data Lake (/mnt/cnpj/raw)
        ↓
DLT Pipeline
   ├── Bronze (dados brutos estruturados)
   ├── Silver (dados tratados)
   └── Gold (dados agregados)
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
│   └── dlt_cnpj_pipeline.py
│
├── config/
│
├── notebooks/
│
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

## Pipeline DLT

O pipeline é definido de forma declarativa utilizando DLT.

---

### Camada Bronze

* Leitura dos arquivos CSV brutos
* Estruturação inicial dos dados

```python
@dlt.table
def bronze_empresas():
    return (
        spark.read
        .format("csv")
        .option("sep", ";")
        .load("/mnt/cnpj/raw/empresas/")
    )
```

---

### Camada Prata

* Limpeza de dados
* Tipagem
* Deduplicação
* Regras de qualidade

```python
@dlt.table
@dlt.expect("cnpj_not_null", "cnpj_basico IS NOT NULL")
def silver_empresas():
    return dlt.read("bronze_empresas").dropDuplicates(["cnpj_basico"])
```

---

### Camada Ouro

* Agregações para análise

```python
@dlt.table
def gold_empresas_por_porte():
    return (
        dlt.read("silver_empresas")
        .groupBy("porte_empresa")
        .count()
    )
```

---

## Qualidade de Dados

O DLT permite definir regras de qualidade diretamente no pipeline:

* `@dlt.expect` → validação
* `@dlt.expect_or_drop` → remove registros inválidos
* `@dlt.expect_or_fail` → falha o pipeline

Exemplo:

```python
@dlt.expect_or_drop("capital_valido", "capital_social >= 0")
```

---

## Orquestração

A orquestração é feita automaticamente pelo DLT com base nas dependências entre tabelas:

```
bronze → silver → gold
```

Não há necessidade de DAGs ou ferramentas externas.

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