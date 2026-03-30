# gitignore builder

An interactive terminal tool for building `.gitignore` files. Instead of writing rules by hand, you navigate your project tree, and everything is **ignored by default** — you only whitelist what you actually want git to track.

<img src="https://github.com/RetributionByRevenue/gitignore_builder/blob/main/1.png?raw=true">
<img src="https://github.com/RetributionByRevenue/gitignore_builder/blob/main/2.png?raw=true">

---

## How it works

When you run the tool it scans the current directory and shows every file and folder in a navigable list. Each item starts out marked as ignored (`✗` red). You use the keyboard to drill into folders and selectively allow items. When you're done, it writes a `.gitignore` that catches everything except what you approved.

The generated file uses a simple two-rule strategy:

```
/*                  ← ignore everything at root
!/src               ← un-ignore approved paths
!/src/main.py       ← un-ignore children too
```

Parent directories are automatically un-ignored so git can recurse into them — you don't have to think about it.

---

## Controls

| Key | Action |
|-----|--------|
| `↑` / `↓` | Move cursor up and down |
| `Space` | Toggle whitelist on/off |
| `→` / `Enter` | Drill into a directory |
| `←` / `Esc` | Go back up |
| `D` | Done — preview and write `.gitignore` |
| `Q` | Quit without writing anything |

### Status indicators

| Symbol | Colour | Meaning |
|--------|--------|---------|
| `✗` | Red | Ignored (default) |
| `✓` | Green | Fully whitelisted |
| `~` | Yellow | Partially whitelisted (some children allowed) |

Toggling a **directory** whitelists or removes the entire subtree at once. You can also drill in and pick individual files.

---

## Install & run

Requires Python 3.10+.

```bash
pip install blessed
python gitignore_builder.py
```

Run it from your **project root** — it always operates on the current working directory.

---

## Compile to a standalone binary (Nuitka)

Nuitka compiles the script and bundles `blessed` into a single native binary with no Python runtime required on the target machine. You need to compile once per target platform.

**Linux / macOS**

```bash
pip install nuitka blessed
nuitka --onefile --include-package=blessed gitignore_builder.py -o gitignore
```

**Windows**

```bash
pip install nuitka blessed
nuitka --onefile --include-package=blessed gitignore_builder.py -o gitignore.exe
```

### Install globally (Linux / macOS)

After compiling, move the binary somewhere on your `$PATH` so you can call it from any project:

```bash
sudo mv gitignore /usr/local/bin/gitignore
sudo chmod +x /usr/local/bin/gitignore
```

Then from any project directory just run:

```bash
gitignore
```

### Install globally (Windows)

Move `gitignore.exe` to a folder that is on your `PATH`, for example `C:\Windows\System32`, or add its folder to your user `PATH` via System Settings.

---

## Using the tree command with your .gitignore

Once your `.gitignore` is written, you can use `tree` to visualise exactly what git sees — virtual environments, build artefacts, and anything else you ignored will be completely absent from the output.

```bash
tree -a --gitignore
```

The `-a` flag includes hidden files (like `.gitignore` itself and `.env`), and `--gitignore` tells tree to respect all `.gitignore` rules. The result is a clean view of only the files git will actually track.

Example output for a typical Python project:

```
.
├── .gitignore
├── 123
│   └── test.txt
├── a.txt
└── gitignore_builder.py

2 directories, 4 files
```

`.venv/` and anything else you left red in the builder won't appear at all.

### Install tree

| Platform | Command |
|----------|---------|
| macOS | `brew install tree` |
| Ubuntu / Debian | `sudo apt install tree` |
| Fedora / RHEL | `sudo dnf install tree` |
| Windows (winget) | `winget install tree` |

> **Note:** `--gitignore` support requires tree v2.1.0 or newer. Check your version with `tree --version`. The default tree shipped with macOS is too old — install a fresh copy via Homebrew.

### Alternative: git ls-files

If tree isn't available, `git ls-files` is the ground truth — it lists every file git is actually tracking after your first commit:

```bash
git ls-files
```

---

## Notes

- `.git/` is always hidden and never appears in the list.
- The `.gitignore` file itself is written after the session ends. If you want git to track it, either re-run the tool and whitelist `.gitignore`, or force-add it once: `git add -f .gitignore`.
- To verify what git will actually track after writing the file: `git ls-files` or `tree --gitignore` (requires tree v2.1+).
