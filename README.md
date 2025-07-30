# 📊 Sales Performance Lakehouse

> Lakehouse moderno para análise de performance de vendas com detecção automática de anomalias

## 🎯 Objetivo

Este projeto implementa um **Lakehouse** robusto que processa dados de vendas diárias, combinando a flexibilidade de um Data Lake com a estrutura e performance de um Data Warehouse. Aplica transformações inteligentes e regras de negócio para gerar insights acionáveis sobre performance comercial e detectar possíveis irregularidades nos preços.

## 🏗️ Arquitetura Lakehouse

### Medallion Architecture
```
📁 Landing Zone     →     📁 Raw Zone     →     📁 Bronze (Clean)     →     📁 Silver (Trusted)     →     📁 Gold (Analytics)
   Arquivo CSV           Parquet bruto        Limpeza básica          Regras de negócio        Dados para consumo
```

> **Lakehouse = Data Lake + Data Warehouse**  
> Combina a flexibilidade do armazenamento de dados não estruturados com a performance e governança de um Data Warehouse tradicional.

### Stack Tecnológica

| Componente | Tecnologia | Finalidade |
|------------|------------|------------|
| **Processamento** | PySpark | Transformações distribuídas dos dados |
| **Plataforma** | Databricks Lakehouse | Ambiente de execução e orquestração |
| **Orquestração** | Databricks Jobs/Tasks/Pipelines | Automação da pipeline |
| **Armazenamento** | DBFS | Lakehouse para todas as camadas |
| **Formato** | Parquet + Delta Tables | Armazenamento ACID e otimizado |

## 📋 Estrutura dos Dados

### Entrada (Landing)
```csv
venda_id,data,vendedor_id,vendedor_nome,produto_id,produto_nome,categoria,preco_unit,qtd,desconto,canal,cidade,estado,comentarios
V001,2024-01-15,123,João Silva,P001,Smartphone X,Eletrônicos,899.90,2,0.10,ecommerce,São Paulo,SP,lorem ipsum
```

### Raw Zone (Parquet)
```
Mesmo conteúdo da Landing, mas convertido para formato Parquet
- Melhor performance de leitura
- Compressão automática
- Preserva tipos de dados originais
```

### Saída (Gold)
```csv
venda_id,data,vendedor_id,produto_id,categoria,valor_total,desconto,score_venda,flag_anomalia
V001,2024-01-15,123,P001,Eletrônicos,1619.82,0.10,VENDA_PREMIUM,FALSE
```

## 🔄 Pipeline de Transformação

### 📥 Landing Zone - Recepção de Dados
- 📂 Recebe arquivo CSV: `vendas_mock.csv`
- 🏷️ Registra metadados do arquivo (nome, tamanho, timestamp)
- 📍 Armazena em `/landing/yyyy/MM/dd/`

### 📦 Raw Zone - Conversão para Parquet
- 🔄 Move arquivo da Landing para Raw
- 📊 Converte CSV para formato Parquet (sem transformações)
- 📍 Salva em `/raw/yyyy/MM/dd/vendas_mock.parquet`
- ✨ Mantém dados exatamente como recebidos

### 🥉 Bronze Layer - Limpeza e Padronização
- ✅ Remoção de registros com campos críticos nulos
- ✅ Conversão e validação de tipos de dados
- ✅ Padronização para snake_case
- ✅ Logging de qualidade dos dados

### 🥈 Silver Layer - Regras de Negócio

#### 💰 Cálculo de Valor Total
```python
valor_total = preco_unit * qtd * (1 - desconto)
```

#### 🏷️ Score de Classificação
- **ALTO_RISCO**: Desconto > 20%
- **VENDA_PREMIUM**: Valor total > R$ 1.000
- **NORMAL**: Demais casos

#### 🚨 Detecção de Anomalias
Identifica preços 30% acima/abaixo da média da categoria:
```python
if abs(preco_unit - preco_medio_categoria) / preco_medio_categoria > 0.30:
    flag_anomalia = True
```

### 🥇 Gold Layer - Dados Analíticos
- 📊 Dataset otimizado para BI e análises
- 🗜️ Apenas campos essenciais
- 📈 Pronto para dashboards e relatórios

## 🔧 Configuração e Execução

### Pré-requisitos
- Databricks Lakehouse Workspace configurado
- Cluster com PySpark habilitado
- Acesso ao DBFS e Unity Catalog

### Estrutura do Projeto
```
project/
├── notebooks/
│   ├── process_file_to_parquet.py
│   ├── bronze_clean.py
│   ├── silver_trusted.py
│   └── gold_final.py
├── config/
│   └── job.yaml              # Configuração da task Databricks
├── data/
│   └── vendas_mock.csv    # Arquivo de exemplo
└── README.md
```

### Deployment
1. **Upload dos notebooks** para o Databricks Workspace
2. **Configuração do Job** usando o arquivo `job.yaml`:
   ```yaml
   name: "Lakehouse Sales Performance Pipeline"
   tasks:
     - task_key: "landing_to_raw"
       notebook_task:
         notebook_path: "/notebooks/00_landing_to_raw"
     - task_key: "raw_to_bronze"
       depends_on:
         - task_key: "landing_to_raw"
       notebook_task:
         notebook_path: "/notebooks/bronze_clean"
        ...
   ```

3. **Trigger** da execução via *File Arrival*

## 📈 Monitoramento e Controle

### Tabela de Auditoria
Cada execução registra métricas de qualidade:

| Métrica | Descrição |
|---------|-----------|
| `linhas_recebidas` | Total de registros no arquivo original |
| `linhas_validas` | Registros após limpeza |
| `colunas_removidas` | Campos descartados na transformação |
| `regras_aplicadas` | Regras de negócio executadas |
| `status` | Sucesso/Erro da execução |

## 🎯 Casos de Uso

### Para Gestores de Vendas
- Identificar vendedores com alta taxa de descontos
- Monitorar performance por categoria de produto
- Detectar padrões sazonais nas vendas

### Para Auditoria
- Rastrear vendas com preços suspeitos
- Validar aplicação de políticas de desconto
- Gerar relatórios de conformidade

### Para Analistas de BI
- Dashboard de performance em tempo real
- Análises de tendência e forecasting
- Segmentação de clientes e produtos

## 🚀 Próximos Passos

- [ ] Implementar ML para detecção avançada de fraudes
- [ ] Implementar alertas automáticos para anomalias
- [ ] Implementar ML + Alertas análise em tempo real com Kafka

---

**Desenvolvido por Eduardo Carvalho**