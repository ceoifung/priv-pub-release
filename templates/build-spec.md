# Build Workflow Spec

This is a **specification**, not a template. Generate the actual YAML based on
the project's language and build system, following these constraints.

## Required structure

Every build workflow must have these steps, in this order:

```
1. 解析版本     — extract TAG and VERSION from dispatch payload or input
2. 拉取源码     — git clone the private repo at the tag
3. 安装依赖     — language-specific, detect from project files
4. 下载资源     — large assets from this repo's Release (if any)
5. 构建         — language-specific build command
6. 测试/冒烟    — verify the artifact works
7. 打包产物     — compress into a versioned zip
8. 创建 Release — gh release create with the zip attached
```

## Step 1: 解析版本

Must set both local variables AND `$GITHUB_ENV` (Pitfall #2):

```bash
if [ "${{ github.event_name }}" = "repository_dispatch" ]; then
  TAG="${{ github.event.client_payload.tag }}"
else
  TAG="${{ github.event.inputs.tag }}"
fi
VERSION="${TAG#v}"
echo "TAG=$TAG" >> $GITHUB_ENV
echo "VERSION=$VERSION" >> $GITHUB_ENV
```

## Step 2: 拉取源码

```bash
git clone --depth 1 --branch "$TAG" \
  "https://x-access-token:${TOKEN}@github.com/OWNER/PRIVATE_REPO.git" app
```

## Step 3: 安装依赖

**Detect from project files**, don't assume:
- `requirements.txt` / `pyproject.toml` → Python (pip install)
- `package.json` + `package-lock.json` → Node (npm ci)
- `go.mod` → Go (go mod download)
- `Cargo.toml` → Rust (cargo build --release)

**Watch for version conflicts** (Pitfall: some packages declare incompatible
dependency ranges). If pip/npm/etc fails with ResolutionImpossible, use
`--no-deps` for the conflicting package and install its deps manually.

**Windows CI encoding**: if the build step prints non-ASCII text (CJK, emoji),
set the language-appropriate encoding env var for that step.

## Step 4: 下载资源 (optional)

If the public repo has a Release with large assets (models, binaries):

```bash
gh release download assets-v1 --repo OWNER/PUB_REPO_NAME \
  --dir resources --clobber
```

## Step 5: 构建

Language-specific. Common patterns:
- Python: `python build.py` or `pyinstaller ...`
- Node: `npm run build` or `pkg .` or `nexe`
- Go: `go build -o app ./...` or `goreleaser`
- Rust: `cargo build --release`

**Don't hardcode paths from the dev machine** (Pitfall #8). Use runtime
detection or relative paths.

## Step 7: 打包产物

The zip filename MUST include the version number:

```
<project>-<VERSION>-<platform>.zip
```

Use `7z a -tzip` on Windows runners, `zip -r` on Linux.

## Step 8: 创建 Release

```bash
gh release create "$TAG" --repo OWNER/PUB_REPO_NAME \
  --title "<project> $TAG" \
  --notes "版本 ${{ env.VERSION }} — 自动构建" \
  --target main \
  "<project>-${{ env.VERSION }}-<platform>.zip"
```

## Common runner choices

| Project type | Runner |
|-------------|--------|
| Windows desktop app | `windows-latest` |
| Linux binary | `ubuntu-latest` |
| macOS app | `macos-latest` |
| Cross-platform | Multiple jobs or matrix |
| Docker image | `ubuntu-latest` + docker build |
