# Pirate Radio Launcher CLI Reference

The Pirate Radio Launcher includes a built-in command-line interface. When the executable is run with a subcommand, it operates in CLI mode. When run without arguments, it launches the graphical interface as usual.

The CLI shares the same configuration files as the GUI. Accounts, settings, and installed mods are accessible from either interface.

## Executable Name

The binary name depends on your platform:

| Platform | Executable |
|----------|------------|
| Windows | `launcher.exe` |
| Linux | `launcher` |

## Global Options

| Option | Description |
|--------|-------------|
| `--output <FORMAT>` | Output format. Accepted values: `human` (default), `json`. The `json` format produces machine-parseable output suitable for scripting and piping into other tools. |
| `--version` | Print the version and exit. |
| `--help` | Print help information. |

## Commands

- [account](#account) -- Manage accounts
- [server](#server) -- List and inspect servers
- [config](#config) -- Read and write configuration
- [mod](#mod) -- Manage mods
- [launch](#launch) -- Launch the game
- [version](#version) -- Show the current version

---

## account

Manage accounts used for authentication and server access. Accounts are shared between the GUI and CLI.

### account list

List all configured accounts.

```
launcher account list
```

### account add

Add a new account. The required options depend on the account type.

```
launcher account add <NAME> --type <TYPE> [OPTIONS]
```

| Argument / Option | Required | Description |
|-------------------|----------|-------------|
| `<NAME>` | Yes | Display name for the account. |
| `--type <TYPE>` | Yes | Account type. One of: `xyz`, `kocitycc`, `raw`. |
| `--auth-url <URL>` | For `xyz`, `kocitycc` | Authentication API base URL. |
| `--news-url <URL>` | For `xyz`, `kocitycc` | News API URL. |
| `--ip <IP>` | For `raw` | Server IP address. |
| `--port <PORT>` | For `raw` | Server port number (1-65535). |

Examples:

```
launcher account add "My KOCity.cc" --type kocitycc --auth-url https://api.kocity.cc --news-url https://api.kocity.cc/news
launcher account add "My Server" --type raw --ip 192.168.1.100 --port 23600
```

### account remove

Remove an account by name.

```
launcher account remove <NAME>
```

### account login

Log in to an account. For `xyz` and `kocitycc` accounts, this opens the authentication URL in the default browser and prompts for the auth code on the command line. Raw accounts do not require authentication.

```
launcher account login <NAME>
```

### account logout

Clear stored credentials for an account.

```
launcher account logout <NAME>
```

### account info

Display detailed information about an account, including authentication status.

```
launcher account info <NAME>
```

### account import

Import an account from a JSON file. The file format matches the GUI import format.

```
launcher account import <PATH>
```

The JSON file must contain at minimum a `name` and `type` field. See [account add](#account-add) for the fields required by each account type.

---

## server

List and inspect game servers.

### server list

List available servers. When no account is specified, servers from all configured accounts are shown.

```
launcher server list [--account <NAME>]
```

| Option | Description |
|--------|-------------|
| `--account <NAME>` | Only show servers for this account. |

### server info

Show detailed information about a specific server.

```
launcher server info <SERVER_ID> --account <NAME>
```

| Argument / Option | Required | Description |
|-------------------|----------|-------------|
| `<SERVER_ID>` | Yes | The server ID (as shown in `server list`). |
| `--account <NAME>` | Yes | The account to use for the lookup. |

---

## config

Read and write launcher configuration. The configuration is stored as JSON files in the platform's application data directory and is shared between the GUI and CLI.

### Configuration directory

| Platform | Path |
|----------|------|
| Windows | `%APPDATA%\de.tandashi.pirateradioclient\config\` |
| macOS | `~/Library/Application Support/de.tandashi.pirateradioclient/config/` |
| Linux | `~/.config/de.tandashi.pirateradioclient/config/` |

### config keys

Show all valid configuration keys and their descriptions.

```
launcher config keys
```

The following keys are available:

| Key | Description |
|-----|-------------|
| `gameDirectory` | Path to the game installation directory. |
| `language` | Game language. Valid values: `en`, `fr`, `de`, `es`, `it`, `pt`, `ru`, `pl`, `ja`, `ko`, `zh`. |
| `clientLogLevel` | Log level passed to the game client on launch. Valid values: `info`, `note`. |
| `modding.cacheDirectory` | Path to the directory where downloaded mods are cached. |

### config get

Read a single configuration value.

```
launcher config get <KEY>
```

### config set

Write a configuration value.

```
launcher config set <KEY> <VALUE>
```

Examples:

```sh
# Windows
launcher config set gameDirectory "C:\Games\KnockoutCity"

# Linux
launcher config set gameDirectory /home/user/games/knockoutcity
```

```
launcher config set language de
```

### config list

Display all configuration values.

```
launcher config list
```

---

## mod

Manage mods: browse available mods, install, uninstall, and apply them to the game. The mod list is fetched from the [PirateRadio-Mods](https://github.com/KnockoutCity-Pirate-Radio/PirateRadio-Mods) repository.

Mods can be identified by either their UUID or their name (case-insensitive).

Prerequisites: `gameDirectory` and `modding.cacheDirectory` must be configured before installing or applying mods. See [config set](#config-set).

### mod list

List available mods from the remote repository. Each entry shows whether it is currently installed.

```
launcher mod list
```

To show only installed mods:

```
launcher mod list --installed
```

### mod info

Show detailed information about a mod.

```
launcher mod info <ID_OR_NAME>
```

### mod install

Download and install a mod. The mod archive is verified against its SHA256 hash before being stored in the cache directory.

```
launcher mod install <ID_OR_NAME>
```

### mod uninstall

Remove a mod from the local cache.

```
launcher mod uninstall <ID_OR_NAME>
```

### mod apply

Apply all installed mods to the game directory. This patches the game files with the installed mods. If the mods have not changed since the last apply, the operation is skipped.

```
launcher mod apply
```

---

## launch

Launch the game on a specific server. This generates the required session token (for authenticated accounts), applies any installed mods, and opens the game through Steam.

```
launcher launch <SERVER_ID> --account <NAME>
```

| Argument / Option | Required | Description |
|-------------------|----------|-------------|
| `<SERVER_ID>` | Yes | The server ID to connect to (as shown in `server list`). |
| `--account <NAME>` | Yes | The account to authenticate with. Must be logged in first. |

The game language and log level are read from the current configuration.

---

## version

Print the current version of the launcher.

```
launcher version
```

---

## JSON Output

All commands that produce output support the `--output json` flag for machine-parseable output. The flag can be placed before or after the subcommand.

```
launcher --output json account list
launcher account list --output json
launcher --output json server list --account "KOCity.cc"
launcher --output json config list
launcher --output json mod list
```

When using JSON output, informational messages are suppressed and only the structured data is written to stdout. Errors are always written to stderr regardless of the output format.
