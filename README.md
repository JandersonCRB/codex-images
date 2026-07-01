# codex-images

A Claude Code plugin that generates UI image assets — hero images, avatars, illustrations, og:images, broken `<img>` placeholders, and so on — by delegating the actual generation to the [Codex MCP server](https://github.com/openai/codex).

Claude does the prompt engineering, picks dimensions, and wires the generated file into your codebase. Codex is treated as a **dumb image-generation pipe**: it takes a verbatim prompt, produces a PNG, saves it to disk, and reports back. No API keys, no post-processing — the only Codex-specific detail Claude passes is a pinned `model`, kept current so ChatGPT-OAuth logins don't hit deprecated defaults.

## What's in the box

- **Skill** (`codex-images`) — auto-triggers when Claude detects the current task involves producing or replacing an image asset (UI work, broken image references, explicit image-generation requests).
- **Slash command** (`/codex-images`) — manual invocation when you want to force it.

Both share the same workflow: analyse the context → decide what images are needed → write rich prompts → call the `codex` MCP tool → wire the saved files into your code.

## Prerequisites

You need the Codex MCP server registered in Claude Code:

```bash
# Install the codex CLI first (https://github.com/openai/codex)
claude mcp add --scope user codex -- codex mcp-server
claude mcp list   # should show: codex  ✓ Connected
```

That's it. No `OPENAI_API_KEY` setup required on Claude's side — Codex handles all of that internally.

## Install

### Option A — via marketplace (recommended)

```bash
claude plugin marketplace add https://github.com/JandersonCRB/codex-images
claude plugin install codex-images@codex-images
```

### Option B — local clone

```bash
git clone https://github.com/JandersonCRB/codex-images ~/dev/codex-images
claude plugin marketplace add ~/dev/codex-images
claude plugin install codex-images@codex-images
```

Restart Claude Code so the skill is picked up.

## Usage

**Auto-triggered** — just describe what you're building. Examples that fire the skill:

- "Build a SaaS landing page with a hero section."
- "The `<img src="/hero.png">` is broken, fix it."
- "Generate an og:image for the blog post."
- "Add an avatar for the testimonial card."

**Manual** — invoke explicitly:

```
/codex-images
```

…then describe what you want. The skill walks through context analysis, prompt construction, and file wiring.

### Reference images

Drop reference images on disk under your project root (e.g. `./refs/style.png`) and mention them in the request. The skill instructs Codex to read them and condition the generation on style/palette/logo/likeness.

## How it stays focused

The skill's prompt to Codex is locked down so Codex doesn't slip into agent mode:

- Use the image description **verbatim** — no rewriting, "improving", or translating.
- Don't explore the repo, run tests, or do anything other than generate + save.
- No post-processing (ImageMagick, PIL, text overlays).
- Fail loudly on errors instead of writing placeholders.

## License

MIT — see [LICENSE](./LICENSE).
