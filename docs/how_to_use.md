# Como Usar

## Prompt Recomendado

Use esta solicitação padrão no VS Code Copilot Chat ou em qualquer agente compatível:

`Execute a skill Dashboard Review deste repositório usando o PBIX atualmente aberto no Power BI Desktop. Use Português, execute primeiro a skill Unused scan, siga o template da skill exatamente e salve as saídas apenas em results/ e unused_collect/ neste workspace.`

Essa redação reduz variações na execução e deixa quatro pontos explícitos:
- a skill a ser executada é `Dashboard Review`
- a origem é o PBIX já aberto no Power BI Desktop
- `Unused scan` deve ser executada antes do relatório final
- as saídas devem permanecer dentro deste workspace do repositório

## Comportamento Esperado

O agente deve:
- conectar ao PBIX aberto no Power BI Desktop
- executar `Unused scan` primeiro
- recriar `results/` e `unused_collect/` automaticamente se não existirem
- salvar o relatório final de performance em `results/`
- salvar a análise de objetos não utilizados em `unused_collect/`
- seguir o template de relatório definido na skill, e não um formato improvisado

## Regra Importante de Caminho

Se o agente precisar do caminho completo do `.pbix`, esse caminho serve apenas para ler o layout e os metadados do PBIX.

A pasta do `.pbix` nunca deve ser usada como local de saída para relatórios gerados.

Todos os arquivos gerados devem ser salvos apenas dentro deste repositório:
- `results/<DashboardName> - Performance Analysis.md`
- `unused_collect/<DashboardName> - Unused Analysis.md`

## Pastas Vazias no GitHub

O Git não mantém diretórios vazios por padrão. Por isso, este repositório versiona:
- `results/.gitkeep`
- `unused_collect/.gitkeep`

Mesmo com `.gitkeep`, as skills também recriam essas pastas automaticamente. Isso protege a execução quando alguém clona o repositório, remove as pastas ou começa a partir de um ambiente limpo.
