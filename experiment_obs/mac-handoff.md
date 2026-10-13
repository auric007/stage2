# stage2 notebook connection handoff

## Local Git setup - 2026-10-14

- Project: /Users/ashutosh.bharti/Documents/Obsidian Vault/Projects/stage2
- Notebook: experiment_obs/
- GitHub: https://github.com/auric007/stage2
- Remote: https://auric007@github.com/auric007/stage2.git
- Branch: main
- Company identity: Ashutosh Bharti <ashutosh.bharti@auricai.in>
- Authentication: repository/account-scoped macOS Keychain; no token stored in files.

Server import confirmed on 2026-10-14. The original notebook files were fetched
from remote main into `/sfs/stage2/experiment_obs`, preserving local research.
The server's separate notebook Git metadata is `/sfs/markdown-sync/git/stage2`.
The company sync controller is configured for a 30-second wait after each cycle;
inspect `bash /sfs/stage2/scripts/experiment_sync_service.sh status` for live results.
This task does not install/change the Mac's own schedule or power on the GPU instance.

## Server use

For a new server checkout, clone the repository at a deliberately chosen path.
For an existing code directory, first inspect Git root, status and remote; do not
blindly replace its remote or overwrite work. Keep experiment_obs/ at the intended
project root and preserve unrelated code and credentials.

## Mac working folder

Only experiment_obs/ should be checked out into Obsidian. Include ordinary nested
files and dotfiles without a Markdown-only filter. Stop on conflicts or unrelated
outgoing commits, and never force-push. GitHub history and local saves are separate
from verified synchronization. Persistent agent loading remains a separate setup.
