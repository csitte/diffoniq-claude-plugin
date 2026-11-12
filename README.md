# Diffoniq plugin for Claude

Lets Claude compare folders, archives and text files on your Windows PC with
[Diffoniq](https://diffoniq.com/), the folder, archive and file comparison for
Windows (formerly WDiff32), and preview what a folder synchronization would do.
Ask "what changed between these two release folders?", "is my backup on the NAS
complete?" or "what would mirroring this folder onto the USB drive delete?" and
Claude answers from Diffoniq's comparison instead of reading the files itself.

## Requirements

- Windows 10 or 11, 64-bit.
- **Diffoniq 2.3 or later installed** ([download](https://diffoniq.com/download/)).
  A portable copy works too when the environment variable `DIFFONIQ_HOME` names
  its folder. The tools work in the free 30-day evaluation; Diffoniq is
  commercial software with a one-time licence.
- Node.js 18 or later, for `npx`.

## What the plugin contains

- **An MCP server entry** (`.mcp.json`) that starts `npx -y diffoniq-mcp@2.4.0`.
- **A skill** (`skills/diffoniq/SKILL.md`) that tells Claude which tool answers
  which question, how to read the summary first, and when to ask before
  downloading files kept online only.

## Tools

All three tools are read-only. None of them changes, copies or deletes a file;
`sync_preview` only shows a plan.

| Tool | What it does |
|---|---|
| `compare_tree` | Compares two folders or archives (ZIP, 7z, archives inside archives): what differs, what exists on one side only, what was renamed or moved |
| `compare_file` | Compares two text files: the lines that differ, with line numbers |
| `sync_preview` | What synchronizing a target folder would create, copy, rename or, in mirror mode, delete |

## What runs, and what goes where

- `npx` downloads the package [`diffoniq-mcp`](https://www.npmjs.com/package/diffoniq-mcp)
  in exactly version 2.4.0 from the npm registry. It is a single MIT-licensed
  JavaScript file with no dependencies, published by the maker of Diffoniq.
- That file looks up where Diffoniq is installed: it reads the
  `App Paths\Diffoniq.exe` registry key that the Diffoniq installer writes,
  through `powershell.exe` and, if PowerShell is locked down, `reg.exe`. Then it
  starts `DiffoniqC.exe /mcp` from that folder (or from `DIFFONIQ_HOME`) and
  passes stdin, stdout and stderr through.
- `DiffoniqC.exe` is part of Diffoniq and signed with it. As an MCP server it
  answers over stdin and stdout only, reads the files and folders a tool call
  names, and sends no usage data.
- Files that a cloud service keeps online only (OneDrive, Dropbox and others
  using the Windows cloud files API, and a drive labelled "Google Drive") are
  not read unless a call asks for it, because reading downloads them; the skill
  tells Claude to ask you first.

## Support

- Guide: <https://diffoniq.com/guides/mcp-server/>
- Help: <https://diffoniq.com/help/automation.htm>
- Mail: support@diffoniq.com

## License

The files in this repository (plugin manifest, MCP entry, skill, this README)
are MIT-licensed, see [LICENSE](LICENSE). Diffoniq itself is commercial software
under its own licence terms: <https://diffoniq.com/eula/>.
