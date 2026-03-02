# Prettier Configuration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add Prettier configuration to the Shopify app template project with React Router v7 + TypeScript optimized settings, while keeping existing ESLint configuration unchanged.

**Architecture:** Create `.prettierrc.json` and `.prettierignore` files, add format scripts to package.json. Prettier operates independently from ESLint - no integration needed. Team can run both tools separately for code quality and formatting.

**Tech Stack:** Prettier 3.6.2 (already installed), npm scripts, JSON configuration

---

### Task 1: Create `.prettierrc.json` configuration file

**Files:**

- Create: `.prettierrc.json`

**Step 1: Create Prettier configuration file**

Create `.prettierrc.json` in project root with the following content:

```json
{
  "semi": true,
  "singleQuote": false,
  "trailingComma": "es5",
  "tabWidth": 2,
  "printWidth": 80,
  "endOfLine": "lf",
  "jsxSingleQuote": false,
  "bracketSpacing": true,
  "arrowParens": "always"
}
```

**Step 2: Verify JSON syntax is valid**

Run: `cat .prettierrc.json | python3 -m json.tool`
Expected: No errors, JSON is valid

**Step 3: Test Prettier can read the configuration**

Run: `npx prettier --check .prettierrc.json`
Expected: No errors (or "No matching files" which is ok)

**Step 4: Commit**

```bash
git add .prettierrc.json
git commit -m "feat: add Prettier configuration with React Router v7 + TypeScript settings"
```

---

### Task 2: Create `.prettierignore` file

**Files:**

- Create: `.prettierignore`

**Step 1: Create Prettier ignore file**

Create `.prettierignore` in project root with the following content:

```
# Dependencies
node_modules/

# Build outputs
dist/
build/
.app-cache/

# Environment
.env
.env.*
!.env.example

# Lock files
package-lock.json
yarn.lock
pnpm-lock.yaml

# Generated files
*.min.js
*.min.css

# Shopify specific
shopify.app.toml
.webpack-cache/

# Logs
*.log

# Cache
.cache/
*.cache

# IDE
.vscode/
.idea/
*.swp
*.swo
```

**Step 2: Verify ignore patterns work**

Run: `npx prettier --check --debug-check .prettierignore 2>&1 | head -20`
Expected: File processed, no critical errors

**Step 3: Commit**

```bash
git add .prettierignore
git commit -m "feat: add Prettier ignore patterns for dependencies and build outputs"
```

---

### Task 3: Add format scripts to package.json

**Files:**

- Modify: `package.json`

**Step 1: Add format scripts to package.json**

Add these two scripts to the `"scripts"` section in `package.json` (after the "lint" script on line 15):

```json
"format": "prettier --write \"**/*.{js,jsx,ts,tsx,json,css,scss,md}\"",
"format:check": "prettier --check \"**/*.{js,jsx,ts,tsx,json,css,scss,md}\""
```

The scripts section should look like this:

```json
"scripts": {
  "build": "react-router build",
  "dev": "shopify app dev",
  "config:link": "shopify app config link",
  "generate": "shopify app generate",
  "deploy": "shopify app deploy",
  "config:use": "shopify app config use",
  "env": "shopify app env",
  "start": "react-router-serve ./build/server/index.js",
  "docker-start": "npm run setup && npm run start",
  "setup": "prisma generate && prisma migrate deploy",
  "lint": "eslint --ignore-path .gitignore --cache --cache-location ./node_modules/.cache/eslint .",
  "format": "prettier --write \"**/*.{js,jsx,ts,tsx,json,css,scss,md}\"",
  "format:check": "prettier --check \"**/*.{js,jsx,ts,tsx,json,css,scss,md}\"",
  "shopify": "shopify",
  "prisma": "prisma",
  "graphql-codegen": "graphql-codegen",
  "vite": "vite",
  "typecheck": "react-router typegen && tsc --noEmit"
}
```

**Step 2: Verify package.json syntax is valid**

Run: `cat package.json | python3 -m json.tool > /dev/null && echo "Valid JSON"`
Expected: "Valid JSON"

**Step 3: Test the new scripts work**

Run: `npm run format:check`
Expected: Either "Checking formatting..." with no errors (if code is already formatted) or list of files that need formatting

**Step 4: Format the project files**

Run: `npm run format`
Expected: Prettier formats all matching files, shows list of formatted files

**Step 5: Verify no files were broken**

Run: `npm run typecheck`
Expected: No TypeScript errors

**Step 6: Commit**

```bash
git add package.json
git commit -m "feat: add Prettier format scripts to package.json"
```

---

### Task 4: Format existing project files

**Files:**

- Modify: All `.js`, `.jsx`, `.ts`, `.tsx`, `.json`, `.css`, `.scss`, `.md` files in project

**Step 1: Run format command**

Run: `npm run format`
Expected: Prettier formats files, may show list of changed files

**Step 2: Review the changes**

Run: `git status`
Expected: List of modified files

Run: `git diff --stat`
Expected: Summary of changes (should be mostly formatting)

**Step 3: Spot check a few files**

Review formatting changes in 2-3 files to ensure they look correct:

```bash
git diff app/routes/app._index.tsx | head -50
git diff package.json | head -50
git difference .eslintrc.cjs | head -50
```

Expected: Only formatting changes (spacing, quotes, commas), no logic changes

**Step 4: Run typecheck to ensure nothing broke**

Run: `npm run typecheck`
Expected: No TypeScript errors

**Step 5: Run linter to ensure code quality is maintained**

Run: `npm run lint`
Expected: No new linting errors (some may already exist)

**Step 6: Commit the formatting changes**

```bash
git add .
git commit -m "style: format existing code with Prettier"
```

---

### Task 5: Verify the setup works end-to-end

**Files:**

- Test: All Prettier functionality

**Step 1: Test format:check detects unformatted files**

Create a test file with intentional formatting issues:

```bash
cat > test-format.js << 'EOF'
const obj={name:"test",items:[1,2,3]}
function test(){return "unformatted"}
EOF
```

Run: `npm run format:check`
Expected: Error showing `test-format.js` needs formatting

**Step 2: Test format command fixes formatting**

Run: `npm run format`
Expected: `test-format.js` is reformatted

Verify the file is now formatted:

```bash
cat test-format.js
```

Expected: Properly formatted code with spacing and quotes

**Step 3: Clean up test file**

Run: `rm test-format.js`

**Step 4: Test that prettierignore works**

Run: `npx prettier --check node_modules/ 2>&1 | head -5`
Expected: No errors (node_modules is ignored)

**Step 5: Final verification**

Run: `npm run format:check`
Expected: No formatting errors (all project files are formatted)

**Step 6: Commit verification** (if any files were modified)

```bash
git add .
git commit -m "test: verify Prettier configuration works correctly"
```

---

## Summary

After completing all tasks, the project will have:

1. ✅ `.prettierrc.json` with React Router v7 + TypeScript optimized settings
2. ✅ `.prettierignore` with standard exclusion patterns
3. ✅ `npm run format` script to auto-format code
4. ✅ `npm run format:check` script to validate formatting
5. ✅ All existing project files formatted
6. ✅ Existing ESLint configuration unchanged

**Team workflow:**

- Developers run `npm run format` before committing
- CI/CD runs `npm run format:check` to validate formatting
- Both `npm run lint` and `npm run format` can be used together

---

## Testing Checklist

- [ ] `.prettierrc.json` is valid JSON and Prettier can read it
- [ ] `.prettierignore` correctly excludes node_modules and build outputs
- [ ] `npm run format` successfully formats files
- [ ] `npm run format:check` validates formatting
- [ ] Existing `.eslintrc.cjs` is unchanged
- [ ] `npm run lint` still works correctly
- [ ] `npm run typecheck` passes after formatting
- [ ] No logic changes in formatted files (only formatting)
