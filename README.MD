# Power BI Performance Assistant

Assistente de análise técnica para modelos Power BI, pensado para ser usado no VS Code em conjunto com o chat do GitHub Copilot ou outro agente compatível com skills de repositório e acesso ao modelo aberto no Power BI Desktop.

A proposta deste repositório é padronizar revisões técnicas de dashboards e acelerar diagnósticos de performance, estrutura de modelo e excesso de objetos não utilizados. Em vez de depender de análises manuais e subjetivas, o assistente orienta a execução por meio de skills com regras explícitas, formato de saída definido e diretórios de resultado previsíveis.

## Objetivo

Este projeto apoia pessoas que desenvolvem dashboards em Power BI e precisam:

- revisar a saúde técnica de um modelo sem partir de uma folha em branco;
- identificar riscos estruturais que impactam performance e custo;
- localizar colunas e medidas possivelmente não utilizadas;
- gerar documentação técnica padronizada para revisão, cleanup ou refatoração;
- usar o VS Code como ambiente central de análise, com o Copilot Chat orquestrando as skills do repositório.

## O Que Este Assistente Faz

Hoje o repositório organiza o trabalho em duas skills principais:

### `Dashboard Review`

Skill principal para revisar o modelo Power BI aberto no Desktop e gerar um relatório técnico estruturado de performance.

Ela foi desenhada para avaliar, entre outros pontos:

- estrutura do modelo;
- volume de dados;
- risco de scans em medidas;
- riscos de serviço, como DirectQuery, modelos compostos e combinações de storage mode.

### `Unused scan`

Skill de apoio usada antes do relatório final.

Ela cruza metadados do modelo com evidências do layout do relatório para detectar:

- colunas não utilizadas;
- medidas não utilizadas;
- concentração desse desperdício por tabela;
- nível de confiança da análise.

## Como as Skills se Relacionam

O fluxo esperado do projeto é este:

1. o agente conecta no PBIX aberto no Power BI Desktop;
2. a skill `Unused scan` roda primeiro;
3. a análise de objetos não utilizados é salva em `unused_collect/`;
4. a skill `Dashboard Review` reaproveita essas evidências;
5. o relatório técnico final é salvo em `results/`.

Essa ordem é importante porque a revisão final depende dos achados da varredura de objetos não utilizados como evidência principal no bloco de volume de dados.

## Contexto Técnico Assumido

Pelas instruções atuais das skills, o assistente foi pensado para um contexto de Power BI/Fabric em que:

- existe preocupação real com pressão de memória e custo de processamento;
- scans grandes, alta cardinalidade e modelagem ineficiente são tratados como riscos relevantes;
- o modo de armazenamento esperado, por padrão, é `Import`;
- uso de `DirectQuery`, modelo composto e misturas de storage mode merecem atenção especial.

## Público-Alvo

Este repositório é voltado principalmente para:

- desenvolvedores de dashboards Power BI;
- analistas BI que mantêm modelos semânticos;
- pessoas que fazem tuning de performance em PBIX;
- times de governança ou sustentação de Power BI;
- consultores que precisam padronizar diagnósticos técnicos.

## Como Usar

### Pré-requisitos

Para o fluxo funcionar como esperado, considere este cenário base:

- `Power BI Desktop` aberto com o PBIX que será analisado;
- `VS Code` aberto na pasta deste repositório;
- `GitHub Copilot Chat` ou agente equivalente com suporte a skills do repositório;
- acesso às ferramentas necessárias para ler o modelo aberto e, quando aplicável, o layout do PBIX;
- permissão de escrita nas pastas `results/` e `unused_collect/`.

Na prática, o assistente foi desenhado para operar com acesso ao modelo ativo no Desktop e com recursos de inspeção de metadados. Sem isso, a análise tende a cair para modo parcial.

### Fluxo recomendado no VS Code

1. Abra este repositório no VS Code.
2. Abra o Power BI Desktop com o arquivo `.pbix` que deseja revisar.
3. No chat do Copilot, use um prompt que invoque explicitamente a skill principal.
4. Aguarde a execução da análise de objetos não utilizados.
5. Aguarde a geração do relatório final de performance.
6. Consulte os arquivos produzidos nas pastas do repositório.

### Prompt recomendado

Use este prompt no Copilot Chat:

```text
Execute the Dashboard Review skill in this repository using the PBIX currently open in Power BI Desktop. Use Portuguese, run the Unused scan skill first, follow the skill template exactly, and save outputs only to results/ and unused_collect/ in this workspace.
```

Esse prompt deixa explícito:

- qual skill deve ser executada;
- que a origem é o PBIX aberto no Desktop;
- que a `Unused scan` precisa rodar primeiro;
- que as saídas devem permanecer dentro deste workspace.

## Formas de Uso

Você pode usar o repositório de formas diferentes, dependendo do objetivo.

### 1. Revisão técnica completa de um dashboard

Use quando quiser um diagnóstico mais amplo do modelo.

Resultado esperado:

- análise estrutural;
- riscos de performance;
- sinais de desperdício de volume;
- recomendações priorizadas de correção.

### 2. Apoio em cleanup de modelo

Use quando o foco for entender o que pode estar sobrando antes de remover colunas ou medidas.

Resultado esperado:

- relatório de colunas e medidas possivelmente não utilizadas;
- evidências por tabela;
- classificação de confiança para orientar validação antes de exclusão.

### 3. Preparação para refatoração

Use antes de reorganizar medidas, revisar relacionamentos, mexer em cardinalidade ou discutir incremental refresh.

Resultado esperado:

- base técnica para decidir o que priorizar;
- visão mais clara de gargalos prováveis;
- registro escrito para alinhar decisões com o time.

### 4. Documentação técnica de um PBIX

Use quando você precisa transformar observações técnicas em um artefato reutilizável.

Resultado esperado:

- um relatório em Markdown com estrutura consistente;
- histórico reproduzível de análise dentro do próprio repositório.

## O Que Esperar da Análise

O assistente foi projetado para ser técnico, objetivo e didático.

Você deve esperar dele:

- linguagem construtiva, sem tom de auditoria punitiva;
- foco em causa e efeito, não apenas em listar problemas;
- relatórios em Markdown, com estrutura fixa;
- recomendações acionáveis para Power BI Desktop;
- separação entre evidência confirmada, provável e preliminar;
- indicação de limitações quando alguma etapa não puder ser concluída.

Você não deve esperar dele:

- tuning perfeito sem contexto do negócio;
- prova absoluta de que um objeto pode ser removido sem validação humana;
- análise confiável se o modelo aberto ou o layout do PBIX não puderem ser lidos;
- liberdade para salvar resultados fora deste repositório.

## Estrutura dos Resultados

As saídas são sempre gravadas dentro deste workspace.

### Relatório final de performance

Arquivo gerado em:

```text
results/<DashboardName> - Performance Analysis.md
```

### Relatório de objetos não utilizados

Arquivo gerado em:

```text
unused_collect/<DashboardName> - Unused Analysis.md
```

## Estrutura do Repositório

```text
.github/skills/
  Dashboard Review/
    SKILL.md
  Unused scan/
    SKILL.md
docs/
  how_to_use.md
results/
unused_collect/
README.MD
```

### Papel de cada pasta

- `.github/skills/`: definição das skills que orientam o comportamento do agente;
- `docs/`: documentação auxiliar de uso;
- `results/`: relatórios finais de análise de performance;
- `unused_collect/`: saídas intermediárias e relatórios de objetos não utilizados.

## Regras Operacionais Importantes

As skills deste repositório definem algumas regras que valem como contrato de execução:

- o trabalho deve acontecer sobre o PBIX atualmente aberto no Power BI Desktop;
- a saída deve ser salva apenas dentro deste repositório;
- o caminho completo do `.pbix`, quando necessário, serve somente como entrada para leitura de layout e metadados;
- o agente não deve salvar arquivos ao lado do `.pbix`;
- `Unused scan` deve ser executada antes de `Dashboard Review`;
- o formato do relatório final não deve ser improvisado;
- relatórios devem ser sobrescritos quando já existirem;
- as pastas `results/` e `unused_collect/` devem ser recriadas automaticamente se estiverem ausentes.

## Tipos de Evidência Considerados

Pelas skills atuais, a análise combina diferentes fontes de evidência, como:

- inventário do modelo;
- relacionamentos;
- dependências entre objetos;
- expressões DAX;
- configurações como `Sort By`;
- layout do relatório dentro do PBIX;
- filtros, slicers, bookmarks e bindings visuais, quando disponíveis.

Isso é importante porque o objetivo não é marcar objetos como não utilizados por suposição simples. A intenção é reduzir falsos positivos cruzando metadados do modelo com o uso real no relatório.

## Idiomas

As skills suportam:

- Português;
- English;
- Spanish.

Para este repositório, o uso principal esperado é em português, inclusive para os relatórios.

## Limitações

Este assistente ajuda bastante, mas ele não substitui validação técnica final.

Considere especialmente estas limitações:

- um objeto pode parecer não utilizado no layout atual e ainda ser necessário em cenários externos ou futuros;
- a qualidade da análise depende do acesso correto ao modelo e ao PBIX;
- referências ambíguas não devem ser forçadas como evidência;
- se a extração do layout falhar, o nível de confiança da análise cai;
- recomendações sobre exclusão de colunas e medidas devem ser validadas antes de aplicação em produção.

## Boas Práticas de Uso

Para obter resultados melhores:

- mantenha apenas o PBIX correto aberto no momento da análise;
- rode a skill com um prompt explícito e estável;
- não mude o diretório de saída para fora do repositório;
- valide objetos marcados como não utilizados antes de removê-los;
- use o relatório como base de priorização, não como verdade absoluta;
- compare os achados com contexto de negócio, uso em páginas ocultas e dependências externas.

## Quando Este Repositório É Mais Útil

Este assistente tende a ser especialmente útil quando:

- o PBIX cresceu e ficou difícil entender o que ainda é usado;
- o modelo apresenta lentidão e ninguém sabe por onde começar;
- há suspeita de excesso de colunas, medidas ou relações complexas;
- o time quer padronizar revisões técnicas;
- você quer usar IA no VS Code com uma estrutura mais controlada do que um prompt livre.

## Referência Rápida

### Resultado esperado após uma execução bem-sucedida

- arquivo de análise de não utilizados em `unused_collect/`;
- arquivo de diagnóstico técnico em `results/`;
- texto em português;
- estrutura padronizada em Markdown;
- recomendações técnicas priorizadas.

### Se algo falhar

Você deve esperar que o agente:

- tente caminhos determinísticos de fallback;
- sinalize execução degradada ou bloqueada;
- não invente métricas ausentes;
- explicite limitações no resultado quando não houver evidência suficiente.

## Documentação Complementar

Consulte também:

- `docs/how_to_use.md`
- `.github/skills/Dashboard Review/SKILL.md`
- `.github/skills/Unused scan/SKILL.md`

## Resumo

Este repositório transforma o VS Code com Copilot Chat em um ponto de apoio para análise técnica de dashboards Power BI. O valor principal não está só em gerar texto, mas em impor um fluxo de trabalho: primeiro localizar desperdícios e evidências de uso, depois consolidar um relatório técnico com foco em performance, manutenção e priorização de melhorias.
