# Contributing

Thank you for helping improve the Upwork Agent Plugin.

## Propose a change

1. Open an issue describing the user need and expected behavior.
2. Create a focused branch from `master`.
3. Keep portable components in the locations defined by the Agent Plugins
   specification.
4. Add or update documentation when behavior changes.
5. Run the validation commands in `README.md`.
6. Open a pull request with a concise summary and test plan.

## Skill guidelines

- Give each skill a lowercase, hyphenated directory name that matches its
  frontmatter `name`.
- Describe both what the skill does and when it should activate.
- Keep `SKILL.md` concise and move detailed material into `references/`.
- Ground Upwork tool names and workflows in the currently published MCP
  interface.
- **Do not add tool, action, or parameter-value inventories to this
  repository.** The MCP server is self-describing — `search_tools` lists what
  an account can use and `get_tool_help` returns any tool's live actions and
  full parameter schema. A hand-maintained copy goes stale quickly, and a stale
  table gets trusted over the real schema, which is worse than having none.
  Document only what a single tool call cannot reveal: identifier families,
  categories of action that finish on upwork.com, role routing by journey,
  cross-tool ordering, and status vocabulary that misleads if read literally.
  Prefer a stated rule over an enumerated list, so a newly added tool or enum
  value is covered without an edit here.
- Do not document rate limits, quotas, or other enforcement thresholds. Steer
  behavior instead: honor `retry_after_seconds`, avoid tight retry loops, and
  note which categories of call are metered more tightly.
- Require explicit confirmation before writes and a separate confirmation for
  final draft submission.
- Never include credentials, access tokens, customer data, private company
  information, or copied participant content.

## Reporting security issues

Do not open a public issue for a vulnerability. Follow `SECURITY.md` instead.
