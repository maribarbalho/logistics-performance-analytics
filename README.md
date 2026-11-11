# Logistics Performance Analytics

### Business Intelligence • Power BI • Logística e Experiência do Cliente

**Um case de Business Intelligence que integra dados operacionais para analisar entregas, pontualidade, custos de frota, movimentação de produtos e atendimento ao cliente.**

[**▶ Acessar dashboard interativo no Power BI**](https://app.powerbi.com/view?r=eyJrIjoiNDI4MjMzNGQtMTk0Mi00NjM4LWE2NzktNjdlYzI3NzYxMjMyIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)

> **Projeto educacional e de portfólio.** Todos os dados, nomes de empresas/clientes, operações e valores são fictícios. Os indicadores abaixo descrevem exclusivamente o cenário simulado e correspondem a referências sem filtros da etapa de desenvolvimento.

---

## O desafio

Em uma operação logística, informações sobre entregas, parceiros, veículos, custos, produtos e reclamações podem ficar distribuídas em múltiplas fontes. O desafio deste case foi organizar os dados, desenvolver indicadores consistentes e construir uma experiência de análise que apoiasse a identificação de gargalos operacionais.

O painel foi estruturado para responder questões como:

- Onde estão os principais atrasos e quais transportadoras merecem atenção?
- Como se distribuem os custos de abastecimento e manutenção da frota?
- Quais produtos e categorias concentram mais movimentações ou devoluções?
- Quais problemas levam os clientes a contatar o atendimento e como isso se relaciona com as avaliações recebidas?

## O que foi desenvolvido

| Página | Análises principais |
|---|---|
| **Home** | Apresentação e navegação do relatório |
| **Visão Executiva** | KPIs gerais, evolução das entregas, pontualidade e geografia |
| **Performance de Entregas** | Atrasos, prazos, centros de distribuição e transportadoras |
| **Rastreamento** | Seleção de entrega, eventos históricos, pontos geográficos e detalhes |
| **Transportadoras & Frota** | Parceiros, frota por categoria, manutenção e abastecimento |
| **Produtos** | Movimentações, categorias, frete unitário e devoluções |
| **Experiência do Cliente** | SAC, motivos de contato, canais, tempo de encerramento e NPS |

O relatório tem **tema claro e escuro**, navegação personalizada e **tooltips de detalhamento** nos principais gráficos.

## Novidades da versão atual

- **7 tooltips de detalhamento** (transportadora, UF, mês, CD, produto/categoria, tipo de veículo e SAC), em tema claro e escuro, conectados a 38 visuais. Cada um mostra indicadores complementares, comparação com a média, minigráfico de tendência e uma frase de insight colorida por regra de negócio. → [Tooltips](docs/tooltips.md)
- **Homologação dos dados com consultas DAX** executadas no modelo: 54 verificações cruzadas e 23 de regressão, todas aprovadas, além de 8 achados documentados sobre a base de origem. → [Homologação](docs/homologacao-dos-dados.md)
- **Relacionamento do SAC com o calendário**, para que o filtro de período passe a filtrar os chamados.
- **Tratamento de mês parcial**: julho/2026 (dados até 10/07) é sinalizado e não gera comparações enganosas.
- **Projeto convertido para PBIP/PBIR**, com medidas e páginas em arquivos de texto versionáveis.

## Indicadores de referência

| KPI | Valor na base simulada |
|---|---:|
| Entregas únicas | **5.500** |
| Entregas pontuais entre concluídas avaliáveis | **55,0%** |
| Frete total | **R$ 27,50 mi** |
| Veículos cadastrados | **45** |
| Produtos cadastrados | **80** |
| Chamados de atendimento | **900** |
| Chamados em aberto | **113** |
| Taxa de avaliação de atendimento | **49,1%** |
| NPS dos atendimentos avaliados | **−46,4** |

**Nota metodológica:** a pontualidade é calculada sobre 4.922 entregas concluídas e avaliáveis, não sobre todas as 5.500. O NPS utiliza apenas chamados que possuem nota. Valores e filtros podem variar no relatório publicado.

## Stack e competências

- **Power BI Desktop / Power BI Service:** modelagem, visualização e publicação do relatório.
- **Power Query (M):** ETL, tipagem, deduplicação, padronização e tratamento de dados de diferentes formatos.
- **DAX:** indicadores operacionais, médias, taxas, contagens distintas e aplicação de contexto de filtro.
- **Modelagem dimensional:** integração de fatos e dimensões, com atenção a relacionamentos e granularidade.
- **Análise geográfica:** visualização e validação inicial de coordenadas de eventos.
- **Data storytelling:** transformação dos indicadores em perguntas e perspectivas de negócio.
- **UX/UI em BI:** tooltips de página com layout padronizado, hierarquia visual e formatação condicional por medida.
- **Testes e homologação:** consultas DAX contra o modelo para reconciliar medidas e testar regressões.
- **PBIP/PBIR e TMDL:** projeto Power BI em formato de texto, editável e versionável.

**Fontes de dados utilizadas no case:** CSV, Excel, JSON e XML.

## Principais desafios técnicos

**1. Deduplicação e qualidade dos fretes.** A base de entregas incluía registros duplicados e diferenças de representação decimal. O tratamento no Power Query permitiu obter 5.500 entregas únicas e corrigir a escala dos valores monetários.

**2. Geolocalização do rastreamento.** As coordenadas precisaram ser convertidas corretamente para números decimais. Pontos geograficamente suspeitos foram sinalizados, preservando o dado original.

**3. Duplicidade gerada por JSON aninhado.** A expansão das interações do SAC repetiu linhas dos chamados e inflou uma contagem inicial de avaliações. A medida foi corrigida para contar **IDs distintos com avaliação não nula**, resultando em 49,1%.

**4. Propagação de filtros.** Para analisar custos por transportadora, algumas medidas utilizam `TREATAS` para aplicar o contexto de parceiros às placas e registros de abastecimento/manutenção, sem criar novos caminhos físicos ambíguos.

**5. Comparações dentro do tooltip.** Cada tooltip recebe o filtro do item sob o mouse e os filtros da página. Medidas com `REMOVEFILTERS` comparam o item com a média, `RANKX` calcula o ranking e `DATEADD` com `REMOVEFILTERS ( d_calendario )` traz o mês anterior mesmo quando o gráfico filtra `Ano` e `Mês`.

**6. Homologação contra o modelo.** As medidas foram validadas com consultas DAX no motor local do Power BI Desktop, comparando cada resultado com um cálculo independente. A homologação revelou, entre outros pontos, um mês parcial (julho/2026) e uma transportadora inativa com o maior volume de entregas.
## Documentação técnica

| Documento | Conteúdo |
|---|---|
| [Contexto de negócio e storytelling](docs/visao-de-negocio.md) | Objetivos, perguntas analíticas e decisões apoiadas |
| [Modelagem e fontes](docs/modelagem-e-fontes.md) | Tabelas, granularidade, integração e relacionamentos |
| [Dicionário de indicadores e DAX](docs/indicadores-e-dax.md) | KPIs, premissas, denominadores e fórmulas |
| [Qualidade e limitações](docs/qualidade-e-limitacoes.md) | Problemas identificados, tratamentos e pontos a auditar |
| [Reprodução e publicação](docs/reproducao-e-publicacao.md) | Etapas de ETL, modelagem, validação e publicação |
| [Tooltips de detalhamento](docs/tooltips.md) | Layout, catálogo, regras dos insights e padrões DAX |
| [Homologação dos dados](docs/homologacao-dos-dados.md) | Método, verificações, achados da base e ajustes |
## Próximas evoluções

- Adicionar capturas de tela das páginas, dos tooltips e do modelo de dados.
- Versionar no repositório o projeto em PBIP/PBIR, sem cache e configurações locais.
- Tratar os achados da homologação: status da Rodonorte, mês parcial no gráfico principal, entregas sem data e motoristas sem cadastro.
- Normalizar `d_transportadora` (uma linha por transportadora) e separar as interações do SAC em uma tabela própria.
- Aprofundar a auditoria dos pontos geográficos e da medida de peso transportado.

## Autoria

**Mariana Barbalho** — Data Analytics

- [Portfólio](https://maribarbalho.github.io/)
- [Relatório Power BI](https://app.powerbi.com/view?r=eyJrIjoiNDI4MjMzNGQtMTk0Mi00NjM4LWE2NzktNjdlYzI3NzYxMjMyIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)

---

*Case construído com dados fictícios para estudo e demonstração de competências em Business Intelligence e Análise de Dados.*
