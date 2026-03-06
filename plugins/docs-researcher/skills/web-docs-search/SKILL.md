---
name: web-docs-search
description: Search strategies and source hierarchies for finding external library documentation on the internet. Used internally by the documentation-researcher-agent. Provides query patterns for npm, PyPI, crates.io, MDN, and GitHub.
user-invocable: false
disable-model-invocation: true
---

# Web Documentation Search

## 1. Purpose

Finding accurate, up-to-date documentation for external libraries and tools requires targeted search strategies. Generic queries return tutorials and blog posts — useful, but often outdated or inaccurate. This skill provides a hierarchy of authoritative sources and proven query patterns to find canonical documentation quickly.

## 2. Documentation Source Hierarchy

Always prefer sources in this order — stop as soon as you find a sufficient answer:

| Priority | Source | When to use |
|----------|--------|-------------|
| 1 | Official documentation site | All cases — highest trust |
| 2 | npm / PyPI / crates.io package page | Package overview, README, links to docs |
| 3 | GitHub repository (`/docs`, `/wiki`, `README.md`) | When official site is lacking or missing |
| 4 | MDN Web Docs | Browser APIs, Node.js built-ins, Web standards |
| 5 | Stack Overflow | Error resolution, edge cases, usage patterns |

**Never use tutorials, blog posts, or Medium articles as a primary source.** They may document outdated APIs.

## 3. Search Query Patterns

### Finding the official docs site

```
[library name] official documentation
[library name] api reference
[library name] docs site:[known-docs-domain]
```

Examples:
```
lodash official documentation
@tanstack/react-query api reference
zod schema validation docs
```

### Finding a specific method or API

```
[library name] [method name] documentation
[library name] [class name] [method] api
site:[docs-domain] [method name]
```

Examples:
```
lodash debounce documentation
react-query useInfiniteQuery options
site:axios-http.com create instance
```

### npm packages (JavaScript / TypeScript)

```
site:npmjs.com [package-name]
[package-name] npm readme
```

Or use Bash for instant metadata:
```bash
npm info <package> homepage        # official docs URL
npm info <package> version         # latest stable version
npm info <package> description     # one-line summary
npm info <package> repository      # GitHub repo URL
```

### Python packages

```
site:pypi.org [package-name]
[package-name] python documentation
[package-name] readthedocs
```

### Rust crates

```
site:docs.rs [crate-name]
[crate-name] rust documentation
```

### MDN Web Docs (browser/Node.js built-ins)

```
site:developer.mozilla.org [API name]
mdn [method name]
```

Examples:
```
site:developer.mozilla.org Promise.all
mdn Array.prototype.flatMap
mdn fetch api
```

### Error-driven searches

When debugging an error from an external library:

```
"[exact error message]" [library name]
[library name] [error type] [method name]
[library name] [version] breaking change [method]
```

Examples:
```
"TypeError: Cannot read properties of undefined" axios
lodash TypeError map undefined
react-query v5 breaking changes useQuery
```

## 4. Fetching Documentation Pages

### URL patterns to prefer

When using WebFetch, target these URL patterns over homepage/landing pages:

| Type | Pattern |
|------|---------|
| API reference | `/api/[method]`, `/reference/[name]` |
| Guide section | `/docs/[topic]`, `/guide/[topic]` |
| GitHub docs | `github.com/[org]/[repo]/blob/main/docs/[file].md` |
| GitHub raw | `raw.githubusercontent.com/[org]/[repo]/main/docs/[file].md` |
| npm README | `npmjs.com/package/[name]` — scroll to README |
| PyPI README | `pypi.org/project/[name]/` |
| Rust docs | `docs.rs/[crate]/latest/[crate]/[module]/struct.Type.html` |

### GitHub raw markdown

For GitHub repositories, fetch raw markdown for clean, structured content:
```
# Replace:
https://github.com/org/repo/blob/main/docs/api.md

# With:
https://raw.githubusercontent.com/org/repo/main/docs/api.md
```

### Changelog and version-specific docs

When version differences matter:
- Check `CHANGELOG.md` in the GitHub repository
- Look for `/docs/migration` or `/docs/upgrading` pages
- Check versioned URL patterns: `/v4/api/`, `/docs/5.x/`

## 5. Quick Reference: Common Ecosystems

### JavaScript / TypeScript (npm)

```bash
# Quick package metadata
npm info <package>

# Find homepage (docs URL)
npm info <package> homepage

# Check if package exists
npm view <package> name
```

Common doc sites:
- React: `react.dev`
- Next.js: `nextjs.org/docs`
- lodash: `lodash.com/docs`
- axios: `axios-http.com/docs`
- @tanstack/react-query: `tanstack.com/query/latest/docs`
- zod: `zod.dev`
- TypeScript: `typescriptlang.org/docs`
- Vitest: `vitest.dev/api`

### Python (PyPI)

Common doc sites:
- Django: `docs.djangoproject.com`
- FastAPI: `fastapi.tiangolo.com`
- Pydantic: `docs.pydantic.dev`
- SQLAlchemy: `docs.sqlalchemy.org`
- Requests: `requests.readthedocs.io`

### Browser / Node.js Built-ins

Always use MDN:
- `developer.mozilla.org/en-US/docs/Web/API/[API]`
- `developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/[type]`

## 6. When Documentation Is Insufficient

If official docs don't cover the specific question:

1. Check the GitHub **Issues** tab — search for the error message or method name
2. Check GitHub **Discussions** or **Wiki**
3. Search Stack Overflow with `[library] [version] [topic]`
4. Look for community documentation: `awesome-[library]` repos, official Discord/Slack links

Always report the source URL and note if the answer came from unofficial sources.
