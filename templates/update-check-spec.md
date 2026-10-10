# Update Check Spec

Specification for the in-app update checker. Implement in the project's
language following this API contract.

## API contract

```
GET https://api.github.com/repos/<owner>/<pub-repo>/releases/latest
Accept: application/vnd.github+json
User-Agent: <app-name>          (GitHub requires a User-Agent header)
```

**No authentication needed** — the public repo is anonymous-readable.

## Response shape (relevant fields)

```json
{
  "tag_name": "v1.2.3",
  "html_url": "https://github.com/.../releases/tag/v1.2.3",
  "body": "Release notes text..."
}
```

## Logic

```
1. Strip "v" prefix from tag_name → remote_version
2. Compare remote_version with local_version (dot-separated integers)
3. If remote > local:
   - Show dialog: "当前 X.X.X → 最新 X.X.X" + release notes
   - Action button opens html_url in browser
4. If remote <= local: "已是最新版"
5. On network error: silently pass (or show "检查更新失败" on manual trigger)
```

## Version comparison algorithm

Compare as arrays of integers, padding with zeros:

```
parse("1.2")  → [1, 2, 0, 0, 0]
parse("1.2.3") → [1, 2, 3, 0, 0]
compare element-wise, first difference wins
```

## UX guidance

- Trigger on app startup (silent, 2s delay to not slow boot)
- Also provide a manual "检查更新" button
- Dialog should show: current version, latest version, release notes excerpt,
  and a button to open the download page
- Keep the timeout short (10s) so slow networks don't hang the app
