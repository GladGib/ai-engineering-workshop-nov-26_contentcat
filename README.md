# 🎬 Studio Manager

An agentic media asset assistant for a solo video creator who shoots 100 clips and publishes 10. It finds footage, drafts storage-saving plans that a human must approve (it never deletes), and writes short-form scripts from real clips.

**Stack:** Google ADK · Gemini `gemini-3.5-flash` · SQLite · Streamlit

```
studio-manager/
├── app.py            # everything: data, MCP tools, guardrails, agents, eval, UI
├── requirements.txt
├── studio.db         # optional: auto-created on first run
├── policies/         # storage_policy, privacy_policy, style_guide, platform_rules, past_captions
└── README.md
```

## Architecture summary

A coordinator agent, `studio_manager`, routes each request to one of three specialists, each wrapped as an `AgentTool`:

- `library_analyst` answers data questions.
- `storage_planner` estimates HEVC savings and drafts an ffmpeg plan as text. Approval happens in the Approvals tab.
- `content_pipeline` is a `SequentialAgent`: clip picker → writer → `ParallelAgent` of hook and platform critics → editor.

The agents get **no static schema**. They discover it at runtime through MCP-style primitives (`mcp_list_tables` → `mcp_describe_table` → `mcp_read_query`) and read rules with `mcp_read_policy`. Clicking **➕ brand_deals** adds a table live, and the agent answers questions about it with zero code change. The destructive `mcp_delete_clips_admin` exists but is never registered with any agent (least privilege).

Safety has **two tiers**:

1. **Input tier.** A `before_model_callback` on every agent redacts emails, phone numbers, IC numbers, card numbers and GPS pairs. On the coordinator, it short-circuits prompt injection and any "delete my clips" request before an LLM call is made.
2. **Data tier.** Every query passes a `sqlparse` AST mutation check (`is_sql_mutation`). It then runs on a `file:studio.db?mode=ro` connection with a SQLite authorizer that NULLs the GPS columns (even inside expressions) and denies writes. Results are redacted (`[REDACTED_GPS]`, `[PRIVATE_LOCATION]`) before the model sees them. This matters because tool results never pass through a text-only guardrail.

An `after_model_callback` tracks tokens (including thinking tokens) and estimated USD per browser session. The **Eval** tab runs 5 golden cases whose expected answers are computed from the DB, scored 1–5 by an LLM judge. A judge parse failure scores 1, not an inflated default.

## Run
```bash
pip install -r requirements.txt
mkdir -p .streamlit && echo 'GOOGLE_API_KEY = "AIza..."' > .streamlit/secrets.toml
streamlit run app.py
```

## Deploy
Push to GitHub → share.streamlit.io → **New app** → `app.py` → **Advanced settings → Secrets**: `GOOGLE_API_KEY = "..."`

Pricing assumption: $1.50 in / $9.00 out per 1M tokens (checked 13 Nov 2026). All data is mock; bitrates are approximate.
