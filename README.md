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

## License

No license specified. Add a LICENSE file if you want to make the repo public with a specific license.
