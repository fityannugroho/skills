---
name: git-sync
description: Synchronize git local with the remote.
author: fityannugroho
license: MIT
---

# Git Sync

Make every git branches in local consistent with the remote.

## When to use

- When requested by the user.
- When you know a pull request has been merged.
- When the local repository is out of sync with the remote.

## What to do

- Examine the local state first.
- Synchronize the local with the remote. Pull the remote changes.
- Clean up any obsolete branches that are no longer tracked on the remote.

## What not to do

- Be careful NOT to delete any local branches that are not yet pushed to the remote.
- DO NOT add new remote branches to the local. Sync the existing branches only.
