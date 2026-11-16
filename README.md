# Skills

Engineering-related skills I personally use for projects.
They may or may not suit your workflow, but feel free to try them out.

## Install

```bash
npx skills add gurvinder-dhaliwal71/skills
```

## Skills

- **callstack-review**: shows the call stacks a pull request changes as diff trees, traced from source. Run `/callstack-review <pr>`; add `--pr` to write the result into the PR description. Needs `gh`. On first run it creates `.callstack/PATH_ALIAS.md` in your repo (uncommitted) to shorten long paths. Review and commit it.
- **feature-design**: writes a technical design for a feature before any code: flows, architecture with call-stack diff trees, interfaces, schema, risks, open points and a stacked PR plan. Ask it to design a feature; it writes `docs/design/<ticket>-<slug>.md` (or a path you give) for your approval and changes no code.
