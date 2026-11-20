# MeaningSpace

**Keep every word. Load only the meaning.**

A local memory layer for people who work with an AI all day, every day.
The full conversation log is never summarized away. A meaning layer with **line-level addresses** sits on top of it, so a new session pulls in only the few sections it needs — and can still walk back to the exact lines they came from.

![storage](https://img.shields.io/badge/storage-DuckDB%20%2B%20Parquet-yellow)
![runs](https://img.shields.io/badge/runs-local%20%2B%20MCP-blue)
![status](https://img.shields.io/badge/status-in%20daily%20use-brightgreen)
![first anchor](https://img.shields.io/badge/first%20public%20anchor-2026--06--06-lightgrey)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24388284.svg)](https://doi.org/10.5281/zenodo.24388284)

---

## The problem

Every new AI session starts from zero. You explain the architecture again. The agent retries the fix that already failed last week. Rebuilding context burns your usage limit before any real work starts.

Today there are two common answers, and both hurt:

| Approach | What happens |
|---|---|
| **Summarize and discard** | Fits in context, but the original reasoning is gone. When your understanding changes, there is nothing to go back to. |
| **Reload everything / grep the files** | Nothing is lost, but every question costs a pile of reading — and keyword search often lands in the wrong file. |

MeaningSpace takes a third path: **keep the ground, carry the meaning.**

## How it works

```mermaid
flowchart LR
    A["Conversation logs<br/>(kept whole, never edited)"] -->|crystallize| B["Crystals<br/>sections with outcome:<br/>decided / overturned / open"]
    B -->|address| A
    B --> C["Skills<br/>light up only above a<br/>similarity threshold"]
    D["Files & code"] --> E["Progress map<br/>purpose, links,<br/>change ledger with reasons"]
    E -->|reason → conversation section| B
```

1. **Ground** — every conversation is stored as a line-numbered log. Nothing is deleted or rewritten.
2. **Crystals** — each conversation is distilled into sections. Every section carries its outcome (*decided / overturned / open*) and the **addresses of the log lines** it came from. Addresses are *selected from a closed candidate set*, not recalled by the model.
3. **Skills** — working procedures are stored as crystals too. A search returns only a title and a query to descend; the body is read only when a skill clears the threshold. Adding skills does not grow every prompt.
4. **Progress map** — files and code get the same treatment: what each file is for, what it links to, and a change ledger where every edit carries a reason and a reverse patch tied to the conversation section that caused it.

Everything — text, vectors, a lexical index, addresses and the map — lives in **one DuckDB file**, so a single query can go from *meaning → section → log line → file → reason for the change*.

## What survives 60 days later

Both approaches keep you from re-explaining everything. The difference is what is still there when a question comes up that nobody expected.

| | Claude Code auto memory | MeaningSpace |
|---|---|---|
| What gets written | Notes Claude decides are worth keeping, while it works | Every conversation, crystallized section by section |
| When "important" is decided | At write time | At read time |
| Original conversation logs | Deleted after the retention period | Kept whole, never edited |
| Can you go back to the exact lines? | Only if the log still exists | Yes — every section carries line addresses |
| Overturned decisions | Stale notes are merged or dropped when noticed | Each section is marked *decided / overturned / open* |
| Loaded at session start | First 200 lines or 25 KB of the index | Only the sections the question pulls in |

Sources for the left column: [Claude Code docs — How Claude remembers your project](https://code.claude.com/docs/en/memory).

Auto memory is good at what it noticed at the time. MeaningSpace is built for what turns out to matter later: the essence of a long history, the decision that was reversed, and the exact line it came from.

## In daily use

- **2,405** conversation crystals over **3,036** logs
- **74** active skills
- Used every day on a Claude Max 5x plan
- Search is lexical (BM25) + vector, fused by rank; an optional GPU reranker sorts the final candidates

## Update 2026-11: a Parquet read layer

Search no longer reads the DuckDB file directly. The meaning layer is exported to a folder of Parquet files every night, and each crystallization adds a small delta file. DuckDB is still the only writer; everyone else reads the Parquet copy.

Measured on the author's PC, November 2026:

| | Before: DuckDB file only | After: Parquet read layer |
|---|---|---|
| One search, 2 threads (median) | 1,551 ms | 457 ms — same top 5 in 30 of 30 queries |
| Data needed to search | 2.3 GB (the whole database) | 157 MB |
| Time to rebuild the read layer | — | 3.6 s |
| 5 readers at once while a writer holds the database for 40 s | 0 of 50 reads succeeded (file locked) | 50 of 50 reads succeeded |

**Multi-user: tested with agents, not yet with people.** We treat each AI agent as one user. A second agent ran on the same host alongside the main one, reading memory and editing code, and passed the same steps three times in a row. The 5-reader test above ran the read path as five separate processes, not five full agents. Writes are serialized by one host, so we expect several human users to work the same way, but that has not been tested yet: who may read what, and what happens when edits collide, are still open.

## What is (and isn't) here

This repository is a **dated public record of the design**, not an installable package. The implementation is not published.
If you want to build something like it, or talk about using it in a team, reach out through **[subtractlab.com](https://subtractlab.com)**.

**Related work.** [Graphiti](https://github.com/getzep/graphiti) also keeps raw input as episodes and traces facts back to them. MeaningSpace differs in what it puts on top: narrative crystals and behavior-changing skills rather than an entity graph, with addresses down to the line.

**More:**
[MeaningSpace overview](https://subtractlab.com/meaningspace) · [Architecture (illustrated)](https://subtractlab.com/architecture.html) · [3Gravity](https://subtractlab.com/3gravity) · [QIS](https://subtractlab.com/qis) · [DP Multi-Agent](dp-multiagent.md) · [Products](https://subtractlab.com/products) · [Prior-art timeline](timeline.md) · [FAQ](https://subtractlab.com/faq)

---

Part of **[SubtractLab](https://github.com/subtractlab)** · Author: 奥田剛司 (Koji Okuda) · [subtractlab.com](https://subtractlab.com) · 日本語: [README.ja.md](README.ja.md)
