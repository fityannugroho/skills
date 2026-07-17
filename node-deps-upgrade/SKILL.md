---
name: node-deps-upgrade
description: Upgrade node.js project dependencies easily using npm-check-updates.
author: fityannugroho
license: MIT
---

# Node.js Dependencies Upgrade

Easily upgrade your Node.js project dependencies using the `npm-check-updates` tool. This skill helps you identify and update outdated packages in your `package.json` file.

## When to use

Use this when you need to:
- Know which dependencies are outdated in your Node.js project.
- Keep your project dependencies up-to-date with the latest versions.
- Upgrade Node.js project dependencies to specific versions.

## Quick start

- Ensure `npm-check-updates` is installed globally. Run `ncu -v` to check if it's available.
- If not, install it using project's preferred Node.js package manager (npm/pnpm/yarn/bun):
  ```bash
  npm i -g npm-check-updates
  ```
- Run `ncu` to check for outdated dependencies.
- Use `ncu -u` to upgrade the `package.json` file with the latest versions.
- Run the package manager's install command to update the lock file and install the new versions.
- Finally, run project checks/tests to ensure everything works as expected.

## Tips and tricks

- Run `ncu --help` to see all available options and flags for customizing the upgrade process.
- Use flags like `--target`, `--filter`, and `--reject` to control which dependencies to upgrade.
- Avoid use interactive mode (`--interactive`, `-i`), especially in automated scripts.
- Beware of potential breaking changes when upgrading major versions of dependencies. Use any relevant tools to find the documentation/changelogs/migration guides of the packages being upgraded.
- Documentation: [npm-check-updates](https://www.npmjs.com/package/npm-check-updates)
