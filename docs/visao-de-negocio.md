# Contexto de negócio e storytelling

## Cenário

O **Logistics Performance Analytics** é um case de Business Intelligence criado para simular a gestão ponta a ponta de uma operação logística. O relatório combina registros de entregas, parceiros logísticos, frota, movimentação de produtos e atendimento ao cliente.

**Objetivo:** transformar dados operacionais em visões que apoiem decisões de pontualidade, qualidade, custos e experiência do cliente.

> Os dados, nomes, valores, trajetos e ocorrências são **fictícios**. As conclusões se referem exclusivamente à base simulada, não a uma empresa real.

## Perguntas de negócio

| Frente | Pergunta | Indicadores e visuais |
|---|---|---|
| Eficiência | Qual é o volume de entregas e sua evolução? | Total de entregas, série temporal |
| Pontualidade | Onde e com quais parceiros ocorrem atrasos? | Entregas no prazo, atrasadas, atraso médio, CD, UF e transportadora |
| Rastreamento | Quais eventos foram registrados para uma entrega? | Seleção por ID, histórico cronológico e pontos geográficos |
| Frota | Como a frota está distribuída e quanto custa operá-la? | Veículos, categorias, abastecimento e manutenção |
| Produtos | Quais produtos/categorias concentram movimento e dificuldades? | Unidades, entregas, frete por unidade, devoluções |
| Atendimento | O que motiva chamados e como a operação responde? | Motivos, canais, tempo de resolução, avaliações e NPS |

## Percurso analítico do relatório

1. **Home:** apresentação e navegação do case.
2. **Visão Executiva:** leitura consolidada dos indicadores de logística.
3. **Performance de Entregas:** investigação de atrasos, prazos e centros de distribuição.
4. **Rastreamento:** consulta de uma entrega individual e seus eventos registrados.
5. **Transportadoras & Frota:** comparação de parceiros, composição dos veículos e custos.
6. **Produtos:** análise do mix de produtos, volume, devoluções e frete unitário.
7. **Experiência do Cliente:** panorama dos chamados, resolução e satisfação dos clientes avaliados.

## Exemplos de decisões que o dashboard apoia

- Priorizar centros de distribuição e transportadoras com maiores concentrações de atrasos.
- Investigar categorias com taxas de devolução acima da média.
- Identificar categorias e parceiros que concentram maiores custos operacionais.
- Investigar motivos de contato associados a avaliações mais negativas.
- Avaliar canais com maior tempo médio até o encerramento dos chamados.

## Cuidados de interpretação

- **Correlação não implica causalidade.** Um motivo de contato com NPS mais baixo não comprova que ele causou a avaliação.
- **Frete não é receita de vendas.** O indicador de frete representa valores da operação simulada.
- **Entregas no prazo** devem usar apenas registros concluídos e avaliáveis quanto à data.
- **NPS de atendimento** usa somente chamados que receberam avaliação.
- **Pontos de rastreamento** são eventos históricos simulados, não localização GPS em tempo real.

Veja também: [Modelagem e fontes](modelagem-e-fontes.md) · [Indicadores e DAX](indicadores-e-dax.md) · [Qualidade e limitações](qualidade-e-limitacoes.md).
