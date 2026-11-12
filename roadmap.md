# Roadmap pessoal — Imersão IA for Devs (Full Cycle)

> Extraído das transcrições brutas de duas aulas da Imersão IA for Devs (via `dtranscript`), para
> eu implementar neste starter repo com apoio de IA. **Cobre só as aulas 1 e 2 — tem mais aulas
> vindo.** Este arquivo é meu, separado do `docs/roadmap.md` do projeto (que é do produto, por
> funcionalidade). Procedimento para manter isto atualizado no final do arquivo.

## Aula 1 — "Como as grandes empresas estão trabalhando com IA" (palestra, com Daniel Ciolfi, Mercado Livre)

Esta aula foi conceitual (sem mão na massa no repo). São princípios de arquitetura de agentes de
IA em produção, relevantes principalmente para quando eu chegar nas funcionalidades F10–F14
(serviço de agentes). Boa parte já está refletida em ADRs existentes (`0012`–`0015`); o resto são
lacunas a endereçar quando a aula 3 (desenvolvimento do agente) chegar.

- [ ] **Guardrails de entrada e saída no agente**, não só no prompt: validar o formato *e* o
      conteúdo do que a LLM retorna antes de confiar (ex.: produto retornado existe no banco?),
      e ter uma camada barata (regra ou LLM pequena) filtrando a entrada antes de chegar na LLM
      cara — mitiga prompt injection e gasto por flood.
- [ ] **Escopo de ferramentas por usuário autenticado**, nunca por parâmetro livre: a ferramenta
      do agente deve ser "buscar dados do usuário atual" (via ID da sessão), não "buscar usuário
      por nome" — evita que alguém se passe por outra pessoa pedindo para o agente.
- [ ] **Observabilidade dedicada a agente**: logs, tracing e métricas cobrindo quais ferramentas
      foram chamadas, quantas vezes, o que cada uma retornou — não só latência/erro HTTP comuns.
      Sem isso não dá para reproduzir um bug não determinístico.
- [ ] **Limite de custo** (rate limit e teto de gasto em dólar por dia/hora) no serviço de
      agentes, para não tomar um susto de fatura (caso real citado: US$100 num fim de semana por
      um agente conversando sozinho com a LLM de um fornecedor).
- [ ] **Human-in-the-loop só onde a ação é irreversível**; preferir dar ao agente ações reversíveis
      (um bom "Ctrl Z") a ficar pedindo aprovação toda hora — pedir aprovação demais gera
      "complacência de automação" (quem aprova passa a clicar sem olhar, piorando o sistema).
- [ ] **Durabilidade**: o agente precisa suportar ser interrompido no meio (queda de servidor) e
      retomar de onde parou — relevante para a Tarefa delegada (F12, que já é assíncrona por
      natureza) e para o Relatório em PDF (F14, processamento longo).
- [ ] **Fallback de modelo testado**, não só configurado: se o provedor cai, o agente trocar de
      LLM não é transparente (prompts/ferramentas calibrados para um modelo respondem diferente
      em outro) — isso precisa ser parte da suíte de testes do agente, igual ao resto do código.
- [ ] **Avaliação do agente não é `assert` de igualdade**: usar um segundo agente/LLM como juiz
      para dar nota/score à resposta, porque a mesma entrada pode gerar respostas ligeiramente
      diferentes a cada execução.
- [ ] **Redação (redact) de dados sensíveis antes de mandar para a LLM**, por LGPD — nunca mandar
      PII crua no prompt.
- [ ] Ao planejar sequências de passos do agente, lembrar que a acurácia cai de forma exponencial
      por passo (ex.: 95% por passo ≈ 60% em 10 passos) — preferir menos passos por tarefa.

## Aula 2 — "Desenvolvendo o Slack com spec-driven no back e no front" (vídeo 2 da imersão)

Esta é a aula prática que se aplica direto a este repo. O repo, no estado atual, já tem feito o
que a aula mostra até o início da implementação da F01: Figma conectado (`figma` plugin habilitado
em `.claude/settings.json`), `docs/design-system.md` criado, `docs/roadmap.md` com as 14
funcionalidades e o mapa de dependências, `GLOSSARY.md`, e a F01 (login e sessão) já com PRD,
spec e as 9 tarefas verticais escritas — só a tarefa 01 (base da API) está **concluída e
implementada**; as tarefas 02 a 09 estão **a fazer**, sem plano ainda. O trabalho que falta é
majoritariamente de execução do workflow, não de planejamento do zero.

### Workflow a repetir em cada tarefa (já configurado nas skills do projeto)

1. `/plan-task` da tarefa → gera `plano.md` com critérios de aceite, contratos e arquivos
   afetados. Revisar o plano antes de seguir.
2. Commit do plano sozinho ("commit de plano") — só ele é commitado, antes de qualquer código.
3. `/implement` da tarefa → invoca a skill de TDD (red-green: teste falha, depois implementa),
   depois `simplify` (agentes revisando complexidade) e `code-review` (sensor inferencial — quem
   julga não deve ser a mesma sessão/modelo de quem implementou).
4. Rodar a "Verificação completa" de cada app (lint, tipagem, testes unitário/integração/E2E —
   já definida em `apps/web/AGENTS.md` e `apps/chat-api/AGENTS.md`).
5. Commit, push, abrir PR. Pedir revisão de um agente diferente do que implementou (ex.: GitHub
   Copilot na PR, ou Claude com outro modelo numa sessão limpa) — nunca a mesma sessão que
   implementou, porque ela já está "viciada" no que acabou de escrever.
6. Resolver os comentários da revisão, commitar a correção.
7. **Merge só com minha autorização explícita** (convenção do `AGENTS.md` do projeto — não pular
   essa etapa mesmo trabalhando sozinho).
8. Sempre que uma decisão nova surgir durante o plano ou a implementação (formato de token, regra
   de bloqueio, Server Actions vs. API direta, etc.), atualizar `docs/lld.md`, `GLOSSARY.md` e
   gerar um ADR em `docs/adr/` **no mesmo momento** — documentos são código vivo, não um registro
   para depois.

### F01 — Login e sessão (terminar)

- [ ] Tarefa 02 — Base da web e dos testes E2E (plano → implementar → revisar → PR)
- [ ] Tarefa 03 — Entrar e ver a home
- [ ] Tarefa 04 — Erros de preenchimento e falha no envio
- [ ] Tarefa 05 — Voltar para onde ia
- [ ] Tarefa 06 — Continuar logada e sessão que acaba
- [ ] Tarefa 07 — Sair
- [ ] Tarefa 08 — Limite de tentativas de login
- [ ] Tarefa 09 — Falha ao verificar a sessão
- [ ] Rodar o Lighthouse (aba Accessibility, no DevTools) nas telas novas — meta é 100, como no
      vídeo.
- [ ] Marcar F01 como "concluída" no `docs/roadmap.md` só depois do PR da última tarefa.

### F02 — Mensagens em canal público (começar do zero)

- [ ] `/prd` da F02 (entrevista guiada pela skill; aceitar as recomendações onde a decisão é
      só técnica).
- [ ] `/to-spec` (ou a skill de grill equivalente) para a especificação técnica, deixando ela
      "envelhecer devagar" (sem citar arquivos, só decisões de alto nível).
- [ ] `/to-tasks` para quebrar em **fatias verticais** (vertical slices) — cada tarefa atravessa
      banco → API → tela, não uma camada só. Revisar a quebra proposta antes de aprovar: se vier
      muito horizontal (uma tarefa só de banco, outra só de endpoint), pedir para re-quebrar.
- [ ] Guardar a informação de "ondas" de execução (quais tarefas podem rodar em paralelo) — não
      precisa agir nisso ainda, mas documentar para não perder depois.
- [ ] Repetir o workflow de tarefa (plano → implementar com TDD → simplify/code-review → PR) para
      cada tarefa da F02.

### F03 — Tempo real via WebSocket (começar do zero)

- [ ] Mesmo fluxo PRD → spec → tasks → implementação da F02.
- [ ] Validar especificamente: handshake de autenticação do socket (token da sessão), join na
      sala/canal, broadcast de mensagem nova sem precisar recarregar a página (F5).
- [ ] Testar reconexão depois de queda (parte do critério de "tempo real" do PRD, já que o
      produto promete "não perder nada depois de uma queda de conexão").

### Meta da aula 2: F01 a F09 prontas antes da aula 3

- [ ] A aula 3 (desenvolvimento do agente) assume o chat funcionando ponta a ponta até pelo menos
      a F09, porque o agente entra na conversa como bot usando essa base. Priorizar terminar F01
      → F02 → F03 antes de seguir para F04–F09 na ordem do mapa de dependências do
      `docs/roadmap.md`.

### Lacunas que a própria aula apontou (não ignorar)

- [ ] **Falta uma skill/guia de testes de unidade.** A aula gerou testes E2E e de integração
      (via TDD), mas nenhum teste de unidade — não porque a IA errou, mas porque ninguém orientou
      isso no harness. Vale decidir, por arquivo, o que merece teste de unidade antes de pedir
      implementação, para manter a pirâmide de testes (poucos E2E no topo, mais unidade na base).
- [ ] **Execução paralela de tarefas (git worktrees + múltiplas sessões) foi só mencionada**, não
      implementada — é um próximo nível de maturidade, não bloqueia o MVP. Não vale a pena
      configurar agora; revisitar só depois que F01–F09 estiverem com o ritmo de revisão estável.

## Como manter este arquivo atualizado (procedimento a repetir a cada aula nova)

1. Gravar a aula com o `dtranscript` (`../dtranscript`): rodar `python app.py`, colar a URL do
   YouTube, escolher modo **AI summary**, template, e marcar a opção **"Append full transcript to
   summary"** — sem isso a transcrição bruta não fica salva no `.md` e eu não consigo extrair as
   tarefas depois.
2. O arquivo sai em `../dtranscript/output/AAAA-MM-DD_<titulo>_<id>_summary.md`. A parte de resumo
   é só para eu ler depois; a seção `## Transcript` no fim (dentro do `<details>`) é a bruta.
3. Me pedir (nesta sessão ou numa nova) para ler a transcrição bruta do novo arquivo e extrair as
   tarefas, do mesmo jeito que foi feito para as aulas 1 e 2 — isto é, comissionar a IA para ler e
   adicionar a seção da aula nova neste `roadmap.md`, sem reescrever as seções já existentes.
