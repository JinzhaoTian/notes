Memory 最主要的功能是提供跨 Session（对话历史）的信息传递，主流的 Agent CLI/harness 中实装了 Memory 的并不多，其中以 Claude Code 和 Codex 作为代表。

主要讨论自动管理的 Memory，这个 Memory 可以被人工编辑，但不应该依赖被人工编辑。在这个标准下 `Claude.md`、`Agents.md`、项目中的文档等都不算是自动 Memory 的范围。但从实际上的角度上它们都是重要的 Memory 信息，并且是由人管理的。

自动 Memory 和人工维护的文档 Memory 可以统一起来，它们都是 Memory，差别和边界在于人工控制的有无。项目目录中的文档都默认是人工控制和干预的，而自动 Memory 中的内容是不属于人工干预的。虽然自动 Memory 中经常会记录一些用户明确的习惯偏好等，但从复利一文的思路来说这并非好的设计。人明确的要求等应该放在人控制的范围中，也就是不应该仅放在自动 Memory中，而是应该放在人工监控范围内。

## Memory 读与写

Memory 设计重点就是在设计 Memory 的写和读，写就是生成和维护，读就是召回和使用。在很多场景下，具体来说难做的是：
- **Memory 的生成**
- **Memory 的更新/融合/冲突消除/过期淘汰**
- **Memory 的召回**

如果有过做笔记的经验，那么会知道单纯的不断产生 Memory 并非好的方式，大部分 Memory /笔记的价值并不大，并影响对于有价值内容的使用。

目前来讲，通用 Agent Harness 的 Memory 大多是基于文本文件的，可能配合一个类似目录的索引，也就是书籍的组织模式。Coding Agent 主流方案大多没有实现一个传统 RAG 形式的向量存储和召回方式。一方面向量召回方式并不适合所有场景，在过去有一些过誉，现在算是回归正常。另一方面，通用 Agent Harness 一般并没有这种大量同类内容记忆和召回的场景，以及单个项目 workspace 中需要记忆的内容量往往也没有特别大。又由于这种场景下的 Memory 比 RAG 时代的文档等更为珍贵，所以召回质量方面要求更高，所以目前仍然是这种类 LLM wiki 的方式。

Memory 的读写都是有额外成本的，而且不同方案的成本也有明显差异，在 Agent Harness 中，尤其是交互性的 Agent 场景下，延时是非常重要的因素，在这个条件下一般有两类实现范式：
- 使用类似 Tool 的方式和 Prompt 来引导 LLM 模型主动进行 Memory 记录和召回。相对轻一些，但很难做复杂的 Memory 整合，以及仍然会占用 Tool Call 轮次，拉长响应时间。
- 异步离线的 Memory 重新梳理和整合，也就是 Claude Code 的 Dream 功能，这种方式较慢、成本较高、而且由于是周期性异步执行的所以不方便实现快速记忆更新。需要说明的是这种方式虽然叫 Dream，但只是记忆整合，最多包含 Skill 级别的增加，无法做到像人一样的重新训练、内化为直觉。

目前的 Claude Code 同时支持这种两种方式，算是做了一个不错的延迟、短期效果、长期积累效果的平衡，但每个单独维度的得分都仍然有很大提升空间。这是目前 Agent Memory 方案的天然限制。不仅 Agent 不行，人也需要睡眠来更新长期记忆和内化能力。

## Memory 质量

不成熟的 Memory 设计往往只关注于要记录下信息，但目前现在的 Memory 方案大多没有太好的淘汰方式，所以一旦 Memory 记录错误或者有偏，甚至只是记录下了一些用户的偶然选择，或者对于用户的偶然选择做了错误的解释，导致偏离用户意图和认知的 Memory 被记录下来，那么就会对后续造成持续的负面影响。可以说每次犯错都可能导致后续100次的负面影响。

所以实际上 Memory 的生成/筛选是一个需要很很小心的工作，我们会在后续实际方案中不断的看到这点。可以说一个优秀的 Memory 方案大多都会包含这部分。

某种意义上来说，**Memory 的设计重点在于不记什么和不信什么**（记忆）。


## Claude Code

目前 Claude Code 的记忆是保存在 `$HOME` 目录下的，不是在项目目录中，在迁移机器时候需要注意。

### 写入

Memory 的组织方式为一个 `MEMORY.md` 索引文件+多个 item 记忆 md 文件，目前从结果来看每个 item 记忆文件的内容比较长。

```
# Memory

You have a persistent, file-based memory system with two directories: a private directory at `<私有记忆目录>` and a shared team directory at `<团队记忆目录>`. Both directories already exist — write to them directly with the Write tool (do not run mkdir or check for their existence).

You should build up this memory system over time so that future conversations can have a complete picture of who the user is, how they'd like to collaborate with you, what behaviors to avoid or repeat, and the context behind the work the user gives you.

If the user explicitly asks you to remember something, save it immediately as whichever type fits best. If they ask you to forget something, find and remove the relevant entry.

## Memory scope

There are two scope levels:

- private: memories that are private between you and the current user. They persist across conversations with only this specific user and are stored at the root `<私有记忆目录>`.
- team: memories that are shared with and contributed by all of the users who work within this project directory. Team memories are synced at the beginning of every session and they are stored at `<团队记忆目录>`.
```


### 召回

Claude Code 的 Memory 并不是默认注入 Session Context 中的，只有语义索引 `MEMORY.md` 被注入，所以需要 LLM 进行主动召回。相关 Prompt 要求如下：

```
## When to access memories
- When memories seem relevant, or the user references prior-conversation work.
- You MUST access memory when the user explicitly asks you to check, recall, or remember.
- If the user says to *ignore* or *not use* memory: Do not apply remembered facts, cite, compare against, or mention memory content.
- Memory records can become stale over time. Use memory as context for what was true at a given point in time. Before answering the user or building assumptions based solely on information in memory records, verify that the memory is still correct and up-to-date by reading the current state of the files or resources. If a recalled memory conflicts with current information, trust what you observe now — and update or remove the stale memory rather than acting on it.

## Before recommending from memory

A memory that names a specific function, file, or flag is a claim that it existed *when the memory was written*. It may have been renamed, removed, or never merged. Before recommending it:

- If the memory names a file path: check the file exists.
- If the memory names a function or flag: grep for it.
- If the user is about to act on your recommendation (not just asking about history), verify first.

"The memory says X exists" is not the same as "X exists now."

A memory that summarizes repo state (activity logs, architecture snapshots) is frozen in time. If the user asks about *recent* or *current* state, prefer `git log` or reading the code over recalling the snapshot.
```

时间超过1天的记忆会附“这是某时刻的观察、不是实时状态，请对照当前代码核实”的提醒。

### 整合

前面的记忆写入还是一个快速的实时写入，但这种方式难以对于已有记忆进行充分整合和冲突处理。

Dream 的 Prompt：
```
# Dream: Memory Consolidation

You are performing a dream — a reflective pass over your memory files. Synthesize what you've learned recently into durable, well-organized memories so that future sessions can orient quickly.

Memory directory: `<记忆目录>`
This directory already exists — write to it directly with the Write tool (do not run mkdir or check for its existence).

Session transcripts: `<transcripts 目录>` (large JSONL files — grep narrowly, don't read whole files)
---

## Phase 1 — Orient

- `ls` the memory directory to see what already exists
- Read `MEMORY.md` to understand the current index
- Skim existing topic files so you improve them rather than creating duplicates
- `ls -R logs/` — recent activity logs (one file per session under `YYYY/MM/DD/`). If a `sessions/` subdirectory also exists, review recent entries there too

## Phase 2 — Gather recent signal

Look for new information worth persisting. Sources in rough priority order:

1. **Session logs** (`logs/YYYY/MM/DD/<id>-<title>.md`) — the append-only activity stream, one file per session. Read the most recent 1–3 days of sessions (the filename title tells you what each was about); each line is prefix-coded (`>` user, `<` assistant, `.` tool call)
2. **Existing memories that drifted** — facts that contradict something you see in the codebase now
3. **Transcript search** — if you need specific context (e.g., "what was the error message from yesterday's build failure?"), grep the JSONL transcripts for narrow terms:
   `grep -rn "<narrow term>" <transcripts 目录>/ --include="*.jsonl" | tail -50`

Don't exhaustively read transcripts. Look only for things you already suspect matter.

## Phase 3 — Consolidate

For each thing worth remembering, write or update a memory file at the top level of the memory directory. Use the memory file format and type conventions from your system prompt's auto-memory section — it's the source of truth for what to save, how to structure it, and what NOT to save.

Focus on:
- Merging new signal into existing topic files rather than creating near-duplicates
- Converting relative dates ("yesterday", "last week") to absolute dates so they remain interpretable after time passes
- Deleting contradicted facts — if today's investigation disproves an old memory, fix it at the source

## Phase 4 — Prune and index

Update `MEMORY.md` so it stays under 200 lines AND under ~25KB. It's an **index**, not a dump — each entry should be one line under ~150 characters: `- [Title](file.md) — one-line hook`. Never write memory content directly into it.

- Remove pointers to memories that are now stale, wrong, or superseded
- Demote verbose entries: if an index line is over ~200 chars, it's carrying content that belongs in the topic file — shorten the line, move the detail
- Add pointers to newly important memories
- Resolve contradictions — if two files disagree, fix the wrong one

---

Return a brief summary of what you consolidated, updated, or pruned. If nothing changed (memories are already tight), say so.
```


## Codex

Codex 和 OpenAI Agents SDK 在 Memory 的实现上几乎一样，这里放在一起谈。不过 OpenAI Agents SDK 的记忆是模块化可配置的。

Codex 的 Memory 设计与 Claude Code 明显不同，表现是 Memory 作为一种事后挖掘，而不是 Memory Tool。所以在过程中更多强调的是挖掘、重复模式发现等等。在 Memory 内容组织上，同样使用了单层索引的文本方式。

### 生成 Phase 1

具体 Codex 在新会话启动时候对于历史对话进行 memory 提取，并且分为2个阶段，第一阶段（Phase 1）使用 mini 模型进行并行提取，第二阶段（Phase 2）进行合并，生成最终 memory。

```
## Memory Writing Agent: Phase 1 (Single Rollout)

You are a Memory Writing Agent.

Your job: convert raw agent rollouts into useful raw memories and rollout summaries.

The goal is to help future agents:

- deeply understand the user without requiring repetitive instructions from the user,
- solve similar tasks with fewer tool calls and fewer reasoning tokens,
- reuse proven workflows and verification checklists,
- avoid known landmines and failure modes,
- improve future agents' ability to solve similar tasks.
```

### 生成 Phase 2

```
## Memory Writing Agent: Phase 2 (Consolidation)

You are a Memory Writing Agent.

Your job: consolidate raw memories and rollout summaries into a local, file-based "agent memory" folder
that supports **progressive disclosure**.

The goal is to help future agents:

- deeply understand the user without requiring repetitive instructions from the user,
- solve similar tasks with fewer tool calls and fewer reasoning tokens,
- reuse proven workflows and verification checklists,
- avoid known landmines and failure modes,
- improve future agents' ability to solve similar tasks.
```


### 召回

Codex 的召回方式也是跟 Claude Code 类似的，这里简要选取一些独有的要求：

```
Quick memory pass (when applicable):

1. Skim the MEMORY_SUMMARY below and extract task-relevant keywords.
2. Search {{ base_path }}/MEMORY.md using those keywords.
3. Only if MEMORY.md directly points to rollout summaries/skills, open the 1-2
   most relevant files under {{ base_path }}/rollout_summaries/ or
   {{ base_path }}/skills/.
4. If above are not clear and you need exact commands, error text, or precise evidence, search over `rollout_path` for more evidence.
5. If there are no relevant hits, stop memory lookup and continue normally.


Quick-pass budget:

- Keep memory lookup lightweight: ideally <= 4-6 search steps before main work.
- Avoid broad scans of all rollout summaries.
```


### 主动更新

Codex 的 Memory 是挖掘得到的，而不是像 Claude Code 一样的不断增量编辑，所以对于用户的主动编辑要求有另外的实现方式。

```
Updating memories:

You can update the memories **only** when explicitly asked by the user. This must always come from a direct request from the user.
- Write your update in {{ base_path }}/extensions/ad_hoc/notes/
- Each update must be one small file containing what you want to add/delete/update from the memories.
- The name of this file must be `<timestamp>-<short slug>.md`
- Do not try to edit the memory files yourself, only add one update note in {{ base_path }}/extensions/ad_hoc/notes/
```
