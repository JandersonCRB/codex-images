# /codex-images — AI Image Generation via Codex

You are being asked to think carefully about whether the current feature,
task, or codebase change would benefit from generated images, and if so,
produce them by delegating to Codex via the `codex` MCP tool.

---

## Step 1 — Analyse the current context

Look at:
- The task or feature description (from the user's message or CLAUDE.md)
- Any open or recently edited files
- Referenced asset paths (img src, background-image, CSS url(), etc.)
- Filenames that suggest images are expected (e.g. hero.png, avatar.jpg)

Then answer internally:

| Question | Signals that say YES |
|---|---|
| Does this feature have a UI layer? | HTML, JSX, Vue, Svelte, CSS, mobile screens |
| Are image assets referenced but missing? | Broken img tags, placeholder paths, 404 assets |
| Would a real image meaningfully improve the feature? | Hero sections, product cards, onboarding, icons |
| Is the user explicitly asking for image generation? | Mentioned in the task description |

**If the answer to all questions is NO** — do not call the tool. Briefly tell the
user that no image generation is needed for this feature and continue with the
main task.

---

## Step 2 — Decide what images to generate

For each image needed:
1. Identify the **output path** relative to the project root (e.g. `public/images/hero.png`)
2. Infer the **visual style** from the surrounding code (check Tailwind theme, CSS variables, existing assets, README)
3. Write a **rich prompt** — at least 2–3 sentences covering: subject, style, palette, mood, intended use

### Prompt quality checklist
- ✅ Names the subject clearly ("a smiling developer at a laptop")
- ✅ Specifies art style ("flat vector illustration, minimal, clean lines")
- ✅ Mentions colour palette if inferable ("uses indigo and slate, white background")
- ✅ States the intended context ("for a SaaS hero section, 1536×1024")
- ❌ Avoids vague terms like "nice", "good", "modern" alone

---

## Step 3 — Generate the images via Codex

Call the `codex` MCP tool once per image. Pass a precise, self-contained
prompt that tells Codex exactly what to generate and where to save it.

Choose dimensions that match the use case:

| Use case | size |
|---|---|
| Square avatar / icon / thumbnail | `1024x1024` |
| Hero / banner / wide card | `1536x1024` |
| Mobile screen / tall card / poster | `1024x1536` |

Use `quality="high"` for final assets; `quality="medium"` for placeholders.

### Reference images (optional)

The `codex` MCP tool only accepts a text `prompt` — you can't attach binary
images to the call. To condition generation on reference images (style match,
logo placement, character consistency, etc.), pass them via the filesystem:

1. Make sure each reference image exists on disk **under the `cwd`** you'll
   pass to Codex. Common conventions: `./refs/`, `./design/references/`,
   or any existing asset folder. If the user provided an image path, use it
   as-is (verify it exists first).
2. List the reference paths explicitly in the prompt and tell Codex what role
   each one plays (style, palette, logo, layout, subject likeness, etc.).
3. Use relative paths from `cwd`, not absolute paths, so the prompt stays
   portable.

Example snippet to include inside the `prompt` field:

```
Reference images (read them from disk before generating):
- ./refs/style.png   — match this illustration style and color palette
- ./refs/logo.svg    — incorporate this logo in the bottom-right corner
- ./refs/subject.jpg — preserve the likeness of the person shown here
```

If no reference images are needed, omit this section entirely.

### Tool call template

Delegate the entire image generation to the `codex` MCP tool. Do NOT specify
a model, do NOT mention OpenAI APIs, do NOT pass API keys — Codex handles all
of that internally with whatever generator it has available. Your job is just
to describe the image and where to save it.

You also MUST NOT post-process the result yourself (no ImageMagick, no PIL,
no text overlays). Whatever Codex saves is the final asset.

```
codex(
  prompt="""
    Generate an image and save it to disk. That is your ONLY job.

    Scope rules — read carefully:
    - Do NOT think about, plan, or critique the request. The creative work
      and prompt engineering have already been done by Claude (the caller).
    - Do NOT rewrite, "improve", expand, translate, or summarize the image
      description below. Pass it to your image generator verbatim.
    - Do NOT explore the repository, read unrelated files, run tests, or
      take any action other than generating the image and writing it to
      the specified path (and reading reference images if listed below).
    - Do NOT ask clarifying questions. If something is genuinely impossible,
      report the error and stop.
    - Keep your reply short: just the saved path and byte count.

    Image description (use verbatim): <your rich prompt here>

    Save the resulting PNG to: <absolute or project-relative output path>

    Requirements:
    - Target dimensions (approximate): <e.g. 1536x1024>
    - Create any missing parent directories before writing.
    - Reply with the absolute path of the saved file and its size in bytes.
    - If image generation fails, report the exact error — do not write a
      placeholder.
  """,
  cwd="<absolute path to the project root>",
  sandbox="workspace-write",
  approval-policy="never"
)
```

After the call returns, verify the file actually exists at the reported path
before wiring it into the codebase. If Codex reports a failure, surface the
error to the user instead of silently moving on.

---

## Step 4 — Wire up the generated assets

After Codex confirms the file was saved:
1. Update any placeholder or broken `src` / `url()` references in the code to
   point at the newly generated file.
2. If the project uses an asset manifest or import system (Vite, webpack, Next.js
   `next/image`, etc.), use the appropriate import or path convention.
3. Add a brief comment noting the image is AI-generated so teammates know it can
   be replaced: `{/* AI-generated placeholder — replace with final asset */}`
4. Tell the user which images were generated, where they were saved, and
   whether they should be committed or .gitignored.

---

## Constraints

- Generate only images that are **directly relevant** to the current feature.
- Do not generate images if the task is backend-only, purely algorithmic, or
  involves no user-facing UI.
- Do not generate images the user already has (check if the file exists first).
- Respect the project's folder conventions for static assets.
- Always set `sandbox="workspace-write"` so Codex can write files to disk.
