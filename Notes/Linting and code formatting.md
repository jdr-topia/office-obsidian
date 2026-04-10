---
created: 2026-04-10
tags: [dx, eslint, prettier, code-quality]
related: [ [[Developer experience (DX) MOC]], [[Git hooks and commit linting]], [[TypeScript in frontend]] ]
status: active
---

# Linting and code formatting

Linting catches code quality and potential error issues. Formatting ensures consistent style. Automating both eliminates an entire class of code review comments and prevents human inconsistency.

## ESLint — The Linter

ESLint statically analyzes your code to find problems: unused variables, missing dependencies in `useEffect`, accessibility violations, and more.

### Setup (Flat Config — ESLint 9+)

```bash
npm install -D eslint @eslint/js typescript-eslint eslint-plugin-react-hooks eslint-plugin-jsx-a11y
```

```js
// eslint.config.mjs
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import reactHooks from 'eslint-plugin-react-hooks';
import jsxA11y from 'eslint-plugin-jsx-a11y';

export default tseslint.config(
  js.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  {
    plugins: {
      'react-hooks': reactHooks,
      'jsx-a11y': jsxA11y,
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      ...jsxA11y.configs.recommended.rules,
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    },
  }
);
```

### Key Plugins for Frontend

| Plugin | What it Catches |
|---|---|
| `typescript-eslint` | TypeScript-specific best practices |
| `eslint-plugin-react-hooks` | Missing deps in `useEffect`, rules of hooks |
| `eslint-plugin-jsx-a11y` | Accessibility issues in JSX |
| `eslint-plugin-import` | Incorrect import paths, circular dependencies |

---

## Prettier — The Formatter

Prettier is an **opinionated** code formatter. It removes all formatting decisions from your team by enforcing a single consistent style.

> **Key insight**: Prettier and ESLint have different jobs. Prettier formats code style (spacing, quotes, line length). ESLint enforces code quality (logic, best practices). Run both.

### Setup

```bash
npm install -D prettier eslint-config-prettier
```

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

`eslint-config-prettier` disables ESLint rules that conflict with Prettier.

### VS Code Integration

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}
```

---

## Running in CI

Add lint and format checks to your CI pipeline so PRs cannot merge with violations:

```json
// package.json
{
  "scripts": {
    "lint": "eslint . --max-warnings 0",
    "format:check": "prettier --check .",
    "format:write": "prettier --write ."
  }
}
```

```yaml
# In GitHub Actions:
- name: Lint
  run: npm run lint
- name: Check formatting
  run: npm run format:check
```

---

## Related Topics
- [[Developer experience (DX) MOC]]
- [[Git hooks and commit linting]]
- [[TypeScript in frontend]]
- [[CI/CD for frontend]]
