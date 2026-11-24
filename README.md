# Claude skills by Amr Khaled Nowahy

Reusable instructions that keep Claude's output consistent across projects.

| Skill | What it does |
|---|---|
| `amr-elnowahy-springboot` | Structure, readability and comment style for Spring Boot backends. |
| `frontend-primeng` | Angular 17+ and PrimeNG patterns: lazy tables, popovers, i18n keys, safe templates. |
| `project-docs` | Short, structured Markdown for features, classes and READMEs. |

## Install

**Claude Code (plugin):**

```
/plugin marketplace add AmrKhaledNowahy/claude-skills
/plugin install amr-skills@amr-skills
```

**Claude Code (manual):** copy any folder from `skills/` into `~/.claude/skills/`.

**Claude.ai:** zip a skill folder (for example `skills/project-docs`) and upload it under Settings, Capabilities, Skills.

## License

MIT. See [LICENSE](LICENSE). Free to use, change and share; keep the copyright notice.
