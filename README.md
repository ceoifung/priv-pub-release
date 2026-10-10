# priv-pub-release

A ZCode skill for building a **private source + public release** CI/CD pipeline — any language, any framework.

## How it works

```
You: push a tag to your private repo
      ↓ (automatic)
Private repo workflow: notifies public repo
      ↓ (automatic)
Public repo workflow: clones private code → builds → publishes Release
      ↓
Anyone: download from Release page (no GitHub account needed)
```

## What makes this different

- **Language-agnostic**: no Python/Node/Go templates to maintain. The skill provides architecture + pitfalls; the AI generates language-specific build steps from your project context.
- **Zero-maintenance**: no language templates to update when ecosystems change.
- **Battle-tested**: 8 pitfalls from real production deployments (not theory).

## Install

```bash
git clone https://github.com/ceoifung/priv-pub-release.git ~/.agents/skills/priv-pub-release
```

## What's inside

| File | Purpose |
|------|---------|
| `SKILL.md` | Architecture + 8 universal pitfalls + setup steps |
| `templates/build-trigger.yml` | Ready-to-use trigger workflow (language-agnostic) |
| `templates/build-spec.md` | Spec for the build workflow — AI generates YAML from this |
| `templates/update-check-spec.md` | Spec for in-app update checker — AI implements in your language |

## Pitfalls covered

1. JSON quoting in curl → use `printf`
2. `$GITHUB_ENV` same-step → set local var too
3. Windows CI non-ASCII encoding → set encoding env var
4. Empty repo can't create Release → seed README
5. Zip missing version → same-step var bug
6. Network resets (proxy) → alternate direct/proxy
7. Retrigger needs Release deletion first
8. Hardcoded dev-machine paths → use runtime detection

## Use it

In any project's agent session:

- "帮我搭建私有仓库发版公开仓库打包的流水线"
- "Set up a dual-repo release pipeline"
- "I want private source but public releases"

## License

MIT
