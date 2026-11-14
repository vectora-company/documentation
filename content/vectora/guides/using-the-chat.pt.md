---
title: Usando o Chat
weight: 1
---

## Enviando mensagens

O input do chat aceita texto, arquivos arrastados (drag-and-drop), colar imagens do clipboard, e
`@menções` de arquivo pra trazer conteúdo específico pro contexto sem precisar descrevê-lo.

## Seletor de modelo

Mostra só os providers com chave de API configurada — Google Gemini, OpenAI, Anthropic, Cohere, ou
Ollama local. Trocar de modelo no meio de uma conversa não perde o histórico.

## Modos de permissão

Controlam o quão automático o agente age antes de pedir sua aprovação:

| Modo                 | Comportamento do gate de aprovação                                                                  |
| -------------------- | --------------------------------------------------------------------------------------------------- |
| **Perguntar sempre** | pausa para ferramentas classificadas como sensíveis ou destrutivas                                  |
| **Aceitar edições**  | permite escrita e edição de arquivos sem pausa; outras ações protegidas continuam sujeitas a HITL   |
| **Autônomo**         | não pausa para aprovação humana comum; policy e bloqueios de segurança continuam ativos             |
| **Plano**            | pausa para a primeira ação protegida do turno; chamadas de mídia com custo exigem aprovação própria |
| **Bypass**           | desativa pausas comuns de HITL; guardrails independentes continuam aplicáveis                       |

`Perguntar sempre`, `Aceitar edições`, `Autônomo`, `Plano` e `Bypass` são modos de permissão, não
níveis de confiança no modelo. Algumas operações, como controle do desktop, ainda podem exigir
confirmação em qualquer modo. Em `Plano`, uma aprovação anterior no mesmo turno permite continuar o
plano; isso não transforma a aprovação em uma autorização permanente. Veja
[Plan Mode, AgentRun e decisões](../plan-mode-and-agentrun) para o prefixo `/plan` e a diferença
entre o Plan Mode e a aba Plan da workbench.

Com isolamento nativo do worker confirmado, o gate HITL dispensa aprovação para `terminal`,
`file_write` e `file_edit`, inclusive na primeira ação do modo Plano. As demais restrições de policy
e das tools continuam valendo.

Uma pré-aprovação é uma concessão separada e explícita. Ela vale uma única vez para uma ferramenta e
seus argumentos exatos, dentro do mesmo workspace e `run_id`, e expira em prazo limitado.
Ferramentas irreversíveis não aceitam essa concessão.

No modo **Autônomo**, um Decision Model só participa quando está habilitado, selecionado e pronto
para a superfície da ação. Sem ele, o runtime não deve ser interpretado como se uma decisão
automatizada adicional tivesse sido tomada: a policy estática ainda determina o que pode executar.
Quando uma avaliação iniciada falha, excede o prazo ou exige revisão humana, a ação não deve ser
liberada como se tivesse sido aprovada.

Veja [Orchestrator & Subagentes](../../concepts/sub-agents) para entender o HITL por trás desses
gates e [Modelos de decisão](../decision-models) para configurar Decision Models e suas superfícies.

## Mensagens enquanto o agente responde

Você pode continuar digitando enquanto o agente trabalha. Uma mensagem enviada durante uma resposta
entra numa fila exibida acima do input, e cada item tem um estado:

| Estado          | Significado                                                                                |
| --------------- | ------------------------------------------------------------------------------------------ |
| **Na fila**     | Aguardando; é enviada quando a resposta atual termina.                                     |
| **Enviando**    | A requisição está a caminho do servidor.                                                   |
| **Não enviada** | A requisição não chegou ao servidor. O item continua na fila.                              |
| **Cancelada**   | Você interrompeu o envio antes de chegar ao servidor. Fica na fila e não segura as demais. |

Um item só sai da fila depois que o servidor confirma. Se o envio falhar, use **Tentar de novo**
para enviá-lo outra vez: o reenvio usa o mesmo identificador de mensagem, então a mensagem nunca é
duplicada, mesmo que a primeira tentativa tenha chegado. Um item que falhou segura os que vêm
depois, o que preserva a ordem; reenvie-o ou remova-o para liberar os demais. Um item cancelado
também pode ser enviado de novo com **Tentar de novo**. Também é possível enviar um item em espera
na hora, o que interrompe antes a resposta atual. A fila tem altura máxima, para não cobrir a
conversa.

Os anexos ainda não enviados ficam guardados por conversa: trocar para outra conversa e voltar
restaura os anexos junto com o texto.

## Stream de resposta confiável

Cada evento de uma resposta leva um número de sequência. Se a conexão repetir ou embaralhar eventos,
o chat ignora os excedentes e, depois do evento final, não aceita mais nada, então o texto nunca
aparece duas vezes. O evento final informa por que a resposta terminou, e a mensagem mostra um
indicador quando ela foi interrompida ou atingiu o limite de passos.

Se a conexão cair enquanto o agente ainda trabalha, o chat reconecta sozinho a partir do último
evento recebido e mostra o que faltou. Só recarrega a conversa salva quando o servidor não tem mais
esses eventos, por exemplo depois de reiniciar. Assim que a resposta final chega, o input fica livre
na hora, sem tempo de espera antes da próxima mensagem.

Se uma mensagem enviada diretamente não chegar ao servidor, o texto e os anexos voltam ao input para
você enviar de novo.

Quando a resposta demora mais de 15 segundos e a janela está oculta, o sistema mostra um alerta de
resposta pronta, no máximo uma vez por mensagem: reenviar da fila ou reconectar o stream não repete
o alerta. Sem permissão de notificações do sistema o alerta não aparece, e isso não impede que ele
apareça depois, se você conceder a permissão. Alertas de push com o app fechado dependem da
plataforma e ainda não estão disponíveis.

## Thinking / raciocínio do orchestrator

O bloco de "pensando" mostra a decisão do orchestrator antes de agir: responder direto ou delegar
pra `coder`/`search`, e por quê. Isso é transparência de verdade, não só um spinner — dá pra ver o
raciocínio, não só esperar o resultado.

## Memória entre sessões

Um badge "🧠 N memórias carregadas" aparece quando o agente usa memórias persistentes de conversas
anteriores nessa resposta. Gerencie fatos persistentes em **Configurações → Preferências →
Memória**; a aba Memory da workbench mostra atividade de recuperação e contexto usado.

## Citações RAG

Respostas baseadas em conteúdo indexado trazem citações `[1] [2]` clicáveis — clique pra ver o
trecho original e a fonte.

## Multi-usuário (Pro)

No modo Pro com chat web multi-usuário, threads podem ser compartilhadas entre membros do mesmo
workspace, com RBAC controlando quem vê o quê.

## Veja também

- [Usando a workbench](../using-the-workbench) — o painel lateral que acompanha o chat
- [Usando as configurações](../using-settings) — onde ficam modelo padrão, idioma, tema
