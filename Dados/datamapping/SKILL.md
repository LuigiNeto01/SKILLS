---
name: datamapping
description: Gera arquivos Excel de Data Mapping para cada microserviço de um projeto, clonando um template com estilos, preenchendo Capa, Error Codes, rotas de serviço e Errors.
---

# Skill: Data Mapping Generator

Gera arquivos `.xlsx` de Data Mapping a partir de um template Excel para cada microserviço de uma API. Cada arquivo contém as abas **Capa**, **{error codes}**, **rotas do serviço** e **{errors}**, todas formatadas com os estilos do template original.

## Quando usar

O usuário pode invocar esta skill com comandos como:
- "Gera o data mapping"
- "Cria os data mappings dos serviços"
- "/datamapping"

## Pré-requisitos

- **Python** com a biblioteca `openpyxl` instalada (`pip install openpyxl`)
- **Código-fonte do projeto** para análise das rotas, modelos e erros de cada serviço

## Arquivos de Referência

Dentro da pasta `scripts/` desta skill existem os seguintes arquivos de referência:

| Arquivo | Descrição |
|---------|----------|
| `scripts/template.xlsx` | **Template base** com toda a formatação de estilos, cores, bordas e merges. Contém as 4 abas: `Capa`, `{error codes}`, `Serviço`, `{errors}`. Este é o arquivo que deve ser clonado via `shutil.copy2()` para gerar cada data mapping. |
| `scripts/exemplo.xlsx` | **Exemplo preenchido** de um data mapping completo (`upload-service`). Use como referência visual para entender como o resultado final deve ficar — quais campos preencher, como as rotas são dispostas e como os error codes são listados. |
| `scripts/generate_datamapping.py` | **Script Python** com todas as funções utilitárias para geração. Use como base, adaptando a seção `__main__` para o projeto. |

> **IMPORTANTE:** Ao gerar os data mappings, o template a ser utilizado é SEMPRE o arquivo `scripts/template.xlsx` desta skill. Copie-o para o diretório de saída antes de preencher. O `scripts/exemplo.xlsx` serve APENAS como referência visual — não o altere nem o use como template.

## Estrutura do Template

### Aba `Capa`
Contém metadados do documento:
- `B1` / `C4`: Nome do serviço
- `C5`: Nome do projeto (ex: `CongonhasHUB`)
- `C6`: Sistema (ex: `[X] CongonhasHUB_API`)
- `C7`: Versão (ex: `1.0`)
- `B17`: Autor → **Luigi Neto**
- `D17`: Data de criação → data atual no formato `DD/MM/YYYY`
- `E17`: Versão `1.0`
- `F17`: `Criação do documento`
- `B25`: Aprovador → **Luigi Neto**
- `D25`: Data de aprovação → data atual no formato `DD/MM/YYYY`
- `E25`: Versão `1.0`
- `F25`: `Criação do Template`

### Aba `{error codes}`
Lista de códigos de erro do serviço:
- **Linha 4** (C4:G4 merged): Título → `MS {nome-do-serviço} - Error Codes`
- **Linha 5** (C5:G5): Headers de coluna → `Código de Erro do Backend`, `Mensagem`, `httpCode`, `detail`, `message`
- **Linha 6+** (C6:G6...): Células de dados, replicadas para cada erro
- Altura da linha 4: `60.75`, linha 5: `18.75`

### Aba `Serviço` (renomeada/duplicada por rota)
Cada rota da API gera um bloco:
- **Linha título** (A:J merged): `[MÉTODO] /rota - REQUEST NomeModel / RESPONSE NomeModel`
  - Altura: `29.25`
- **Linha labels** (A:E merged = `REQUEST`, F:J merged = `RESPONSE`)
  - Altura: `21.0`
- **Linha headers**: 
  - Request: `Tipo de Container`, `Localização/Objeto`, `Propriedade`, `Regras`, `Exemplo`
  - Response: `Tipo de Container`, `Localização/Objeto`, `Propriedade`, `Regras`, `Obrigatório?`
- **Linhas de dados**: Uma linha por propriedade do request/response
- Largura das colunas: `A:18.0`, `B:21.14`, `C:18.14`, `D:90.43`, `E:72.29`, `F:18.0`, `G:41.57`, `H:66.29`, `I:72.71`, `J:71.0`

### Aba `{errors}`
Erros padrão do framework (HTTPException, ValidationError):
- Mesma estrutura de título/labels/headers/dados da aba Serviço
- Bloco 1: `HTTPException - RESPONSE (detail)` com campo `detail`
- Bloco 2: `ValidationError (422) - RESPONSE (detail[])` com campos `loc`, `msg`, `type`

## Procedimento Passo a Passo

### 1. Localizar o Template

O template está disponível dentro desta skill em:
```
scripts/template.xlsx   (relativo a esta skill)
```

Use o caminho absoluto resolvido a partir do diretório desta skill. Por exemplo:
```
.agent/skills/dados/datamapping/scripts/template.xlsx
```

Para referência visual de como o resultado deve ficar, consulte:
```
scripts/exemplo.xlsx    (exemplo preenchido de upload-service)
```

### 2. Analisar o Código-Fonte do Projeto

Leia o código do projeto para identificar:
- **Cada microserviço** (ex: `auth-service`, `user-service`, etc.)
- **Rotas de cada serviço**: método HTTP, path, modelo de request, modelo de response
- **Modelos/schemas**: propriedades, tipos, regras de validação, obrigatoriedade
- **Error codes**: status HTTP, detail, mensagens de cada serviço

### 3. Criar o Script Python de Geração

Use o script de referência em `scripts/generate_datamapping.py` como base. São as funções utilitárias de cópia de estilo e preenchimento de cada aba. Adapte para o projeto em questão:

#### Funções principais:
- `copy_cell_style(src, dst)` — Copia fonte, preenchimento, borda, alinhamento e formato numérico
- `get_template_styles(template_path)` — Extrai células de referência para cada tipo de estilo
- `fill_capa(ws, service_name)` — Preenche a aba Capa
- `write_error_codes_sheet(ws, styles, service_name, errors)` — Preenche {error codes}
- `write_service_sheet(ws, styles, routes)` — Preenche abas de rotas do serviço
- `write_errors_sheet(ws, styles)` — Preenche {errors} com padrões FastAPI
- `generate_service(styles, service_name, error_codes, route_sheets)` — Orquestra tudo

### 4. Definir os Dados de Cada Serviço

Para cada microserviço, construa as estruturas de dados:

```python
# Error codes: lista de tuplas (httpCode, detail, message)
error_codes = [
    (401, 'invalid_credentials', 'Credenciais inválidas'),
    (404, 'user_not_found', 'Usuário não encontrado'),
]

# Route sheets: dict { nome_da_aba: [rotas] }
# Cada rota: { 'title': str, 'request': [...], 'response': [...] }
route_sheets = {
    'register': [{
        'title': '[POST] /auth/register - REQUEST RegisterRequest / RESPONSE UserProfile',
        'request': [
            ('JSON Body', 'root', 'email', 'Email válido, obrigatório', 'user@example.com'),
            ('JSON Body', 'root', 'password', 'Mínimo 8 caracteres', '********'),
        ],
        'response': [
            ('JSON Body', 'root', 'id', 'UUID', 'S'),
            ('JSON Body', 'root', 'email', 'Email do usuário', 'S'),
        ]
    }],
    'login': [{
        'title': '[POST] /auth/login - REQUEST LoginRequest / RESPONSE TokenResponse',
        'request': [...],
        'response': [...]
    }],
}
```

### 5. Gerar os Arquivos

Para cada serviço:
1. Copiar o template para `CongonhasHUB_API - DataMapping - {nome-do-serviço}_1.0.xlsx`
2. Preencher a aba Capa
3. Preencher a aba {error codes}
4. Renomear a aba "Serviço" para a primeira rota e criar abas adicionais para outras rotas
5. Preencher a aba {errors}
6. Salvar o arquivo

### 6. Apresentar Resumo

Após gerar todos os arquivos, apresentar:

```
✅ Data Mappings gerados com sucesso!

📋 Resumo:
1. auth-service - 3 rotas, 5 error codes → CongonhasHUB_API - DataMapping - auth-service_1.0.xlsx
2. user-service - 4 rotas, 3 error codes → CongonhasHUB_API - DataMapping - user-service_1.0.xlsx

Total: X serviços, Y rotas, Z arquivos gerados
```

## Regras Importantes

- **SEMPRE** copie os estilos do template usando `copy_cell_style()`. Nunca crie estilos do zero.
- **SEMPRE** replique as alturas de linha conforme documentado acima.
- **SEMPRE** replique as larguras de coluna conforme documentado acima.
- **SEMPRE** use merges conforme a estrutura do template.
- **NUNCA** altere o template original — sempre copie primeiro com `shutil.copy2()`.
- O nome do autor é **Luigi Neto** em todos os documentos.
- A data deve ser a data atual da execução.
- Nomes de arquivo seguem o padrão: `CongonhasHUB_API - DataMapping - {serviço}_1.0.xlsx`
- As abas de serviço devem ser nomeadas de forma descritiva (ex: `register`, `login`, `alertas`).
- Para cada propriedade dos modelos, identifique: tipo de container (`JSON Body`, `Query Param`, `Path Param`, `Header`), localização/objeto, propriedade, regras/tipo e exemplo ou obrigatoriedade.
