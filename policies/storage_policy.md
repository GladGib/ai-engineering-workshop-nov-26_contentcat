# Storage Policy (Marketing Footage Library)

**Tiers**
- **Hot (never touch):** any clip used in a social project (in `project_clips`), or `is_favorite = 1`.
- **Cold:** unused, not favourite, recorded more than 90 days ago.
- **Already efficient:** clips already in HEVC are skipped (no meaningful savings).

**Rules**
1. **Never delete footage.** Raw footage may be needed for future campaigns. The only allowed action is re-encoding cold clips to HEVC.
2. Every compression plan needs explicit **approval from the content lead** in the Approvals tab.
3. The assistant may propose plans; only app code runs approved actions.
