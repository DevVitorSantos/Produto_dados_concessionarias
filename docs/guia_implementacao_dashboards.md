# Guia de Implementação dos Dashboards — Nova Drive

> **Fonte de dados:** `gold.vw_fato_completa` (view única com todos os JOINs)
> **Ferramenta:** Looker Studio
> **Atalho:** vá direto ao dashboard desejado: [Executivo](#1-dashboard-executivo) · [Comercial](#2-dashboard-comercial) · [Clientes](#3-dashboard-clientes) · [Regional](#4-dashboard-regional) · [Temporal](#5-dashboard-temporal)

---

## 1. Dashboard Executivo

### Scorecard — Faturamento Total
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `SUM(valor_pago)` |
| Formato | Moeda (R$) — `#.##0,00` |
| Prefixo | `R$ ` |
| Abreviação compacta | Ativar (milhar = K, milhão = M) |
| Dica extra | Ativar "Mostrar comparação com período anterior" |

### Scorecard — Total de Vendas
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `COUNT(id_venda)` |
| Formato | Número — `#.##0` |

### Scorecard — Ticket Médio
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `SUM(valor_pago) / COUNT(id_venda)` |
| Formato | Moeda (R$) — `#.##0,00` |

### Scorecard — Desconto Médio
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `AVG(percentual_desconto)` |
| Formato | Porcentagem — `0,00%` |

### Scorecard — Clientes Atendidos
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `COUNT(DISTINCT id_cliente)` |
| Formato | Número — `#.##0` |

### Gráfico de Série Temporal — Faturamento ao Longo do Tempo
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Série temporal** / **Time series** |
| Dimensão (Eixo X) | `data` (granularidade: mês) |
| Métrica (Eixo Y) | `SUM(valor_pago)` |
| Linha de tendência | Ativar (polinomial 2º grau) |
| Linha de referência | Adicionar `meta_mensal` como parâmetro (R$ 200K) — linha tracejada vermelha |
| Cor da série | Azul `#1967d2` |
| Rótulos | Desativar (poluição visual) |
| Dica | Incluir média móvel 3 meses como 2ª série (opcional) |

### Gráfico de Rosca — Vendas por Concessionária
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Rosca** / **Donut** |
| Dimensão | `concessionaria` |
| Métrica | `SUM(valor_pago)` |
| Legenda | Posição: direita |
| Mostrar total no centro | Ativar (exibir `R$ 2,4M`) |
| Cores | Alpha `#1967d2`, Beta `#ea8600`, Gamma `#34a853`, Delta `#d93025` |

### Gráfico de Barras — Modelos Mais Vendidos
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `modelo` |
| Métrica | `COUNT(id_venda)` |
| Ordenação | Decrescente por métrica |
| Limite | Top 10 |
| Rótulos | Mostrar valor (`#0`) + percentual |
| Cor | Gradiente azul (`#1967d2` → `#c6dafc`) |
| Cabeçalho do eixo Y | "Modelo" |
| Cabeçalho do eixo X | "Vendas" |

### Gráfico de Barras — Faturamento por Modelo
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `modelo` |
| Métrica | `SUM(valor_pago)` |
| Ordenação | Decrescente |
| Limite | Top 10 |
| Formato da métrica | Moeda (R$) |
| Cor | Gradiente azul escuro (`#1967d2` → `#aecbfa`) |

---

## 2. Dashboard Comercial

### Scorecard — Vendedores Ativos
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `COUNT(DISTINCT id_vendedor)` |
| Filtro sugerido | `ano_mes = Mês Atual` (apenas vendedores com venda no mês) |
| Formato | Número — `#.##0` |
| Subtexto | "de 25 cadastrados" (valor fixo no texto do gráfico) |

### Medidor — Meta do Mês
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Medidor** / **Gauge** |
| Métrica 1 (Realizado) | `SUM(valor_pago)` com filtro `ano_mes = Mês Atual` |
| Métrica 2 (Meta) | Parâmetro `meta_mensal` (valor padrão: `200000`) |
| Faixas | Vermelho < 70% · Amarelo 70–90% · Verde ≥ 90% |
| Rótulo central | `{valor_atual / meta}` formatado como % |
| Mostrar valores | Meta: R$ 200K · Real: R$ 180K |

### Scorecard — Ticket Médio Geral
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `SUM(valor_pago) / COUNT(id_venda)` |
| Formato | Moeda (R$) |
| Subtexto | "Maior: R$ 18,2K (Ana)" (usar tabela auxiliar) |

### Gráfico de Barras — Ticket Médio por Vendedor
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `vendedor` |
| Métrica | `SUM(valor_pago) / COUNT(id_venda)` |
| Ordenação | Decrescente |
| Rótulos | Mostrar valor em R$ |
| Cor | Azul (`#1967d2` → `#c6dafc`) |

### Tabela com Badges — Performance dos Vendedores
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Tabela** / **Table** |
| Dimensão | `vendedor` |
| Coluna 1 | `COUNT(id_venda)` — rótulo "Vendas" |
| Coluna 2 | `SUM(valor_pago)` — rótulo "Faturamento", formato R$ |
| Coluna 3 (Performance) | **Campo calculado:** `CASE WHEN SUM(valor_pago) >= PERCENTILE_CONT(SUM(valor_pago), 0.8) OVER () THEN '⭐ Destaque' WHEN SUM(valor_pago) >= PERCENTILE_CONT(SUM(valor_pago), 0.5) OVER () THEN '👍 Bom' ELSE '⚠ Atenção' END` |
| Cor condicional | Destaque = fundo verde · Bom = fundo azul claro · Atenção = fundo amarelo |

**Nota:** Looker Studio não tem `PERCENTILE_CONT`. Alternativa prática: crie 3 scorecards separados com filtros percentuais (Top 20%, 20–50%, 50%+) ou use um campo calculado com `RANK()`.

### Gráfico de Barras — Desconto Médio Concedido
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `vendedor` |
| Métrica | `AVG(percentual_desconto)` |
| Ordenação | Decrescente |
| Cor | Gradiente verde→amarelo→vermelho (menor desconto = verde `#34a853`, maior = vermelho `#ea4335`) |

### Gráfico de Barras — Distribuição de Vendas por Concessionária
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `concessionaria` |
| Métrica | `COUNT(id_venda)` |
| Rótulos | Mostrar valor + percentual (ex: "65 (42%)") |
| Cor | Azul (`#1967d2` → `#8ab4f8`) |

---

## 3. Dashboard Clientes

### Scorecard — Total de Clientes
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `COUNT(DISTINCT id_cliente)` |
| Formato | Número — `#.##0` |
| Comparação | Período anterior |

### Scorecard — Ticket Médio por Cliente
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `SUM(valor_pago) / COUNT(DISTINCT id_cliente)` |
| Formato | Moeda (R$) |

### Scorecard — Cliente Top 1
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Tabela** (1 linha sem cabeçalho) / **Table** |
| Dimensão | `cliente` |
| Métrica | `SUM(valor_pago)` — renomear "Gasto Total" |
| Ordenação | Decrescente, limite 1 |
| Formato | Primeira célula: nome do cliente (texto grande), segunda célula: valor em R$ |
| Dica | Esconder cabeçalho da tabela e ajustar tamanho da fonte |

### Scorecard — Clientes Recorrentes
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `COUNT(DISTINCT CASE WHEN COUNT(id_venda) OVER (PARTITION BY id_cliente) > 1 THEN id_cliente END)` |
| Alternativa | Pré-calcular flag `cliente_recorrente` no BigQuery |
| Subtexto | "17,2% dos clientes" — campo calculado: `[Recorrentes] / COUNT(DISTINCT id_cliente)` formatado como % |

### Gráfico de Barras — Top Clientes por Gasto Total
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `cliente` |
| Métrica | `SUM(valor_pago)` |
| Ordenação | Decrescente |
| Limite | Top 10 |
| Formato | Moeda (R$) |
| Cor | Azul escuro (`#1967d2` → `#e0ecff`) |

### Gráfico de Rosca — Clientes por Faixa de Gasto
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Rosca** / **Donut** |
| Dimensão | **Campo calculado:** `CASE WHEN SUM(valor_pago) >= 100000 THEN '> R$ 100K' WHEN SUM(valor_pago) >= 50000 THEN 'R$ 50-100K' WHEN SUM(valor_pago) >= 20000 THEN 'R$ 20-50K' ELSE '< R$ 20K' END` |
| Métrica | `COUNT(DISTINCT id_cliente)` |
| Legenda | Direita, com percentual e valor absoluto |
| Cores | Escala sequencial azul (`#1967d2` → `#e0ecff`) |

### Gráfico de Áreas Empilhadas — Recorrentes vs Novos
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de áreas empilhadas** / **Stacked area chart** |
| Dimensão (Eixo X) | `ano_mes` |
| Série 1 | `COUNT(DISTINCT CASE WHEN [cliente_recorrente] THEN id_cliente END)` — cor azul `#1967d2` |
| Série 2 | `COUNT(DISTINCT CASE WHEN NOT [cliente_recorrente] THEN id_cliente END)` — cor cinza `#dadce0` |
| Dica | Pré-calcular flag `cliente_recorrente` no BigQuery para simplicidade |
| Eixo Y | Número de clientes |
| Legenda | Topo |

### Gráfico de Barras — Ticket Médio por Cliente (Top 5)
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `cliente` |
| Métrica | `SUM(valor_pago)` |
| Limite | Top 5 |
| Ordenação | Decrescente |
| Cor | Azul (`#1967d2` → `#aecbfa`) |

---

## 4. Dashboard Regional

### Scorecard — Estados Atendidos
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `COUNT(DISTINCT estado)` |
| Subtexto | Lista dos estados (SP, RJ, MG, RS, PR) |

### Scorecard — Cidades Atendidas
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | `COUNT(DISTINCT cidade)` |
| Subtexto | "Em 5 estados" |

### Scorecard — Faturamento Top 1 Estado
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Tabela** (1 linha sem cabeçalho) / **Table** |
| Dimensão | `estado` |
| Métrica | `SUM(valor_pago)` |
| Ordenação | Decrescente, limite 1 |
| Subtexto | "R$ 980K — 40% do total" (percentual calculado: `SUM(valor_pago) / SUM(valor_pago) OVER ()`) |

### Scorecard — Concentração Top 3 Estados
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Métrica | **Campo calculado:** `SUM(CASE WHEN estado IN ('SP','RJ','MG') THEN valor_pago ELSE 0 END) / SUM(valor_pago)` |
| Formato | Porcentagem — `0%` |
| Subtexto | "R$ 1,91M dos R$ 2,45M" |

### Gráfico de Mapa de Árvore — Faturamento por Estado
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Mapa de árvore** / **Treemap** |
| Dimensão | `estado` |
| Métrica | `SUM(valor_pago)` |
| Cor | Gradiente: menor = azul claro (`#aecbfa`), maior = azul escuro (`#1967d2`) |
| Rótulos | Mostrar sigla do estado + valor + percentual |

### Gráfico de Barras — Faturamento por Cidade (Top 5)
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `cidade` |
| Métrica | `SUM(valor_pago)` |
| Ordenação | Decrescente |
| Limite | Top 5 |
| Cor | Azul (`#1967d2` → `#aecbfa`) |

### Gráfico de Colunas — Evolução por Estado (Top 3)
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de colunas agrupadas** / **Grouped column chart** |
| Dimensão (Eixo X) | `ano_mes` |
| Série 1 | `SUM(CASE WHEN estado = 'SP' THEN valor_pago END)` — cor `#1967d2` |
| Série 2 | `SUM(CASE WHEN estado = 'RJ' THEN valor_pago END)` — cor `#ea8600` |
| Série 3 | `SUM(CASE WHEN estado = 'MG' THEN valor_pago END)` — cor `#34a853` |
| Legenda | Topo |

### Tabela — Detalhamento por Estado
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Tabela** / **Table** |
| Dimensão | `estado` |
| Coluna 1 | `SUM(valor_pago)` — "Faturamento", formato R$ |
| Coluna 2 | `COUNT(id_venda)` — "Vendas" |
| Coluna 3 | `SUM(valor_pago) / SUM(SUM(valor_pago)) OVER ()` — "Participação", formato % |
| Coluna 4 | `SUM(valor_pago) / COUNT(id_venda)` — "Ticket Médio", formato R$ |
| Linha de total | Ativar (soma das colunas) |

---

## 5. Dashboard Temporal

### Scorecard — Faturamento do Mês
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Filtro | `ano_mes = Último mês com dados` |
| Métrica | `SUM(valor_pago)` |
| Comparação | Mês anterior |
| Subtexto | Nome do mês (ex: "Junho/2026") |

### Scorecard — Faturamento YTD
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Cartão** / **Scorecard** |
| Filtro | Intervalo de datas = **Acumulado no ano (YTD)** |
| Métrica | `SUM(valor_pago)` |
| Comparação | YTD ano anterior (ex: +27%) |

### Gráfico de Medidor — Meta Anual
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Medidor de progresso** (barra horizontal) / **Progress bar** |
| Métrica 1 | `SUM(valor_pago)` com filtro YTD |
| Métrica 2 | Parâmetro `meta_anual` (valor padrão: `3000000`) |
| Fórmula visual | "R$ 1,4M de R$ 3,0M" |
| Barra | Preenchimento gradiente azul com label "47%" no centro |

### Scorecard — Mês + Rentável (Safra)
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Tabela** (Top 1 sem cabeçalho) / **Table** |
| Dimensão | `nome_mes` (agregado histórico, ignorando ano) |
| Métrica | `SUM(valor_pago)` |
| Ordenação | Decrescente |
| Limite | 1 |
| Subtexto | "R$ 280K — maior faturamento" |

### Gráfico de Colunas com Linha de Referência — Faturamento Mensal vs Meta
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de colunas** + **Linha de referência** / **Column chart + Reference line** |
| Dimensão (Eixo X) | `ano_mes` (últimos 12 meses) |
| Métrica | `SUM(valor_pago)` — colunas azuis `#1967d2` |
| Linha de referência | Parâmetro `meta_mensal` (R$ 200K) — linha tracejada vermelha `#d93025` |
| Rótulos de dados | Mostrar valores nas colunas |
| Eixo Y | Moeda (R$) começando em 0 |
| Dica extra | Adicionar 2ª série: média móvel 3 meses (linha, cor laranja) |

### Gráfico de Barras — Meses Mais Rentáveis (Ranking)
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de barras** / **Bar chart** |
| Orientação | Horizontal |
| Dimensão | `nome_mes` |
| Métrica | `SUM(valor_pago)` |
| Ordenação | Decrescente |
| Limite | Top 6 |
| Rótulos | Mostrar valor em R$ + medalha (🥇 🥈 🥉) |
| Cor | Gradiente azul (1º = `#1967d2`, último = `#c6dafc`) |

### Gráfico de Área — Acumulado YTD
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de área** / **Area chart** |
| Dimensão (Eixo X) | `ano_mes` |
| Métrica | `SUM(valor_pago)` com **Running sum** (acumulado) |
| Ordenação | `ano_mes` ascendente |
| Cor | Azul `#1967d2` com opacidade 50% |
| Linha de meta | Adicionar linha em R$ 3,0M como referência |
| Rótulos | Mostrar valor apenas no último ponto |

### Gráfico de Colunas Agrupadas — Crescimento YoY
| Campo | Valor |
|---|---|
| Tipo (PT / EN) | **Gráfico de colunas agrupadas** / **Grouped column chart** |
| Dimensão (Eixo X) | `mes` (1 a 12) |
| Série 1 | `SUM(CASE WHEN ano = 2024 THEN valor_pago END)` — cor `#aecbfa` |
| Série 2 | `SUM(CASE WHEN ano = 2025 THEN valor_pago END)` — cor `#669df6` |
| Série 3 | `SUM(CASE WHEN ano = 2026 THEN valor_pago END)` — cor `#1967d2` |
| Legenda | Topo, com rótulos "2024", "2025", "2026" |
| Dica extra | Scorecard auxiliar: `(SUM(CASE WHEN ano = 2026 THEN valor_pago END) - SUM(CASE WHEN ano = 2025 THEN valor_pago END)) / SUM(CASE WHEN ano = 2025 THEN valor_pago END)` formatado como % — exibindo "+27%" |

---

## Resumo de Tipos de Gráfico Utilizados

| Nome (PT) | Nome (EN) | Ocorrências |
|---|---|---|
| Cartão | Scorecard | 18 |
| Gráfico de barras | Bar chart (horizontal) | 10 |
| Gráfico de colunas | Column chart | 1 |
| Gráfico de colunas agrupadas | Grouped column chart | 2 |
| Série temporal | Time series | 1 |
| Rosca | Donut chart | 2 |
| Mapa de árvore | Treemap | 1 |
| Medidor | Gauge | 2 |
| Tabela | Table | 3 |
| Gráfico de áreas empilhadas | Stacked area chart | 1 |
| Gráfico de área | Area chart | 1 |
| Barra de progresso | Progress bar | 1 |

---

## Configuração Global

### Filtros por Dashboard

| Dashboard | Filtro 1 | Filtro 2 | Filtro 3 |
|---|---|---|---|
| Executivo | Cidade (dropdown) | Mês (dropdown) | Vendedor (dropdown) |
| Comercial | Mês (dropdown) | Concessionária (dropdown) | — |
| Clientes | Período (dropdown) | Cidade (dropdown) | — |
| Regional | Ano (dropdown) | Vendedor (dropdown) | Concessionária (dropdown) |
| Temporal | Ano (dropdown) | Concessionária (dropdown) | Data range |

### Paleta de Cores (Google Material)

| Função | Cor | Hex |
|---|---|---|
| Primário (destaque) | Azul Google | `#1967d2` |
| Secundário | Azul médio | `#4285f4` |
| Terciário | Azul claro | `#669df6` |
| Container | Azul muito claro | `#e8f0fe` |
| Sucesso / Meta OK | Verde | `#34a853` |
| Atenção | Amarelo | `#fbbc04` |
| Alerta / Crítico | Vermelho | `#ea4335` |
| Neutro (fundo de track) | Cinza claro | `#f1f3f4` |

### Parâmetros de Meta (criar em Recurso > Gerenciar parâmetros)

| Nome | Tipo | Valor padrão | Descrição |
|---|---|---|---|
| `meta_mensal` | Número | `200000` | Meta de faturamento mensal (em R$) |
| `meta_anual` | Número | `3000000` | Meta de faturamento anual (em R$) |
