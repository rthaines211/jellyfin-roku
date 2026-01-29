# Jellyfin Roku Enhanced — Fork Strategy

This is a feature-enhanced fork of the [official Jellyfin Roku client](https://github.com/jellyfin/jellyfin-roku).

## Branch Structure

| Branch | Purpose |
|--------|---------|
| `master` | Tracks upstream `jellyfin/jellyfin-roku:master` — never commit directly |
| `enhanced-fork` | Main development branch for enhanced features |
| `feature/*` | Individual feature branches off `enhanced-fork` |
| `upstream-pr/*` | Clean branches for PRs back to upstream (cherry-picked, no fork-specific changes) |

## Syncing with Upstream

```bash
# Fetch latest upstream
git fetch upstream

# Update local master
git checkout master
git merge upstream/master

# Rebase enhanced-fork onto updated master
git checkout enhanced-fork
git rebase master
```

Do this regularly to stay current with official bug fixes and features.

## Upstreaming Changes

When a feature is mature and suitable for the official repo:

```bash
# Create clean branch from current upstream master
git checkout -b upstream-pr/feature-name master

# Cherry-pick only the relevant commits (no fork-specific changes like manifest rename)
git cherry-pick <commit-hash>

# Push and create PR against jellyfin/jellyfin-roku
git push origin upstream-pr/feature-name
gh pr create --repo jellyfin/jellyfin-roku --base master
```

## Fork-Specific Changes

These changes exist only in the fork and should **never** be upstreamed:

- `manifest` — title changed to `jellyfin-enhanced` (allows coexistence with official app)
- `FORK.md` — this file
- Any branding/icon changes

## Sideloading

The renamed channel title means both the official Jellyfin app and this enhanced version can be installed on the same Roku device simultaneously for comparison testing.
