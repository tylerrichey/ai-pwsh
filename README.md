# `ai`

`ai` is a .NET global tool that uses OpenRouter by default, or any OpenAI-compatible chat-completions endpoint you configure, to turn natural-language goals into shell commands or answer questions about your current workspace.

It is cross-platform and targets your current shell — PowerShell, bash, or zsh — automatically.

## Install

Install the global tool from NuGet:

```bash
dotnet tool install --global Ai.Cli
```

Update to the latest published version:

```bash
dotnet tool update --global Ai.Cli
```

This puts the `ai` command on your PATH. It works the same on Windows, Linux, and macOS.

## Features

- `ai <goal...>` generates a shell command for your current shell, displays it, and prompts for Enter-to-execute.
- `ai -q <question...>` / `ai --question <question...>` prints a plain text answer instead of generating a command.
- `ai -r <follow-up...>` / `ai --resume <follow-up...>` continues from the last history entry, sending the prior conversation as context for a multi-turn exchange.
- `ai --bash <goal...>` generates a bash command body and prints it as `bash -lc "<command>"`.
- `ai --shell <target> <goal...>` generates a command for the specified shell (`powershell`, `bash`, `zsh`).
- `ai -x <goal...>` / `ai --execute <goal...>` generates a command, displays it, prompts for Enter-to-confirm, and runs it directly via the target shell.
- `ai -f <path> ...` / `ai --file <path> ...` includes up to 3 files as additional context for command generation or `-q` answers.
- `ai -hs` / `ai --history` shows recent history (most recent 50 entries). Append search tokens to filter: `ai -hs <terms...>`. Resume entries are shown with the `resume` label.
- `ai -nh` / `ai --no-history` skips recording the current invocation in history.
- `ai --models` lists available model IDs from the configured provider alphabetically.
- `ai --model <model-id> <goal...>` overrides the configured default model.
- `ai --version` prints the built tool version.
- `ai --timing <goal...>` prints timing information to stderr, including the AI call duration.

## Configuration

The tool reads JSON config from:

- Windows: `%USERPROFILE%\.config\ai\config.json`
- Unix: `$XDG_CONFIG_HOME/ai/config.json` or `~/.config/ai/config.json`

History is stored as JSONL (one entry per line) in the same directory:

- Windows: `%USERPROFILE%\.config\ai\history.jsonl`
- Unix: `$XDG_CONFIG_HOME/ai/history.jsonl` or `~/.config/ai/history.jsonl`

Supported keys:

```json
{
  "apiKey": "your-openrouter-api-key",
  "baseUrl": "https://openrouter.ai/api/v1/",
  "defaultModel": "openai/gpt-5-mini",
  "defaultShell": "bash"
}
```

`baseUrl` is optional. When omitted, `ai` defaults to `https://openrouter.ai/api/v1/`. For LM Studio, set `baseUrl` to `http://localhost:1234/v1/` and choose a local `defaultModel`.

The `defaultShell` key accepts `powershell`, `bash`, or `zsh`. When omitted, the default is platform-specific: PowerShell on Windows, bash on Linux, zsh on macOS.

Environment overrides:

- `OPENROUTER_API_KEY` overrides `apiKey`
- `OPENROUTER_BASE_URL` overrides `baseUrl`
- `--model` overrides `defaultModel`
- `--shell` or `--bash` overrides `defaultShell`

OpenRouter requires an API key. Local providers such as LM Studio may not. If no model can be resolved, `ai` exits with a setup error.

## Usage

Generate a command and run it (prompts for confirmation before executing):

```bash
ai list all files in the current directory, remove everything after the underscore and give me a distinct list
```

The generated command is displayed, and you are prompted:

```text
Press Enter to execute, any other key to cancel
```

Press `Enter` to run it, or any other key to cancel.

Generate and execute a command directly with `-x`:

```bash
ai -x list all files in the current directory
```

Generate a bash command wrapper:

```bash
ai --bash list all markdown files modified in the last day
```

Ask a question without generating a command:

```bash
ai -q what does src/Ai.Cli/AiApplication.cs do
```

Include files in the request context:

```bash
ai -q -f README.md -f src/Ai.Cli/AiApplication.cs summarize how the CLI behaves
```

`-f` accepts up to 3 files and shares a 12,000-character budget across all included file contents.

List models from the configured provider:

```bash
ai --models
```

Print the installed tool version:

```bash
ai --version
```

Print timing information while generating a command:

```bash
ai --timing list all files in the current directory
```

Continue from the last history entry (multi-turn conversation):

```bash
ai -q why would i want to use git lfs
ai -r i am not using github
ai -r what about for large video assets
```

Each `-r` invocation chains from the previous entry so the full conversation context is sent. Entries are saved to history with the `resume` kind and a back-link to the entry they continued from.

Search history for commands involving "dotnet":

```bash
ai -hs dotnet
```

Run a command without recording it to history:

```bash
ai -nh list all files in the current directory
```

## Build

Restore and test:

```bash
dotnet restore tests/Ai.Cli.Tests/Ai.Cli.Tests.csproj
dotnet test tests/Ai.Cli.Tests/Ai.Cli.Tests.csproj --no-restore
```

Pack the global tool:

```bash
dotnet pack src/Ai.Cli/Ai.Cli.csproj -c Release
```

Install from the local package output:

```bash
dotnet tool install --global --add-source ./src/Ai.Cli/bin/Release Ai.Cli
```

## Release

Pushing a tag matching `v*` (e.g. `v1.0.0`) triggers a GitHub Actions workflow that packs the tool and publishes the NuGet package to nuget.org. The tag name (minus the `v` prefix) is used as the package version.
