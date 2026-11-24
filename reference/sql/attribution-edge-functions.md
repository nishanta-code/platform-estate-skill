# Brain project: edge functions

Live snapshot taken 2026-11-24 from <DATA_REF> via list_edge_functions. Grouped by apparent subsystem (grouping is by name only). verify_jwt=false means the function performs its own auth (most carry an internal_token in the body or a webhook signature); created/updated are the deploy timestamps (UTC date). Entrypoints of the form file:///tmp/user_fn_... are dashboard/MCP deploys, supabase/functions/<name>/index.ts are CLI deploys.

