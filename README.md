# Upwork Agent Plugin

Portable agent workflows for hiring and finding work on Upwork, backed by the
[Upwork MCP server](https://mcp.upwork.com/mcp).

This repository follows the
[Agent Plugins specification v1.0.0](https://agent-plugins.org/specification).
Compatible clients discover the remote MCP server from `mcp.json` and Agent
Skills from immediate subdirectories of `skills/`.

## Included components

- `mcp.json` connects to Upwork over the Streamable HTTP transport.
- `skills/write-job-post` guides clients from a rough need to a reviewed job
  post, then through publishing, updating, and closing it.
- `skills/hire-on-upwork` guides clients from candidates to a contract through
  freelancer search, proposal review, invitations, and offers.
- `skills/write-proposal` creates tailored, evidence-based proposals and guides
  freelancers through Connects, attachments, boosting, and submission review.
- `skills/upwork-workflows` defines the cross-cutting conventions: account
  selection, tool discovery, role routing, identifier handling, draft-confirm
  chains, pagination, and error handling.

## Install

Install this repository with any client that supports the portable Agent
Plugins format. The client is responsible for Upwork authentication and
authorization; the plugin does not contain credentials.

After installation, ask the agent to connect to Upwork and describe the task,
for example:

- "Help me turn this project brief into an Upwork job post."
- "Write a proposal for this Upwork job using my real work history."
- "Show my active Upwork contracts and any milestones needing attention."

The agent must request confirmation before write operations. Draft-based
workflows require a second confirmation before the final marketplace action.

## Structure

```text
.
├── plugin.json
├── mcp.json
└── skills/
    ├── hire-on-upwork/
    │   └── SKILL.md
    ├── upwork-workflows/
    │   └── SKILL.md
    ├── write-job-post/
    │   └── SKILL.md
    └── write-proposal/
        └── SKILL.md
```

## Validate

Validate the manifests against the canonical schemas:

```bash
curl -fsS https://agent-plugins.org/schemas/1.0.0/plugin.schema.json \
  -o /tmp/agent-plugin.schema.json
curl -fsS https://agent-plugins.org/schemas/1.0.0/mcp.schema.json \
  -o /tmp/agent-plugin-mcp.schema.json

npx --yes ajv-cli@5.0.0 validate --spec=draft2020 \
  -s /tmp/agent-plugin.schema.json \
  -d plugin.json

npx --yes ajv-cli@5.0.0 validate --spec=draft2020 \
  -s /tmp/agent-plugin-mcp.schema.json \
  -d mcp.json
```

Validate each skill with the
[Agent Skills reference validator](https://agentskills.io/specification):

```bash
python3 -m pip install skills-ref==0.1.1

for skill in skills/*/; do
  agentskills validate "$(pwd)/${skill%/}"
done
```

## Contributing

Keep portable components in the standard fixed locations:

- `plugin.json` at the repository root
- `mcp.json` at the repository root
- `skills/<skill-name>/SKILL.md`, one level below `skills/`

Do not add secrets, access tokens, or fixed authorization headers to
`mcp.json`.
