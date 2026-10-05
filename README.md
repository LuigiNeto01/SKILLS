# SKILLS

Repositorio pessoal de skills para agentes. Cada skill fica em uma pasta propria,
com o arquivo principal chamado `SKILL.md`.

## Estrutura

```text
Dados/
  datamapping/
    SKILL.md
    scripts/
Desenvolvimento/
  compactador-de-contexto/
    SKILL.md
  humanizer/
    SKILL.md
Git/
  auto-commit/
    SKILL.md
```

## Skills disponiveis

- `Dados/datamapping` - gera arquivos Excel de Data Mapping a partir de um template.
- `Desenvolvimento/compactador-de-contexto` - compacta o contexto de uma conversa em Markdown reutilizavel.
- `Desenvolvimento/humanizer` - reescreve textos com cara de IA para uma voz mais natural, preservando o conteudo.
- `Git/auto-commit` - cria commits locais agrupados por responsabilidade, com mensagens em portugues.

## Padrao para adicionar skills

1. Crie uma pasta por skill dentro da categoria correta.
2. Coloque o conteudo principal em `SKILL.md`.
3. Deixe scripts, templates e exemplos em subpastas da propria skill.
4. Atualize este README quando adicionar ou mover uma skill.

Exemplo:

```text
Categoria/nome-da-skill/SKILL.md
```

