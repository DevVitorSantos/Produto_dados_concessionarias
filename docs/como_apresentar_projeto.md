# Como Apresentar o Projeto Nova Drive

## Antes de tudo — domine o pitch de 1 minuto (vale para todos os públicos)

> *"Projetei e construí um pipeline de dados completo no GCP para uma rede de concessionárias. Os dados brutos (8 tabelas) passam por um processo ELT no Dataform — limpeza, enriquecimento e modelagem — até virar um modelo estrela no BigQuery. Isso alimenta 5 dashboards no Looker Studio que respondem perguntas reais da diretoria: faturamento, performance de vendedores, clientes mais valiosos, análise regional e tendências temporais."*

Mude o tom e a profundidade conforme o público abaixo.

---

## 1. Para Profissionais da Área (pares técnicos, entrevistas para vagas)

**Objetivo:** demonstrar domínio técnico, boas práticas de engenharia de dados, capacidade de设计 arquiteturas escaláveis.

### O que enfatizar

| Tópico | Como falar |
|---|---|
| **Arquitetura Medalhão** (Bronze → Silver → Gold) | Explique por que separar em camadas: bronze mantém dado original (rastreabilidade), silver aplica limpeza e enriquecimento (reuso), gold organiza em star schema (performance de query e BI). Isso mostra que você conhece o padrão ouro de data lakes modernos. |
| **Medalhões vs Camadas simples** | Diferencie seu projeto de um "ETL qualquer": você tem separação clara de responsabilidades, cada camada tem um propósito distinto. |
| **Modelo Estrela** | Explique a decisão de usar star schema na gold: fato central (fato_vendas) com 5 dimensões ao redor. Mostre que entende o trade-off: desnormalização vs performance de query. Cite que a view `vw_fato_completa` foi criada para suprir a limitação de 5 tabelas do Blend do Looker Studio. |
| **Dataform (ELT)** | Destaque que usou Dataform — SQL puro, versionamento, dependências entre tabelas resolvidas automaticamente. Isso mostra que você entende de engenharia de dados moderna (não está preso a ferramentas low-code). |
| **BigQuery** | Fale sobre diferenças de custo: tabelas particionadas (se aplicável), slot allocation, e por que escolheu NUMERIC em vez de FLOAT64 para valores monetários. |
| **Cálculos de Negócio** | Mostre as métricas calculadas: `percentual_desconto`, `faixa_preco`, `categoria_desconto`. Isso demonstra que você não só move dados — você os transforma em informação de negócio. |

### O que não enfatizar (ou simplificar)

- Nomes de colunas e schemas detalhados (a não ser que perguntem)
- Wireframes HTML (é um bônus visual, não o core técnico)
- Guia do Looker Studio passo a passo (muito operacional para uma conversa técnica)

### Potenciais perguntas técnicas e respostas

| Pergunta | Resposta esperada |
|---|---|
| *Por que star schema e não snowflake?* | "Performance de query no BigQuery e simplicidade para o BI. O Looker Studio se beneficia de menos JOINs. Como o BigQuery é columnar e paga por bytes escaneados, star schema reduz o número de tabelas no caminho crítico." |
| *Como garantir a consistência dos dados entre as camadas?* | "O Dataform gerencia as dependências — cada tabela silver só executa depois que a bronze correspondente terminou. Uso LEFT JOINs para não perder vendas órfãs. A `data_processamento` timestampa cada execução para rastreabilidade." |
| *Por que criar a view vw_fato_completa se o star schema já está na gold?* | "Limitação do Looker Studio: o Blend permite no máximo 5 tabelas, e meu modelo tem 6 (1 fato + 5 dims). A view resolve isso sem perder a normalização do modelo estrela — é uma camada de apresentação, não de armazenamento." |
| *Como tratou dados duplicados ou inconsistentes?* | "Na silver, uso `CAST` explícito para NUMERIC/DATE para garantir tipos. Na enriched, uso `GREATEST(valor_tabela - valor_pago, 0)` para evitar descontos negativos. LEFT JOINs garantem que vendas com IDs órfãos não se percam." |
| *Qual a estratégia de testes?* | (Se aplicável) "O Dataform permite `assertions` — criei `rowConditions` para valor_pago > 0, data_venda não nula, e uniqueness de IDs nas dimensões." |

### Formato de apresentação sugerido

1. **Contexto (30s):** "Rede de concessionárias, 8 tabelas brutas, dados espalhados."
2. **Problema (30s):** "Diretoria tinha 7 perguntas de negócio sem resposta integrada."
3. **Solução (2min):** Mostre a arquitetura bronze → silver → gold. Aponte cada decisão de design.
4. **Resultado (1min):** "5 dashboards, cross-filtering funcional, dados atualizados diariamente."

> ⚠ **Dica para entrevistas:** Se for在一 vaga presencial, leve o diagrama da arquitetura impresso (ou no notebook). Use o arquivo `wireframe_dashboards.html` como demonstração visual rápida.

---

## 2. Para Gestores e Coordenadores

**Objetivo:** mostrar impacto no negócio, capacidade de traduzir perguntas da diretoria em entregáveis de dados, organização do projeto.

### O que enfatizar

| Tópico | Como falar |
|---|---|
| **Perguntas de negócio → Dashboards** | Mostre a tabela de correspondência: cada pergunta da diretoria vira um gráfico. Ex: "Qual concessionária fatura mais?" → gráfico de pizza no dashboard executivo. "Como evolui o faturamento?" → série temporal no dashboard temporal. Isso mostra que você pensa no negócio primeiro. |
| **5 dashboards, 1 modelo** | Destaque que com um único modelo estrela você atende 5 públicos diferentes (executivo, comercial, clientes, regional, temporal). Isso é eficiência — não precisou criar 5 modelos diferentes. |
| **Cross-filtering** | "Qualquer filtro de cidade, mês ou vendedor propaga para todos os gráficos automaticamente." Isso é o que gestores querem ouvir — análise dinâmica sem esforço manual. |
| **Tecnologias Google** | "GCP + BigQuery + Dataform + Looker Studio = stack 100% nativa Google, sem custo de licenças extras, escalável sob demanda." Gestores se importam com custo e integração. |
| **Metas e acompanhamento** | Mostre que o dashboard temporal inclui "Meta Anual" e "Faturamento YTD". Gestores vivem de meta vs realizado. |
| **Manutenibilidade** | "O Dataform versiona tudo no Git. Se um vendedor muda de concessionária, atualiza a dim_vendedor e todos os dashboards refletem automaticamente." |

### O que não enfatizar

- Diferença entre NUMERIC e FLOAT64 (muito técnico)
- O nome de cada coluna ou tabela
- Código SQL (mostre o resultado, não a implementação)
- Blends e limitação de 5 tabelas (a solução já foi implementada — view única)

### Como estruturar a conversa

| Momento | O que dizer |
|---|---|
| **1. O problema** | "A diretoria tinha 7 perguntas sobre o negócio — faturamento, vendedores, clientes, regiões — mas os dados estavam espalhados em 8 tabelas diferentes, sem integração." |
| **2. O que foi feito** | "Estruturei os dados em 3 camadas: uma cópia bruta, uma camada tratada e enriquecida, e um modelo estrela otimizado para BI — tudo no BigQuery, orquestrado pelo Dataform." |
| **3. O resultado** | Abra o `wireframe_dashboards.html` e mostre os 5 dashboards. "Com um clique, o executivo vê faturamento total, o comercial vê performance dos vendedores, o regional vê concentração por estado." |
| **4. O valor** | "Decisões baseadas em dados, não em achismo. Tempo de resposta: de dias (pedir relatório para TI) para segundos (abrir o dashboard e filtrar)." |

### Métricas de impacto para citar

- **Tempo de resposta:** de 2-3 dias (solicitação manual) para tempo real
- **Unificação:** 8 tabelas brutas → 1 modelo estrela → 5 dashboards
- **Automação:** pipeline ELT auto-executável no Dataform (não precisa intervenção manual)
- **Custo:** BigQuery paga por consulta — star schema reduz bytes escaneados

---

## 3. Para RH / Recrutadores (não técnicos)

**Objetivo:** provar que você sabe o que está fazendo, tem experiência prática com ferramentas relevantes, e consegue se comunicar claramente.

### O que enfatizar

| Tópico | Como falar |
|---|---|
| **Stack relevante** | "Trabalhei com GCP, BigQuery, Dataform e Looker Studio." São palavras-chave que o RH busca. |
| **Papel no projeto** | "Projetei a arquitetura, implementei todo o pipeline de dados (do raw ao dashboard), e documentei o projeto." Mostra protagonismo. |
| **Tamanho e complexidade** | "8 tabelas brutas integradas em 1 modelo de dados, gerando 5 dashboards para públicos diferentes." Mostra escala. |
| **Linguagem** | "SQL — padrão de mercado para engenharia de dados." |
| **Metodologia** | "Modelagem medalhão (bronze, silver, gold), ELT com Dataform, versionado no Git." Palavras-chave que aparecem em descrições de vaga. |
| **Resultado de negócio** | "As perguntas da diretoria passaram a ser respondidas em tempo real, sem depender de TI para gerar relatórios." |

### Palavras-chave para currículo e LinkedIn

Extraia estas palavras do projeto para usar no seu resumo:

```
GCP | BigQuery | Dataform | Looker Studio | SQL
Modelagem de Dados | Star Schema | ELT
Pipeline de Dados | Data Warehouse | Medalhon Architecture
Dashboard | Visualização de Dados | Analytics
Engenharia de Dados | Análise de Negócio | BI
```

### Possíveis perguntas do RH e respostas

| Pergunta | Resposta |
|---|---|
| *O que esse projeto faz?* | "Organiza dados de vendas de uma rede de concessionárias e cria dashboards para a diretoria acompanhar faturamento, vendedores, clientes e regiões — tudo em tempo real." |
| *Qual tecnologia você usou?* | "Google Cloud Platform — BigQuery como banco, Dataform para organizar o pipeline, Looker Studio para os gráficos." |
| *Qual foi seu papel?* | "Fiz tudo — desde entender as perguntas de negócio, modelar os dados, escrever o SQL, até criar os dashboards e documentar." |
| *É um projeto real ou acadêmico?* | (Se for acadêmico/portfolio) "É um projeto pessoal que simula um cenário real de uma rede de concessionárias. Usei dados consistentes e segui as melhores práticas do mercado — arquitetura medalhão, star schema, pipeline ELT." |
| *Quanto tempo levou?* | "O desenvolvimento do pipeline e dashboards foi feito de forma estruturada — a modelagem, a implementação SQL e a documentação." (Mencione horas/semanas se aplicável.) |
| *Tem link para ver?* | "O código está documentado e organizado em pastas. Posso compartilhar o repositório ou uma apresentação resumida." |

### Como descrever o projeto no LinkedIn (sugestão de texto)

> **Projeto Nova Drive — Pipeline de Dados no GCP**
> 
> Projetei e implementei um pipeline de dados completo no ecossistema Google Cloud para transformar dados brutos de vendas de uma rede de concessionárias em dashboards estratégicos.
> 
> **O que foi feito:**
> - Arquitetura medalhão (Bronze → Silver → Gold) com modelagem star schema
> - ELT com Dataform + BigQuery (SQL puro, versionado)
> - 7 perguntas de negócio da diretoria respondidas em 5 dashboards no Looker Studio
> - Métricas: faturamento, ticket médio, performance de vendedores, clientes mais valiosos, análise regional, safra temporal
> - Cross-filtering entre todos os gráficos
> 
> **Stack:** GCP | BigQuery | Dataform | Looker Studio | SQL | Star Schema | ELT

---

## Resumo: o que mostrar para cada público

| Público | Entregar | Formato |
|---|---|---|
| **Técnico** | Arquitetura + SQL + decisões de modelagem + trade-offs | Código no repositório + diagrama da arquitetura |
| **Gestor** | Dashboards + perguntas respondidas + impacto no negócio | HTML interativo (`wireframe_dashboards.html`) + explicação oral |
| **RH** | Stack + palavras-chave + descrição clara do projeto | LinkedIn + currículo + pitch de 1 minuto |

### Checklist de preparação

- [ ] Tenho o pitch de 1 minuto decorado (funciona para qualquer público)
- [ ] O repositório está organizado (pastas silver/, gold/, docs/)
- [ ] Consigo explicar o star schema desenhando num guardanapo
- [ ] Sei responder "por que Dataform e não Airflow?" (ou similar)
- [ ] O wireframe HTML abre no navegador sem erros
- [ ] Meu LinkedIn tem as palavras-chave: GCP, BigQuery, Dataform, Star Schema, ELT
