# Genesis — Base MCP Catalog

Genesis uses this catalog when creating agents. For each new agent, Genesis selects
the relevant MCPs from this list based on the agent's domain and responsibilities.

## MCP Registry

### Filesystem & Documents

| ID | MCP | Install | Auth | Description |
|----|-----|---------|------|-------------|
| `filesystem` | @modelcontextprotocol/server-filesystem | `npx -y @modelcontextprotocol/server-filesystem /path` | none | Read/write/search files with directory access control |
| `document-loader` | awslabs.document-loader-mcp-server | `uvx awslabs.document-loader-mcp-server@latest` | none | Extract text from PDF, DOCX, XLSX, PPTX, images |
| `pdf-reader` | @sylphx/pdf-reader-mcp | `npx @sylphx/pdf-reader-mcp` | none | Advanced PDF extraction: text + images + metadata, parallel |
| `markdownify` | markdownify-mcp (zcaceres) | `pnpm install && pnpm start` (clone) | none | Convert PDF, DOCX, HTML, images to clean Markdown |
| `pandoc` | mcp-pandoc (vivekVells) | `uvx mcp-pandoc` | none | Convert between Markdown, LaTeX, PDF, DOCX, EPUB, HTML via Pandoc |

### Productivity & Collaboration

| ID | MCP | Install | Auth | Description |
|----|-----|---------|------|-------------|
| `google-workspace` | google_workspace_mcp (taylorwilsdon) | `uvx workspace-mcp --tool-tier core` | OAuth2 | Gmail + Calendar + Drive + Sheets + Docs + Slides + Forms + Tasks |
| `ms365` | @softeria/ms-365-mcp-server | `npx -y @softeria/ms-365-mcp-server` | OAuth2 | 200+ tools: Outlook, Calendar, OneDrive, Excel, Planner, Contacts |

### Computation

| ID | MCP | Install | Auth | Description |
|----|-----|---------|------|-------------|
| `calculator` | mcp-server-calculator (githejie) | `uvx mcp-server-calculator` | none | Precise numerical calculations: arithmetic, scientific functions |

### Reasoning & Knowledge

| ID | MCP | Install | Auth | Description |
|----|-----|---------|------|-------------|
| `sequential-thinking` | @modelcontextprotocol/server-sequential-thinking | `npx -y @modelcontextprotocol/server-sequential-thinking` | none | Step-by-step reasoning with branching for complex problems |
| `memory` | @modelcontextprotocol/server-memory | `npx -y @modelcontextprotocol/server-memory` | none | Persistent knowledge graph in local JSON — entities and relations |

### Infrastructure & DevOps

| ID | MCP | Install | Auth | Description |
|----|-----|---------|------|-------------|
| `ssh` | @fangjunjie/ssh-mcp-server (classfang) | `npx -y @fangjunjie/ssh-mcp-server` | SSH key | Remote commands, file transfer, multi-server management |
| `git` | mcp-server-git (official) | `uvx mcp-server-git` | none | Git operations: status, diff, log, commit, branch |
| `github` | github-mcp-server (official) | Docker or `npx` | PAT | Full GitHub API: repos, issues, PRs, files, search |

## Relevance Matrix

When creating an agent, Genesis selects MCPs based on this matrix.
`x` = assign by default, `?` = propose to user, empty = skip.

| MCP | secretary | ceo | admin | hr | marketing | legal | finance | technician | engineer | site-mgr | estimator | backup | monitoring | doc-keeper |
|-----|-----------|-----|-------|-----|-----------|-------|---------|------------|----------|----------|-----------|--------|------------|------------|
| filesystem | | | x | | | x | | x | x | x | x | x | x | x |
| document-loader | | | x | | | x | | x | x | x | x | | | x |
| pdf-reader | | | x | | | x | | x | x | | x | | | x |
| markdownify | | | | | | | | | | | | | | x |
| pandoc | | | | | x | x | | | x | | | | | x |
| google-workspace | x | x | x | x | x | ? | x | | | ? | | | | |
| ms365 | x | x | x | x | x | ? | x | | | ? | | | | |
| calculator | | | | | | | x | x | x | | x | | | |
| sequential-thinking | | x | | | | x | x | | x | | | | | |
| memory | | x | | | | | | | x | | | | | |
| ssh | | | | | | | | | | | | x | x | |
| git | | | | | | | | | | | | x | x | x |
| github | | ? | | | | | | | | | | x | ? | x |

Legend:
- `x` = auto-assign (high relevance, always useful)
- `?` = propose to user (useful but depends on their setup)
- empty = not relevant for this agent

## How Genesis Uses This Catalog

### During Agent Creation (genesis-create-agent, Step 5)

1. Read this file
2. Find the new agent's name in the relevance matrix
3. Collect all MCPs marked `x` → auto-assign
4. Collect all MCPs marked `?` → ask user
5. For each selected MCP:
   a. Check if already configured in settings.json
   b. If not, add the MCP config
   c. Add `mcp__{id}__*` to permissions
   d. Reference in agent's tools list
6. If the agent doesn't match any column → ask user which MCPs to assign

### During Agent Update (genesis-update-agents)

1. Read this file
2. For each agent, compare current MCPs vs matrix
3. Flag missing MCPs (should have but doesn't)
4. Flag extra MCPs (has but shouldn't per matrix)
5. Propose changes for user approval

### Adding New MCPs to the Catalog

When `/genesis-discover-mcp` finds a new MCP the user approves:
1. Add it to the appropriate section above
2. Add a column or update the relevance matrix
3. Run `/genesis-update-agents` to propagate
