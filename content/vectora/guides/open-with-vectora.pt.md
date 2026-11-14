---
title: Abrir com Vectora
weight: 3
---

No app desktop, o Vectora aparece como opção em **Abrir com** para os arquivos que ele sabe usar
como anexo. Ele nunca se torna o aplicativo padrão de nenhum tipo: é só mais uma opção na lista.

## Quais arquivos

- **Imagens:** PNG, JPEG, GIF e WebP.
- **Áudio:** MP3, WAV, M4A, OGG, FLAC e WebM de áudio.
- **Vídeo:** MP4, WebM, MOV e M4V.
- **Documentos:** PDF, Word, Excel, PowerPoint e os formatos OpenDocument (ODT, ODS, ODP).
- **Texto e código:** texto simples, Markdown, CSV, JSON, XML, HTML, CSS, YAML, Python, JavaScript,
  TypeScript, SQL, shell e as demais linguagens listadas no seletor de anexos.
- **Pastas:** no Linux, o gerenciador de arquivos oferece o Vectora para pastas.

## O que acontece ao abrir

- **Arquivos** abrem uma conversa nova com eles já anexados, do mesmo jeito que se você os
  arrastasse para o campo de mensagem: cada arquivo respeita o limite de tamanho do seu tipo, e os
  que passam do limite avisam sem impedir os outros. Se você já tinha um rascunho na conversa nova,
  os arquivos se somam a ele.
- **Pastas** viram um workspace. O Vectora não confia na pasta sozinho: liberar escrita, terminal e
  git continua sendo uma ação sua, no diálogo de confiança do workspace.
- Se o Vectora já está aberto, o arquivo vai para a janela que existe, sem abrir um segundo app.

Também funciona arrastando o arquivo sobre o ícone do app e pela linha de comando
(`vectora caminho/do/arquivo`).

## Limites

- No Windows e no macOS, a pasta não aparece no menu de contexto do sistema. Arraste-a sobre o ícone
  do Vectora ou abra pela linha de comando.
- O Vectora lê no máximo 50 MB por arquivo ao abri-lo desta forma, e cada tipo tem ainda o seu
  próprio limite de anexo.

## Links do tipo `vectora://`

O link `vectora://session/<id>` abre a conversa com esse identificador no app desktop. Outros
destinos não são aceitos.
