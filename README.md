# vish

vish is a tiny helper script to quickly create or edit small executable shell scripts in a system-friendly location. It's written to be Termux-friendly (uses `$PREFIX` when available) and safe for quick edits — if you leave the file essentially empty, vish will remove it.

## Features / behavior

- Places scripts in `$PREFIX/local/bin` (Termux) or `/usr/local/bin` by default.
- If the target file does not exist, vish creates it with a `#!/usr/bin/env bash`-style shebang (it uses `${PREFIX:-/usr}` so Termux uses its prefix).
- Opens the file in `nano` at line 3 (so your cursor is after the shebang and a blank line).
- After editing, if the file contains only the shebang (or is otherwise empty / contains just the shebang), vish deletes it to avoid leaving empty files around.
- Otherwise, it makes the script executable (`chmod +x`) and prints the saved file path.

> Note: The script uses `nano` as the editor. If you prefer another editor, edit the script to replace `nano` with your editor of choice.

## Installation

1. Copy `vish` to a directory in your PATH, e.g. `/usr/local/bin` (or let the script create it under `$PREFIX/local/bin`).
2. Make it executable:
   - chmod +x /usr/local/bin/vish

Or, from this repository root:

- Option A (install to /usr/local/bin):
  - sudo cp vish /usr/local/bin/vish
  - sudo chmod +x /usr/local/bin/vish

- Option B (Termux install):
  - cp vish "$PREFIX/local/bin/vish"
  - chmod +x "$PREFIX/local/bin/vish"

## Usage

Basic usage:

```
vish myscript
```

- This will create or open `$PREFIX/local/bin/myscript` (or `/usr/local/bin/myscript`).
- The file will already have a `#!/usr/bin/bash`-style shebang inserted.
- After editing, if you leave the file essentially empty vish will delete it; otherwise it will make it executable.

Examples:

- Create a small helper:
  - vish greet
  - Add: echo "Hello from greet" ; save and exit — vish will make it executable.

- Abandon creating a script:
  - vish temp
  - Save without adding content — the script will be removed.

## Companion tool: cash

Once you have `vish`, a great companion command is **`cash`** — a safe way to view the source code of any command in your PATH (perfect for inspecting scripts you created with `vish`).

`cash` shows the script source for text-based commands, refuses to dump binaries by default (to avoid garbling your terminal), and has a `-f/--force` flag if you really need to view a binary.

### Quick install & create cash with vish itself

```bash
vish cash
```

Then paste the full script code (from below) into the editor, save, and exit. `vish` will make it executable automatically.

```bash
#!/usr/bin/env bash

set -euo pipefail

SCRIPT_NAME="$(basename "$0")"

show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [-f | --force] <command>
       $SCRIPT_NAME -h | --help

Display the source code of a command/script if it is a text file.

By default, refuses to display compiled binaries to prevent garbled terminal output.

Options:
  -f, --force      Force display even if the file is binary (may mess up terminal)
  -h, --help       Show this help message and exit

Examples:
  $SCRIPT_NAME cash          # Show this script (safe)
  $SCRIPT_NAME ls            # Warn: ls is binary
  $SCRIPT_NAME -f ls         # Force display ls binary (dangerous!)
EOF
}

FORCE=0

while [[ $# -gt 0 ]]; do
    case "$1" in
        -h|--help)
            show_help
            exit 0
            ;;
        -f|--force)
            FORCE=1
            shift
            ;;
        -*)
            echo "Error: Unknown option: $1" >&2
            echo "Try '$SCRIPT_NAME --help' for usage." >&2
            exit 1
            ;;
        *)
            break
            ;;
    esac
done

if [[ $# -ne 1 ]]; then
    echo "Error: Exactly one command name required." >&2
    echo "Try '$SCRIPT_NAME --help' for usage." >&2
    exit 1
fi

COMMAND="$1"
COMMAND_PATH="$(which "$COMMAND" 2>/dev/null)" || {
    echo "Error: '$COMMAND' not found in PATH." >&2
    exit 1
}

if [[ ! -f "$COMMAND_PATH" ]]; then
    echo "Error: '$COMMAND_PATH' is not a regular file." >&2
    exit 1
fi

if [[ ! -r "$COMMAND_PATH" ]]; then
    echo "Error: '$COMMAND_PATH' is not readable." >&2
    exit 1
fi

is_text_file() {
    if file -b --mime-type "$COMMAND_PATH" | grep -qE '^text/|^application/x-shellscript|^inode/x-empty|^application/x-empty'; then
        return 0
    fi
    if file "$COMMAND_PATH" | grep -qiE 'script|interpreted'; then
        return 0
    fi
    return 1
}

if is_text_file || [[ $FORCE -eq 1 ]]; then
    echo "# Source of '$COMMAND' ($COMMAND_PATH):"
    echo
    cat "$COMMAND_PATH"
else
    echo "'$COMMAND' appears to be a compiled binary or non-text file."
    echo "Displaying it with 'cat' would produce garbage and could mess up your terminal."
    echo
    echo "Location: $COMMAND_PATH"
    echo
    echo "To force display anyway (not recommended), use:"
    echo "  $SCRIPT_NAME -f $COMMAND"
    echo
    echo "Tip: Use 'file \"$COMMAND_PATH\"' to inspect the file type."
    exit 1
fi
```

Now you can do things like:

```bash
cash cash     # view the cash script itself
cash greet    # view the greet helper you made earlier
cash ls       # safe warning (binary)
```

## Notes & portability

- Uses `$PREFIX` if set (Termux). Otherwise uses `/usr/local/bin`.
- Uses `nano` and `perl` — both must be available on the system. If `perl` isn't available, you can replace the cleanup check with a small `sed`/`awk` or Bash-only test.
- The script assumes you have permission to write into the target directory; use `sudo` when necessary for system locations (or install into a local bin directory).

## Security & safety

- The script uses `chmod +x` on files created — be careful what content you save into system-wide bin directories.
- If you want to vet scripts before making them executable, edit the script to skip `chmod +x` and perform manual checks.

## Contributing / improvements

- Add support for a configurable editor via `$EDITOR` or an environment variable.
- Add an option to choose shebang (e.g., `#!/usr/bin/env bash` vs an explicit interpreter path).
- Add a `--dry-run` or `--no-chmod` flag.
