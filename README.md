# GPT Image - Azure AI Foundry Skill

Two CLI scripts wrapped as an AI coding assistant skill: `generate.py` for text-to-image with `gpt-image-2`, and `edit.py` for editing or compositing up to 10 reference images — powered by Azure AI Foundry. Dependencies install on first run via `uv` and PEP 723 inline metadata. No glue code. No boilerplate.

## Requirements

| Requirement | Purpose |
|---|---|
| **[uv](https://docs.astral.sh/uv/)** | Runs the scripts and manages dependencies automatically. No `pip install` needed. |
| **Azure AI Foundry project** | A project with a `gpt-image-2` model deployment. You'll need the project endpoint URI and an API key. |
| **AI coding assistant** (optional) | Only needed if you want the assistant to invoke the skill automatically. The scripts also work standalone. |

### Installing uv

`uv` is a fast Python package manager from Astral. It replaces `pip` + `venv` + `pyenv` with a single tool that's roughly 10-100x faster. These scripts rely on `uv run` to fetch dependencies on first invocation, so it must be installed.

**macOS / Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows (PowerShell):**
```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Homebrew:**
```bash
brew install uv
```

**Verify the install:**
```bash
uv --version
```

If `uv` isn't on your PATH after install, restart your shell or follow the on-screen instructions from the installer. Full docs: <https://docs.astral.sh/uv/getting-started/installation/>.

## Setup

**1. Clone or copy the skill into your project or skills directory.**

```bash
git clone https://github.com/nicholasgriffintn/azure-gpt-image-2-skill.git
```

The skill itself lives in the `gpt-image/` subdirectory of this repo.

**2. Add your Azure AI Foundry credentials.**

The scripts read `AI_FOUNDRY_PRJ_URI` and `AI_FOUNDRY_PRJ_API_KEY` from the environment. You have two options:

**Option A - export in your shell** (recommended):

```bash
# Add to ~/.zshrc or ~/.bashrc so it sticks across sessions
export AI_FOUNDRY_PRJ_URI="https://<your-project>.services.ai.azure.com/api/v1"
export AI_FOUNDRY_PRJ_API_KEY="your-api-key-here"
```

Reload your shell (`source ~/.zshrc`) or open a new terminal.

**Option B - `.env` file in the skill directory** (good for project-scoped keys):

```bash
cp gpt-image/.env.example gpt-image/.env
# then edit gpt-image/.env and paste your credentials
```

The scripts look for shell-exported variables first and fall back to `.env`. Both can coexist; the shell export wins. `.env` is gitignored so your credentials won't end up in commits.

**3. Done.** First invocation triggers `uv` to install `openai` (takes a couple of seconds). Every subsequent run is instant.

## Usage

### As a Claude Code skill (recommended)

Just ask Claude normally. The skill description covers generation, editing, compositing, logos, ads, UI mockups, infographics, product mockups, virtual try-on, style transfer, lighting/weather changes, object removal, translation, sketch-to-render, comic strips, and more. Claude picks the right script, crafts the prompt, and saves the result.

### As standalone CLI scripts

Generate from scratch:
```bash
uv run gpt-image/scripts/generate.py \
  --prompt "A vintage 1960s travel poster for Kyoto in autumn" \
  --size 1024x1536 \
  --quality high \
  --output kyoto.png
```

Edit an existing image:
```bash
uv run gpt-image/scripts/edit.py \
  --prompt "Replace the sky with a dramatic thunderstorm. Keep everything else identical." \
  --images photo.jpg \
  --output photo-stormy.png
```

Multi-image compositing:
```bash
uv run gpt-image/scripts/edit.py \
  --prompt "Place the dog from Image 2 next to the woman in Image 1. Match lighting and scale." \
  --images scene.png dog.png \
  --output composite.png
```

Four logo variants at once:
```bash
uv run gpt-image/scripts/generate.py \
  --prompt "Original logo for 'Field & Flour', a local bakery. Warm, simple, timeless. Flat vector shapes, strong silhouette, plain background." \
  --n 4 \
  --output logo.png
# → logo_1.png, logo_2.png, logo_3.png, logo_4.png
```

## Flag reference

### `generate.py`

| Flag | Required | Default | Values |
|------|----------|---------|--------|
| `--prompt` | yes | - | The generation prompt |
| `--output` | no | `output.<format>` | File path for the result |
| `--size` | no | `1024x1024` | `auto` or any valid `WxH` (see [size constraints](#size-constraints)) |
| `--quality` | no | `medium` | `low`, `medium`, `high`, `auto` |
| `--output-format` | no | `png` | `png`, `jpeg`, `webp` |
| `--output-compression` | no | - | `0`-`100` (only with jpeg/webp) |
| `--n` | no | `1` | `1`-`4` variants |
| `--env-file` | no | `../.env` | Path to `.env` with `AI_FOUNDRY_PRJ_URI` and `AI_FOUNDRY_PRJ_API_KEY` |

### `edit.py`

All of the above, plus:

| Flag | Required | Default | Values |
|------|----------|---------|--------|
| `--images` | yes | - | 1-10 input image paths |

## Size constraints

`gpt-image-2` accepts any resolution that satisfies all of these rules:

- Both edges must be **multiples of 16**
- Max edge **≤ 3840 px**
- Long-to-short ratio **≤ 3:1**
- Total pixels **655,360 to 8,294,400**

Above **2560x1440** (2K / 3.69M pixels) is officially experimental. Expect more variance. The scripts print a warning but don't block.

Common sizes that satisfy every constraint:

| Use case | Size |
|----------|------|
| Square (default) | `1024x1024` |
| Portrait | `1024x1536` |
| Landscape | `1536x1024` |
| Widescreen / slide | `1536x864` |
| 2K / QHD | `2560x1440` |
| 4K landscape | `3840x2160` |
| 4K portrait | `2160x3840` |

Pass `--size auto` to let the model pick.

## Quality guide

| Setting | When to use |
|---------|-------------|
| `low` | Speed-critical, high-volume batches, previews, rapid iteration |
| `medium` | Default. Good balance of quality and speed. |
| `high` | Dense text, infographics, close-up portraits, identity-sensitive edits |
| `auto` | Let the model pick based on prompt |

## Prompting

See [`gpt-image/references/prompting-guide.md`](gpt-image/references/prompting-guide.md) for core patterns, and the "Workflow Patterns" section of [`gpt-image/SKILL.md`](gpt-image/SKILL.md) for copy-pasteable recipes covering infographics, logos, ads, UI mockups, photorealism, virtual try-on, sketch-to-render, scene compositing, and more.

Three principles that actually change output quality:

1. **Structure beats creativity.** Scene, subject, details, constraints, in that order. Prompts that wander produce images that wander.
2. **For edits, restate what to preserve on every iteration.** Skip this and the model drifts: faces change, layouts shift, text mutates. The "preserve" list matters more than the "change" list.
3. **Quote literal text and use `--quality high` for dense typography.** Otherwise you get close-but-not-verbatim output, usually on the most important word.

## Limitations & intentional omissions

A couple of things this skill deliberately doesn't do, and why.

- **No transparent backgrounds.** `gpt-image-2` does not currently support `background: "transparent"`. The API rejects the request. If you need alpha transparency, run a downstream background-removal step on the opaque PNG output.

- **No `input_fidelity` flag.** The API documentation is internally inconsistent on this parameter. The formal API reference states that `gpt-image-2` processes every image input at high fidelity automatically. Rather than expose a flag that might work or might 400, this skill omits it entirely.

## Other notes

- **Azure AI Foundry project required.** You need an Azure AI Foundry project with a `gpt-image-2` model deployed. If you get a 403 or 401, verify your endpoint URI and API key are correct.
- **Cost is token-based.** `gpt-image-2` pricing scales with quality × size. `low` is dramatically cheaper than `high`. See [Azure AI Foundry pricing](https://azure.microsoft.com/pricing/details/cognitive-services/openai-service/) for current rates.
- **Latency.** Complex prompts or high quality settings can take up to 2 minutes. `low` is typically a few seconds.

## Project layout

```
gpt-image/
├── SKILL.md                   # AI assistant instructions for using the skill
├── .env                       # Your Azure AI Foundry credentials (gitignored, you create this)
├── .env.example               # Template
├── scripts/
│   ├── generate.py            # Text → image
│   └── edit.py                # Image(s) → image
└── references/
    └── prompting-guide.md     # Detailed prompting patterns
```

## Packaging as a `.skill` file

This skill can be packaged as a zipped `.skill` bundle for portability. Useful if you want to move a pre-configured copy to another machine or share it with a teammate.

### 1. Install `uv`

`uv` is the runtime the scripts depend on.

**macOS / Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows (PowerShell):**
```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Homebrew:**
```bash
brew install uv
```

Verify:
```bash
uv --version
```

### 2. Get your Azure AI Foundry credentials

1. Go to [Azure AI Foundry](https://ai.azure.com) and open your project.
2. Navigate to **Management > Models + endpoints** and ensure you have a `gpt-image-2` deployment.
3. Copy the **Endpoint URI** and an **API key** from the deployment details.

### 3. Provide the credentials

The scripts check shell-exported `AI_FOUNDRY_PRJ_URI` and `AI_FOUNDRY_PRJ_API_KEY` first, then fall back to a `.env` file inside the skill directory. Pick whichever fits your flow:

**Shell export** (no `.env` needed; credentials travel with your shell, not the bundle):

```bash
export AI_FOUNDRY_PRJ_URI="https://<your-project>.services.ai.azure.com/api/v1"
export AI_FOUNDRY_PRJ_API_KEY="your-api-key-here"
# Add the same lines to ~/.zshrc or ~/.bashrc if you want them to persist.
```

**`.env` inside the skill** (credentials travel with the bundle; convenient for personal use, but strip before sharing):

```bash
cp gpt-image/.env.example gpt-image/.env
# Edit gpt-image/.env and paste your credentials
cat gpt-image/.env  # confirm
```

### 4. Zip into a `.skill` bundle

From the repo root:

```bash
zip -r gpt-image.skill gpt-image/
```

That produces `gpt-image.skill`, a single file you can move, share, or install into another Claude Code environment. Drop `.DS_Store` files from the archive while you're at it:

```bash
zip -r gpt-image.skill gpt-image/ -x "*.DS_Store"
```

**⚠ Sharing with others?** Strip your `.env` first so your credentials don't travel with the bundle. Recipients repeat step 3 on their own machine.

```bash
rm gpt-image/.env
zip -r gpt-image.skill gpt-image/ -x "*.DS_Store"
```

### 5. Install the bundle

Unzip the `.skill` into your Claude Code skills directory. Same as the normal [Setup](#setup) flow, just from a bundle instead of a clone:

```bash
unzip gpt-image.skill -d ~/.claude/skills/
```

Claude Code picks it up on next launch. If you stripped the `.env` in step 4, repeat step 3 on the target machine.

## Contributing

PRs welcome. If it misbehaves on a specific workflow, open an issue with the prompt, flags, and output.

## License

MIT. Do whatever you want.
