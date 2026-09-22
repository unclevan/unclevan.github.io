# Static sites and production deployment

Last reviewed: 2026-09-22

## Default publication policy

- The canonical production platform for both sites is the owner's Alibaba
  Cloud ECS, not GitHub Pages.
- Do not publish either site to `unclevan.github.io` unless the user explicitly
  requests it for the current task.
- Do not treat `git push` as a deployment step. Because this repository may
  still have GitHub Pages automation attached, obtain an explicit user request
  before pushing changes to its remote.
- Local commits are acceptable when requested or useful for preserving a
  completed change. Use clear English Conventional Commit messages.
- Production metadata, canonical URLs, Open Graph URLs, and internal links
  should use the relevant `enlightened.cn` hostname.

## Sites

| Product | Production URL | Source in this repository | Server document root | Nginx configuration |
| --- | --- | --- | --- | --- |
| DoggoLedger / 黑狗记账 | `https://heygo.enlightened.cn/` | Repository root (`index.html`, `privacy.html`, `support.html`, `styles.css`, and root assets) | `/srv/www/heygo.enlightened.cn/public/` | `/etc/nginx/conf.d/heygo.enlightened.cn.conf` |
| Doggo Compass / 黑狗罗盘 | `https://zhinan.enlightened.cn/` | `doggo-compass/` | `/srv/www/zhinan.enlightened.cn/public/` | `/etc/nginx/conf.d/zhinan.enlightened.cn.conf` |

Keep the two products isolated. A change for one site must not rewrite,
replace, or deploy the other site's content.

## Current Doggo Compass publication

- The iPhone App Store listing is `https://apps.apple.com/cn/app/%E9%BB%91%E7%8B%97%E7%BD%97%E7%9B%98/id6813387560`. The header icon and brand name on the home, support, and privacy pages link to it; the home page also has an App Store button.
- The home-page headline is `知经纬，定方向。`.
- The header and hero use `doggo-compass/assets/app-icon-v4.webp` (512 × 512). Open Graph uses the same image; Apple Touch Icon uses `doggo-compass/assets/app-icon-v4-180.png` (180 × 180). Both are derived from `DoggoCompass/Branding/assets/doggocompass-app-icon-polished-v4-sparse-ticks-1024.png` in the adjacent app repository.
- The new asset filenames avoid a stale image under the site's seven-day static cache. The older image files remain available for previously cached pages.
- The 2026-09-22 production changes were deployed to the ECS from this local source. Recoverable backups are `/data/backups/apps/zhinan.enlightened.cn-pre-appstore-links-20260922.tar.gz` and `/data/backups/apps/zhinan.enlightened.cn-pre-v4-icon-headline-20260922.tar.gz`.
- After the latest deployment, `nginx -t` passed, Nginx was reloaded and remained active, HTTP redirected to HTTPS, the HTML/CSS/image responses had the expected status and Content-Type, production file hashes matched the source, and the site-specific error log was empty. More detail is recorded in `/root/SERVER_NOTES.md` on the server.

## Server access and sources of truth

- Production server: `139.129.26.245`
- Routine administrative login: `ssh victor@139.129.26.245`
- Use `sudo` for privileged operations.
- Before server work, read `/Users/victor/.codex/SERVER_INFRASTRUCTURE.md` on
  the administration Mac.
- After logging in, read `/root/SERVER_NOTES.md` before making changes.
- The server notes and actual server state take precedence over this summary
  if they differ. Update this file and the server notes when durable topology
  or deployment policy changes.
- Do not change the SSH port, root SSH policy, MySQL, UFW, or unrelated
  services as part of a website deployment.

This file intentionally contains no password, SSH private key, API credential,
DNS credential, or TLS private key. Do not add any.

## Required deployment workflow

1. Inspect the relevant source and current server state; do not infer privacy
   or product behavior from marketing copy alone.
2. Confirm the intended hostname and DNS record.
3. Before changing production files or Nginx, create a recoverable backup under
   `/data/backups/apps/` with a descriptive timestamped name.
4. Stage and inspect files before installing them into the document root.
5. Preserve ownership `root:www-data`, directory mode `0755`, and file mode
   `0644` unless the actual server configuration requires otherwise.
6. Run `sudo nginx -t` before every Nginx reload. Reload only after the test
   succeeds, then confirm `nginx.service` remains active.
7. Verify public HTTP redirects, HTTPS status, Content-Type values, page
   content, internal links, mail links, images, and both desktop and mobile
   layouts.
8. Check the site-specific Nginx error log after validation.
9. When TLS changes, verify the certificate hostname and expiry, ensure
   `certbot.timer` is enabled and active, and run an appropriate renewal dry
   run.
10. Record durable production changes in `/root/SERVER_NOTES.md`.

## Current TLS arrangement

- `heygo.enlightened.cn` has its own Let's Encrypt certificate.
- `zhinan.enlightened.cn` has its own Let's Encrypt certificate.
- The certificates are renewed by the existing `certbot.timer` service.
- Do not assume a certificate can be reused for a new hostname; inspect its SAN
  list first and request a separate certificate when the hostname is absent.

## Repository boundaries

- For DoggoLedger work, preserve all files under `doggo-compass/` unless the
  user also requests a Doggo Compass change.
- For Doggo Compass work, preserve the repository-root DoggoLedger pages and
  their existing public URLs.
- Keep the websites static unless the user explicitly requests a different
  architecture. Do not add analytics, advertising, cookies, external fonts, or
  unnecessary third-party resources without an explicit product requirement.
