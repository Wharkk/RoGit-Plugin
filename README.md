> ⚠️ **Deprecation notice:** This plugin will be deprecated and archived in favor of **RoGit CLI**, which is currently in development.
>
> See [Known Limitations](#known-limitations) for the current plugin's constraints.

![RoGit Logo](images/Small_RoGit_Icon_Semi_Focused.png)

# RoGit: GitHub integration for Roblox Studio

RoGit is a Luau plugin for syncing scripts and supported instances between Roblox Studio and GitHub. Review changes, commit selected files, pull updates, and manage branches from Studio through the GitHub REST API. No local Git installation or external sync process is required.

## Features

- **Push and pull:** select changed files for a commit, or use Quick Push with confirmation. Pull compares local files with cached and newly fetched remote content and offers whole-file conflict resolution.
- **Review changes:** Luau syntax highlighting, line diffs, commit and file history, and search across serialized project files.
- **GitHub workflow:** create and select branches, request branch merges, create and list pull requests and releases, and view workflow runs and recent commit activity.
- **Project setup:** choose which supported services to sync and exclude instances by name. Optional auto-backup checks for changes every five minutes and pushes through the commit workflow while connected and idle.
- **Request handling:** cached blobs are reused by SHA, tree writes are batched, and collapsed metadata panels defer their requests. These reduce requests; GitHub and Studio limits still apply.

## Installation

1. Install [RoGit from the Roblox Creator Hub](https://create.roblox.com/store/asset/79063996456836/RoGit).
2. Open Roblox Studio and click **RoGit** in the plugin toolbar.

Source code and version history are available in this repository and on the [releases page](https://github.com/Wharkk/RoGit-Plugin/releases).

## Quick Start

### Connect a project

1. Create a GitHub repository with an initial commit, such as a README, or use an existing repository.
2. Create a [personal access token](https://github.com/settings/tokens). The plugin's setup guide uses a classic token with the `repo` scope; your account must have access to the repository.
3. In RoGit, enter the repository owner, name, branch, and token, then click **Connect**. The first-time guide covers welcome, repository creation, and token creation before opening this connection panel.
4. Use **Exclude Services** to omit services you do not want to sync. **Remember token** is optional and stores the token in Studio plugin settings.

To import an existing RoGit-formatted project, choose **Clone Repository** and enter `owner/repo` or its GitHub URL. Clone applies the serialized files to the open place; use an empty place for a new import.

### Commit changes

1. Make changes in Studio and review the **Changed Files** list.
2. Select the files to include and enter a commit message.
3. Click **Commit & Push** and check the output for the result.

### Pull updates

Click the pull button to fetch remote files and apply changes. Files detected as changed both locally and remotely open in the conflict resolver, where you choose **Accept Mine** or **Accept Theirs** for each file.

Pull preserves detected local edits when applying remote file content, but remote deletions are handled separately and can remove locally edited instances. Save a separate copy of the place before pulling or cloning. See [Known Limitations](#known-limitations).

## Serialization

Scripts use `.server.luau`, `.client.luau`, or `.luau` files. Supported instance properties, attributes, and tags use JSON. Instances with children use a directory containing an `_init` file; script metadata uses `.meta.json` sidecars when needed.

```text
ServerScriptService/
  MyScript.server.luau
  MyScript.meta.json
  MyFolder/
    _init.json
    ChildModule.luau
Workspace/
  Model/
    _init.json
    Part.json
    WeldConstraint.json
README.md
```

- Property coverage is defined in [PropertyMap.luau](ServerScriptService/RoGit/Init/PropertyMap.luau). The format does not capture every property or replace a complete place backup.
- Supported object references, including `PrimaryPart`, `Part0`/`Part1`, attachments, and `Adornee`, are resolved after instances are applied.
- File paths sanitize instance names and disambiguate duplicate siblings with numbered suffixes. Original names are stored in JSON or script metadata when renamed.
- `ServerStorage/README` can supply the repository's root `README.md`.
- A `_gitignore` ModuleScript under a synced service can return a list of exact instance names to exclude within that service. This is not Git's pattern-based `.gitignore` format.
- Downloaded blobs normalize CRLF and CR line endings to LF. The line-diff viewer also ignores trailing whitespace on each line.

## Known Limitations

- **Serialization coverage:** Terrain and Camera instances are skipped, and only the properties listed in `PropertyMap` are serialized. Asset references do not embed the referenced assets.
- **Object references and names:** references use dot-separated instance paths. Names containing dots and duplicate sibling names can make reference resolution ambiguous. Filename disambiguation does not remove this limitation.
- **Packages:** PackageLink restoration is incomplete. The import path attempts to load a package asset and can replace its parent model; do not rely on it to preserve package links, versions, or local package edits.
- **Pull and conflict handling:** resolution selects an entire local or remote file; there is no per-line merge. Remote deletion reconciliation does not check for local edits and has limitations with renamed or duplicate instances. Pull and clone attempt Studio undo recordings, but these are not a guarantee of atomic application or recovery after an error.
- **Branch selection:** selecting a branch changes the remote target and refreshes its cached files; it does not check out that branch into the place. Wait for ongoing operations to finish before changing branches.
- **Incomplete downloads and large projects:** a truncated tree response or failed blob download can leave an incomplete remote file set. Pull can continue with that set, including deletion reconciliation. Caching and batching do not guarantee successful synchronization of large projects; review the output and keep a separate place backup.

## License and Author

[MIT License](LICENSE). Created by **wharkk**.

[Portfolio](https://wharkk-developer.framer.website) · [Release history](https://github.com/Wharkk/RoGit-Plugin/releases)
