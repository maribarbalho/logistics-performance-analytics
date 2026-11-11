# Tooltips de detalhamento

## Objetivo

Os tooltips personalizados respondem à pergunta que o gráfico sozinho não responde: **"esse valor é bom ou ruim, comparado com o quê?"**. Ao passar o mouse sobre uma barra, um estado ou um mês, o usuário vê o contexto, uma comparação com a média e uma frase de leitura do resultado, sem precisar trocar de página.

Critério adotado: **tooltip personalizado só onde agrega informação**. Gráficos autoexplicativos ficaram com o tooltip padrão (às vezes enriquecido com um campo) e tabelas detalhadas ficaram sem tooltip, porque elas já são o detalhe.

## Estrutura visual (padrão único)

Todas as páginas de tooltip seguem o mesmo layout de **360 × 340 px**, em versão clara e escura:

```text
┌──────────────────────────────────────────┐
│        RÓTULO DA DIMENSÃO (eyebrow)      │  faixa de destaque + degradê
│            Título dinâmico               │  ex.: "São Paulo (SP)"
│     linha de contexto (CD, SLA, ...)     │
├────────────┬─────────────┬───────────────┤
│  KPI 1     │  KPI 2      │  KPI 3        │  valor principal + legenda
│ ────────── │ ─────────── │ ───────────── │
│ comparação │ comparação  │ comparação    │  vs. média, participação, ranking
├──────────────────────────────────────────┤
│  minigráfico (tendência mensal ou barras)│
├──────────────────────────────────────────┤
│ ▌ insight colorido (▲ bom · ● neutro · ▼)│
└──────────────────────────────────────────┘
```

| Elemento | Tema claro | Tema escuro |
|---|---|---|
| Fundo | `#FFFFFF` | `#131C30` |
| Blocos | `#F8FAFD` | `#182338` |
| Texto principal | `#0F1B33` | `#E2E8F0` |
| Texto secundário | `#5F6B80` | `#94A3B8` |
| Destaque | `#2563EB` | `#3B82F6` |
| Positivo / atenção / negativo | `#15803D` / `#D97706` / `#DC2626` | `#4ADE80` / `#FBBF24` / `#F87171` |

O fundo (cabeçalho, blocos arredondados e faixa do insight) é uma **imagem PNG em resolução 3x** usada como plano de fundo da página, a mesma técnica das páginas principais do relatório. Os cartões ficam transparentes por cima, alinhados aos blocos.

## Catálogo

| Tooltip | Visuais de origem | Indicadores | Comparações | Minigráfico |
|---|---|---|---|---|
| **Transportadora** | Pontualidade por Transportadora · Atraso Médio por Transportadora · Volume e Pontualidade por Transportadora · Custo de Abastecimento por Transportadora | % no prazo · entregas · atraso médio | p.p. vs. média da operação · % do volume · posição no ranking | Pontualidade mensal vs. média |
| **UF** | Atrasos por UF (mapa) | % atrasadas · entregas atrasadas · atraso médio | p.p. vs. Brasil · % dos atrasos do país · pior transportadora na UF | % de atrasos mensal vs. Brasil |
| **Mês** | Performance Mensal de Entregas · Evolução do Tempo Médio de Entrega | entregas concluídas · % no prazo · tempo médio | variação vs. mês anterior (volume, p.p. e dias) | Pontualidade por transportadora no mês |
| **CD** | Pontualidade por Centro de Distribuição · % de Entregas Atrasadas por CD e UF | entregas · % no prazo · atraso médio | % do volume · p.p. vs. média dos CDs · UF com mais atrasos | Pontualidade mensal vs. média dos CDs |
| **Produto / Categoria** | Top 10 Produtos · Volume por Categoria · Taxa de Devolução por Categoria · Frete por Unidade por Categoria | unidades · % devoluções · frete por unidade | % das unidades · p.p. vs. média · frete vs. média | Unidades movimentadas por mês |
| **Tipo de Veículo** | Frota por Tipo de Veículo · Custo de Manutenção por Tipo de Veículo | veículos · manutenção por veículo · entregas por veículo | % da frota · manutenção vs. média · abastecimento por veículo | Veículos do tipo por transportadora |
| **SAC** | Principais Motivos de Contato · NPS por Motivo · Tempo Médio de Resolução por Canal · Evolução Mensal de Chamados | chamados · NPS · tempo de resolução | % dos chamados · pontos de NPS vs. média · % resolvidos | Chamados por prioridade |

Ao todo: **7 tooltips × 2 temas = 14 páginas ocultas**, conectadas a **38 visuais**. O gráfico **Status das Entregas** usa o tooltip padrão com o campo extra "% do total".

Sem tooltip personalizado, por decisão de design: mapa e tabelas da página **Rastreamento** (dados de uma única entrega, sem base de comparação) e tabelas detalhadas (**Comparativo de Transportadoras**, **Detalhamento dos Chamados**).

## Regras dos insights

Cada tooltip termina com uma frase gerada por DAX e colorida por uma medida de cor (formatação condicional por *valor do campo*). Há uma faixa neutra para evitar alarmes por variações pequenas:

| Tooltip | Métrica | Faixa neutra | Direção boa |
|---|---|---|---|
| Transportadora | p.p. vs. média da operação | ± 2 p.p. | maior pontualidade |
| UF | p.p. de atraso vs. Brasil | ± 2 p.p. | menor atraso |
| Mês | p.p. vs. mês anterior | ± 2 p.p. | maior pontualidade |
| CD | p.p. vs. média dos CDs | ± 2 p.p. | maior pontualidade |
| Produto | p.p. de devolução vs. média | ± 1 p.p. | menor devolução |
| Tipo de veículo | manutenção por veículo vs. frota | ± 10% | menor custo |
| SAC | pontos de NPS vs. média | ± 5 pts | maior NPS |

Mês incompleto: quando a base termina no meio do mês (julho/2026 vai até 10/07/2026), o tooltip mostra "**(parcial)**" no título, oculta as comparações com o mês anterior e exibe *"◐ Mês parcial: comparação não conclusiva"*. A regra é automática e deixa de valer quando a base for atualizada.

## Padrões DAX utilizados

**1. Comparação com a média, mantendo os demais filtros.** O tooltip recebe o filtro do item sob o mouse (ex.: uma transportadora) e os filtros da página. `REMOVEFILTERS` retira só a dimensão comparada:

```dax
% No Prazo (Geral) =
CALCULATE (
    [% Entregas no Prazo],
    REMOVEFILTERS ( d_transportadora )
)

Dif. Pontualidade vs Geral (p.p.) =
VAR EntregasAvaliadas = [Entregas no Prazo] + [Entregas Atrasadas]
RETURN
    IF ( EntregasAvaliadas > 0, ( [% Entregas no Prazo] - [% No Prazo (Geral)] ) * 100 )
```

**2. Ranking com transição de contexto.** `RANKX` recalcula a pontualidade de cada transportadora e ignora as que não têm entregas avaliadas no filtro atual:

```dax
Ranking Pontualidade =
VAR TransportadorasAvaliadas =
    FILTER (
        ALL ( d_transportadora[nome] ),
        [Entregas no Prazo] + [Entregas Atrasadas] > 0
    )
VAR Posicao =
    RANKX ( TransportadorasAvaliadas, [% Entregas no Prazo], , DESC, DENSE )
RETURN
    IF (
        HASONEVALUE ( d_transportadora[nome] )
            && [Entregas no Prazo] + [Entregas Atrasadas] > 0,
        Posicao
    )
```

**3. Mês anterior sem marcar tabela de datas.** O gráfico filtra `Ano` e `MesAbrev`; `REMOVEFILTERS ( d_calendario )` retira esses filtros e `DATEADD` aplica as datas do mês anterior:

```dax
Entregas Concluídas (Mês Anterior) =
CALCULATE (
    [Entregas Concluídas],
    DATEADD ( d_calendario[Data], -1, MONTH ),
    REMOVEFILTERS ( d_calendario )
)
```

**4. "Pior item" dentro do contexto, com amostra mínima.** Evita eleger como pior uma transportadora com 1 ou 2 entregas na UF:

```dax
TT UF Pior Transportadora =
VAR Ranking =
    FILTER (
        ADDCOLUMNS (
            VALUES ( d_transportadora[nome] ),
            "@Avaliadas", [Entregas no Prazo] + [Entregas Atrasadas],
            "@PctAtraso", [% Entregas Atrasadas]
        ),
        [@Avaliadas] >= 5
    )
VAR Pior = TOPN ( 1, Ranking, [@PctAtraso], DESC, d_transportadora[nome], ASC )
RETURN
    IF (
        COUNTROWS ( Ranking ) > 0,
        "Pior: " & MAXX ( Pior, d_transportadora[nome] ) & " ("
            & FORMAT ( MAXX ( Pior, [@PctAtraso] ), "0%" ) & ")"
    )
```

**5. Mês parcial.**

```dax
Última Data com Entregas =
CALCULATE ( MAX ( f_entregas[data_entrega] ), ALL ( f_entregas ) )

Mês Parcial =
VAR InicioMes = MIN ( d_calendario[Data] )
VAR UltimaData = [Última Data com Entregas]
RETURN
    HASONEVALUE ( d_calendario[AnoMes] )
        && NOT ISBLANK ( InicioMes )
        && UltimaData >= InicioMes
        && UltimaData < EOMONTH ( InicioMes, 0 )
```

Todas as medidas dos tooltips (82) estão na pasta de exibição **Tooltips** da tabela `medidas`, com comentários explicando a lógica.

## Boas práticas aplicadas

- **Altura mínima dos cartões:** o cartão clássico centraliza o texto e corta conteúdo em caixas baixas. Regra usada: altura ≥ 2,5 × o tamanho da fonte em pixels, com espaçamento interno zerado.
- **Contagens antes de percentuais:** toda comparação percentual é feita sobre somas de contagens, nunca sobre médias de percentuais.
- **Desempenho:** cada tooltip dispara consultas a cada passagem do mouse; por isso foram mantidos 10 visuais por página e nenhum tooltip em tabelas grandes.
- **Construção reproduzível:** com o projeto salvo em PBIP/PBIR, páginas, visuais e medidas são arquivos de texto. Os tooltips foram gerados por script a partir de uma especificação única, o que garante o mesmo layout nos 14 painéis e facilita ajustes futuros.

Veja também: [Homologação dos dados](homologacao-dos-dados.md) · [Indicadores e DAX](indicadores-e-dax.md).
