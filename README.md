# Guia completo de estudo — Devin CLI para desenvolvimento com IA

## Objetivo do guia

Este guia é para você usar o **Devin CLI**, também chamado na documentação de **Devin for Terminal**, de uma forma mais profissional no desenvolvimento de software.

A ideia é sair deste uso:

```text
"Devin, faz essa feature."
```

E ir para este fluxo:

```text
Entender → Especificar → Planejar → Quebrar em tarefas → Implementar → Validar → Revisar
```

O Devin for Terminal possui modos como **Normal**, **Accept Edits**, **Bypass** e **Autonomous**, além dos modos de agente **Normal**, **Plan** e **Ask**. Os comandos principais que vamos usar são `/normal`, `/ask`, `/plan` e `/accept-edits`. A documentação também mostra que alguns nomes podem aparecer de forma unificada em ambientes integrados ao Windsurf, como **Code**, **Ask**, **Plan**, **Accept Edits** e **Bypass Permissions**, mas para este guia vamos manter os comandos do Devin CLI. ([Devin][1])

---

# Visão geral das 6 semanas

| Semana   | Tema                           | Resultado esperado                                                                        |
| -------- | ------------------------------ | ----------------------------------------------------------------------------------------- |
| Semana 1 | Modos básicos + SDD local      | Usar `/plan`, `/ask`, `/normal`, `/accept-edits` e criar `spec.md`, `plan.md`, `tasks.md` |
| Semana 2 | `AGENTS.md` + prompts melhores | Criar regras permanentes do projeto e melhorar planejamento                               |
| Semana 3 | Skills                         | Criar comandos reutilizáveis: `review-diff`, `generate-tests`, `architecture-review`      |
| Semana 4 | Subagents                      | Criar agentes especializados: `reviewer` e `researcher`                                   |
| Semana 5 | Permissions + Hooks            | Controlar segurança e bloquear comandos perigosos                                         |
| Semana 6 | MCP                            | Conectar Devin a ferramentas externas, como GitHub ou documentação interna                |

---

# Antes de começar: o que é SDD?

Neste guia, vamos usar **SDD** como **Spec-Driven Development**, ou seja, desenvolvimento guiado por especificação.

Em vez de começar pelo código, você começa pela clareza.

```text
Demanda
  ↓
spec.md
  ↓
plan.md
  ↓
tasks.md
  ↓
implementação
  ↓
validação
  ↓
revisão
```

O objetivo é evitar que a IA implemente algo sem entender direito a regra de negócio, o escopo, os riscos e os critérios de aceite.

Pense assim:

```text
spec.md  = o que precisa funcionar
plan.md  = como vamos implementar
tasks.md = em quais passos vamos dividir
```

---

# Estrutura final recomendada no projeto

Ao longo das semanas, você pode chegar nesta estrutura:

```text
meu-projeto/
├── AGENTS.md
├── docs/
│   └── sdd/
│       ├── spec.md
│       ├── plan.md
│       └── tasks.md
├── .devin/
│   ├── config.json
│   ├── hooks.v1.json
│   ├── skills/
│   │   ├── review-diff/
│   │   │   └── SKILL.md
│   │   ├── generate-tests/
│   │   │   └── SKILL.md
│   │   └── architecture-review/
│   │       └── SKILL.md
│   └── agents/
│       ├── reviewer/
│       │   └── AGENT.md
│       └── researcher/
│           └── AGENT.md
└── scripts/
    └── check-command.sh
```

## O que cada arquivo representa?

| Arquivo                    | Para que serve                          |
| -------------------------- | --------------------------------------- |
| `AGENTS.md`                | Manual do projeto para agentes de IA    |
| `docs/sdd/spec.md`         | Especificação da funcionalidade         |
| `docs/sdd/plan.md`         | Plano técnico de implementação          |
| `docs/sdd/tasks.md`        | Lista de tarefas pequenas e executáveis |
| `.devin/skills/*/SKILL.md` | Comandos reutilizáveis para o Devin     |
| `.devin/agents/*/AGENT.md` | Subagents especializados                |
| `.devin/config.json`       | Configuração de permissões e MCP        |
| `.devin/hooks.v1.json`     | Configuração de hooks automáticos       |
| `scripts/check-command.sh` | Script para bloquear comandos perigosos |

Analogia simples:

```text
AGENTS.md          = manual da empresa
spec.md            = descrição da demanda
plan.md            = plano técnico
tasks.md           = checklist de execução
SKILL.md           = procedimento reutilizável
AGENT.md           = especialista de uma área
config.json        = regras de permissão
hooks.v1.json      = sistema de fiscalização
check-command.sh   = segurança na porta
MCP                = conexão com sistemas externos
```

---

# Semana 1 — Modos básicos + SDD local

## Objetivo da semana

Nesta semana, você vai aprender a controlar o Devin antes de mandar ele codar.

Você precisa dominar:

```text
1. /normal
2. /ask
3. /plan
4. /accept-edits
5. spec.md
6. plan.md
7. tasks.md
```

---

## 1. `/normal`

## O que é?

O `/normal` é o modo padrão e mais seguro para começar.

No modo Normal, o Devin pode fazer leituras dentro do diretório atual, mas pede permissão para escrever arquivos ou executar comandos. ([Devin][1])

## Para que serve?

Serve para trabalhar com controle.

Use quando você quer que o Devin ajude, mas não quer que ele saia alterando arquivos sozinho.

## Como pensar nele?

```text
"Pode me ajudar, mas me peça permissão antes de mexer."
```

## Exemplo de uso

```text
/normal

Analise a estrutura do projeto e me explique:
1. Qual é a linguagem principal
2. Qual framework está sendo usado
3. Quais camadas existem
4. Onde ficam controllers, use cases, domain e infrastructure

Não altere nenhum arquivo.
```

---

## 2. `/ask`

## O que é?

O `/ask` serve para fazer perguntas rápidas sem mudar o modo atual. A documentação descreve `/ask <question>` como um comando para perguntar algo sem alterar código. ([Devin][2])

## Para que serve?

Serve para estudar, tirar dúvida ou entender a codebase.

## Como pensar nele?

```text
"Me explique, mas não mexa em nada."
```

## Exemplos

```text
/ask Qual é a diferença entre application e infrastructure neste projeto?
```

```text
/ask Onde faz mais sentido validar e-mail único em arquitetura hexagonal?
```

```text
/ask Me explique o fluxo de cadastro de responsável neste projeto.
```

---

## 3. `/plan`

## O que é?

O `/plan` coloca o Devin em modo de planejamento.

Esse é o modo mais importante para SDD, porque você usa antes da implementação. A documentação lista `/plan` como um dos modos/comandos principais para trocar o comportamento do agente. ([Devin][2])

## Para que serve?

Serve para fazer o Devin entender o projeto, analisar impacto e propor uma estratégia antes de codar.

## Como pensar nele?

```text
"Antes de codar, vamos entender e planejar."
```

## Exemplo

```text
/plan

Preciso implementar cadastro de responsável.

Antes de alterar qualquer arquivo:
1. Analise a estrutura do projeto.
2. Identifique padrões existentes.
3. Gere uma proposta de spec.md.
4. Gere uma proposta de plan.md.
5. Gere uma proposta de tasks.md.
6. Não implemente nada ainda.
```

---

## 4. `/accept-edits`

## O que é?

O `/accept-edits` permite que o Devin edite arquivos dentro do workspace com menos interrupções, mas ainda mantém mais controle do que modos totalmente automáticos. Na documentação, Accept Edits aparece como um dos modos principais do Devin for Terminal. ([Devin][1])

## Para que serve?

Serve para quando você já revisou o plano e quer deixar o Devin implementar a tarefa.

## Como pensar nele?

```text
"Pode editar os arquivos, mas ainda siga o plano."
```

## Exemplo

```text
/accept-edits

Implemente somente a Task 1 do arquivo docs/sdd/tasks.md.

Regras:
1. Não avance para a Task 2.
2. Altere apenas os arquivos necessários.
3. Explique o que alterou.
4. Mostre o diff resumido ao final.
5. Peça permissão antes de executar comandos.
```

---

## 5. Evite `/bypass` no começo

O modo Bypass autoaprova mais ações e reduz sua intervenção. Ele pode ser útil em contextos confiáveis, mas não é ideal para aprender.

Na fase inicial, evite:

```text
/bypass
/yolo
/dangerous
```

Seu foco agora é:

```text
Entender → Planejar → Executar com controle
```

---

# Arquivo da Semana 1: `docs/sdd/spec.md`

## O que é?

O `spec.md` é a especificação da funcionalidade.

Ele responde:

```text
O que precisa ser feito?
Por que precisa ser feito?
Quais regras precisam ser respeitadas?
Como saberemos que está pronto?
```

## Para que serve?

Serve para evitar que o Devin comece a codar com uma demanda vaga.

Exemplo de demanda vaga:

```text
Crie cadastro de responsável.
```

Isso deixa várias dúvidas:

```text
- Quais campos são obrigatórios?
- E-mail precisa ser único?
- Senha precisa ser criptografada?
- O responsável começa ativo?
- Qual retorno da API?
- Precisa de teste?
```

O `spec.md` força essas decisões a ficarem claras.

## Como pensar no `spec.md`?

Pense nele como um documento de **produto + regra de negócio**.

Ele não deve explicar profundamente como codar. Ele deve explicar o comportamento esperado.

## Modelo

```md
# Spec — Nome da Feature

## Objetivo
Descrever o objetivo da funcionalidade.

## Contexto
Explicar o problema atual e por que essa alteração é necessária.

## Escopo
O que será feito agora.

## Fora de escopo
O que não será feito agora.

## Regras de negócio
- Regra 1
- Regra 2
- Regra 3

## Critérios de aceite
- Dado X, quando Y, então Z.
- Deve impedir determinada ação quando uma regra for violada.
- Deve retornar erro claro quando os dados forem inválidos.

## Dúvidas abertas
- Dúvida 1
- Dúvida 2
```

## Exemplo prático

```md
# Spec — Cadastro de Responsável

## Objetivo
Permitir cadastrar um responsável no sistema.

## Contexto
Atualmente o sistema precisa registrar responsáveis para associá-los a crianças, consultas ou outros recursos.

## Escopo
- Receber nome, e-mail e senha.
- Validar campos obrigatórios.
- Impedir cadastro com e-mail duplicado.
- Criar responsável com status ativo.

## Fora de escopo
- Login.
- Recuperação de senha.
- Edição de responsável.
- Exclusão de responsável.

## Regras de negócio
- Nome é obrigatório.
- E-mail é obrigatório.
- E-mail deve ser único.
- Senha é obrigatória.
- Todo responsável novo deve ser criado como ativo.

## Critérios de aceite
- Deve cadastrar responsável com dados válidos.
- Deve retornar erro quando e-mail já existir.
- Deve retornar erro quando nome, e-mail ou senha estiverem vazios.

## Dúvidas abertas
- A senha deve ser criptografada nesta tarefa?
- Qual status HTTP deve ser retornado em caso de e-mail duplicado?
```

---

# Arquivo da Semana 1: `docs/sdd/plan.md`

## O que é?

O `plan.md` é o plano técnico.

Ele responde:

```text
Como vamos implementar a spec?
Quais arquivos serão alterados?
Quais camadas serão impactadas?
Quais testes serão necessários?
Quais riscos existem?
```

## Para que serve?

Serve para transformar a especificação em uma estratégia técnica.

O `spec.md` diz **o que precisa acontecer**.

O `plan.md` diz **como vamos fazer isso no código**.

## Como pensar no `plan.md`?

Pense nele como o planejamento técnico de um desenvolvedor antes de começar a codar.

Em arquitetura hexagonal, por exemplo, ele deve separar o impacto por camada:

```text
Domain
Application
Infrastructure
Tests
Database
```

## Modelo

```md
# Plan — Nome da Feature

## Estratégia técnica
Explicar a abordagem geral.

## Arquivos impactados
- caminho/arquivo1.java
- caminho/arquivo2.java

## Alterações por camada

### Domain
O que muda no domínio.

### Application
O que muda nos casos de uso e portas.

### Infrastructure
O que muda em controllers, repositories, mappers, banco etc.

## Testes necessários
- Teste unitário X
- Teste de integração Y
- Teste manual Z

## Riscos
- Risco 1
- Risco 2

## Plano de validação
- Rodar comando X
- Conferir comportamento Y
- Revisar resposta da API
```

## Exemplo prático

```md
# Plan — Cadastro de Responsável

## Estratégia técnica
Implementar o cadastro seguindo arquitetura hexagonal, mantendo regra de negócio no domínio/use case e detalhes de persistência na infraestrutura.

## Arquivos impactados
- src/main/java/.../domain/Responsavel.java
- src/main/java/.../application/ports/input/CadastrarResponsavelInputPort.java
- src/main/java/.../application/ports/output/ResponsavelOutputPort.java
- src/main/java/.../application/usecase/CadastrarResponsavelUseCase.java
- src/main/java/.../infrastructure/adapters/input/controller/ResponsavelController.java
- src/main/java/.../infrastructure/adapters/output/repository/ResponsavelJpaRepository.java
- src/main/java/.../infrastructure/adapters/output/ResponsavelDataProvider.java

## Alterações por camada

### Domain
- Garantir criação válida de Responsavel.
- Definir status ativo na criação.

### Application
- Criar caso de uso de cadastro.
- Validar duplicidade de e-mail via output port.

### Infrastructure
- Criar endpoint REST.
- Implementar persistência usando JPA.
- Criar mapper entre domain e entity.

## Testes necessários
- Teste unitário do domínio.
- Teste unitário do use case.
- Teste do controller ou integração, se o projeto já possuir padrão.

## Riscos
- Vazamento de JPA para domínio.
- Validação duplicada em camadas erradas.
- Falta de tratamento adequado para e-mail duplicado.

## Plano de validação
- Rodar testes unitários.
- Rodar build.
- Testar cadastro com payload válido.
- Testar cadastro com e-mail duplicado.
```

---

# Arquivo da Semana 1: `docs/sdd/tasks.md`

## O que é?

O `tasks.md` é a quebra do plano em tarefas pequenas.

Ele responde:

```text
Qual é a sequência de implementação?
O que será feito primeiro?
O que pode ser validado separadamente?
```

## Para que serve?

Serve para impedir que o Devin tente implementar tudo de uma vez.

Esse é um dos pontos mais importantes para trabalhar bem com IA.

Pedido arriscado:

```text
Implemente toda a feature.
```

Pedido melhor:

```text
Implemente somente a Task 1.
Depois pare e mostre o diff.
```

## Como pensar no `tasks.md`?

Pense nele como um checklist de execução.

Cada task deve:

```text
- alterar poucos arquivos
- ter objetivo claro
- ter validação clara
- evitar misturar muitos assuntos
```

## Modelo

```md
# Tasks — Nome da Feature

## Task 1 — Nome da task
Objetivo:
- Descrever o objetivo.

Arquivos:
- caminho/arquivo.java

Alterações:
- O que deve ser alterado.

Validação:
- Como validar que funcionou.

## Task 2 — Nome da task
Objetivo:
- Descrever o objetivo.

Arquivos:
- caminho/arquivo.java

Alterações:
- O que deve ser alterado.

Validação:
- Como validar que funcionou.
```

## Exemplo prático

```md
# Tasks — Cadastro de Responsável

## Task 1 — Ajustar domínio
Objetivo:
- Garantir que Responsavel tenha criação válida.

Arquivos:
- src/main/java/.../domain/Responsavel.java

Alterações:
- Criar ou ajustar método de criação.
- Validar nome, e-mail e senha obrigatórios.
- Definir ativo como true.

Validação:
- Criar ou ajustar teste unitário da entidade.

## Task 2 — Criar portas de aplicação
Objetivo:
- Definir contratos de entrada e saída para cadastro.

Arquivos:
- src/main/java/.../application/ports/input/CadastrarResponsavelInputPort.java
- src/main/java/.../application/ports/output/ResponsavelOutputPort.java

Alterações:
- Criar método cadastrar.
- Criar método para verificar e-mail existente.
- Criar método para salvar responsável.

Validação:
- Verificar se os contratos não dependem de infraestrutura.

## Task 3 — Criar use case
Objetivo:
- Orquestrar o cadastro do responsável.

Arquivos:
- src/main/java/.../application/usecase/CadastrarResponsavelUseCase.java

Alterações:
- Validar e-mail duplicado.
- Chamar regra de criação do domínio.
- Salvar pela output port.

Validação:
- Criar teste unitário do use case.

## Task 4 — Criar adapter de persistência
Objetivo:
- Implementar a saída para banco de dados.

Arquivos:
- src/main/java/.../infrastructure/adapters/output/ResponsavelDataProvider.java
- src/main/java/.../infrastructure/adapters/output/repository/ResponsavelJpaRepository.java
- src/main/java/.../infrastructure/adapters/output/entity/ResponsavelEntity.java
- src/main/java/.../infrastructure/adapters/output/mapper/ResponsavelMapper.java

Alterações:
- Implementar busca por e-mail.
- Implementar salvamento.
- Mapear domain para entity.

Validação:
- Rodar testes de persistência, se existirem.

## Task 5 — Criar endpoint REST
Objetivo:
- Expor cadastro via HTTP.

Arquivos:
- src/main/java/.../infrastructure/adapters/input/controller/ResponsavelController.java
- src/main/java/.../infrastructure/adapters/input/dto/CadastrarResponsavelRequest.java
- src/main/java/.../infrastructure/adapters/input/dto/ResponsavelResponse.java

Alterações:
- Criar endpoint POST.
- Validar request.
- Chamar input port.
- Retornar resposta adequada.

Validação:
- Testar payload válido.
- Testar payload inválido.
```

---

# Prompts da Semana 1

## Prompt para gerar `spec.md`, `plan.md` e `tasks.md`

```text
/plan

Quero praticar SDD local neste projeto.

Tarefa:
[descreva a tarefa]

Antes de codar:
1. Analise o projeto.
2. Gere uma proposta de docs/sdd/spec.md.
3. Gere uma proposta de docs/sdd/plan.md.
4. Gere uma proposta de docs/sdd/tasks.md.
5. Não implemente nada ainda.
```

## Prompt para implementar uma task

```text
/accept-edits

Implemente somente a Task 1 do arquivo docs/sdd/tasks.md.

Regras:
1. Não avance para a Task 2.
2. Altere apenas os arquivos necessários.
3. Siga o plano.
4. Ao final, mostre o resumo do diff.
5. Informe como validar.
```

## Checklist da Semana 1

```text
[ ] Sei usar /ask para tirar dúvidas
[ ] Sei usar /plan antes de codar
[ ] Sei usar /normal para trabalhar com aprovação
[ ] Sei usar /accept-edits para implementar com controle
[ ] Sei criar spec.md
[ ] Sei criar plan.md
[ ] Sei criar tasks.md
[ ] Sei implementar uma task por vez
```

---

# Semana 2 — `AGENTS.md` + prompts de planejamento

## Objetivo da semana

Criar um arquivo com regras permanentes do projeto e melhorar a qualidade dos prompts usados com Devin.

A documentação do Devin CLI descreve Rules como instruções persistentes que moldam o comportamento do Devin em cada sessão. Ela também recomenda `AGENTS.md` como uma abordagem principal para regras de projeto, por ser versionado junto com o código. ([Devin][1])

---

# Arquivo da Semana 2: `AGENTS.md`

## O que é?

O `AGENTS.md` é o manual do projeto para agentes de IA.

Pense nele como a explicação que você daria para um novo desenvolvedor entrando no time:

```text
Este projeto usa esta arquitetura.
Estas são as regras.
Estes são os comandos.
Estas coisas você não deve fazer.
Este é o padrão esperado.
```

## Para que serve?

Serve para evitar repetir sempre as mesmas instruções.

Sem `AGENTS.md`, você acaba escrevendo toda hora:

```text
Lembre que o projeto usa arquitetura hexagonal.
Não coloque regra de negócio no controller.
Não use JPA no domínio.
Siga o padrão dos arquivos existentes.
Rode os testes antes de finalizar.
```

Com `AGENTS.md`, essas regras ficam documentadas no projeto.

## O que colocar nele?

Coloque regras duradouras:

```text
- arquitetura
- linguagem
- framework
- padrão de pastas
- comandos de build/teste
- regras de implementação
- regras do que evitar
- padrão de validação
```

## O que não colocar nele?

Não coloque tarefa pontual.

Errado:

```md
# AGENTS.md

Implemente cadastro de responsável.
```

Certo:

```md
# AGENTS.md

Controllers não devem conter regra de negócio.
Use cases devem depender de portas.
Domínio não deve depender de Spring ou JPA.
```

## Template completo

```md
# AGENTS.md

## Projeto
Este projeto usa Java, Spring Boot, PostgreSQL e arquitetura hexagonal.

## Objetivo das regras
Estas regras orientam agentes de IA a trabalhar neste projeto com segurança, mantendo padrões de arquitetura, código e validação.

## Arquitetura

### Domain
- Não deve depender de Spring, JPA, HTTP, banco de dados ou frameworks externos.
- Deve conter entidades, value objects, regras de negócio e comportamentos do domínio.

### Application
- Deve conter casos de uso e portas.
- Casos de uso devem depender de abstrações, não de adapters concretos.
- Portas de entrada representam ações disponíveis no sistema.
- Portas de saída representam dependências externas necessárias.

### Infrastructure
- Deve conter controllers, repositories, mappers, entidades JPA, clients externos e configurações.
- Adapters de entrada recebem requisições externas e chamam portas de entrada.
- Adapters de saída implementam portas de saída.

## Regras de implementação
- Antes de alterar código, leia os arquivos relacionados.
- Siga padrões já existentes no projeto.
- Não crie abstrações genéricas sem necessidade.
- Não altere contratos públicos sem explicar impacto.
- Não remova testes existentes para fazer build passar.
- Não ignore erros de compilação.

## Validação
Antes de finalizar uma tarefa:
- Explique os arquivos alterados.
- Informe os comandos executados.
- Informe testes que passaram ou falharam.
- Mostre riscos ou pendências.

## Comandos úteis
- Build: ./mvnw clean package
- Testes: ./mvnw test
- Teste específico: ./mvnw -Dtest=NomeDoTeste test

## Fluxo recomendado
1. Entender a demanda.
2. Gerar spec.md.
3. Gerar plan.md.
4. Gerar tasks.md.
5. Implementar uma task por vez.
6. Validar.
7. Revisar diff.
```

---

# Melhorando prompts de planejamento

## Prompt ruim

```text
Crie o cadastro de responsável.
```

Esse prompt é ruim porque não define:

```text
- escopo
- regra de negócio
- critérios de aceite
- arquivos impactados
- padrão esperado
- validação
```

## Prompt bom

```text
/plan

Preciso implementar o cadastro de responsável.

Contexto:
- O projeto usa Java, Spring Boot e arquitetura hexagonal.
- Quero seguir os padrões existentes.
- O e-mail do responsável deve ser único.
- A senha deve ser obrigatória.
- O cadastro deve marcar o responsável como ativo.

Antes de codar:
1. Leia o AGENTS.md.
2. Analise a estrutura atual.
3. Identifique arquivos semelhantes.
4. Gere spec.md.
5. Gere plan.md.
6. Gere tasks.md.
7. Liste dúvidas abertas.
8. Não altere arquivos ainda.
```

---

# Usando Devin task por task

Depois que o Devin gerar o plano, você implementa uma tarefa por vez.

```text
/accept-edits

Implemente somente a Task 1 de docs/sdd/tasks.md.

Regras:
1. Siga o AGENTS.md.
2. Não avance para outras tasks.
3. Altere apenas arquivos necessários.
4. Ao final, mostre resumo do diff.
5. Informe como validar.
6. Peça permissão antes de executar comandos.
```

Depois revise:

```text
/normal

Revise a implementação da Task 1.

Verifique:
1. Se seguiu o AGENTS.md.
2. Se ficou dentro do escopo.
3. Se há risco de bug.
4. Se faltou teste.
5. Se posso seguir para a Task 2.
```

## Checklist da Semana 2

```text
[ ] Criei AGENTS.md
[ ] Documentei arquitetura do projeto
[ ] Documentei comandos de build e teste
[ ] Documentei o que o Devin não deve fazer
[ ] Melhorei meus prompts de planejamento
[ ] Usei Devin para implementar task por task
```

---

# Semana 3 — Skills

## Objetivo da semana

Criar comandos reutilizáveis para tarefas que você faz repetidamente.

Skills são unidades reutilizáveis que agrupam instruções, ferramentas, permissões e um fluxo de trabalho. No Devin CLI, elas podem ser chamadas como comandos personalizados. ([Devin][1])

---

# Arquivo da Semana 3: `.devin/skills/*/SKILL.md`

## O que é?

Um `SKILL.md` define uma habilidade reutilizável para o Devin.

Pense em uma Skill como um comando personalizado:

```text
/review-diff
/generate-tests
/architecture-review
```

## Para que serve?

Serve para transformar prompts repetidos em comandos reutilizáveis.

Em vez de escrever toda hora:

```text
Revise o diff atual procurando bug, falta de teste, problema de arquitetura e alteração fora do escopo.
```

Você cria:

```text
/review-diff
```

## Quando criar uma Skill?

Crie uma Skill quando a tarefa se repete.

Bons exemplos:

```text
- review-diff
- generate-tests
- architecture-review
- security-review
- prepare-pr-summary
```

Exemplos ruins:

```text
- criar-cadastro-responsavel
- corrigir-bug-do-dia-15
- alterar-nome-da-classe-x
```

Esses exemplos ruins são tarefas pontuais, não habilidades reutilizáveis.

---

# Skill 1 — `review-diff`

## O que ela faz?

Revisa o diff atual procurando:

```text
- bugs
- alterações fora do escopo
- falta de testes
- problemas de segurança
- quebra de arquitetura
- inconsistência com padrões do projeto
```

## Quando usar?

Depois que o Devin implementar uma task.

Fluxo:

```text
1. Devin implementa Task 1
2. Você chama /review-diff
3. Devin revisa o que foi alterado
4. Você corrige problemas antes de continuar
```

## Arquivo

```text
.devin/skills/review-diff/SKILL.md
```

## Conteúdo

```md
---
name: review-diff
description: Revisa o diff atual procurando bugs, problemas de arquitetura, segurança e falta de testes
allowed-tools:
  - read
  - grep
  - glob
  - exec
triggers:
  - user
permissions:
  allow:
    - Exec(git status)
    - Exec(git diff)
    - Exec(git diff --staged)
  deny:
    - write
    - edit
---

Revise o diff atual do projeto.

Passos:
1. Rode `git status`.
2. Rode `git diff --staged`.
3. Se não houver staged changes, rode `git diff`.
4. Analise:
   - bugs de lógica
   - quebra de arquitetura
   - falta de testes
   - problemas de segurança
   - inconsistência com padrões do projeto
   - alterações fora do escopo

Responda com:
- resumo geral
- problemas encontrados
- gravidade de cada problema
- sugestão objetiva de correção
- arquivos impactados
- recomendação: aprovar, ajustar ou bloquear
```

## Explicando o arquivo

```yaml
name: review-diff
```

É o nome da Skill. Normalmente será o comando que você chama.

```yaml
description: Revisa o diff atual...
```

Ajuda o Devin a entender quando essa Skill é útil.

```yaml
allowed-tools:
  - read
  - grep
  - glob
  - exec
```

Define quais ferramentas a Skill pode usar.

```yaml
permissions:
  allow:
    - Exec(git status)
    - Exec(git diff)
  deny:
    - write
    - edit
```

Permite leitura e comandos de Git, mas impede edição de arquivos.

---

# Skill 2 — `generate-tests`

## O que ela faz?

Gera ou melhora testes para a alteração atual.

Ela deve:

```text
- analisar o diff
- encontrar testes parecidos
- identificar cenários faltantes
- criar testes seguindo o padrão do projeto
- evitar mexer em produção sem necessidade
```

## Quando usar?

Depois de implementar código de produção.

Fluxo:

```text
1. Implementa a feature
2. Chama /generate-tests
3. Devin identifica lacunas
4. Devin cria ou melhora testes
5. Você roda os testes
```

## Arquivo

```text
.devin/skills/generate-tests/SKILL.md
```

## Conteúdo

```md
---
name: generate-tests
description: Gera ou melhora testes para a alteração atual seguindo os padrões existentes do projeto
allowed-tools:
  - read
  - grep
  - glob
  - edit
  - write
  - exec
triggers:
  - user
permissions:
  allow:
    - Exec(git status)
    - Exec(git diff)
    - Exec(./mvnw test)
    - Exec(mvn test)
  ask:
    - write
    - edit
---

Gere ou melhore testes para a alteração atual.

Passos:
1. Analise o diff atual.
2. Encontre testes semelhantes no projeto.
3. Identifique cenários não cobertos.
4. Crie ou ajuste testes seguindo o padrão existente.
5. Priorize testes de unidade para regras de negócio.
6. Use testes de integração apenas quando necessário.
7. Não altere código de produção sem explicar o motivo.

Ao final:
- liste os testes criados ou alterados
- explique os cenários cobertos
- informe comandos de teste recomendados
- informe riscos ou lacunas restantes
```

## Por que ela é importante?

IA costuma implementar o caminho feliz e esquecer cenários de erro.

Essa Skill força o Devin a pensar em:

```text
- campo obrigatório
- e-mail duplicado
- exceções
- regra de negócio
- retorno esperado
```

---

# Skill 3 — `architecture-review`

## O que ela faz?

Revisa se a alteração respeita a arquitetura do projeto.

No seu caso, é muito útil para arquitetura hexagonal.

Ela verifica:

```text
- domínio sem Spring/JPA
- controller sem regra de negócio
- use case dependendo de portas
- infraestrutura implementando adapters
- mappers no lugar correto
- dependências corretas entre camadas
```

## Quando usar?

Antes de considerar uma task concluída.

Fluxo:

```text
1. Implementa Task 1
2. Chama /review-diff
3. Chama /architecture-review
4. Corrige problemas
5. Só então segue para Task 2
```

## Arquivo

```text
.devin/skills/architecture-review/SKILL.md
```

## Conteúdo

```md
---
name: architecture-review
description: Revisa se a alteração respeita arquitetura hexagonal, separação de camadas e dependências corretas
allowed-tools:
  - read
  - grep
  - glob
  - exec
triggers:
  - user
permissions:
  allow:
    - Exec(git status)
    - Exec(git diff)
    - Exec(git diff --staged)
  deny:
    - write
    - edit
---

Revise a arquitetura da alteração atual.

Considere as regras:
1. Domain não deve depender de Spring, JPA, HTTP ou frameworks externos.
2. Application deve depender de portas e domínio.
3. Infrastructure pode depender de frameworks e implementar adapters.
4. Controllers não devem conter regra de negócio.
5. Repositories JPA não devem vazar para o domínio.
6. Mappers devem ficar na infraestrutura.
7. Use cases devem orquestrar o fluxo sem conter detalhes técnicos de banco ou HTTP.

Passos:
1. Rode `git status`.
2. Analise o diff.
3. Identifique arquivos alterados por camada.
4. Verifique dependências indevidas.
5. Verifique se nomes e pacotes estão coerentes.

Responda com:
- status geral da arquitetura
- violações encontradas
- explicação do impacto
- sugestões de correção
- severidade: baixa, média ou alta
```

## Checklist da Semana 3

```text
[ ] Criei review-diff
[ ] Criei generate-tests
[ ] Criei architecture-review
[ ] Usei /review-diff em uma alteração real
[ ] Usei /generate-tests em uma feature real
[ ] Usei /architecture-review antes de finalizar uma tarefa
```

---

# Semana 4 — Subagents

## Objetivo da semana

Aprender a delegar tarefas para agentes especializados.

Subagents são agentes auxiliares que podem executar subtarefas com um contexto próprio. Eles são úteis para pesquisa, revisão, análise de impacto e outras tarefas que você quer separar do agente principal. ([Devin][1])

---

# Arquivo da Semana 4: `.devin/agents/*/AGENT.md`

## O que é?

Um `AGENT.md` dentro de `.devin/agents/` define um subagent especializado.

Pense nele como uma pessoa especialista dentro do time:

```text
- reviewer
- researcher
- tester
- security-reviewer
```

## Diferença entre Skill e Subagent

| Conceito | O que é                      | Exemplo        |
| -------- | ---------------------------- | -------------- |
| Skill    | Um procedimento reutilizável | `/review-diff` |
| Subagent | Um agente especializado      | `reviewer`     |

Uma Skill é como um comando.

Um subagent é como um especialista.

---

# Subagent 1 — `reviewer`

## O que ele faz?

Revisa código sem alterar arquivos.

Ele procura:

```text
- bugs
- problemas de arquitetura
- problemas de segurança
- falta de testes
- regressões prováveis
- alterações fora do escopo
```

## Quando usar?

Depois de uma implementação feita pelo agente principal.

## Arquivo

```text
.devin/agents/reviewer/AGENT.md
```

## Conteúdo

```md
---
name: reviewer
description: Revisa alterações de código procurando bugs, problemas de arquitetura, segurança e falta de testes
model: sonnet
allowed-tools:
  - read
  - grep
  - glob
  - exec
permissions:
  allow:
    - Exec(git status)
    - Exec(git diff)
    - Exec(git diff --staged)
  deny:
    - write
    - edit
---

Você é um subagent especialista em code review.

Seu papel é revisar alterações sem modificar arquivos.

Analise:
1. Bugs de lógica
2. Problemas de arquitetura
3. Problemas de segurança
4. Falta de testes
5. Regressões prováveis
6. Alterações fora do escopo

Sempre responda com:
- resumo
- problemas encontrados
- severidade
- evidências em arquivos
- sugestão objetiva de correção
```

## Prompt de uso

```text
Use o subagent reviewer para revisar o diff atual.
Não altere arquivos.
```

---

# Subagent 2 — `researcher`

## O que ele faz?

Pesquisa a codebase antes da implementação.

Ele deve descobrir:

```text
- onde começa determinado fluxo
- quais classes participam
- quais padrões existentes devem ser copiados
- quais arquivos serão impactados
- quais riscos existem
```

## Quando usar?

Antes de implementar algo em uma área que você não conhece bem.

## Arquivo

```text
.devin/agents/researcher/AGENT.md
```

## Conteúdo

```md
---
name: researcher
description: Pesquisa a codebase para entender fluxos, dependências, padrões existentes e impacto de mudanças
model: sonnet
allowed-tools:
  - read
  - grep
  - glob
permissions:
  deny:
    - write
    - edit
    - exec
---

Você é um subagent pesquisador de codebase.

Seu papel é entender o sistema antes da implementação.

Pesquise:
1. Onde determinado fluxo começa.
2. Quais classes participam.
3. Quais padrões existentes devem ser copiados.
4. Quais arquivos provavelmente serão impactados.
5. Quais riscos existem.

Não altere arquivos.
Não execute comandos.
Sempre cite caminhos de arquivos relevantes.
```

## Prompt de uso

```text
Use o subagent researcher para descobrir como o fluxo de cadastro funciona neste projeto.
Não altere arquivos.
```

---

# Usando Skills com Subagents

Você pode ter uma Skill que delega a revisão para um subagent.

Exemplo:

```md
---
name: review-diff-subagent
description: Revisa o diff atual usando o subagent reviewer
agent: reviewer
triggers:
  - user
---

Revise o diff atual.

Verifique:
1. Bugs
2. Segurança
3. Arquitetura
4. Testes
5. Regressões

Não altere arquivos.
```

Uso:

```text
/review-diff-subagent
```

## Quando isso faz sentido?

Use Skill normal quando a tarefa é simples.

Use subagent quando você quer separar contexto ou especialização.

```text
Skill normal:
- revisão simples
- geração de teste
- resumo de diff

Subagent:
- pesquisa mais profunda
- revisão isolada
- análise em paralelo
- investigação de fluxo complexo
```

## Checklist da Semana 4

```text
[ ] Criei subagent reviewer
[ ] Criei subagent researcher
[ ] Usei researcher para entender um fluxo
[ ] Usei reviewer para revisar um diff
[ ] Entendi diferença entre Skill e Subagent
```

---

# Semana 5 — Permissions + Hooks

## Objetivo da semana

Controlar o que o Devin pode fazer automaticamente e proteger o projeto contra ações perigosas.

O sistema de permissões permite configurar regras como `allow`, `ask` e `deny`, controlando quais ações podem ser executadas automaticamente, quais precisam de aprovação e quais devem ser bloqueadas. ([Devin][3])

---

# Arquivo da Semana 5: `.devin/config.json`

## O que é?

O `.devin/config.json` é o arquivo de configuração do Devin CLI para o projeto.

Ele pode controlar:

```text
- permissões
- MCP servers
- hooks
- configurações específicas do projeto
```

## Para que serve?

Serve para definir o que o Devin pode fazer sozinho, o que precisa perguntar e o que nunca pode fazer.

## Como pensar nele?

```text
allow = pode fazer sozinho
ask   = precisa pedir permissão
deny  = nunca pode fazer
```

## Exemplo

```json
{
  "permissions": {
    "allow": [
      "Read(**)",
      "Exec(git status)",
      "Exec(git diff)",
      "Exec(git diff --staged)",
      "Exec(git log)"
    ],
    "ask": [
      "Write(**)",
      "Edit(**)",
      "Exec(./mvnw test)",
      "Exec(mvn test)",
      "Exec(npm test)",
      "Exec(npm run lint)"
    ],
    "deny": [
      "Exec(rm -rf **)",
      "Exec(sudo **)",
      "Exec(git push **)",
      "Exec(git reset --hard **)",
      "Exec(git clean -fd **)",
      "Exec(docker system prune **)"
    ]
  }
}
```

## Política recomendada

Permitir automaticamente:

```text
- leitura de arquivos
- git status
- git diff
- git log
```

Pedir permissão:

```text
- editar arquivos
- criar arquivos
- rodar testes
- rodar build
```

Bloquear:

```text
- rm -rf
- sudo
- git push
- git reset --hard
- comandos destrutivos
```

---

# Arquivo da Semana 5: `.devin/hooks.v1.json`

## O que é?

O `hooks.v1.json` configura hooks do Devin CLI.

Hook é uma ação automática disparada em determinado momento.

Exemplo:

```text
Antes de executar um comando shell, rode este script de validação.
```

## Para que serve?

Serve para criar proteções automáticas.

Mesmo que uma permissão esteja mal configurada, o hook pode bloquear algo perigoso.

## Como pensar nele?

Pense no hook como um porteiro automático.

```text
Antes do Devin executar algo, o hook verifica se aquilo é permitido.
```

## Exemplo

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

Esse arquivo diz:

```text
Antes de usar a ferramenta exec,
rode o script ./scripts/check-command.sh.
```

---

# Arquivo da Semana 5: `scripts/check-command.sh`

## O que é?

É um script chamado pelo hook para verificar comandos perigosos.

## Para que serve?

Serve para bloquear comandos como:

```text
rm -rf
git push
git reset --hard
sudo
docker system prune
DROP DATABASE
TRUNCATE TABLE
```

## Como pensar nele?

É uma camada extra de segurança.

Mesmo que o Devin tente executar algo perigoso, esse script pode impedir.

## Conteúdo

```bash
#!/usr/bin/env bash

set -euo pipefail

input="$(cat)"

blocked_patterns=(
  "rm -rf"
  "sudo"
  "git push"
  "git reset --hard"
  "git clean -fd"
  "docker system prune"
  "DROP DATABASE"
  "TRUNCATE TABLE"
)

for pattern in "${blocked_patterns[@]}"; do
  if echo "$input" | grep -qi "$pattern"; then
    echo "Blocked dangerous command: $pattern"
    exit 1
  fi
done

exit 0
```

Depois, no Linux/macOS/Git Bash:

```bash
chmod +x scripts/check-command.sh
```

## Checklist da Semana 5

```text
[ ] Criei .devin/config.json
[ ] Configurei allow, ask e deny
[ ] Bloqueei comandos perigosos
[ ] Criei .devin/hooks.v1.json
[ ] Criei scripts/check-command.sh
[ ] Testei se comandos perigosos são bloqueados
```

---

# Semana 6 — MCP

## Objetivo da semana

Conectar o Devin CLI a ferramentas externas.

MCP, ou **Model Context Protocol**, permite conectar o Devin a ferramentas e serviços externos. Depois de configuradas, essas ferramentas podem ser usadas pelo agente como capacidades adicionais. ([Devin][1])

---

# MCP dentro do `.devin/config.json`

## O que é?

MCP é uma ponte entre o Devin e outros sistemas.

Exemplos:

```text
- GitHub
- Jira
- Linear
- Notion
- documentação interna
- APIs internas
```

## Para que serve?

Serve para o Devin acessar informações que não estão no código local.

Sem MCP:

```text
Devin entende o código local e o que você escreve no prompt.
```

Com MCP:

```text
Devin pode buscar contexto em ferramentas externas.
```

## Cuidado

MCP aumenta o poder do agente.

Por isso, no começo use:

```text
- permissões restritas
- tokens com escopo mínimo
- acesso somente leitura
- hooks de segurança
```

---

# Exemplo: GitHub MCP

## Arquivo

```text
.devin/config.json
```

## Exemplo conceitual

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_seu_token_aqui"
      }
    }
  },
  "permissions": {
    "allow": [
      "Read(**)",
      "Exec(git status)",
      "Exec(git diff)",
      "mcp__github__search_repositories",
      "mcp__github__get_file_contents"
    ],
    "ask": [
      "mcp__github__create_issue",
      "mcp__github__create_pull_request"
    ],
    "deny": [
      "mcp__github__delete_repo"
    ]
  }
}
```

## Como pensar nessa configuração?

```text
mcpServers = quais ferramentas externas existem
permissions = o que Devin pode fazer com elas
```

Permita primeiro só leitura.

Depois, com maturidade, libere ações de escrita com `ask`.

## Prompt seguro para usar MCP

```text
Use o MCP do GitHub apenas para ler issues relacionadas a esta tarefa.

Faça:
1. Busque issues abertas com o termo "responsavel".
2. Resuma o problema.
3. Relacione com os arquivos locais.
4. Não crie issue.
5. Não altere PR.
6. Não execute ações de escrita no GitHub.
```

## Checklist da Semana 6

```text
[ ] Entendi o que é MCP
[ ] Configurei um MCP simples
[ ] Controlei MCP com permissions
[ ] Bloqueei ações perigosas
[ ] Usei MCP apenas para leitura no primeiro teste
[ ] Integrei MCP ao fluxo SDD
```

---

# Fluxo completo para uma feature real

Use este fluxo no dia a dia:

```text
1. Abrir Devin CLI no projeto
2. Entrar em /normal
3. Usar /ask para dúvidas
4. Entrar em /plan
5. Gerar spec.md
6. Gerar plan.md
7. Gerar tasks.md
8. Revisar manualmente
9. Entrar em /accept-edits
10. Implementar Task 1
11. Rodar testes
12. Usar /review-diff
13. Usar /architecture-review
14. Ajustar problemas
15. Repetir para próxima task
16. Finalizar com resumo e validação
```

---

# Prompt mestre para qualquer feature

```text
/plan

Quero implementar a seguinte demanda usando SDD local:

[descreva a demanda]

Antes de alterar qualquer arquivo:
1. Leia o AGENTS.md.
2. Analise a estrutura do projeto.
3. Identifique padrões existentes.
4. Encontre arquivos semelhantes.
5. Gere ou atualize docs/sdd/spec.md com:
   - objetivo
   - contexto
   - escopo
   - fora de escopo
   - regras de negócio
   - critérios de aceite
   - dúvidas abertas
6. Gere ou atualize docs/sdd/plan.md com:
   - estratégia técnica
   - arquivos impactados
   - alterações por camada
   - riscos
   - testes necessários
   - plano de validação
7. Gere ou atualize docs/sdd/tasks.md com:
   - tasks pequenas
   - objetivo
   - arquivos envolvidos
   - validação esperada
8. Não implemente nada ainda.
```

---

# Prompt para implementar uma task

```text
/accept-edits

Implemente somente a Task [número] do arquivo docs/sdd/tasks.md.

Regras:
1. Siga o AGENTS.md.
2. Não avance para outras tasks.
3. Altere apenas arquivos necessários.
4. Preserve padrões existentes.
5. Não remova testes.
6. Não mude contratos públicos sem explicar.
7. Ao final, mostre:
   - arquivos alterados
   - resumo do diff
   - comandos recomendados para validação
   - riscos ou pendências
```

---

# Prompt para revisão final

```text
/review-diff
```

Depois:

```text
/architecture-review
```

Ou manualmente:

```text
/normal

Revise a alteração atual.

Verifique:
1. Bugs de lógica
2. Quebra de arquitetura
3. Falta de testes
4. Segurança
5. Alterações fora do escopo
6. Se os critérios de aceite da spec foram atendidos

Não altere arquivos.
```

---

# Ordem de maturidade

Não tente usar tudo de uma vez.

Siga esta ordem:

```text
Nível 1:
- /ask
- /plan
- /normal
- /accept-edits

Nível 2:
- spec.md
- plan.md
- tasks.md
- AGENTS.md

Nível 3:
- Skills

Nível 4:
- Subagents

Nível 5:
- Permissions
- Hooks

Nível 6:
- MCP
```

---

# Resumo mental final

```text
AGENTS.md        = regras permanentes do projeto
spec.md          = o que será feito
plan.md          = como será feito
tasks.md         = sequência de execução
SKILL.md         = comando reutilizável
AGENT.md         = especialista delegado
config.json      = permissões e integrações
hooks.v1.json    = automações de segurança
check-command.sh = bloqueio de comandos perigosos
MCP              = acesso a ferramentas externas
```

O ponto central do estudo é este:

```text
Não use Devin como digitador de código.

Use Devin como engenheiro assistente:
ele entende, planeja, executa, valida e revisa.
```

Para começar de forma prática, implemente apenas estes três itens no seu projeto atual:

```text
1. AGENTS.md
2. docs/sdd/spec.md, plan.md e tasks.md
3. .devin/skills/review-diff/SKILL.md
```

Depois disso, avance para:

```text
4. generate-tests
5. architecture-review
6. subagents
7. permissions
8. hooks
9. MCP
```

[1]: https://cli.devin.ai/docs/essential-commands?utm_source=chatgpt.com "Essential Commands - Devin for Terminal"
[2]: https://cli.devin.ai/docs/reference/commands?utm_source=chatgpt.com "Commands & Flags - Devin for Terminal"
[3]: https://cli.devin.ai/docs/reference/permissions?utm_source=chatgpt.com "Permissions - Devin for Terminal"
