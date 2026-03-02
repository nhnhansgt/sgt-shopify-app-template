# Prettier Configuration Design

**Date:** 2025-03-02
**Status:** Approved
**Approach:** Option B - React Router v7 + TypeScript Modern

## Overview

Add Prettier configuration to the Shopify app template project with React Router v7 and TypeScript-optimized settings. Keep existing ESLint configuration unchanged.

## Design Decisions

### Configuration Philosophy

- Modern TypeScript/React Router v7 standards
- Double quotes (more common in React ecosystem)
- 80 character line width (TypeScript standard, better readability)
- ES5 trailing commas (balanced approach)

### Files to Create

#### 1. `.prettierrc.json` - Main Configuration

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

**Rationale:**

- `singleQuote: false` - Double quotes are more common in React ecosystem
- `trailingComma: "es5"` - Trailing commas in ES5-valid locations (objects, arrays)
- `printWidth: 80` - TypeScript standard, easier to read in code reviews
- `jsxSingleQuote: false` - Consistency with JS/TS files
- `bracketSpacing: true` - Better readability: `{ foo: bar }`
- `arrowParens: "always"` - Clearer intent: `(x) => x`

#### 2. `.prettierignore` - Exclude Patterns

Standard exclusions for:

- Dependencies (node_modules)
- Build outputs (dist/, build/)
- Environment files (.env)
- Lock files
- Generated/minified files
- Shopify-specific files (shopify.app.toml, .webpack-cache)
- Logs and cache directories
- IDE configuration

#### 3. `package.json` Scripts

Add two npm scripts:

- `format` - Auto-format all files
- `format:check` - Check formatting without changes (CI/CD)

### Integration Strategy

- **No ESLint changes** - Keep existing `.eslintrc.cjs` unchanged
- Prettier and ESLint operate independently
- Team can run both tools: `npm run lint` and `npm run format`

## File Extensions Covered

- JavaScript: `.js`, `.jsx`
- TypeScript: `.ts`, `.tsx`
- Data: `.json`
- Styles: `.css`, `.scss`
- Docs: `.md`

## Success Criteria

1. ✓ Prettier configuration file created
2. ✓ Prettier ignore file created
3. ✓ Format scripts added to package.json
4. ✓ Existing ESLint configuration unchanged
5. ✓ `npm run format` successfully formats files
6. ✓ `npm run format:check` validates formatting

## Future Considerations

- Optional: Add pre-commit hooks with Husky + lint-staged
- Optional: Integrate Prettier into CI/CD pipeline
- Optional: Add editor integration (VS Code settings)
