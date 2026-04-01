# mobilidade-urbana-nyc
Análise de ~20 milhões de corridas de táxi em Nova York com Power BI, DAX e Apache Parquet. Dashboards de visão geral, análise temporal, perfil de passageiros e distribuição geográfica.
# 🚖 O Pulso da Mobilidade Urbana em Nova York
### Análise Completa das Corridas de Táxi em NYC com Power BI

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Parquet](https://img.shields.io/badge/Apache%20Parquet-50ABF1?style=for-the-badge&logo=apacheparquet&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-brightgreen?style=for-the-badge)

---

## 📌 Visão Geral do Projeto

Este projeto apresenta uma análise completa das corridas de táxi da cidade de Nova York, utilizando uma base de dados pública com **quase 20 milhões de registros**. O objetivo foi extrair insights estratégicos sobre mobilidade urbana, padrões temporais, comportamento dos passageiros e distribuição geográfica das corridas.

> **Desafio técnico:** A base de dados original possui dezenas de milhões de linhas, exigindo processamento inteligente, segmentação em arquivos Parquet e otimização de modelo no Power BI para garantir performance e escalabilidade.

---

## 📊 Dashboards Desenvolvidos

O projeto é composto por **4 painéis interativos**, acessíveis por uma tela de navegação central:

### 1. 🗂️ Visão Geral
Painel executivo com os principais KPIs do período analisado (janeiro a junho).

| Métrica | Valor |
|---|---|
| Total de corridas | 19.817.561 |
| Receita total | $414,24 Mi |
| Distância média (KM) | 5,95 |
| Duração média (min) | 89,19 |
| Ticket médio | $20,90 |
| % com gorjeta | 76,28% |

**Visuais:** Gráfico de linha (corridas por mês), gráfico de barras (receita por mês), barras horizontais (ticket médio por mês).

---

### 2. ⏱️ Análise Temporal
Exploração dos padrões de uso ao longo do tempo — por dia da semana e por horário.

- **Dia mais movimentado:** Quinta-feira (3.090.890 corridas)
- **Mapa de árvore (Treemap):** Distribuição de corridas por dia da semana com codificação de cores por intensidade
- **Duração média por horário:** Gráfico de área identificando picos às 15h–17h (~19,9 min)
- **Duração por dia da semana:** Quinta-feira lidera com média de 17,5 minutos por corrida

---

### 3. 👤 Perfil dos Usuários
Análise do comportamento e perfil dos passageiros.

- **Média de passageiros por corrida:** 1
- **% com gorjeta:** 76,28% | Valor médio: $2,69
- **Horário de melhor gorjeta:** 05:00 ($3,48) e 04:00 ($3,12)
- **Formas de pagamento:** 79,62% cartão de crédito / 20,38% dinheiro
- **Pico de uso:** 16h–17h (1,43 Mi corridas)
- **Horário de melhor receita:** 15h–19h (acima de $26 Mi por hora)

---

### 4. 🗺️ Análise Geográfica
Distribuição das corridas e receita por regiões de Nova York.

| Região | Corridas | Duração Média (min) |
|---|---|---|
| EWR (Aeroporto Newark) | 3,8 Mi | 21,43 |
| Brooklyn | 3,8 Mi | 14,79 |
| Manhattan | 3,2 Mi | 14,95 |
| Staten Island | 2,8 Mi | 17,87 |
| Bronx | 2,4 Mi | 15,33 |
| Queens | 1,7 Mi | 15,50 |

- **Custo total com pedágios:** $1.496.235,25
- **Valor médio do pedágio:** $0,50
- **Mapa de calor:** Regiões com maior intensidade de tráfego em Nova York

---

## 🛠️ Stack Técnica e Arquitetura

### Processamento de Dados
```
Base Pública NYC Taxi Trips
        │
        ▼
  Extração e limpeza
        │
        ▼
  Segmentação em arquivos .parquet
  (particionamento por mês/região)
        │
        ▼
  Importação no Power BI via
  conector Parquet / Power Query
        │
        ▼
  Modelagem estrela (Star Schema)
        │
        ▼
  Camada DAX (medidas e colunas calculadas)
        │
        ▼
  Dashboards Interativos
```

### Por que Parquet?
- ✅ Compressão colunar eficiente (até 90% menor que CSV)
- ✅ Leitura seletiva de colunas (melhor performance no Power BI)
- ✅ Suporte nativo a tipos de dados complexos
- ✅ Ideal para grandes volumes de dados analíticos

---

## 📐 Modelagem DAX — Principais Medidas

### KPIs Principais
```dax
-- Total de Corridas
Total de Corridas = COUNTROWS(fato_corridas)

-- Receita Total
Receita Total = SUM(fato_corridas[total_amount])

-- Ticket Médio
Ticket Médio = DIVIDE([Receita Total], [Total de Corridas])

-- % de Gorjetas
% Gorjetas = 
DIVIDE(
    COUNTROWS(FILTER(fato_corridas, fato_corridas[tip_amount] > 0)),
    [Total de Corridas]
)
```

### Análise Temporal
```dax
-- Duração Média por Horário
Duração Média (min) = 
AVERAGEX(
    fato_corridas,
    DATEDIFF(fato_corridas[tpep_pickup_datetime], 
             fato_corridas[tpep_dropoff_datetime], MINUTE)
)

-- Hora Formatada
Hora Formatada = FORMAT(fato_corridas[tpep_pickup_datetime], "HH:00")

-- Dia Mais Movimentado
Dia Mais Movimentado = 
CALCULATE(
    FIRSTNONBLANK(dim_calendario[dia_semana], 1),
    TOPN(1, VALUES(dim_calendario[dia_semana]), [Total de Corridas])
)
```

### Análise Geográfica
```dax
-- Custo Total com Pedágios
Custo Total Pedágios = SUM(fato_corridas[tolls_amount])

-- Valor Médio do Pedágio
Valor Médio Pedágio = 
DIVIDE(
    [Custo Total Pedágios],
    COUNTROWS(FILTER(fato_corridas, fato_corridas[tolls_amount] > 0))
)

-- Corridas por Região
Corridas por Região = 
CALCULATE([Total de Corridas], ALLEXCEPT(dim_zona, dim_zona[Borough]))
```

### Perfil do Usuário
```dax
-- Valor Médio de Gorjeta
Valor Médio Gorjeta = 
AVERAGEX(
    FILTER(fato_corridas, fato_corridas[tip_amount] > 0),
    fato_corridas[tip_amount]
)

-- Horário com Melhor Gorjeta
Melhor Gorjeta por Hora = 
CALCULATE(
    [Valor Médio Gorjeta],
    ALLEXCEPT(fato_corridas, fato_corridas[hora_pickup])
)
```

---

## 🗃️ Estrutura do Modelo de Dados

```
fato_corridas              dim_calendario
├── trip_id           ──►  ├── data
├── pickup_datetime         ├── mes
├── dropoff_datetime        ├── dia_semana
├── passenger_count         └── hora
├── trip_distance
├── total_amount       dim_zona
├── tip_amount    ──►  ├── LocationID
├── tolls_amount        ├── Borough (região)
├── payment_type        └── Zone
├── PULocationID
└── DOLocationID
```

---

## 📁 Estrutura de Arquivos

```
📦 nyc-taxi-powerbi/
 ┣ 📂 data/
 ┃ ┣ 📂 parquet/
 ┃ ┃ ┣ 📄 yellow_taxi_2024_01.parquet
 ┃ ┃ ┣ 📄 yellow_taxi_2024_02.parquet
 ┃ ┃ ┣ 📄 yellow_taxi_2024_03.parquet
 ┃ ┃ ┣ 📄 yellow_taxi_2024_04.parquet
 ┃ ┃ ┣ 📄 yellow_taxi_2024_05.parquet
 ┃ ┃ └── 📄 yellow_taxi_2024_06.parquet
 ┃ ┗ 📂 lookup/
 ┃   └── 📄 taxi_zone_lookup.csv
 ┣ 📂 dax/
 ┃ ┣ 📄 medidas_kpi.dax
 ┃ ┣ 📄 medidas_temporal.dax
 ┃ ┣ 📄 medidas_geograficas.dax
 ┃ └── 📄 medidas_perfil_usuario.dax
 ┣ 📂 screenshots/
 ┃ ┣ 📄 00_home.png
 ┃ ┣ 📄 01_visao_geral.png
 ┃ ┣ 📄 02_analise_temporal.png
 ┃ ┣ 📄 03_perfil_usuarios.png
 ┃ └── 📄 04_analise_geografica.png
 ┣ 📄 nyc_taxi_analysis.pbix
 ┗ 📄 README.md
```

---

## 🔍 Principais Insights Encontrados

1. **Crescimento consistente de receita:** A receita saltou de $47 Mi em janeiro para $79 Mi em junho — crescimento de **68%** no semestre.

2. **Quinta-feira domina em volume:** Com 3,09 Mi de corridas, quintas-feiras superam todos os outros dias, inclusive finais de semana.

3. **Pico de duração à tarde:** Entre 15h e 17h, a duração média das corridas atinge ~20 minutos, provavelmente reflexo do rush comercial de Nova York.

4. **Madrugada é mais lucrativa para gorjetas:** Corridas às 05:00 geram a maior gorjeta média ($3,48), possivelmente corridas ao aeroporto.

5. **Cartão de crédito é dominante:** 79,62% das corridas são pagas com cartão, impactando diretamente a previsibilidade de receita.

6. **EWR (Newark) tem maior duração:** Corridas com origem/destino no aeroporto de Newark têm duração média de 21,43 min — 44% acima da média de Manhattan.

---

## 🚀 Como Usar Este Projeto

### Pré-requisitos
- Power BI Desktop (versão 2.120+)
- Python 3.8+ (para scripts de extração Parquet, se necessário)
- Mínimo 8 GB de RAM recomendado

### Passos
1. Clone o repositório:
   ```bash
   git clone https://github.com/seu-usuario/nyc-taxi-powerbi.git
   ```
2. Abra o arquivo `nyc_taxi_analysis.pbix` no Power BI Desktop
3. Atualize o caminho dos arquivos Parquet nas configurações da fonte de dados
4. Clique em **Atualizar** para carregar os dados
5. Navegue pelos painéis pela tela inicial

---

## 📚 Fonte dos Dados

- **NYC TLC Trip Record Data** — [nyc.gov/tlc](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- Dados públicos disponibilizados pela prefeitura de Nova York
- Período analisado: Janeiro a Junho de 2024

---

## 👨‍💻 Autor

Desenvolvido como projeto de análise de dados com Power BI, DAX e engenharia de dados com Apache Parquet.

---

*Projeto desenvolvido para demonstração de habilidades em Business Intelligence, modelagem de dados e análise de grandes volumes de dados.*
