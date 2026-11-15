---
title: Context Graph
weight: 2
---

The Context Graph is a **native workspace knowledge graph** — it complements RAG with structural
context that embeddings alone can't capture (who calls whom, what depends on what, which concepts
relate to each other).

## How it's built

1. **AST parsing** via [tree-sitter](https://tree-sitter.github.io/), with grammars for Python,
   JavaScript, TypeScript, Go, Rust, Java, C, C++, and JSON.
2. **Semantic extraction** by LLM on top of the parsing result — identifies concepts, relationships,
   and the relative importance of nodes.
3. **Community and "god node" detection** — excessively coupled files/symbols, which tend to be risk
   points during refactors.
4. **Suggested questions** — the graph suggests what's worth asking about the workspace, based on
   its topology.

## Indexing modes

Configurable in the workbench's **Context Graph** tab:

- **By file type** — code, documents, papers — you choose what enters the graph (e.g., markdown
  only, leaving code to pure RAG).
- **Semantic vs. AST** — AST is faster and structural; semantic leans more on the LLM to capture
  relationships syntax alone doesn't show.

## Pausable build

Building the graph for a large workspace consumes LLM calls. The build is **pausable and resumable**
by quota — if you hit a rate/cost limit mid-process, it picks up where it left off instead of
starting over.

## Where artifacts live

`.vectora/context-graph/` inside the workspace itself — the same files `.vectoraignore` hides from
the rest of Vectora also stay out of the graph.

## Visual stages (workbench)

While the graph builds, each file shows a stage indicator:

| Stage    | Color (dark / light)                  | Means                          |
| -------- | ------------------------------------- | ------------------------------ |
| AST      | light blue `#4a9eff` / blue `#1f6feb` | structural parsing in progress |
| Semantic | purple `#b66dff` / purple `#8957e5`   | LLM extraction in progress     |
| Done     | green `#5ec26a` / green `#2da44e`     | file processed                 |

## Finding symbols

After the graph is built, enter the exact name of a function or class in the viewer search and press
Enter or select **Resolve symbol**. Text search still highlights matches on the canvas; symbol
resolution queries exact code definitions in the graph. If the same name appears in multiple files,
choose a candidate by its path and location. Selecting one opens the node on the canvas and reveals
its community, if present. Vectora does not choose for you when the result is ambiguous.

The agent can also use `graph_resolve_symbol` to query the workspace graph. The tool returns a
bounded list of candidates and does not modify the saved graph files.

## Export and remote destination

The tab can download GraphML, Cypher, Obsidian Canvas, SVG, and an Obsidian vault as a ZIP. SVG and
vault exports have their own size limits to prevent oversized exports.

You can also configure a FalkorDB instance and send the graph after a confirmation that identifies
the host and the node and edge counts. Sending uses TLS, checks the destination against the
workspace egress policy, and does not happen when the tab opens. The host, port, and graph name are
stored in local preferences; the username and password stay in the encrypted vault and are never
returned by the API. The workspace policy must allow the configured host and port.

The graph can contain repository-derived names and content. Review the destination before confirming
the transfer.

## See also

- [Hybrid RAG](../rag) — similarity retrieval, complementary to the graph
- [Using the workbench](../../guides/using-the-workbench) — the Context Graph tab in detail
