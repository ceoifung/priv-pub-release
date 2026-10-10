# Skill: priv-pub-release

## Description

Build a "private source + public release" CI/CD pipeline for any project, any language. Use when the user wants to keep source code private but publish downloadable releases publicly, set up dual-repo auto-build, create a tag-triggered release pipeline, or separate code hosting from distribution. Triggers on: "私有仓库发版", "公开仓库打包", "private repo public release", "dual repo CI/CD", "auto build and release", "release channel", "发版流水线", or when setting up release infrastructure for any new or existing project.

## Architecture

```
你：本地打 tag → push 到私有仓库
      ↓ （自动）
私有仓库 workflow：收到 tag → curl 通知公开仓库
      ↓ （自动）
公开仓库 workflow：拉私有代码 → 构建 → 创建 Release 挂产物
      ↓
任何人：Release 页直接下载（无需 GitHub 账号）
```

## Prerequisites

- gh CLI installed and authenticated (`gh auth status` shows `repo` + `workflow` scopes)
- A private repo with the source code
- Network can reach GitHub (directly or via proxy)

## Setup steps

### 1. Create the public repo

```bash
gh repo create <name>-pub --public
```

Seed with a README so the Release API works on empty repos.

### 2. Configure secrets

```bash
TOKEN=$(gh auth token)
gh secret set PRIVATE_PULL_TOKEN --repo <owner>/<name>-pub --body "$TOKEN"
gh secret set PUB_TRIGGER_TOKEN --repo <owner>/<name> --body "$TOKEN"
```

### 3. Write two workflows

Copy [templates/build-trigger.yml](templates/build-trigger.yml) to the private repo
(replace `OWNER/PUB_REPO_NAME`). This file is language-agnostic.

For the public repo's build workflow, **generate it from scratch** based on the
project's language and build system. The structure is always the same but the
build/dependency steps differ. See [templates/build-spec.md](templates/build-spec.md)
for the required skeleton and constraints.

### 4. Version management

The project needs a single source of truth for the version number that the tag
must match. Detect the project's language and use its convention (package.json,
Cargo.toml, pyproject.toml, version file, etc). Add an in-app update checker
that anonymously queries the public repo's `releases/latest` endpoint — see
[templates/update-check-spec.md](templates/update-check-spec.md).

### 5. Test

```bash
git tag v0.1.0 && git push origin v0.1.0
gh run watch <run-id> --repo <owner>/<name>-pub
```

## Universal pitfalls (language-independent)

These affect every project regardless of stack:

| # | Pitfall | Fix |
|---|---------|-----|
| 1 | JSON quoting in curl breaks | Use `printf` to construct JSON, never inline `'{"key":"'$VAR'"}'` |
| 2 | `$GITHUB_ENV` doesn't affect the same step | Set a local variable AND write to `$GITHUB_ENV` |
| 3 | Windows CI can't print non-ASCII output | Set the appropriate encoding env var for the language |
| 4 | Empty repo can't create Release via API | Push a README first |
| 5 | Zip filename missing version | Same-step variable issue (see #2) |
| 6 | Git push fails with SSL/reset (unstable network) | Try with and without proxy: `git -c http.proxy=... push` |
| 7 | Retrigger requires deleting old Release first | `gh api -X DELETE repos/.../releases/<id>` then re-tag |
| 8 | Build script uses hardcoded tool paths from dev machine | Use runtime detection instead (the CI environment differs) |

## Release workflow (after setup)

```bash
# 1. Update the version source of truth
# 2. Commit and push
# 3. Tag and push
git tag v1.0.0 && git push origin v1.0.0
```

## Key principle

This skill provides the architecture and the pitfalls — the things the AI
doesn't inherently know. Language-specific build commands, dependency
installation, and version file conventions are left to the AI to derive from
the project context. Don't hardcode them here; they change and vary.
