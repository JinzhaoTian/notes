```
                  ┌─────────────────────────────┐
                  │           UI / API          │
                  │   Web / Headless / SDK ...  │
                  └──────────────┬──────────────┘
                                 │
                         session/event
                                 │
              ┌──────────────────▼──────────────────┐
              │           Cordis Kernel             │
              │  plugin tree / dependency / effects │
              └──────────────────┬──────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
   ctx.agentLoop             ctx.sessions              ctx.llm
        │                        │                        │
 Agent / Turn / Step     Append-only Event Log       Model Adapters
        │                        │
        │                 deriveMessages()
        │                        │
        └───────────────► Model Context
                                 │
                          Model Response
                                 │
                          Tool Calls
                                 │
                  ┌──────────────▼──────────────┐
                  │        ctx.tools            │
                  │ Tool Execution Pipeline     │
                  ├─────────────────────────────┤
                  │ pre-execute                 │
                  │ permissions / sandbox       │
                  │ monotonic guards            │
                  │ execute                     │
                  │ timeout / retry / metrics   │
                  │ post-execute                │
                  │ result normalization        │
                  └──────────────┬──────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
       FS                     Shell                   Subagents
        │                        │                        │
 local/remote             local/sandbox       DSH / Claude / Codex
```


