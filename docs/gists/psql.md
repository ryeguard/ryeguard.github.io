# `psql` on macOS

## Installation

`psql` is the command line interface for PostgreSQL. It can be installed, along with the rest of PostgreSQL, with Homebrew:

```bash
brew install postgresql
```

However, you may want to go lightweight and install only `psql` without the rest of PostgreSQL. You can do this with:

```bash
brew install libpq
```

If doing this, Homebrew will not automatically the utilities to your PATH because if you were to install `postgresql` there'd be some conflict. Therefore, Homebrew will tell you how to do this by adding something like `export PATH="/usr/local/opt/libpq/bin:$PATH"` to your shell's configuration file (e.g., `~/.zshrc`). If you ever need to install `postgresql` in the future, you'll need to simply remove this line from your shell's configuration file.

You can also force Homebrew to link `libpq` to your PATH anyway with:

```bash
brew link --force libpq
```

If you do this, unlink it if you ever need to install `postgresql` in the future with:

```bash
brew unlink libpq
```

Also consider uninstalling `libpq` if `psql` is included in `postgresql`:

```bash
brew uninstall libpq
```
