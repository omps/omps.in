# WordPress GitHub-Driven Workflow Boilerplate (omps.in)

## 🛠 Requirements
- A WordPress site hosted on Namecheap
- GitHub for source control (posts + theme/plugins)
- WP REST API or WP-CLI support
- [Lando](https://lando.dev) for local dev (optional)
- Composer (for managing plugins/themes if needed)
- Google Docs for writing content

---

## ✅ Workflow Overview

```text
Google Docs → Markdown
            ↓
    GitHub Repo (Markdown + Theme/Plugins)
            ↓
 GitHub Actions (Validation + CI/CD)
            ↓
     Push to WordPress (via API or FTP)
```

---

## 📥 Local Development Setup (Optional: Lando + WP-CLI)

### 1. Install [Lando](https://docs.lando.dev)

```bash
brew install lando # avoid Homebrew if it causes issues
```

### 2. Clone the GitHub repo:
```bash
git clone git@github.com:yourusername/omps-site.git
cd omps-site
```

### 3. Initialize Lando with WordPress recipe:
```bash
lando init --source cwd --recipe wordpress --name omps
```

### 4. Configure `.lando.yml`:
```yaml
name: omps
recipe: wordpress
config:
  webroot: .
  php: '8.1'
  via: apache
  database: mariadb
proxy:
  appserver:
    - omps.lndo.site
services:
  node:
    type: 'node:18'
  composer:
    type: 'composer:2'
tooling:
  wp:
    service: appserver
  composer:
    service: composer
  npm:
    service: node
  sniff:
    service: appserver
    cmd: /app/vendor/bin/phpcs -ns
  fix:
    service: appserver
    cmd: /app/vendor/bin/phpcbf -s
  debug:
    service: appserver
    cmd: 'touch /app/web/wp-content/debug.log && tail -f /app/web/wp-content/debug.log'
    description: 'Get real-time WP debug log output'
```

### 5. Start environment
```bash
lando start
```

### 6. Install dependencies
```bash
lando composer install
```

---

## 🔐 JWT Auth Plugin Setup (for REST API Access)

### 1. Install JWT Authentication for WP REST API plugin:
- Download from GitHub: https://github.com/usefulteam/jwt-auth
- Upload and activate the plugin, or use this in your theme/plugin setup.

### 2. Edit `wp-config.php`:
```php
define('JWT_AUTH_SECRET_KEY', 'your-very-secure-secret');
define('JWT_AUTH_CORS_ENABLE', true);
```

### 3. Add the following to `.htaccess` if using Apache:
```apache
RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule ^(.*) - [E=HTTP_AUTHORIZATION:%1]
```

### 4. Generate a JWT Token:
Use Postman or CLI:
```bash
curl -X POST https://omps.in/wp-json/jwt-auth/v1/token \
  -H "Content-Type: application/json" \
  -d '{"username": "yourusername", "password": "yourpassword"}'
```
The response will include your JWT token to use in GitHub Actions or scripts.

---

## 📝 Writing & Publishing Posts

### 1. Write content in Google Docs
Use the **Docs to Markdown** extension or [gdoc-down](https://workspace.google.com/marketplace/app/docs_to_markdown/700168918607).

### 2. Save Markdown into `/content/posts/your-post.md`

### 3. Markdown Compatibility Check
Twenty Twenty-Five supports basic Markdown via Gutenberg blocks **if Markdown plugin is enabled**. However, to ensure consistent formatting, it's recommended to convert `.md` to HTML before publishing.

---

## 🛠 Markdown-to-HTML Conversion Script

Create a file `convert-md-to-html.sh` in the root:
```bash
#!/bin/bash
mkdir -p output
for file in content/posts/*.md; do
  html_name=$(basename "$file" .md).html
  npx marked "$file" > output/"$html_name"
done
```

Make it executable:
```bash
chmod +x convert-md-to-html.sh
```

Run it manually or integrate it in GitHub Actions before pushing to WordPress via REST API.

---

## 🚀 GitHub Actions Workflow

### Structure:
```text
/content/posts         # markdown content
/theme/                # optional custom theme
/plugin/               # optional custom plugins
/.github/workflows     # GitHub Actions
```

### Secrets required:
- `WP_URL`: yoursite.com
- `WP_JWT`: JWT token for WordPress REST API
- `FTP_HOST`, `FTP_USER`, `FTP_PASS`: (if using FTP)

---

## 🔄 Theme/Plugin Development

1. Custom themes live in `/theme/`
2. Plugins live in `/plugin/`
3. Use `lando wp theme activate your-theme` to switch
4. Compile assets:
```bash
cd theme
lando npm install
lando npm run build
```

---

## 🧪 Code Quality Tools

- PHP Sniffer (WordPress standards):
```bash
lando sniff
lando fix
```
- Markdown Linting via GitHub Actions

---

## 🔐 Security Best Practices
- Use `.env` file for credentials locally
- Add sensitive files to `.gitignore`:
```
wp-config.php
.env
/wp-content/uploads/
```
- Store secrets securely in GitHub → Settings → Secrets

---

## ✅ Deployment Options

### A. Using WP REST API
- Converts markdown → HTML
- Uses POST to `https://yourdomain.com/wp-json/wp/v2/posts`
- Auth via JWT or App Password

### B. Using FTP (for code)
- GitHub Action pushes theme/plugin via FTP
- Only for file sync — no database

### C. Using WP Pusher Plugin (optional)
- Install WP Pusher on WordPress site
- Connect to your GitHub repo
- Pushes theme/plugin changes on merge

---

## 🧪 Staging Site Setup (Optional)
- Create a subdomain `staging.omps.in`
- Clone site for testing changes
- Push to staging before production

---

## 📦 Composer Package Management (Optional)
If using Composer-managed WordPress (Bedrock or custom setup):
```bash
lando composer update
```

---

## 🔁 Manual Trigger for Cron (e.g., custom syncs)
```bash
lando wp cron event run --all
```

## 🧹 Reset Caches/Transients
```bash
lando wp transient delete --all
```

---

## ✅ Final Notes
- Start: `lando start`
- Stop: `lando stop`
- Shutdown Docker: `lando poweroff`

You're now set up to run a GitOps-powered WordPress publishing workflow!
