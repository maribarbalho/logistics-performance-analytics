# Dicionário de indicadores e exemplos DAX

## Regras de leitura

- Os valores abaixo são **referências sem filtros** observadas na etapa de construção/validação do projeto. Podem variar após seleção de período, produto ou transportadora.
- Nome de medida, formato e condição de elegibilidade importam tanto quanto o valor.
- Os exemplos de DAX são **referências didáticas**; conferir correspondência de nomes de campos e contexto de filtros no PBIX.
- Os valores das tabelas abaixo foram **conferidos por consultas DAX no modelo** na [homologação dos dados](homologacao-dos-dados.md).

## Operação logística

| Indicador | Referência | Regra de negócio |
|---|---:|---|
| Total de entregas | 5.500 | Entregas únicas, após remoção de duplicatas exatas |
| Entregas no prazo | 2.708 | Entregas concluídas e elegíveis, dentro da data prevista |
| Entregas atrasadas | 2.214 | Entregas concluídas e elegíveis, após a data prevista |
| Pontualidade | 55,0% | 2.708 / (2.708 + 2.214) |
| Atraso médio | ~2,72 dias | Média entre entregas consideradas atrasadas |
| Frete total | ~R$ 27,50 mi | Soma do valor de frete na base de entregas |

**Denominador da pontualidade:** 4.922 entregas concluídas avaliáveis. A base possui 4.962 entregas com status entregue, mas 40 carecem de informação suficiente para comparar os prazos. Portanto 55,0% **não é** a porcentagem de todas as 5.500 entregas.

## Frota, custos e produtos

| Indicador | Referência no desenvolvimento | Observação |
|---|---:|---|
| Transportadoras ativas | 7 | Cadastro; gráficos podem incluir registros de parceiros inativos |
| Veículos cadastrados | 45 | Cadastro atual, não série histórica |
| Custo de manutenção | ~R$ 3,60 mi | Soma de serviços de manutenção |
| Custo de abastecimento | ~R$ 3,24 mi | Litros × valor por litro em cada abastecimento |
| Produtos cadastrados | 80 | Produtos distintos |
| Unidades movimentadas | 141.240 | Soma de quantidades nas entregas |
| Taxa de devolução | 6,07% | Percentual conforme critério de devolução usado na base |

```dax
Custo Total Abastecimento =
SUMX(
    f_abastecimentos,
    f_abastecimentos[litros] * f_abastecimentos[valor_litro]
)

Frete por Unidade =
DIVIDE(
    [Frete Total],
    [Unidades Movimentadas]
)
```

**Atenção:** o indicador de peso transportado apresentou um valor visualmente suspeito durante o desenvolvimento e requer validação da unidade, granularidade e soma. Não foi incluído entre os valores auditados deste documento.

## Experiência do cliente

| Indicador | Referência | Regra |
|---|---:|---|
| Total de chamados | 900 | `DISTINCTCOUNT(chamado_id)` |
| Chamados em aberto | 113 | `fechamento` vazio |
| Taxa de encerramento | 87,4% | 787 / 900 |
| Tempo médio até encerramento | 78,5 h | Apenas chamados com fechamento |
| Chamados avaliados | 442 | Nota NPS não nula, por ID distinto |
| Taxa de avaliação | 49,1% | 442 / 900 |
| NPS de atendimento | −46,4 | (promotores − detratores) / avaliados × 100 |

**Nota terminológica:** o campo de fechamento demonstra **encerramento**, mas não garante solução efetiva do problema. Prefira “taxa de encerramento” ao interpretar esse indicador.

### Medidas resistentes à duplicação por interações

```dax
Total Chamados =
DISTINCTCOUNT(f_sac_chamados[chamado_id])

Chamados Avaliados =
CALCULATE(
    DISTINCTCOUNT(f_sac_chamados[chamado_id]),
    NOT ISBLANK(f_sac_chamados[avaliacao_nps])
)

Taxa de Avaliação =
DIVIDE([Chamados Avaliados], [Total Chamados])
```

### NPS: regra de classificação

- **Promotores:** notas 9 e 10.
- **Neutros:** notas 7 e 8.
- **Detratores:** notas de 0 a 6.
- **Sem avaliação:** valor nulo, excluído do denominador do NPS.
- A nota **0 é válida** e integra o grupo de detratores.

```dax
NPS Atendimento (IDs distintos) =
VAR Promotores =
    CALCULATE(
        DISTINCTCOUNT(f_sac_chamados[chamado_id]),
        FILTER(
            f_sac_chamados,
            NOT ISBLANK(f_sac_chamados[avaliacao_nps])
                && f_sac_chamados[avaliacao_nps] >= 9
        )
    )
VAR Detratores =
    CALCULATE(
        DISTINCTCOUNT(f_sac_chamados[chamado_id]),
        FILTER(
            f_sac_chamados,
            NOT ISBLANK(f_sac_chamados[avaliacao_nps])
                && f_sac_chamados[avaliacao_nps] <= 6
        )
    )
RETURN
    DIVIDE(Promotores - Detratores, [Chamados Avaliados]) * 100
```

**Nota da homologação:** a medida `NPS Atendimento` publicada conta linhas avaliadas (`COUNTROWS`), não IDs distintos. Como a nota fica registrada em uma única linha por chamado, as duas versões produzem o mesmo resultado (**−46,4**). A versão por IDs distintos acima é a mais robusta caso a estrutura de interações mude.

## Indicadores comparativos dos tooltips

As medidas da pasta **Tooltips** reutilizam as medidas acima e acrescentam comparações:

| Medida | Regra |
|---|---|
| `% No Prazo (Geral)` | Pontualidade sem o filtro de transportadora (mantém período, UF etc.) |
| `Dif. Pontualidade vs Geral (p.p.)` | (% da transportadora − % geral) × 100 |
| `Ranking Pontualidade` | Posição entre transportadoras com entregas avaliadas |
| `% Atrasadas (Nacional)` | % de atraso sem filtros geográficos |
| `Entregas Concluídas (Mês Anterior)` | `DATEADD` de −1 mês, removendo os filtros de calendário |
| `Mês Parcial` | Verdadeiro quando a base termina antes do fim do mês analisado |
| `Custo Manutenção por Veículo` | Custo de manutenção ÷ veículos da frota |
| `Dif. NPS vs Geral` | NPS do motivo/canal − NPS geral do atendimento |
| `% do Total de Entregas` | Participação de cada status no total |

Detalhes, regras de cor e código: [Tooltips de detalhamento](tooltips.md).

## Boas práticas para comparação

1. Compare taxa e volume lado a lado (uma taxa elevada em pequena amostra é instável).
2. Valide que totais por categoria/transportadora correspondam aos cartões sob os mesmos filtros.
3. Trate indicadores de cadastro (veículos ativos) separadamente de indicadores históricos (entregas mensais).
4. Confirme a data utilizada para segmentar cada fato.
4. Some contagens antes de dividir; nunca faça média de percentuais.
4. Sinalize meses incompletos antes de compará-los com meses fechados.
5. Evite usar o nome “NPS dos clientes” para avaliações coletadas apenas dos atendimentos.
