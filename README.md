# apps.shawonmajid.com

Landing page + release pipeline for desktop apps I ship. Currently hosts:

- **Agents Messaging** — Pake-wrapped desktop client for `agent.vyg.app`

## Architecture

```
                 ┌────────────────────────────────┐
                 │  apps.shawonmajid.com (Vercel) │
                 │  static index.html             │
                 └──────────────┬─────────────────┘
                                │ download links point to
                                ▼
        ┌────────────────────────────────────────────┐
        │  github.com/shawon-majid/apps-shawonmajid  │
        │  /releases/latest/download/<asset>         │
        └──────────────┬─────────────────────────────┘
                       │ assets uploaded by
                       ▼
        ┌────────────────────────────────────────────┐
        │  GitHub Actions: .github/workflows/        │
        │    build-pake.yml                          │
        │  Matrix: macOS-arm64, Windows-x64          │
        └────────────────────────────────────────────┘
```

The site has zero build step — it's a single `index.html` served by Vercel. Binaries live on GitHub Releases (free CDN, versioned, direct URLs).

## Cutting a new release

1. Tag the commit:
   ```bash
   git tag v1.1.0
   git push origin v1.1.0
   ```
2. The `build-pake.yml` workflow fires automatically and attaches the macOS DMG + Windows MSI to the release.
3. The `/releases/latest/download/Agents-Messaging-mac-arm64.dmg` link auto-resolves to the newest asset — no site changes needed.

## Manual Windows build (if local Mac build already exists)

When you've built the Mac DMG locally and only need CI to produce the Windows MSI:

```bash
gh workflow run build-pake.yml -f tag=v1.0.0
```

That dispatches the workflow and attaches the Windows artifact to an existing release.

## Local dev (preview the site)

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## Signing (todo)

Both builds are currently **unsigned**. Users need to bypass Gatekeeper (right-click → Open) on macOS and SmartScreen on Windows the first time. To remove these prompts:

- **macOS**: Apple Developer ID Application certificate (~$99/yr) + add `--sign` flag to Pake build
- **Windows**: Code-signing certificate from a CA (~$200+/yr)
