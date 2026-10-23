# BOSSFYME — Contexto do projeto

> Arquivo de contexto permanente. Leia antes de qualquer tarefa de código ou design.
> Fonte da verdade: este arquivo. Itens marcados **(a validar)** ainda não são decisão final.

## O produto
App de produtividade com personalidade de **chefe caricato**: autoritário, seco, irônico, exigente, mas justo. Nunca ofensivo ou humilhante. Ele não é amigo do usuário: organiza tarefas com prazo e cobra a execução, **uma tarefa por vez**. O pomodoro é um recurso de execução, não o centro do app.

- Mascote: **Dr. Alberto**.
- Idioma da interface: português do Brasil.
- Plataforma do MVP: celular (Expo). As telas antigas no Figma são desktop. **(a validar)**
- Referência de fluxo: WERKFY (disponível → em andamento ⇄ pausado → concluído; pausa com motivo; servidor fecha/avisa sozinho).
- Referência visual: arquivo Figma "BossFyme" (Modo Pressão, Central de Desculpas, Café Expresso, Almoço Relâmpago). O tom caricato do Figma está aprovado e deve ser expandido para todas as telas.

## Três momentos
1. **Planejar (modo chefe):** cria tarefas com prazo, estimativa e tolerância de atraso. Assinar o plano trava as tarefas.
2. **Executar (modo funcionário):** uma tarefa por vez, escolhida pelo usuário. Pausa sempre com motivo e limite de tempo.
3. **Prestar contas:** fechamento do dia (Bater o ponto) e relatório (Dossiê).

## Telas e navegação
Barra inferior: **Prancheta · Modo Pressão · Cobranças (com contador) · Dossiê · Regulamento**.

- **Prancheta do Chefe** — planejamento. Cria tarefas e "assina o plano". Barra de capacidade (soma das estimativas x horas do expediente).
- **Modo Pressão** — execução. Fila travada, tela de foco, relógio, pomodoro como camada. **Café Expresso** (pausa curta) e **Almoço Relâmpago** (pausa longa) são estados de pausa dentro dele, não páginas.
- **Central de Desculpas** (aba Cobranças) — o usuário explica por que não cumpriu; o chefe dá o veredito. Não gera desculpas prontas.
- **Dossiê de Produtividade** — relatório do dia e da semana.
- **Regulamento** — configurações.
- **Bater o ponto** — momento do dia, não aba. Aberto por notificação ou botão.

**Regra de abertura do app:** primeiro uso → Primeiro uso; tarefa em execução ou pausada → Modo Pressão com a tarefa; expediente encerrado e ponto não batido → Bater o ponto; fila assinada → Modo Pressão para escolher; sem plano → Prancheta. **Notificação ignora a regra e abre direto no assunto.**

## User flows
0 Abertura do app · 1 Planejar e assinar o plano · 2A Começar uma tarefa · 2B Durante o foco · 3 Justificar · 4 Bater o ponto · 5 Primeiro uso.

- Página dos fluxos: https://claude.ai/artifact/1Pq6awYeJBU3m7tapsAqF4
- FigJam: https://www.figma.com/board/x4uBysh3LEROfGneQCVe3B (fluxos 0 e 5 revisados; 2A, 2B, 3 e 4 não conferidos após o último ajuste).

## Estados da tarefa
Máquina de estados explícita; transições inválidas bloqueadas.

| De | Para | Quem |
|---|---|---|
| rascunho | na_fila | usuário assina o plano **(a validar)** |
| na_fila | em_execucao | usuário inicia |
| em_execucao | pausada | usuário pausa (motivo obrigatório) |
| pausada | em_execucao | usuário retoma |
| em_execucao | concluida | usuário entrega — **concluída nunca reabre** |
| na_fila | atrasada | sistema: prazo venceu |
| pausada | atrasada | sistema: pausa estourou o limite (**não** gera abandono direto) |
| em_execucao | atrasada_em_execucao | sistema: prazo venceu, trabalho segue **(a validar)** |
| atrasada | em_execucao | usuário retoma (continua devendo) |
| atrasada | abandonada | sistema: atraso passou da tolerância |

`abandonada` é final no MVP. Estados do sistema são **derivados dos timestamps** (prazo, início da pausa, limite, tolerância), recalculados ao abrir o app e ao voltar ao primeiro plano. Notificações locais agendadas para os mesmos momentos.

## Regras de negócio
**Decididas**
- Uma tarefa por vez. Enquanto uma está em execução ou pausada, as outras ficam bloqueadas ("Uma de cada vez.").
- Pomodoro não é obrigatório, mas é estimulado: ligado por padrão (25/5/15, pausa longa a cada 4 ciclos), o chefe comenta quando é desligado, destaque no relatório.
- Pausa estourada gera atraso, não abandono.
- Cada tarefa tem tolerância de atraso que define quando vira abandono; a contagem aparece durante a execução.

**Propostas (a validar)**
1. Toda falha vira cobrança, explicável na hora ou até bater o ponto. O dia só fecha com tudo explicado.
2. Dois tipos de desculpa: **rápida** para pausa (categoria + motivo); **formal** (com plano se-então) para atraso, abandono e mudança no plano.
3. Só se executa o que foi assinado. Tarefa nova entra como rascunho. Adicionar tarefa não exige justificativa; mudar prazo, estimativa, tolerância ou excluir tarefa assinada exige, e fica num histórico que não pode ser apagado.
4. Prazo vencido não interrompe o trabalho. A tolerância só corre com a tarefa parada.
5. Relógios de pausa e tolerância só correm dentro do horário de trabalho; prazos valem como data e hora.
6. O veredito do Dr. Alberto não bloqueia nada; vai para o histórico e o Dossiê.
7. Primeiro uso tem "contrato de trabalho" (consentimento explícito do tom e das regras) e pré-permissão de notificação antes do pedido do sistema.
- Pausa do pomodoro não pede motivo nem conta contra; só o excedente vira pausa não programada e gera cobrança.
- Tarefa atrasada não bloqueia: o usuário pode "trabalhar primeiro" e explicar até bater o ponto.
- Bater o ponto antes da hora é permitido, mas fica anotado. Plano acima da capacidade pode ser assinado "assumindo o risco" (registrado).

## Central de Desculpas (a validar)
- Três campos: (1) categoria — interrupção externa, subestimei o tempo, travei/procrastinei, cansaço, prioridade mudou; (2) relato livre; (3) "da próxima vez: se X, eu faço Y" (opcional, pesa no veredito).
- Veredito por regra, não aleatório: primeira vez na categoria na semana = **Tolerada**; repetida com plano = **Suspeita**; repetida sem plano = **Rejeitada**.
- Dossiê mostra padrões ("interrupção externa 7x esta semana").
- Base: intenções de implementação (Gollwitzer), WOOP (Oettingen), autoperdão e procrastinação (Wohl, Pychyl e Bennett, 2010), autocompaixão (Breines e Chen, 2012).

## Tom de voz
- Curto, seco, exigente, irônico. Elogio raro e curto. Microinterações reforçam cobrança e entrega, não celebração.
- Exemplos aprovados: "Eu não sou seu amigo. Sou seu chefe." · "Sem contrato, sem chefe." · "Plano assinado. Agora é contrato." · "Uma de cada vez." · "Passou do tempo. Isso vai para as cobranças." · "Ponto batido. Relógios parados até amanhã."
- Evitar "chicote" e qualquer coisa humilhante.

## Para código (Claude Code)
- Stack: Expo (React Native) + TypeScript, armazenamento local, sem backend e sem login, `expo-notifications`, backup JSON (exportar/importar).
- Toda copy num único arquivo, por chave e contexto (ex.: `pausa.motivoObrigatorio`), com variações. O usuário vai reescrever esse arquivo.
- Cores, tipografia e espaçamentos num arquivo de tema com tokens, claro e escuro.
- Testes unitários cobrindo todas as transições e os cálculos de atraso e abandono.
- Modelo de dados: `Task` (id, titulo, prazo, estimativaMin, toleranciaAtrasoMin, status, ordem, criadaEm, concluidaEm) · `Interval` (taskId, tipo: trabalho | pausa_pomodoro | pausa_tarefa, inicio, fim, motivo, pomodoro) · `Interruption` (taskId, tipo: interna | externa | ideia, texto, data) · `PlanChange` (taskId, campo, valorAnterior, valorNovo, justificativa, data) · `DayClose` (data, reflexao, veredito) · `Settings` (limitePausaMin=15, toleranciaPadraoMin, pomodoro 25/5/15/4, horarioTrabalho).
- Fora do MVP: login, nuvem, equipe, apostas com dinheiro, gamificação, integrações, subtarefas, categorias, prioridades, recorrência, hábitos.

## Para design (wireframes e telas)
- Celular 390 x 844, uma prancheta por tela, nomeada "nº do fluxo · nome da tela" (ex.: "2A · Tela de foco").
- Wireframe: tons de cinza, sem cor de marca; Dr. Alberto como placeholder circular com legenda.
- Textos reais em português, no tom do chefe. Nada de lorem ipsum.
- Prioridade: núcleo do Modo Pressão (fila, modal de atraso, tela de foco, estimativa estourada, pausas, desculpa rápida, tarefa pausada).

## Em aberto
- O que acontece depois do abandono (definitivo ou recriável, e com que custo).
- Tolerância por tarefa, global ou as duas (padrão global + ajuste por tarefa).
- Nome do modo de execução ("Modo Pressão" é o recomendado).
- Plataforma (celular no MVP x telas atuais em desktop).

## Como trabalhar comigo
- Responda em português.
- Antes de escrever código ou montar telas grandes, mostre o plano e espere aprovação.
- Ao fim de cada etapa: o que foi feito, como testar, o que vem a seguir.
- Se uma decisão aqui estiver marcada (a validar), siga a proposta, mas sinalize.
