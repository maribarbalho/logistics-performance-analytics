# Qualidade dos dados, validações e limitações

## Princípios

Este case usa **dados sintéticos** para fins educacionais e de portfólio. Erros de qualidade foram tratados ou sinalizados explicitamente; posições e ocorrências simuladas não devem ser confundidas com dados de operações reais.

## Entregas e frete

**Problema:** arquivos de entrega possuíam 5.800 linhas, sendo 300 duplicatas exatas. Após tratamento, a análise passou a contar **5.500 entregas únicas**.

**Problema:** diferentes separadores decimais em fretes produziram somatórios incompatíveis com a escala esperada.

**Tratamento:** normalização dos números no Power Query, conversão com cultura explícita e validação dos totais contra registros amostrais. Referência do painel sem filtros: aproximadamente **R$ 27,50 milhões em fretes**.

## Rastreamento e coordenadas

**Problema:** latitude e longitude foram inicialmente importadas como inteiros, distorcendo a posição geográfica.

**Tratamento:** conversão para número decimal, mantendo o sinal e a precisão. Em auditoria posterior, foram sinalizados registros geograficamente suspeitos e possíveis inconsistências da ordem dos eventos.

**Estratégia:** conservar `latitude`/`longitude` originais, manter indicadores de qualidade e usar `latitude_mapa`/`longitude_mapa` somente para pontos aprovados na verificação inicial.

**Limitações:** o contorno simplificado usado na checagem não comprova se cada ponto fica sobre uma estrada; posições litorâneas e áreas fronteiriças exigem validação adicional. O mapa não representa GPS ao vivo nem rotas oficiais. Alguns eventos sintéticos aparecem em sequência operacional pouco plausível.

## SAC e avaliações

**Problema:** o JSON contém uma lista de interações em cada chamado. Expandir essa lista diretamente em `f_sac_chamados` duplica linhas de certos chamados. Por isso, `COUNT(avaliacao_nps)` produziu incorretamente **98,2%** de avaliações.

**Tratamento:** contagem de IDs distintos com avaliação não nula, resultando em **442 de 900 chamados (49,1%)**. A nota **0** é válida e não pode ser confundida com avaliação ausente.

**Melhoria recomendada:** separar cabeçalhos de chamados das interações em duas tabelas distintas.

## Achados da homologação

A [homologação dos dados](homologacao-dos-dados.md) confirmou os indicadores e identificou características da base que afetam a leitura:

- **Rodonorte** está cadastrada como inativa, mas tem 723 entregas (o maior volume).
- **Julho/2026 é parcial** (dados até 10/07/2026); os tooltips sinalizam o mês e não fazem comparações com ele.
- **244 entregas sem data de entrega** saem da análise quando há filtro de período (o total cai de 5.500 para 5.256).
- **40 entregas "ENTREGUE" sem data de entrega** ficam fora da pontualidade.
- **12 entregas com data de entrega anterior à saída** ficam fora do tempo médio, mas contam na pontualidade.
- **60 entregas com motorista sem cadastro.**
- **`d_transportadora` com 55 linhas para 8 transportadoras**, gerando relação muitos-para-muitos.

## Granularidade e significado dos indicadores

- O denominador de pontualidade inclui somente entregas concluídas e com informações suficientes para comparar datas.
- “Taxa de resolução” derivada da presença de fechamento deve ser interpretada como **taxa de encerramento**, e não comprova satisfação nem solução.
- O NPS está restrito aos **atendimentos avaliados**, não à população inteira de clientes.
- Quantidades de frota e transportadoras ativas descrevem o cadastro no conjunto carregado, não necessariamente a situação histórica.
- Custos por parceiro dependem do vínculo de placa com a transportadora.
- A medida de **peso transportado** merece auditoria adicional de unidades e granularidade antes de ser usada como resultado validado.

## Checklist de publicação e revisão

- [x] Aplicar deduplicação de entregas.
- [x] Corrigir importação decimal de frete e geolocalização.
- [x] Corrigir taxa de avaliação do SAC utilizando chamados distintos.
- [x] Publicar relatório interativo no Power BI.
- [x] Homologar indicadores e medidas com consultas DAX (54 verificações + 23 de regressão).
- [x] Relacionar os chamados do SAC ao calendário.
- [x] Sinalizar mês parcial nos tooltips.
- [x] Converter o projeto para PBIP/PBIR.
- [ ] Conferir 100% das coordenadas e trajetórias em fonte geográfica apropriada.
- [ ] Auditar a medida de peso transportado.
- [ ] Revisar dimensões com IDs duplicados e caminhos ambíguos no modelo (`d_transportadora`).
- [ ] Separar interações de SAC em fato próprio.
- [ ] Decidir o tratamento da Rodonorte, do mês parcial no gráfico principal, das entregas sem data e dos motoristas sem cadastro.
- [ ] Adicionar capturas de tela e o projeto PBIP ao repositório.

## Privacidade e publicação

Todos os nomes, clientes, placas, mensagens, avaliações e demais dados do case são fictícios. Ao reaproveitar a estrutura em ambiente real, remover dados pessoais, segredos, caminhos locais e credenciais antes de versionar ou publicar.
