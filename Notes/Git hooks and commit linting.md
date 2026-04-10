---
created: 2026-04-10
tags: [dx, git, husky, conventional-commits, lint-staged]
related: [ [[Developer experience (DX) MOC]], [[Linting and code formatting]], [[CI/CD for frontend]] ]
status: active
---

# Git hooks and commit linting

Git hooks run scripts at specific points in the Git workflow (before a commit, before a push). Combined with commit message linting, they enforce code quality and a consistent commit history automatically — with no discipline required.

---

## Husky — Git Hooks Made Easy

Husky makes it trivially easy to set up Git hooks in a Node.js project.

```bash
npm install -D husky
npx husky init
```

This creates a `.husky/` directory with a `pre-commit` file.

---

## lint-staged — Only Lint What Changed

Running ESLint and Prettier over the **entire codebase** on every commit is slow. `lint-staged` runs linters only on the files that are **staged for commit**.

```bash
npm install -D lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix --max-warnings 0", "prettier --write"],
    "*.{json,css,md}": ["prettier --write"]
  }
}
```

```sh
# .husky/pre-commit
npx lint-staged
```

**Result**: Every commit is guaranteed to be lint-free and correctly formatted.

---

## Commitlint — Enforcing Conventional Commits

The **Conventional Commits** spec standardizes commit message format, enabling automated changelogs and semantic versioning.

### Format
```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

### Types
| Type | Use When |
|---|---|
| `feat` | Introducing a new feature |
| `fix` | Patching a bug |
| `refactor` | Restructuring code without behavior change |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `docs` | Documentation changes only |
| `chore` | Build scripts, dependency updates |
| `ci` | CI/CD configuration changes |
| `style` | Formatting, whitespace (no logic change) |

### Examples
```
feat(auth): add social login with Google OAuth
fix(dashboard): correct pagination offset calculation
perf(images): switch to next/image for automatic optimization
chore(deps): upgrade react-query to v5
```

### Setup

```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

```js
// commitlint.config.js
export default { extends: ['@commitlint/config-conventional'] };
```

```sh
# .husky/commit-msg
npx --no -- commitlint --edit "$1"
```

### Benefits Beyond Style

- **`semantic-release`**: Reads commit types to auto-determine the next version (`patch`, `minor`, `major`) and publish to npm.
- **Automated changelogs**: Tools like `conventional-changelog` generate `CHANGELOG.md` from commit history.
- **Better `git log`**: Uniform, readable history makes bisecting bugs much easier.

---

## The Complete DX Pre-Commit Flow

```
git commit -m "feat(cart): add quantity selector"
  └─ husky pre-commit
       └─ lint-staged
            ├─ ESLint --fix on staged .ts/.tsx files
            └─ Prettier --write on staged files
  └─ husky commit-msg
       └─ commitlint validates message format
  └─ Commit succeeds ✅
```

---

## Related Topics
- [[Developer experience (DX) MOC]]
- [[Linting and code formatting]]
- [[CI/CD for frontend]]
