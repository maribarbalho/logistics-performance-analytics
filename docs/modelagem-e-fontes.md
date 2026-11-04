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
| Medidas | `_medidas` | Organização das medidas DAX |

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

## Próximas melhorias de modelagem

1. Separar `f_sac_chamados` (uma linha por chamado) e `f_sac_interacoes` (uma por interação), relacionadas por `chamado_id`.
2. Revisar duplicidades e unicidade de chaves de dimensões antes de impor cardinalidade 1:*.
3. Documentar quais datas filtram quais fatos; avaliar papéis de data separados.
4. Revisar relações inativas, caminhos ambíguos e uso de `TREATAS` em medidas específicas.
5. Versionar o projeto Power BI, preferencialmente como **PBIP/PBIR**, conforme compatibilidade do ambiente.

> Este documento descreve a arquitetura do case, não garante que todos os relacionamentos tenham sido tecnicamente auditados no PBIX publicado.
