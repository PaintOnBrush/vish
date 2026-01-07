# vish

vish is a tiny helper script to quickly create or edit small executable shell scripts in a system-friendly location. It's written to be Termux-friendly (uses `$PREFIX` when available) and safe for quick edits — if you leave the file essentially empty, vish will remove it.

## Features / behavior

- Places scripts in `$PREFIX/local/bin` (Termux) or `/usr/local/bin` by default.
- If the target file does not exist, vish creates it with a `#!/usr/bin/env bash`-style shebang.
- Opens the file in `nano` at line 3 (cursor ready after shebang + blank line).
- After saving, if the file contains only the shebang (or is essentially empty), vish deletes it.
- Otherwise, it makes the script executable (`chmod +x`) and prints the path.

> Note: Uses `nano` as the editor. Change it in the script if you prefer something else.

## Installation

```bash
# System-wide
sudo cp vish /usr/local/bin/vish
sudo chmod +x /usr/local/bin/vish

# Termux / user-local
cp vish "$PREFIX/local/bin/vish"
chmod +x "$PREFIX/local/bin/vish"
```

## Usage

```bash
vish myscript        # create or edit ~/local/bin/myscript or /usr/local/bin/myscript
```

- Add your code, save and quit → script becomes executable.
- Save without adding anything → file is automatically removed.

## Companion tool: cash

A perfect companion for `vish` is **`cash`** — a safe way to view the source of any command in your `$PATH`.

It shows script source for text files, refuses to dump binaries by default (to protect your terminal), and offers `-f/--force` if you really need to see a binary.

### Install cash with vish itself

Just run:

```bash
vish cash
```

Then paste the contents of the `cash` file from this repository into the editor, save, and quit. `vish` will make it executable automatically.

Now you can:

```bash
cash cash        # view the cash script itself
cash myscript    # quickly inspect any script you made with vish
cash ls          # safe warning that ls is a binary
cash -f ls       # force view binary (not recommended)
```

The full, up-to-date `cash` source lives in this repository as `cash` (next to `vish` and this README).

## Notes & portability

- Detects `$PREFIX` for Termux compatibility.
- Requires `nano` (for editing) and `file` (used by the recommended `cash` tool).
- Assumes write permission to the target bin directory (use `sudo` when needed).

## Security & safety

- `vish` runs `chmod +x` automatically — only save code you trust in system-wide bin directories.

## Contributing / improvements

Ideas welcome:
- Support `$EDITOR` or `-e` flag for custom editor
- Configurable shebang options
- `--no-chmod` or `--dry-run` flags
- Better empty-file detection without `perl`

Enjoy quick scripting!
