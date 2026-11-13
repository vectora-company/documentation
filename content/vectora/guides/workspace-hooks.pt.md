---
title: Hooks do Workspace
weight: 11
---

Hooks são scripts determinísticos que rodam em pontos fixos do loop do agente. Use-os para impor uma
regra que não dá para confiar que o modelo siga sozinho: bloquear um comando perigoso, conferir o
resultado de uma ferramenta ou recusar uma resposta final que quebra uma convenção. Os hooks estão
disponíveis a partir da versão 0.1.26.

## Definindo um hook

Os hooks ficam no `vectora.toml` do workspace, uma tabela `[[hooks.rules]]` por regra:

```toml
[[hooks.rules]]
id = "bloqueia-rm"
event = "pre_tool"                       # "pre_tool" | "post_tool" | "pre_response"
tools = ["run_command"]                  # padrões fnmatch; vazio vale para todas
command = ["python", ".vectora/hooks/check.py"]   # lista de argumentos, sem shell
mode = "blocking"                        # "blocking" | "informative"
timeout_s = 10                           # de 0,5 a 60 segundos
```

| Campo       | Significado                                                                                                    |
| ----------- | -------------------------------------------------------------------------------------------------------------- |
| `id`        | Letras minúsculas, dígitos, `-` e `_`, até 64 caracteres.                                                      |
| `event`     | `pre_tool` roda antes de uma ferramenta, `post_tool` depois, `pre_response` antes de aceitar a resposta final. |
| `tools`     | Padrões de nome de ferramenta. Ignorado em `pre_response`, que não tem ferramenta.                             |
| `command`   | O programa e seus argumentos, em lista. O Vectora nunca passa isso por um shell.                               |
| `mode`      | `blocking` pode impedir a ação; `informative` só registra o resultado.                                         |
| `timeout_s` | Limite de tempo do script.                                                                                     |

Uma regra com campo inválido é descartada com um aviso, para que um erro de digitação não derrube o
resto da configuração.

## O que o script recebe e devolve

O script lê um objeto JSON da entrada padrão: `version`, `event`, `hook_id`, `run_id`, `thread_id`
e, em seguida, `tool` (`name` e `args`) mais `result` no `post_tool`, ou `response` no
`pre_response`. Os valores de chaves sensíveis (tokens, senhas, chaves de API) são substituídos
antes de chegarem ao script, e resultados longos são truncados. O ambiente é mínimo: algumas
variáveis básicas mais `VECTORA_HOOK_ID` e `VECTORA_HOOK_EVENT`.

Ele responde na saída padrão com `{"decision": "allow"}` ou `{"decision": "deny", "reason": "..."}`.
Saída vazia com código 0 equivale a `allow`.

```python
import json, sys

payload = json.load(sys.stdin)
command = payload["tool"]["args"].get("command", "")
if "rm -rf" in command:
    print(json.dumps({"decision": "deny", "reason": "rm -rf não é permitido aqui"}))
```

## O que acontece em cada evento

- **`pre_tool`**: uma negação impede a chamada, e o agente recebe o erro `hook_denied` com o seu
  motivo. A permissão para executar a ferramenta é conferida de novo depois que o hook termina; uma
  mudança de política que chegue enquanto o hook roda continua valendo.
- **`post_tool`**: uma negação bloqueante retém o resultado da ferramenta, que não chega ao modelo.
- **`pre_response`**: uma negação devolve o seu motivo ao modelo e pede que ele revise a resposta.
  Isso acontece no máximo duas vezes por execução; depois disso, a última resposta é entregue.

Os hooks valem também para as ferramentas que um subagent executa.

## Falha fechada

No modo `blocking`, uma negação, um timeout, um código de saída diferente de zero ou uma resposta
que não é JSON válido impedem a ação. Um script quebrado nunca deixa a ação passar por acidente. No
modo `informative`, o resultado só é registrado. Cada execução deixa uma entrada `hook_run` nas
evidências da run, sem os argumentos da chamada.

Se o sandbox está ativo no workspace, o script roda dentro do mesmo worker isolado das ferramentas.
Quando um script estoura o tempo, a árvore inteira de processos é encerrada, não só o processo
principal.

## Aprovação

Um `vectora.toml` pode vir de um repositório clonado, então os comandos dele não rodam sozinhos. O
workspace precisa ser aprovado antes, e a aprovação fica presa a um digest das regras: mudar um
comando, evento, lista de ferramentas, modo ou timeout exige aprovar de novo. O mesmo digest cobre
os comandos de `[hooks].post_file_write`. Enquanto isso, os hooks aparecem como pendentes e são
ignorados.

Revise e aprove as regras na seção **Hooks do workspace** das configurações do workbench Terminal.

## Testando pela linha de comando

```bash
vectora hooks list --workspace .
vectora hooks test bloqueia-rm --tool run_command --args '{"command": "rm -rf /tmp/x"}'
```

`list` mostra cada regra e o estado da aprovação. `test` executa uma regra uma vez com uma chamada
sintética e imprime a decisão, sem afetar nenhuma execução. Use `--result` para `post_tool` e
`--response` para `pre_response`. Como você mesmo escolhe a regra, o `test` não exige a aprovação do
workspace.

## Veja também

- [Usando o Chat](../using-the-chat) — modos de permissão e aprovações
- [Políticas de egress](../egress-policies)
