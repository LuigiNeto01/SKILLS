---
name: auto-commit
description: Realiza commits automáticos segregando arquivos por pasta/responsabilidade, com prefixos emoji e descrições detalhadas em português.
---

# Skill: Auto-Commit Inteligente

Esta skill realiza commits automáticos no Git, agrupando arquivos por pasta ou responsabilidade similar, com mensagens descritivas em português e prefixos com emoji.

## Quando usar

O usuário pode invocar esta skill com comandos como:
- "Commita pra mim"
- "Faz o commit"
- "Commita tudo"
- "/auto-commit"

## Formato da Mensagem de Commit

```
prefix (emoji): texto descritivo
```

### Prefixos e Emojis

Use os seguintes prefixos baseados no tipo de mudança:

| Prefixo        | Emoji | Uso                                      |
|----------------|-------|------------------------------------------|
| feat           | ✨    | Nova funcionalidade ou arquivo novo       |
| fix            | 🐛    | Correção de bug                           |
| refactor       | ♻️    | Refatoração de código                     |
| style          | 💄    | Alterações visuais, CSS, formatação       |
| docs           | 📝    | Documentação                              |
| test           | 🧪    | Testes                                    |
| config         | ⚙️    | Configurações, env, build, CI/CD          |
| chore          | 🔧    | Tarefas gerais, manutenção                |
| deps           | 📦    | Dependências (package.json, requirements) |
| perf           | ⚡    | Melhorias de performance                  |
| remove         | 🗑️    | Remoção de arquivos ou código             |
| init           | 🎉    | Commit inicial / setup do projeto         |
| data           | 🗃️    | Dados, migrations, seeds, schemas         |
| security       | 🔒    | Segurança                                 |
| assets         | 🖼️    | Imagens, ícones, fontes, mídia            |

## Procedimento Passo a Passo

### 1. Verificar Status do Git

Execute `git status --porcelain` para listar todos os arquivos modificados, novos ou deletados.

Se não houver mudanças, informe ao usuário que não há nada para commitar.

### 2. Analisar e Agrupar Arquivos

Agrupe os arquivos por **pasta ou responsabilidade semelhante**. Critérios de agrupamento:

1. **Por diretório**: Arquivos no mesmo diretório que fazem parte do mesmo módulo/componente
2. **Por tipo**: Arquivos de configuração juntos, documentação junta, etc.
3. **Por funcionalidade**: Se arquivos em pastas diferentes fazem parte da mesma feature, agrupe-os

**Exemplos de agrupamento:**
- `src/components/Header.jsx` + `src/components/Header.css` → mesmo commit (componente Header)
- `src/api/users.js` + `src/api/auth.js` → mesmo commit (camada de API)
- `package.json` + `package-lock.json` → mesmo commit (dependências)
- `README.md` + `docs/setup.md` → mesmo commit (documentação)
- `.env.example` + `vite.config.js` → mesmo commit (configurações)

### 3. Construir a Mensagem de Commit

Para cada grupo, construa a mensagem seguindo este formato:

```
prefix (emoji): descrição geral do grupo

Arquivos incluídos:
- caminho/do/arquivo1.ext - descrição específica do que foi feito neste arquivo
- caminho/do/arquivo2.ext - descrição específica do que foi feito neste arquivo
```

#### Regras para a mensagem:
- A primeira linha (subject) deve ser **concisa e em português**
- Inclua uma **linha em branco** após o subject
- Liste **cada arquivo** com uma descrição detalhada do que foi feito
- Descreva o **conteúdo/propósito** do arquivo, não apenas "arquivo adicionado"
- Use verbos no **passado** (adicionou, corrigiu, atualizou, refatorou)

#### Exemplo completo:
```
feat (✨): implementação dos componentes de autenticação

Arquivos incluídos:
- src/components/LoginForm.jsx - Criado formulário de login com validação de email e senha, estados de loading e mensagens de erro
- src/components/LoginForm.css - Estilização do formulário com design responsivo, animações de foco e tema escuro
- src/hooks/useAuth.js - Hook customizado para gerenciar estado de autenticação, login, logout e verificação de token
```

### 4. Executar os Commits

Para cada grupo, execute na ordem:

```bash
# Adicionar apenas os arquivos do grupo
git add caminho/arquivo1 caminho/arquivo2

# Fazer o commit com a mensagem formatada
git commit -m "prefix (emoji): descrição" -m "Arquivos incluídos:" -m "- arquivo1 - descrição" -m "- arquivo2 - descrição"
```

> **IMPORTANTE:** Use `git add` especificando CADA ARQUIVO individualmente. **NUNCA** use `git add .` ou `git add --all`, pois isso misturaria arquivos de grupos diferentes.

> **IMPORTANTE:** Se a mensagem de commit for muito grande para um único `-m`, use múltiplos `-m` flags ou escreva a mensagem em um arquivo temporário e use `git commit -F arquivo_temp`.

### 5. Confirmar ao Usuário

Após todos os commits, apresente um resumo:

```
✅ Commits realizados com sucesso!

📋 Resumo:
1. feat (✨): implementação dos componentes de autenticação (3 arquivos)
2. style (💄): estilização da página inicial (2 arquivos)
3. config (⚙️): configurações do projeto (2 arquivos)

Total: 3 commits, 7 arquivos
```

## Notas Importantes

- Sempre verifique se o repositório Git está inicializado antes de começar
- Se o repositório não estiver inicializado, pergunte ao usuário se deseja inicializar com `git init`
- Arquivos no `.gitignore` devem ser ignorados automaticamente
- Em caso de conflitos ou problemas, informe o usuário imediatamente
- Se houver arquivos staged (já adicionados), considere-os no agrupamento
- Prefira commits menores e focados a commits grandes e genéricos
