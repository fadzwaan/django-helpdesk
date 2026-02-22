# How to run django-helpdesk (Python 3.12.4 + uv)

This guide uses **Python 3.12.4** and **uv** for the virtual environment.

---

## 1. Prerequisites

- **Python 3.12.4** — [python.org](https://www.python.org/downloads/) or `py -3.12` (ensure you have 3.12.4 installed)
- **uv** — Install: `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"`  
  Or: `pip install uv`
- **Node.js + Yarn** (for front-end assets) — [nodejs.org](https://nodejs.org/), then `npm install -g yarn`

---

## 2. Update the repository (optional)

From the project root:

```powershell
git pull origin main
```

---

## 3. Create and use the uv virtual environment

All commands below are from the **project root** (where `pyproject.toml` and `manage.py` are).

### Create the venv (first time only)

Use Python 3.12.4 explicitly:

```powershell
uv venv --python 3.12.4
```

If you already have a venv and want to recreate it with 3.12.4:

```powershell
uv venv --python 3.12.4 --clear
```

### Install/update dependencies (use this after clone or pull)

```powershell
uv sync --all-extras --dev --group test --group teams
```

- `--all-extras`: includes optional features (e.g. teams).
- `--dev --group test --group teams`: dev and test deps plus pinax-teams for the demo.

This uses the project’s `.venv` and updates all packages to the latest versions allowed by `pyproject.toml`.

### Activate the venv (optional)

You can run commands with `uv run` without activating. If you prefer an activated shell:

**PowerShell:**

```powershell
.\.venv\Scripts\Activate.ps1
```

**Cmd:**

```cmd
.\.venv\Scripts\activate.bat
```

Then you can use `python` and `pip` directly instead of `uv run`.

---

## 4. Run the demo app

The **demodesk** project is the demo. Run everything from the **project root**.

### Option A: Using Make (if you have `make`, e.g. Git Bash, WSL, or Chocolatey)

One-time setup + run:

```bash
make rundemo
```

This will:

1. Run `yarn install` and copy front-end assets.
2. Run `uv sync` with extras and groups.
3. Run migrations and load demo data.
4. Start the server on **http://localhost:8080**.

### Option B: Manual steps (PowerShell, no Make)

Run these in order from the project root.

**Step 1 – Front-end dependencies and static files**

```powershell
yarn install
```

Then copy vendor assets. If you have **Git Bash** or **WSL**:

```bash
make static-vendor
```

If you don’t have `make`, you can skip this for a quick test; the app may run with some missing CSS/JS (e.g. DataTables). For a full demo, use Git Bash/WSL for `make static-vendor` or see the Makefile `static-vendor` target to replicate it.

**Step 2 – Python dependencies (if not already done)**

```powershell
uv sync --all-extras --dev --group test --group teams
```

**Step 3 – Database and demo data**

```powershell
uv run python manage.py migrate --noinput
uv run python manage.py loaddata emailtemplate.json
uv run python manage.py loaddata demo.json
```

**Step 4 – Start the server**

Because of a dependency issue with pinax-teams (missing `pkg_resources`), run with teams mode disabled:

```powershell
$env:HELPDESK_TEAMS_MODE_ENABLED="false"; uv run python manage.py runserver 8080
```

Open: **http://localhost:8080**

### Demo login

- **URL:** http://localhost:8080  
- **Username:** `admin`  
- **Password:** `Pa33w0rd`

---

## 5. Useful commands (all from project root)

| Task              | Command |
|-------------------|--------|
| Run dev server    | `$env:HELPDESK_TEAMS_MODE_ENABLED="false"; uv run python manage.py runserver 8080` |
| Run tests         | `uv run quicktest.py` or `make test` |
| Create migrations | `uv run python manage.py makemigrations` |
| Apply migrations  | `uv run python manage.py migrate` |
| Django shell      | `uv run python manage.py shell` |
| Install/update deps | `uv sync --all-extras --dev --group test --group teams` |

---

## 6. Troubleshooting

- **“No module named 'demodesk'”**  
  Run commands from the **project root** (where `manage.py` and `pyproject.toml` are), not from `demodesk/` or `src/`.

- **“No module named 'helpdesk'”**  
  Run `uv sync --all-extras --dev --group test --group teams` so the `helpdesk` package is installed from `src/`.

- **Python version**  
  Check: `uv run python --version`. To use 3.12.4: `uv venv --python 3.12.4 --clear` then run `uv sync` again.

- **Port 8080 in use**  
  Use another port: `uv run python manage.py runserver 8081`
