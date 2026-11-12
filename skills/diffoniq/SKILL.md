---
name: diffoniq
description: Compare folders, archives (ZIP, 7z, nested) and text files on Windows with Diffoniq, and preview what a folder sync would copy or delete. Use when the user asks what changed between two folders, releases, backups or archives, whether a backup is complete, which lines differ between two files, or what a mirror/backup run would do. Needs the Diffoniq MCP server (tools compare_tree, compare_file, sync_preview).
---

# Comparing with Diffoniq

Diffoniq's MCP server offers three read-only tools. None of them changes a
file; `sync_preview` only shows a plan.

| Question | Tool |
|---|---|
| What changed between these two folders, releases or archives? Is the backup complete? | `compare_tree` |
| Which lines differ between these two files? | `compare_file` |
| What would copying/mirroring this folder onto that one do? What would it delete? | `sync_preview` |

## Rules

- **Absolute Windows paths only** (`C:\...`, `\\server\share\...`). Resolve a
  relative path against the user's working folder yourself before calling.
- **Read the summary first.** Every answer starts with `summary` counting the
  whole comparison; the list (`entries`, `lines` or `actions`) is kept short
  (`limit`, and a size limit of about 20,000 characters). When
  `summary.omitted` is above 0, the answer's `hint` says how to see the rest -
  never present a cut list as complete.
- **Large trees: overview first, then zoom in.** `compare_tree` names the
  first-level `folders` with the most differences. To see one of them, call
  again with the same roots and `path` set to it (relative to both roots,
  e.g. `"Docs\\2024"`); `scope` then counts what lies below it. Prefer this
  to narrower `left`/`right`: a file moved between folders stays one pair
  instead of one entry on each side. A folder that exists on one side
  only is one entry with `items` and `bytes` - zoom into it with `path` if
  the user needs its files. When every difference lies in one chain of
  single folders, `foldersBelow` names its end and `folders` lists what is
  below it - zoom there directly.
- **Long lists: page, filter, exclude.** The hint names the `offset` of the
  next page (same filter). `only: ["renamed"]` lists the moved and renamed
  files. `summary.renamed` counts identical moved files too, which the
  default list leaves out (`renamedUnlisted`) - never report it as the
  number of listed pairs. `exclude` (like `["*.bak", ".obsidian"]`) leaves out noise on both
  sides; it is counted nowhere, so tell the user what you left out.
- **`notCompared` and `unverified` are not "identical".** Without
  `content: true`, files of equal size are not read and come back as
  `notCompared`. When the user needs certainty ("is the backup really the
  same?"), call again with `content: true`; Diffoniq's hash cache makes a
  repeat run cheap.
- **Files kept online only are not read.** A cloud placeholder (OneDrive,
  Dropbox and other services that keep files online only) is never
  downloaded by default: its pair stays `unverified` with `onlineOnly`,
  `summary.onlineOnly` counts them, `compare_file` refuses such a file and
  `sync_preview` skips it (reason `onlineOnly`). Tell the user how many
  were left out. Set `include_online_only: true` only after the user agreed
  to download them, and prefer a `path` that narrows the comparison.
  Recognized are the attributes of the Windows cloud files API (OneDrive,
  Dropbox and others) and, as a whole, a drive labelled "Google Drive",
  because streaming mode sets no attributes: such a root carries
  `cloudDrive`, and the hint names the side. A renamed Google Drive is not
  recognized; there `content: true` downloads the files - ask the user
  first. When the server's instructions say it is configured to read files
  kept online only (`DIFFONIQ_MCP_ONLINE=read`), the user allowed it once
  and for all: read them without asking.
- **Archives are folders here.** A ZIP or 7z file can be either side of
  `compare_tree`, also against a plain folder; archives inside archives are
  looked into.
- **`sync_preview` defaults to `update`** (copy new and changed files, never
  delete). Use `mode: "mirror"` only when the user means an exact copy, and
  point out the `delete` count before anything else - those files would be
  removed from the target.
- **What a sync would do comes only from `sync_preview`.** Never derive it
  from a `compare_tree` answer or from an earlier preview of other folders;
  call the tool and report its plan.
- The tools use Diffoniq's factory settings plus what the call asks for, not
  the user's window settings: `ignore_case`, `ignore_whitespace` and
  `ignore_trailing_blanks` of `compare_file` are off unless set. The answer's
  `options` list says what was in effect.
- `compare_file` cuts a line text at 400 characters (`leftChars`/`rightChars`
  give the full length). A binary file has a few info lines (size, hash,
  first differing offset) instead of text; an image has no lines at all.

## Typical calls

Release diff, only what was added or removed:

```json
{ "left": "D:\\releases\\v1", "right": "D:\\releases\\v2", "only": ["onlyLeft", "onlyRight"] }
```

Large tree, look into one folder of the overview:

```json
{ "left": "D:\\Projects", "right": "E:\\backup\\Projects", "path": "Customer\\2026" }
```

Backup check by content:

```json
{ "left": "D:\\Photos", "right": "\\\\nas\\backup\\Photos", "content": true }
```

What a mirror would do:

```json
{ "source": "D:\\work", "target": "E:\\backup\\work", "mode": "mirror" }
```

## Reporting back

Lead with the counts in plain words ("3 files changed, 12 only in v2,
nothing removed"), then name the entries that matter. For a file comparison,
quote the differing lines with their line numbers. If a tool returns
`isError`, its text names the reason (a missing folder, a relative path);
fix the call or tell the user.
