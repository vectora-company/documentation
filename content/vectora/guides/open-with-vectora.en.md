---
title: Open with Vectora
weight: 3
---

In the desktop app, Vectora shows up under **Open with** for the files it can use as attachments. It
never becomes the default application for any type: it is just one more option in the list.

## Which files

- **Images:** PNG, JPEG, GIF and WebP.
- **Audio:** MP3, WAV, M4A, OGG, FLAC and WebM audio.
- **Video:** MP4, WebM, MOV and M4V.
- **Documents:** PDF, Word, Excel, PowerPoint and the OpenDocument formats (ODT, ODS, ODP).
- **Text and code:** plain text, Markdown, CSV, JSON, XML, HTML, CSS, YAML, Python, JavaScript,
  TypeScript, SQL, shell and the other languages listed in the attachment picker.
- **Folders:** on Linux, the file manager offers Vectora for folders.

## What happens when you open

- **Files** open a new conversation with them already attached, the same way as if you dragged them
  into the message box: each file respects the size limit of its type, and the ones over the limit
  warn without blocking the others. If the new conversation already had a draft, the files are added
  to it.
- **Folders** become a workspace. Vectora does not trust the folder on its own: allowing writes,
  terminal and git is still your action, in the workspace trust dialog.
- If Vectora is already open, the file goes to the existing window, without starting a second app.

It also works by dragging the file onto the app icon.

## Limits

- On Windows and macOS, the folder does not appear in the system context menu. Drag it onto the
  Vectora icon.
- Vectora reads at most 50 MB per file when opening it this way, and each type also has its own
  attachment limit.

## `vectora://` links

The `vectora://session/<id>` link opens the conversation with that identifier in the desktop app.
Other destinations are not accepted.
