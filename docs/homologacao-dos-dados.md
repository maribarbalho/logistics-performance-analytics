# Homologação dos dados

## Objetivo

Confirmar, antes da publicação, que **os números do dashboard estão corretos** e que as medidas novas dos tooltips seguem as mesmas regras de negócio das medidas originais. A homologação também levantou características da base que afetam a leitura dos indicadores.

## Método

1. O relatório foi aberto no **Power BI Desktop**, que executa localmente um motor Analysis Services com o modelo carregado.
2. Consultas **DAX** (`EVALUATE`, `SUMMARIZECOLUMNS`, `ROW`) foram executadas diretamente nesse motor, via biblioteca ADOMD, de forma **somente leitura**.
3. Os resultados foram comparados de três maneiras:
   - **Reconciliação:** a medida contra um cálculo independente sobre as linhas (ex.: `[Frete Total]` × `SUMX` linha a linha).
   - **Consistência:** partes somando o todo (participações = 100%, atrasos por UF = total de atrasos, entregas por transportadora = total).
   - **Recalculo independente:** regras dos tooltips refeitas fora do Power BI (ranking, "pior transportadora" por UF, valores do mês anterior).
4. Após os ajustes, as verificações principais foram executadas de novo como **teste de regressão**.

Exemplo de consulta usada:

```dax
EVALUATE
SUMMARIZECOLUMNS (
    d_transportadora[nome],
    "Total", [Total Entregas],
    "Pct", [% Entregas no Prazo],
    "Geral", [% No Prazo (Geral)],
    "Dif", [Dif. Pontualidade vs Geral (p.p.)],
    "Rank", [Ranking Pontualidade]
)
```

## Resultado

| Etapa | Verificações | Resultado |
|---|---:|---|
| Verificações cruzadas das medidas | 54 | ✅ todas aprovadas |
| Regressão após os ajustes | 23 | ✅ todas aprovadas |

### Indicadores do dashboard (sem filtros)

| Indicador | Valor | Conferência |
|---|---:|---|
| Total de entregas | 5.500 | = linhas de `f_entregas` = IDs distintos |
| Entregas no prazo | 2.708 | ✅ |
| Entregas atrasadas | 2.214 | ✅ |
| % no prazo | 55,0% | = 2.708 / (2.708 + 2.214) |
| Frete total | R$ 27.497.042,39 | = soma linha a linha |
| Atraso médio | 2,72 dias | = média da coluna `Dias de Atraso` |
| Custo de abastecimento | R$ 3.238.585,85 | = soma por transportadora (via `TREATAS`) |
| Custo de manutenção | R$ 3.601.235,81 | = soma por transportadora e por tipo de veículo |
| Veículos da frota | 45 | = soma da frota por transportadora e por tipo |
| Chamados de SAC | 900 | = soma por motivo e por canal |

### Medidas dos tooltips

- Participações (volume, atrasos, unidades, frota, chamados, status) somam **100%** em todas as dimensões.
- O **ranking** de transportadoras segue exatamente a ordem da pontualidade (RápidoSul 59,2% em 1º; TransBrasil 52,6% em 8º).
- A **pior transportadora por UF** confere com o recálculo independente nas 13 UFs.
- Os valores de **mês anterior** conferem com o mês real anterior em toda a série (jan/2025 a jul/2026), tanto no contexto `AnoMes` quanto em `Ano` + `MesAbrev` (usado no gráfico combinado).
- O relacionamento novo **SAC → calendário** funciona: 596 chamados em 2025 tanto pelo calendário quanto pela data de abertura; nenhum chamado fica fora do calendário.

### NPS e tempo de resolução do SAC

A tabela `f_sac_chamados` tem uma linha por **interação** (1.687 linhas para 900 chamados). Havia a suspeita de que medidas calculadas linha a linha dariam peso extra a chamados com mais interações. A homologação mostrou que **não há distorção**: a nota e o fechamento ficam em uma única linha por chamado, então o cálculo atual e o cálculo por chamado dão o mesmo resultado (NPS **−46,4** e tempo médio de **78,5 h** nos dois). Mesmo assim, separar chamados e interações continua recomendado para robustez.

## Achados na base de origem

Nenhum dos itens abaixo é erro de DAX; são características dos dados que mudam a leitura dos indicadores.

| # | Achado | Impacto |
|---:|---|---|
| 1 | **Rodonorte** está cadastrada como inativa (`ativo = FALSE`), mas tem **723 entregas**, o maior volume | O cartão mostra 7 transportadoras ativas enquanto 8 operam |
| 2 | **Julho/2026 é parcial**: dados até 10/07/2026 (último pedido em 30/06) | O mês aparece com 35 entregas e queda de 87,6%; tratado nos tooltips como mês parcial |
| 3 | **40 entregas "ENTREGUE" sem data de entrega** | Ficam fora da pontualidade (4.962 entregues, 4.922 avaliáveis) |
| 4 | **244 entregas sem data de entrega** (146 em trânsito, 58 extraviadas, 40 do item 3) | O calendário se liga à data de entrega: com qualquer filtro de período, o total cai de 5.500 para 5.256 |
| 5 | **12 entregas com data de entrega anterior à saída** (ex.: ENT003856, saída 21/06/2025, entrega 19/06/2025) | Ficam fora do tempo médio, mas contam na pontualidade |
| 6 | **60 entregas com motorista sem cadastro** em `d_motoristas` | Análises por motorista ficam incompletas |
| 7 | **`d_transportadora` tem 55 linhas para 8 transportadoras** (uma por região × contrato) | Relação muitos-para-muitos bidirecional; números corretos hoje, mas filtros por região ou contrato podem enganar |
| 8 | Textos sem acento na origem (`Cobranca`, `Endereco`) | Apenas visual |

Verificações sem ocorrência: nenhuma entrega com transportadora, produto, veículo ou cliente sem cadastro; nenhum frete ≤ 0; nenhuma quantidade ≤ 0; nenhuma saída antes do pedido; nenhuma nota de NPS fora de 0 a 10; nenhum chamado fechado antes da abertura; nenhum evento de rastreamento sem entrega.

## Ajustes aplicados após a homologação

- Tooltip de **Mês**: título "(parcial)", comparações ocultas e aviso *"◐ Mês parcial"* quando a base termina no meio do mês.
- Tooltip de **UF**: concordância "1 cidade atendida" / "N cidades atendidas".
- Título do mês tratado para entregas sem data.

## Pendências (dependem de decisão de negócio)

- Corrigir o status da Rodonorte ou documentar a regra de "ativa".
- Decidir se julho/2026 deve sair do gráfico Performance Mensal.
- Definir tratamento para entregas sem data e com datas incoerentes.
- Completar o cadastro de motoristas.
- Normalizar `d_transportadora` (uma linha por transportadora + tabela de regiões/contratos).

Veja também: [Qualidade e limitações](qualidade-e-limitacoes.md) · [Tooltips](tooltips.md).
