# Guia para uso de Devin CLI usando as melhores praticas com SDD

```text
Devin CLI = agente local no terminal
Foco principal:
1. Modes
2. AGENTS.md / Rules
3. Skills
4. Subagents
5. Permissions
6. Hooks
7. MCP
8. Workflow SDD local
```

O Devin CLI tem modos como **Normal**, **Accept Edits**, **Bypass**, **Autonomous**, além dos agent-modes **Plan** e **Ask**. Para estudo e trabalho seguro, o mais importante é aprender bem `/plan`, `/ask`, `/normal` e `/accept-edits`; deixe `/bypass` para casos muito confiáveis, porque ele autoaprova chamadas de ferramenta, incluindo comandos e edições. ([Devin][1])

## Ordem certa para estudar

### 1. `/plan` e SDD local

Esse deve ser seu primeiro foco.

No Devin CLI, use `/plan` para fazer o agente **pensar antes de alterar código**. A documentação descreve Plan como um modo de planejamento com ferramentas read-only, ideal para arquitetura, entendimento de codebase e planejamento antes da implementação. ([Devin][2])

Seu fluxo base deveria ser:

```text
1. /plan
2. Entender a tarefa
3. Gerar spec.md
4. Gerar plan.md
5. Gerar tasks.md
6. Sair do plan
7. Implementar task por task
8. Rodar testes
9. Revisar diff
```

Prompt inicial para usar no Devin CLI:

```text
/plan

Quero implementar a seguinte feature: [descreva a feature].

Antes de alterar qualquer código:
1. Analise a estrutura do projeto.
2. Identifique arquitetura, padrões e convenções.
3. Gere uma proposta de spec.md com:
   - objetivo
   - escopo
   - regras de negócio
   - critérios de aceite
   - dúvidas abertas
4. Gere uma proposta de plan.md com:
   - arquivos impactados
   - estratégia técnica
   - riscos
   - testes necessários
5. Gere uma proposta de tasks.md com tarefas pequenas.

Não altere arquivos ainda.
```

Esse é o uso prático de **SDD no Devin CLI**: você transforma uma ideia em especificação, plano e tarefas antes de deixar o agente codar.

---

### 2. `AGENTS.md`

Depois, aprenda `AGENTS.md`.

Ele é o arquivo de regras permanentes do projeto. O Devin CLI lê automaticamente `AGENTS.md`, `AGENT.md` e também pode importar regras de ferramentas como Cursor, Windsurf e Claude Code. A documentação recomenda `AGENTS.md` como abordagem principal para regras de projeto, mas também recomenda manter essas regras pequenas e usar Skills quando possível. ([Devin][3])

Exemplo bom para seu contexto Java/Spring/Hexagonal:

```md
# AGENTS.md

## Projeto
Este projeto usa Java, Spring Boot, PostgreSQL e arquitetura hexagonal.

## Regras de arquitetura
- Domínio não deve depender de Spring, JPA, controllers ou frameworks.
- Casos de uso ficam na camada application/usecase.
- Portas de entrada ficam em application/ports/input.
- Portas de saída ficam em application/ports/output.
- Adapters REST ficam em infrastructure/adapters/input.
- Adapters JPA ficam em infrastructure/adapters/output.

## Antes de codar
- Leia os arquivos relacionados.
- Identifique padrões existentes.
- Não crie abstrações desnecessárias.
- Não altere contrato público sem explicar impacto.

## Validação
- Rode testes relacionados.
- Explique arquivos alterados.
- Informe comandos executados.
```

Regra prática:

```text
AGENTS.md = regras pequenas que sempre valem
Skills = processos maiores que só devem entrar quando necessário
```

---

### 3. Skills

Depois de `AGENTS.md`, foque em **Skills**.

No Devin CLI, Skills são unidades reutilizáveis com prompt, permissões, ferramentas permitidas e workflow. Elas podem ser chamadas por você com `/nome-da-skill` ou pelo próprio agente quando forem relevantes. ([Devin][4])

Estrutura:

```text
.devin/
  skills/
    review-diff/
      SKILL.md
```

Exemplo de Skill:

```md
---
name: review-diff
description: Revisa o diff atual procurando bugs, problemas de arquitetura e falta de testes
allowed-tools:
  - read
  - grep
  - glob
  - exec
permissions:
  allow:
    - Exec(git diff)
    - Exec(git status)
  deny:
    - write
    - edit
triggers:
  - user
---

Revise o diff atual.

Passos:
1. Rode `git status`.
2. Rode `git diff`.
3. Analise:
   - bugs de lógica
   - quebra de arquitetura
   - falta de testes
   - problemas de segurança
   - inconsistência com padrões do projeto

Responda com:
- resumo geral
- problemas encontrados
- gravidade
- sugestão objetiva de correção
- arquivos impactados
```

Você usaria assim:

```text
/review-diff
```

Skills podem ficar no projeto em `.devin/skills/<nome>/SKILL.md` ou globalmente na sua máquina, por exemplo em `%APPDATA%\devin\skills\<nome>\SKILL.md` no Windows. ([Devin][4])

---

### 4. Subagents

Depois estude **subagents**.

Subagents são trabalhadores independentes que o agente principal pode criar para tarefas específicas. Eles compartilham o contexto da codebase, mas trabalham em uma conversa separada e não herdam todo o histórico do agente principal. Isso é útil para pesquisa, revisão, testes ou análise paralela. ([Devin][5])

Use subagent para:

```text
- pesquisar como uma parte do sistema funciona
- revisar um diff
- rodar testes
- analisar impacto de uma mudança
- investigar bug sem poluir a conversa principal
```

Exemplo de subagent customizado:

```text
.devin/
  agents/
    reviewer/
      AGENT.md
```

```md
---
name: reviewer
description: Revisa código procurando bugs, problemas de arquitetura, segurança e testes
model: sonnet
allowed-tools:
  - read
  - grep
  - glob
  - exec
permissions:
  allow:
    - Exec(git diff)
    - Exec(git status)
  deny:
    - write
    - edit
---

Você é um subagent especialista em code review.

Revise alterações sem modificar arquivos.

Procure:
1. Erros de lógica
2. Quebra de arquitetura
3. Problemas de segurança
4. Falta de testes
5. Inconsistências com padrões do projeto

Sempre cite arquivos e explique o impacto.
```

Custom subagents são definidos com `AGENT.md` dentro de `.devin/agents/<nome>/`, mas a própria documentação marca custom subagents como recurso experimental. ([Devin][5])

---

### 5. Permissions

Depois estude permissões.

O sistema de permissões controla o que o Devin pode fazer sem pedir aprovação. Ele suporta regras de `allow`, `deny` e `ask`; regras `deny` têm prioridade, depois `ask`, depois `allow`. ([Devin][6])

Exemplo seguro para projeto:

```json
{
  "permissions": {
    "allow": [
      "Read(**)",
      "Exec(git status)",
      "Exec(git diff)",
      "Exec(git log)"
    ],
    "ask": [
      "Write(**)",
      "Edit(**)",
      "Exec(mvn test)",
      "Exec(npm test)"
    ],
    "deny": [
      "Exec(rm -rf **)",
      "Exec(git push **)",
      "Exec(sudo **)"
    ]
  }
}
```

No Devin CLI, configurações de projeto podem ficar em `.devin/config.json`; no config de projeto, a documentação lista suporte para `permissions`, `mcpServers`, `read_config_from` e `hooks`. ([Devin][7])

---

### 6. Hooks

Depois vá para **Hooks**.

Hooks permitem rodar lógica em eventos do agente, como antes de usar uma ferramenta, depois de executar algo, no início da sessão ou quando há pedido de permissão. Eles servem para bloquear comandos perigosos, adicionar contexto, logar ações, integrar com sistemas externos ou modificar permissões dinamicamente. ([Devin][8])

Exemplo:

```text
.devin/
  hooks.v1.json
scripts/
  check-command.sh
```

```json
{
  "PreToolUse": [
    {
      "matcher": "exec",
      "hooks": [
        {
          "type": "command",
          "command": "./scripts/check-command.sh",
          "timeout": 10
        }
      ]
    }
  ]
}
```

Esse hook roda antes de comandos shell. O script recebe os dados do evento via stdin e pode bloquear a ação retornando uma decisão como `block`. ([Devin][8])

---

### 7. MCP

MCP é o último da lista, porque aumenta bastante o poder do agente.

No Devin CLI, MCP conecta o agente a ferramentas externas, como APIs, bancos, issue trackers e serviços que você expõe por um MCP server. Depois de configurado, as ferramentas aparecem para o agente como ferramentas nativas. ([Devin][9])

Use MCP quando fizer sentido o Devin consultar ou agir sobre:

```text
- GitHub
- Jira
- Linear
- Notion
- documentação interna
- APIs internas
- banco de dados de desenvolvimento
```

Mas só depois de dominar permissões e hooks, porque MCP amplia o risco. A própria documentação mostra que ferramentas MCP entram no sistema de permissões com nomes como `mcp__github__create_issue`, permitindo liberar ou negar ferramentas específicas. ([Devin][9])

---

## Seu roteiro de estudo prático

Eu faria assim:

```text
Semana 1:
- Aprender /plan, /ask, /normal, /accept-edits
- Praticar SDD local com spec.md, plan.md e tasks.md

Semana 2:
- Criar AGENTS.md para um projeto real
- Melhorar prompts de planejamento
- Usar Devin para implementar task por task

Semana 3:
- Criar 3 Skills:
  1. review-diff
  2. generate-tests
  3. architecture-review

Semana 4:
- Criar subagent reviewer
- Criar subagent researcher
- Usar skills rodando com subagent

Semana 5:
- Configurar permissions
- Criar hooks para bloquear comandos perigosos

Semana 6:
- Estudar MCP
- Conectar uma ferramenta simples, como GitHub ou documentação interna
```

## O que você deve dominar primeiro

Prioridade real para você:

```text
1. /plan + SDD local
2. AGENTS.md
3. Skills
4. Permissions
5. Subagents
6. Hooks
7. MCP
```

Minha recomendação: **não comece por MCP, hooks ou subagents**. Comece dominando `/plan`, `AGENTS.md` e Skills. Isso já vai melhorar muito a qualidade das entregas com Devin CLI.

O primeiro exercício que eu faria agora seria criar um `AGENTS.md` e uma skill `/review-diff` no seu projeto atual. Depois disso, a gente evolui para `/architecture-review` e um subagent `reviewer`.

[1]: https://cli.devin.ai/docs/essential-commands "Essential Commands - Devin for Terminal"
[2]: https://cli.devin.ai/docs/reference/commands "Commands & Flags - Devin for Terminal"
[3]: https://cli.devin.ai/docs/extensibility/rules "Rules & AGENTS.md - Devin for Terminal"
[4]: https://cli.devin.ai/docs/extensibility/skills/overview "Skills Overview - Devin for Terminal"
[5]: https://cli.devin.ai/docs/subagents "Subagents - Devin for Terminal"
[6]: https://cli.devin.ai/docs/reference/permissions "Permissions - Devin for Terminal"
[7]: https://cli.devin.ai/docs/reference/configuration/config-file "Configuration File - Devin for Terminal"
[8]: https://cli.devin.ai/docs/extensibility/hooks/overview "Hooks - Devin for Terminal"
[9]: https://cli.devin.ai/docs/extensibility/mcp/overview "MCP Overview - Devin for Terminal"
