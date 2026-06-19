# Contributing Guide

This project is in the early stages of development. External contributions are not yet accepted, but the internal development workflow is documented here.

## Branch Strategy

- `main` — stable, always releasable
- `develop` — integration branch for ongoing development
- `feature/<name>` — feature branches
- `fix/<name>` — bug fixes
- `docs/<name>` — documentation changes

## Commit Message Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>
```

Common `type` values: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.

Examples:
- `feat(editor): add vertical layout to NSTextView`
- `docs(design): update class diagram in detailed design`

## Pull Requests

1. Branch off `develop` into a `feature/...` branch.
2. Implement and test.
3. Open a PR following the template.
4. Merge after review.

## Issues

Use the appropriate template for bug reports and feature requests.

## Documentation Changes

Documents under `docs/` are versioned per design phase. For significant changes, append a change log entry to the top of the affected document.
