# 🍪 Agent API Cookiecutter

My base template for new AI agent projects (FastAPI + clean/layered architecture, Docker, uv, ruff, pytest).
Forked from [neural-maze/agent-api-cookiecutter](https://github.com/neural-maze/agent-api-cookiecutter) — upstream credit to Miguel Otero Pedrido.

> **This repo is a template, not a runnable project.** Never edit code inside `{{cookiecutter.package_name}}/` expecting to run it — generate a project first, then work in the generated copy.

## Table of Contents

- [Quick start](#quick-start)
- [Template variables](#template-variables)
- [What gets generated](#what-gets-generated)
- [What each folder is for](#what-each-folder-is-for)
- [After generating: first steps](#after-generating-first-steps)
- [Known gaps in the template](#known-gaps-in-the-template)
- [Editing this template](#editing-this-template)
- [License](#license)

## Quick start

```bash
# 1. Install cookiecutter once (pipx / uv tool keep it isolated)
pip install -U cookiecutter          # or: uv tool install cookiecutter

# 2. Generate a project from this template
cookiecutter <this-repo-url>         # from GitHub
cookiecutter path/to/AGENT-API-COOKIECUTTER   # from a local clone

# Skip the prompts and accept defaults / pass values directly
cookiecutter <this-repo-url> --no-input package_name=my-agent project_name="My Agent"
```

The generated project lands in the **current directory**, in a folder named after `package_name`. Use `-o <dir>` to put it elsewhere. **Don't generate inside the template folder itself.**

## Template variables

Defined in [cookiecutter.json](cookiecutter.json). You are prompted for each one.

| Variable | Default | Notes |
|---|---|---|
| `full_name` | author name | Used for attribution |
| `package_name` | `agent-api` | Folder / distribution name. Use hyphens: `esp-design-agent` |
| `project_name` | `Agent API` | Human-readable title used in the generated README |
| `project_slug` | derived from `package_name` | Python import name (hyphens → underscores). **Must be a valid module name** — `hooks/pre_gen_project.py` aborts otherwise |
| `project_short_description` | template blurb | Goes into `pyproject.toml` and the README |
| `first_version` | `0.1.0` | Initial version in `pyproject.toml` |

## What gets generated

Example for `package_name=my-agent`:

```
my-agent
├── Dockerfile
├── Makefile                  # docker compose shortcuts
├── README.md
├── pyproject.toml            # deps, ruff config, CLI entry point
├── docker-compose.yaml
├── .env.example              # copy to .env and fill in
├── .pre-commit-config.yaml
├── .github/workflows/test.yml
├── data/                     # local data (docs, CSVs, ...)
├── notebooks/                # exploration
├── static/
├── src
│   └── my_agent
│       ├── config.py         # pydantic-settings
│       ├── application       # use cases / services
│       │   ├── chat_service
│       │   ├── evaluation_service
│       │   ├── ingest_documents_service
│       │   └── reset_memory_service
│       ├── domain            # core logic, no framework deps
│       │   ├── exceptions.py
│       │   ├── utils.py
│       │   ├── memory
│       │   ├── prompts
│       │   └── tools
│       └── infrastructure    # adapters to the outside world
│           ├── api           # FastAPI app (main.py, models.py)
│           ├── db
│           ├── llm_providers
│           ├── mcp_clients
│           └── monitoring
└── tests
    ├── conftest.py
    └── test_my_agent.py
```

## What each folder is for

The layering is the point: dependencies flow **infrastructure → application → domain**, never the other way.

- **`domain/`** — the agent's brain, independent of FastAPI/LLM SDKs.
  - `prompts/` system prompts · `tools/` functions the agent can call · `memory/` conversation state · `exceptions.py`, `utils.py`.
- **`application/`** — one sub-folder per use case (chat, evaluation, document ingestion, memory reset). Orchestrates domain pieces (e.g. the LangGraph graph lives in `chat_service/`).
- **`infrastructure/`** — everything that talks to the outside world.
  - `api/` FastAPI routes + request/response models · `llm_providers/` OpenAI/Groq/etc. clients · `db/` vector store / SQL · `mcp_clients/` MCP servers · `monitoring/` Opik/Langfuse/logging.
- **`config.py`** — settings loaded from `.env` via pydantic-settings.
- **`data/`, `notebooks/`, `static/`** — non-code assets, kept out of `src/`.

Delete any empty folder you don't need — the `.gitkeep` files are placeholders only.

## After generating: first steps

```bash
cd my-agent
git init && git add . && git commit -m "Initial commit"

cp .env.example .env          # add your API keys (never commit .env)
uv sync                       # create .venv and install deps (incl. dev group)
uv run pre-commit install     # optional: lint on commit

uv run fastapi dev src/my_agent/infrastructure/api/main.py   # run the API locally
uv run pytest                 # run tests
uv run ruff check . && uv run ruff format .

make start-project            # docker compose up --build -d
make stop-project
```

Typical build order for a new agent: `config.py` → `domain/prompts` → `domain/tools` → `application/chat_service` (graph/agent) → wire it into `infrastructure/api/main.py` → tests.

## Known gaps in the template

Things I've hit and should fix once here instead of in every project:

- **No `cli.py`, but `pyproject.toml` declares `<slug> = "<slug>.cli:app"`.** Installing the package works, but running the script fails until you add `src/<slug>/cli.py` (a Typer `app`). Either add a stub to the template or remove the `[project.scripts]` entry.
- **No agent dependencies.** Add what you need (`langgraph`, `langchain`, `langchain-openai`, `copilotkit`, ...) to `pyproject.toml`.
- **`.env.example` is minimal** (`OPENAI_API_KEY` only). Add `LLM_PROVIDER`, `LLM_MODEL`, etc. as projects need them.
- **Placeholder text** in the generated README (`Features: TODO`) and `classifiers = [# TODO]`.
- `[tool.ty]` section in `pyproject.toml` is only a commented stub.

## Editing this template

- Files under `{{cookiecutter.package_name}}/` are **Jinja templates**. `{{ cookiecutter.x }}` is replaced at generation time; a literal `{{`/`{%` in the file (e.g. GitHub Actions `${{ ... }}`) must be wrapped in `{% raw %}...{% endraw %}`.
- `hooks/pre_gen_project.py` validates input before anything is written; `hooks/post_gen_project.py` runs after.
- The root `pyproject.toml`, `main.py`, `Pipfile` and `uv.lock` belong to the template repo itself (upstream's dev tooling), not to generated projects.
- Test changes by generating into a throwaway dir **outside** this repo: `cookiecutter . --no-input -o /tmp/test-out`, then inspect and delete it.
- To pull upstream improvements: `git remote add upstream https://github.com/neural-maze/agent-api-cookiecutter.git && git fetch upstream && git merge upstream/main --allow-unrelated-histories`.

## License

MIT — see [LICENSE](LICENSE).
