## Agent skills

### Issue tracker

Issues and specs live as GitHub issues in cypherkunp/resume-builder. Use the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Five canonical labels, each string equal to its role: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `GLOSSARY.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.

## Git

Use [Conventional Commits](https://www.conventionalcommits.org/) for every commit message, and the same type vocabulary for every branch name.

### Commit messages

```
type(scope): subject
```

- `type` is one of `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- `scope` is optional and names the area, as in `feat(preview): add page thumbnails`.
- `subject` is imperative, lowercase, and has no trailing period.
- Add a body when the subject does not say why.
- Mark a breaking change with `type(scope)!: subject` and a `BREAKING CHANGE:` footer.

### Branches

```
type/kebab-case-summary
```

Use the same `type` list. Examples: `feat/resume-preview`, `fix/pdf-page-break`.
