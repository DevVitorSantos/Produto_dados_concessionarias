# Produto de dados para Concessionárias

Pipeline ELT no ecossistema Google Cloud que transforma dados brutos de vendas de uma rede de concessionárias em um modelo estrela na camada gold, alimentando 5 dashboards estratégicos no Looker Studio.

![Produto de Dados]([file:///C:/Users/Usu%C3%A1rio/Documents/01_Ag_Abobora_Digital/OpenCode/novaDrive/docs/dahsboard%20clientes.JPG](https://github.com/DevVitorSantos/Produto_dados_concessionarias/blob/68bba8db265beaa08c7ce25f0684a703ba55c6e0/img/infografico.png))
---

## Objetivo do projeto:

O projeto deve ter a capacidade de dar apoio a diretoria, time de negócios e performance a **tomar decisões mais assertivas** no dia a dia.

Atualmente o projeto responde a **7 perguntas de negócio** da diretoria através de **5 dashboards** com cross-filtering entre todos os gráficos:

- Qual concessionária fatura mais?
- Quais modelos são mais vendidos?
- Quem são os vendedores com melhor desempenho?
- Existe concentração excessiva em poucos vendedores?
- Quem são os clientes mais valiosos?
- Como evolui o faturamento?

## Status do Projeto

**Fase:** Implementado / Pronto para uso  
**Custo estimado:** R$ 60 a R$ 100 por mês (BigQuery + Looker Studio) — sem servidor, sem licenças fixas.

### Custo atual do meu projeto <br>
este foi o custo atual do projeto para 7 dias de execução

![x](file:///C:/Users/Usu%C3%A1rio/Documents/01_Ag_Abobora_Digital/OpenCode/novaDrive/docs/dahsboard%20clientes.JPG)

---

## Stack Tecnológica

| Componente | Tecnologia | Custo |
|---|---|---|
| Cloud | Google Cloud Platform (GCP) | Incluso no free tier / $300 crédito inicial |
| Data Ingestion | Cloud Run  | R$ 60 - 80/mês  |
| Data Warehouse | Google BigQuery | R$ 10-20/mês — pay-per-query |
| ELT / Orquestração | Dataform (nativo GCP) | Gratuito (integrado ao BigQuery) |
| Visualização | Looker Studio | Gratuito |
| Versionamento | Git + GitHub | Gratuito |
| Linguagem | SQL (BigQuery dialect) | — |
| Linguagem | Python | — |

### Por que esta stack?


- **Processamento escalável** —  escalabilidade automática instantânea (com redução para zero quando ocioso), cobrança baseada no uso real (sem pagar nada quando não está em uso), e eliminação de gerenciamento de infraestrutura
- **Custo proporcional ao uso** — você paga apenas pelos bytes processados nas consultas
- **Ferramentas nativas Google** — Dataform e Looker Studio são gratuitos e integram nativamente com o BigQuery
- **Tudo em SQL e Python** — não depende de ferramentas proprietárias ou linguagens exóticas

---

## Arquitetura: Medalhão (Bronze → Silver → Gold)

```
┌────────────────────────────────────────────────────────────────────┐
│                          BRONZE (Raw)                              │
│  8 tabelas brutas: clientes, vendas, veiculos, vendedores,        │
│  concessionarias, cidades, estados, customers                     │
│  Dados como ingeridos, sem tratamento.                            │
└────────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │         Dataform ELT          │
              │  Limpeza, casts, JOINs,       │
              │  enriquecimento, cálculos     │
              └───────────────┬───────────────┘
                              ▼
┌────────────────────────────────────────────────────────────────────┐
│                          SILVER (Tratado)                          │
│  5 dimensões: dim_clientes, dim_veiculos, dim_vendedores,         │
│  dim_concessionarias, dim_calendario                               │
│  2 fatos: fato_vendas (limpa), fato_vendas_enriquecida (wide)     │
│  Métricas calculadas: desconto, faixa_preco, categoria_desconto,  │
│  ticket, percentuais.                                              │
└────────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │         Dataform ELT          │
              │  Renomeação, padronização,    │
              │  organização star schema      │
              └───────────────┬───────────────┘
                              ▼
┌────────────────────────────────────────────────────────────────────┐
│                   GOLD (Modelo Estrela)                            │
│                                                                     │
│                    ┌──────────┐                                     │
│                    │dim_tempo │                                     │
│                    └────┬─────┘                                     │
│                         │                                           │
│   ┌──────────┐          │          ┌──────────┐                    │
│   │dim_cliente│─────────┤─────────│dim_veiculo│                    │
│   └──────────┘          │          └──────────┘                    │
│                         │                                           │
│                    ┌────┴─────┐                                    │
│                    │fato_vendas│                                    │
│                    └────┬─────┘                                    │
│                         │                                           │
│   ┌──────────┐          │          ┌──────────────┐                │
│   │dim_vendedor│────────┤─────────│dim_concessionaria│             │
│   └──────────┘          │          └──────────────┘                │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │ vw_fato_completa (VIEW) — consulta única já com todos       │  │
│   │ os JOINs, pronta para Looker Studio                         │  │
│   └─────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │  Looker Studio   │
                    │  5 dashboards    │
                    │  Cross-filtering  │
                    └──────────────────┘
```

### O que cada camada faz

| Camada | Finalidade | Exemplos de transformação |
|---|---|---|
| **Bronze** | Raw copy — dados como chegam | Nenhuma transformação |
| **Silver** | Limpeza, tipagem, enriquecimento | `CAST(valor AS NUMERIC)`, JOINs com cidades/estados, geração de calendário (2020-2030), cálculo de `valor_desconto`, `percentual_desconto`, `faixa_preco`, `categoria_desconto` |
| **Gold** | Modelo estrela para BI | Renomeação para padrão singular (`id_cliente`, `id_veiculo`), separação em dimensões, view consolidada `vw_fato_completa` |

---

## Modelo de Dados — Gold (Star Schema)

### Dimensões

| Tabela | Chave | Atributos |
|---|---|---|
| `gold.dim_tempo` | `id_tempo` | `data`, `ano`, `semestre`, `trimestre`, `mes`, `nome_mes`, `ano_mes`, `ano_mes_texto`, `semana`, `dia`, `dia_ano`, `numero_dia_semana`, `nome_dia_semana`, `fim_de_semana`, `dia_util` |
| `gold.dim_cliente` | `id_cliente` | `cliente`, `endereco`, `id_concessionaria` |
| `gold.dim_veiculo` | `id_veiculo` | `modelo`, `tipo`, `valor_tabela` |
| `gold.dim_vendedor` | `id_vendedor` | `vendedor`, `id_concessionaria` |
| `gold.dim_concessionaria` | `id_concessionaria` | `concessionaria`, `cidade`, `estado`, `uf` |

### Fato

| Tabela | Chave | Medidas |
|---|---|---|
| `gold.fato_vendas` | `id_venda` | `valor_pago`, `valor_desconto`, `percentual_desconto`, `ticket_medio`, `quantidade_venda`, `faixa_preco`, `venda_com_desconto`, `categoria_desconto` |

### Chaves estrangeiras

```
fato_vendas.id_cliente         → dim_cliente.id_cliente
fato_vendas.id_veiculo         → dim_veiculo.id_veiculo
fato_vendas.id_vendedor        → dim_vendedor.id_vendedor
fato_vendas.id_concessionaria  → dim_concessionaria.id_concessionaria
fato_vendas.id_tempo           → dim_tempo.id_tempo
```

### View para Looker Studio

A `gold.vw_fato_completa` realiza todos os 5 JOINs e expõe todas as colunas em uma única tabela, resolvendo a limitação de máximo 5 tabelas no Blend do Looker Studio.

---

## Métricas Calculadas

| Métrica | Fórmula | Classificação |
|---|---|---|
| `valor_desconto` | `GREATEST(valor_tabela - valor_pago, 0)` | Nenhuma |
| `percentual_desconto` | `(valor_desconto / valor_tabela) × 100` | Leve (≤5%), Moderado (≤10%), Agressivo (>10%) |
| `faixa_preco` | Baseada em `valor_pago` | Popular (<R$30K), Padrão (R$30-50K), Alto Padrão (R$50-100K), Premium (≥R$100K) |
| `ticket_medio` | `valor_pago / quantidade_venda` | Nenhuma |
| `venda_com_desconto` | `CASE WHEN valor_tabela > valor_pago THEN 1 ELSE 0 END` | Binário |
| `semestre` | `CASE WHEN mes BETWEEN 1 AND 6 THEN 1 ELSE 2 END` | 1º ou 2º semestre |

---

## Dashboards (Looker Studio)

O projeto responde a **7 perguntas de negócio** da diretoria através de **5 dashboards** com cross-filtering entre todos os gráficos:

### 1. Dashboard Executivo
**Perguntas:** <br>
Qual concessionária fatura mais? <br>
Quais modelos são mais vendidos?  
**Métricas:** Faturamento total, total de vendas, ticket médio, desconto médio, clientes atendidos, modelos mais vendidos  
**Gráficos:** 5 scorecards + série temporal de faturamento + donut por concessionária + barras de modelos

![My Screenshot](file:///C:/Users/Usu%C3%A1rio/Documents/01_Ag_Abobora_Digital/OpenCode/novaDrive/docs/dahsboard%20clientes.JPG)

### 2. Dashboard Comercial
**Perguntas:** <br>
Quem são os vendedores com melhor desempenho? <br>
Existe concentração excessiva em poucos vendedores?  
**Métricas:** Vendedores ativos, meta vs realizado, ticket médio por vendedor, desconto médio, performance (destaque/bom/atenção)  
**Gráficos:** Gauge de meta + barras de ticket por vendedor + tabela de performance com badges + barras de desconto

### 3. Dashboard Clientes
**Perguntas:** <br>
Quem são os clientes mais valiosos?  
**Métricas:** Total de clientes, ticket médio por cliente, cliente top 1 (gasto total), clientes recorrentes  
**Gráficos:** 4 scorecards + barras top 10 clientes + donut por faixa de gasto + área empilhada recorrentes vs novos

### 4. Dashboard Regional
**Perguntas:** <br>
Quais estados/cidades têm maior potencial? <br>
Existe concentração excessiva em poucas regiões?  
**Métricas:** Estados atendidos, cidades atendidas, faturamento top 1 estado, concentração top 3 estados  
**Gráficos:** 4 scorecards + treemap por estado + barras por cidade + série temporal por estado + tabela detalhada

### 5. Dashboard Temporal
**Perguntas:** Como evolui o faturamento?  
**Métricas:** Faturamento do mês, YTD, meta anual, safra (mês mais rentável)  
**Gráficos:** 4 scorecards + barras mensais vs meta + ranking de safra + progressão YTD + YoY

### Cross-filtering

Qualquer filtro aplicado (cidade, mês, vendedor, concessionária) propaga automaticamente para todos os gráficos do dashboard. Isso é possível porque todos os gráficos consultam a mesma view `vw_fato_completa`.



---

## Análise complementar 
Essa análise tem objetivo de complementar o dash com dados mais complexos que podem ser feitos de forma mensal, trimestral ou semestral, obetivo é gerar análises passadas detalhadas para determinar se existem padrões e também analises de predição futuras ( forecast ) para dar um norte de crescimento baseado em dados para o futuro.

### Forecast
![My Screenshot](file:///C:/Users/Usu%C3%A1rio/Documents/01_Ag_Abobora_Digital/OpenCode/novaDrive/docs/dahsboard%20clientes.JPG)

objetivo: lorem ipsum lorem

---

## Custo Mensal Detalhado

| Recurso | Estimativa | Detalhamento |
|---|---|---|
| **Cloud run** | R$40 - R$60 / mês | Os $300 de crédito inicial + free tier mensal (10 GB de armazenamento, 1 TB de consultas/mês) cobrem os primeiros meses |
| **BigQuery** | R$ 10-20 / mês | Payu as you go. Custo por consulta e armazenamento. Estimativa para ~50 GB processados/mês em consultas e ~1 GB armazenado. O modelo estrela reduz bytes escaneados vs tabelas wide. |
| **Dataform** | R$ 0 | Gratuito (nativo BigQuery, pago apenas se usar o Dataform standalone, que não é o caso) |
| **Looker Studio** | R$ 0 | Gratuito (plano Pro é pago mas o gratuito cobre o uso deste projeto) |

| **Total estimado** | **R$ 60-100/mês** | Após o período gratuito, com uso moderado (consultas diárias dos dashboards + atualização do Dataform) |

> **Nota:** Este valor considera o BigQuery on-demand (pay-per-query). Para workloads intensas, o BigQuery Editions (capacidade reservada) pode ser mais econômico a partir de ~R$ 500/mês.

---

## Estrutura do Repositório

```
Produto_Dados/
├── definitions/
│   ├── silver/                               # 7 tabelas da camada silver (Dataform .sqlx)
│   │   ├── dim_clientes.sqlx
│   │   ├── dim_veiculos.sqlx
│   │   ├── dim_vendedores.sqlx
│   │   ├── dim_concessionarias.sqlx
│   │   ├── dim_calendario.sqlx
│   │   ├── fato_vendas.sqlx
│   │   └── fato_vendas_enriquecida.sqlx
│   └── gold/                                 # 6 tabelas + 1 view da camada gold
│       ├── dim_cliente.sqlx
│       ├── dim_veiculo.sqlx
│       ├── dim_vendedor.sqlx
│       ├── dim_concessionaria.sqlx
│       ├── dim_tempo.sqlx
│       ├── fato_vendas.sqlx
│       └── vw_fato_completa.sqlx             # View consolidada para Looker Studio
├── docs/
│   ├── wireframe_dashboards.md               # Wireframe ASCII dos 5 dashboards
│   ├── wireframe_dashboards.html             # Wireframe interativo em HTML
│   ├── wireframe_lookerstudio.html           # Wireframe no estilo Looker Studio
│   └── guia_looker_studio.md                 # Guia de implementação no Looker Studio
│   
├── img/                                      # Imagens do projeto
├── projeto_nova_drive_modelagem_BG.txt       # Documento de requisitos original
└── README.md                                 # Este arquivo
```

---

## Como Executar

### Pré-requisitos

1. Conta GCP com BigQuery ativado
2. Dataform habilitado no projeto GCP
3. Looker Studio (gratuito, conta Google)

### Passo a passo

1. **Crie as tabelas bronze** no BigQuery com os schemas descritos em `projeto_nova_drive_modelagem_BG.txt`
2. **Copie as definições** da pasta `definitions/` para o repositório do Dataform
3. **Execute o Dataform** na ordem:
   - Primeiro as dimensões silver, depois `fato_vendas`, depois `fato_vendas_enriquecida`
   - Depois as dimensões gold, depois `fato_vendas`, depois `vw_fato_completa`
4. **Conecte o Looker Studio** à view `gold.vw_fato_completa`
5. **Monte os gráficos** seguindo o guia em `docs/guia_looker_studio.md`

### Dependências entre tabelas (ordem de execução)

```
bronze.* ──→ silver.dim_* ──→ silver.fato_vendas ──→ silver.fato_vendas_enriquecida
                                                                      │
                                                                      ▼
                                                   gold.dim_* ──→ gold.fato_vendas ──→ gold.vw_fato_completa
```

O Dataform resolve automaticamente a ordem de execução através das referências `FROM` nos arquivos `.sqlx`.

---

## Licença

Projeto de portfólio. Livre para uso e modificação.
