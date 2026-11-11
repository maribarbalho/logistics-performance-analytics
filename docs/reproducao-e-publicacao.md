# Como reproduzir e publicar

## Pré-requisitos

- Power BI Desktop atualizado.
- Arquivos de origem do case (CSV, Excel, JSON e XML), disponibilizados separadamente conforme as permissões de uso.
- Acesso à publicação no Power BI Service, se desejar publicar sua própria versão.

> O relatório online permite **visualizar** as análises. Não substitui o arquivo fonte PBIX/PBIP nem fornece automaticamente os dados e consultas Power Query.

## Processo geral

1. **Preparar os arquivos:** organizar as fontes em uma pasta local. Não versionar caminhos pessoais, dados confidenciais ou credenciais.
2. **Ingerir no Power Query:** importar as tabelas e conferir codificação UTF-8, delimitadores, locais/culturas, tipos de dados e nulos.
3. **Aplicar tratamentos:** remover duplicatas exatas; padronizar identificadores; corrigir tipos monetários, data/hora, latitude e longitude; sinalizar inconsistências.
4. **Modelar:** relacionar dimensões e fatos por chaves apropriadas. Conferir unicidade e cardinalidade antes de ativar filtros.
5. **Criar medidas DAX:** usar medidas para soma, contagem distinta, percentuais e médias; checar denominadores e contextos de filtro.
6. **Montar relatório:** criar páginas executivas, análises detalhadas, segmentações e navegação.
7. **Validar:** verificar resultados dos cartões versus tabelas, filtros, valores ausentes e comportamento de gráficos. Para uma validação reproduzível, executar consultas DAX no modelo (Exibição de consulta DAX ou motor local do Desktop) e comparar com cálculos independentes, como descrito em [Homologação dos dados](homologacao-dos-dados.md).
8. **Publicar:** enviar o relatório ao Power BI Service e configurar o compartilhamento conforme a política da conta e da organização. Ao republicar no mesmo workspace, escolher **Substituir**: o link de "Publicar na web" continua o mesmo (a atualização pode levar até cerca de 1 hora para aparecer no link público).

## Observações sobre a implementação

- Medidas que usam `TREATAS` podem aplicar filtros virtuais entre tabelas, mas **não substituem** a revisão da qualidade do modelo.
- O formato **PBIP/PBIR** facilita controle de versão quando suportado pela instalação do Power BI Desktop.
- Visuais geográficos podem depender das permissões do locatário. Na construção do projeto, o **Azure Mapas** encontrou restrições administrativas, e foi utilizado o mapa tradicional como alternativa. Verifique a disponibilidade e as políticas do visual na sua própria conta.
- Os gráficos de SAC devem usar `DISTINCTCOUNT(chamado_id)` quando as interações tiverem sido expandidas.
- Em gráficos mensais, crie uma coluna `MesAno` e outra com `Ano*100+Mes`, usando **Classificar por coluna**, para ordenar os meses cronologicamente.

## Exemplo de coluna auxiliar de mês

```dax
MesAno Abertura =
FORMAT(f_sac_chamados[abertura], "MMM/yyyy")

Ordem MesAno Abertura =
YEAR(f_sac_chamados[abertura]) * 100 +
MONTH(f_sac_chamados[abertura])
```

## Estrutura sugerida para futuras versões

```text
logistics-performance-analytics/
├── README.md
├── docs/
│   ├── visao-de-negocio.md
│   ├── modelagem-e-fontes.md
│   ├── indicadores-e-dax.md
│   ├── qualidade-e-limitacoes.md
│   └── reproducao-e-publicacao.md
├── assets/                 # Capturas autorizadas do dashboard
├── powerbi/                # Arquivo PBIP/PBIR ou PBIX, se disponibilizado
└── dados-exemplo/          # Somente dados sintéticos autorizados
```

As pastas `assets`, `powerbi` e `dados-exemplo` são sugestões para evolução do repositório, não arquivos disponibilizados nesta documentação.
