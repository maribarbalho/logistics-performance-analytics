# Modelagem de dados e fontes

## Abordagem

O relatório foi desenvolvido no **Power BI Desktop**, com ingestão e transformação de dados no **Power Query (M)**, modelagem e cálculos em **DAX**. O modelo integra dimensões com fatos operacionais de diferentes granularidades, preservando `f_entregas` como eixo principal das análises logísticas.

**Não se trata de um esquema estrela puro:** existem múltiplas tabelas fato e relações com caminhos potencialmente ambíguos. Em análises específicas, foram utilizadas medidas com `TREATAS` para controlar a propagação de filtros sem alterar relacionamentos existentes.

## Tabelas identificadas no modelo

| Papel | Tabela | Granularidade / finalidade |
|---|---|---|
| Fato central | `f_entregas` | Registro de entrega e seus atributos operacionais |
| Fato | `f_rastreamento` | Evento de movimentação associado a uma entrega |
| Fato | `f_abastecimentos` | Abastecimento realizado por veículo |
| Fato | `f_manutencoes` | Serviço/manutenção de veículo |
| Fato | `f_sac_chamados` | Chamado de atendimento (JSON inclui lista de interações) |
| Legado | `f_pedidos_legado` | Informações de pedidos legados |
| Dimensão | `d_calendario` | Datas e atributos calendáricos |
| Dimensão | `d_clientes` | Cadastro de clientes |
| Dimensão | `d_produtos` | Cadastro de produtos e categorias |
| Dimensão | `d_transportadora` | Parceiros logísticos e seus atributos |
| Dimensão | `d_veiculos` | Placa, tipo, capacidade e transportadora |
| Dimensão | `d_motoristas` | Cadastro de motoristas |
| Dimensão | `d_oficinas` | Cadastro de oficinas |
| Medidas | `medidas` | Organização das medidas DAX (as dos tooltips ficam na pasta de exibição **Tooltips**) |

As entidades refletem as tabelas utilizadas durante o desenvolvimento; nomes e tipos devem ser conferidos no arquivo PBIX caso a fonte evolua.

## Tipos de arquivos integrados

A prática de ETL incluiu arquivos **CSV, Excel, JSON e XML**. O principal exemplo de dado semiestruturado é `sac_chamados.json`, que possui uma lista `interacoes` dentro de cada chamado.

## Relacionamentos e filtros

- O `entrega_id` identifica a entrega e permite relacionar fatos de rastreamento/atendimento à operação.
- A `placa` relaciona os veículos aos registros de abastecimento e manutenção.
- IDs de produto, cliente, motorista e transportadora são chaves para análises dimensionais.
- O calendário depende da **data escolhida para cada análise**. Abertura de chamado, data de entrega, manutenção e abastecimento representam eventos distintos.
- Evitar usar filtro bidirecional indiscriminadamente: relações muitos-para-muitos e caminhos alternativos podem causar ambiguidade.
- Para despesas por transportadora, as medidas de análise aplicam os IDs da transportadora às placas e às tabelas de custos, por `TREATAS`.

### Exemplo conceitual de filtro virtual

```dax
Custo Abastecimento por Transportadora =
VAR TransportadorasSelecionadas =
    VALUES(d_transportadora[id_transportadora])
VAR PlacasTransportadora =
    CALCULATETABLE(
        VALUES(d_veiculos[placa]),
        TREATAS(
            TransportadorasSelecionadas,
            d_veiculos[transportadora_id]
        )
    )
RETURN
    CALCULATE(
        [Custo Total Abastecimento],
        KEEPFILTERS(
            TREATAS(PlacasTransportadora, f_abastecimentos[placa])
        )
    )
```

### Quais datas filtram quais fatos

| Fato | Coluna ligada a `d_calendario` | Observação |
|---|---|---|
| `f_entregas` | `data_entrega` | Entregas sem data de entrega (em trânsito, extraviadas e 40 entregues sem data) saem da análise quando há filtro de período |
| `f_sac_chamados` | `Data Abertura` | Coluna calculada com a data de abertura sem a hora; relacionamento criado para o filtro de período funcionar na página Experiência do Cliente |
| `f_abastecimentos`, `f_manutencoes` | — | Sem relação com o calendário; os custos não respondem ao filtro de período |

```dax
Data Abertura =
DATE (
    YEAR ( f_sac_chamados[abertura] ),
    MONTH ( f_sac_chamados[abertura] ),
    DAY ( f_sac_chamados[abertura] )
)
```

### Granularidade de `d_transportadora`

A dimensão é montada a partir de um XML com listas de contratos e regiões. A expansão gera **55 linhas para 8 transportadoras** (uma por região × contrato), por isso o relacionamento com `f_entregas` é **muitos-para-muitos e bidirecional**. A homologação confirmou que os totais por transportadora estão corretos, mas filtros por região ou contrato podem enganar. Os tooltips usam `CONCATENATEX` sobre valores distintos para exibir regiões, modal e SLA.

## Formato do projeto

O projeto foi salvo como **PBIP**, com relatório em **PBIR** (páginas e visuais em JSON) e modelo semântico em **TMDL** (tabelas e medidas em texto). Isso permite revisar diferenças de medidas em controle de versão e automatizar alterações repetitivas, como as 14 páginas de tooltip.

## Próximas melhorias de modelagem

1. Separar `f_sac_chamados` (uma linha por chamado) e `f_sac_interacoes` (uma por interação), relacionadas por `chamado_id`.
2. Normalizar `d_transportadora` em uma linha por transportadora, com tabelas auxiliares de regiões e contratos, e trocar o relacionamento por 1:*.
3. Avaliar papéis de data separados (pedido, saída, prevista, entrega) para análises que não dependam da data de entrega.
4. Desativar a opção **Data/hora automática**, que criou tabelas de datas ocultas desnecessárias, e marcar `d_calendario` como tabela de datas.
5. Revisar relações inativas, caminhos ambíguos e uso de `TREATAS` em medidas específicas.

> Os relacionamentos principais e as medidas foram verificados na [homologação dos dados](homologacao-dos-dados.md). Itens de modelagem ainda pendentes estão listados acima.
