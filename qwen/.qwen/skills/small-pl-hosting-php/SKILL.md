---
name: small-pl-php
description: Use when developing PHP applications hosted on small.pl (devil hosting) — configuring PHP versions, .user.ini, MySQL connections, cron jobs, caching, logging, or running devil CLI commands for www options.
---

# PHP Development on small.pl (Devil Hosting)

## Overview

small.pl uses the **Devil** hosting platform with NGiNX + PHP-FPM. Configuration is managed via `.user.ini` files and the `devil` CLI. Default PHP version: **8.3**.

## PHP Versions

Available: 5.6, 7.0–7.4, 8.0–8.4

**CLI binaries:** `php`, `php56`, `php70`, `php71`, `php72`, `php73`, `php74`, `php80`, `php81`, `php82`, `php83`, `php84`
**CGI variants:** `php-cgi`, `php56-cgi` … `php84-cgi`

**Set version via `.htaccess`:**
```apacheconf
AddType application/x-httpd-php83 .php
```

## PHP Configuration (.user.ini)

Place in `~/.user.ini` (global) or `~/domains/DOMAIN/public_html/.user.ini` (per-domain).
Only `PHP_INI_PERDIR` / `PHP_INI_ALL` directives are editable.

```ini
; Performance
memory_limit = 256M
max_execution_time = 60
max_input_vars = 3000

; Error handling
error_reporting = E_ALL
display_errors = Off
log_errors = On
error_log = /usr/home/LOGIN/domains/DOMAIN/logs/php_errors.log

; File uploads
upload_max_filesize = 64M
post_max_size = 64M
upload_tmp_dir = /usr/home/LOGIN/tmp

; Security
open_basedir = /usr/home/LOGIN/domains/DOMAIN:/tmp

; Extra modules (amqp, sodium, grpc, yaml, sysvsem, swoole — version-dependent)
anp.extensions = yaml:sodium
```

## Devil CLI — PHP/WWW Options

```bash
# Enable/disable eval()
devil www options DOMAIN php_eval on|off

# Enable/disable exec() and shell functions
devil www options DOMAIN php_exec on|off

# Set open_basedir (default dirs always included)
devil www options DOMAIN php_openbasedir /path1:/path2

# Enable GZIP compression
devil www options DOMAIN gzip on|off

# Force HTTPS
devil www options DOMAIN ssl force on|off
```

## MySQL

**Connection host:** `mysqlX.small.pl` where X = your server number (e.g. server `s2.small.pl` → `mysql2.small.pl`)

**PHP PDO / mysqli connection:**
```php
$pdo = new PDO(
    'mysql:host=mysql2.small.pl;dbname=DBNAME;charset=utf8mb4',
    'DB_USER',
    'DB_PASS'
);
```

**Devil CLI:**
```bash
devil mysql db add DBNAME                          # create database
devil mysql user add LOGIN                         # create user
devil mysql passwd LOGIN                           # change password
devil mysql access add LOGIN@HOST                  # allow host
devil mysql privileges LOGIN DBNAME ALL            # grant permissions
devil mysql list -v                                # list databases/users

# Export
mysqldump --host mysql2.small.pl -u USER -p DBNAME --no-tablespaces > dump.sql
# Import
mysql --host mysql2.small.pl -u USER -p DBNAME < dump.sql
```

## Cron Jobs

**Via crontab:**
```bash
crontab -e
```

```cron
SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin

# Run PHP script every hour
0 * * * * php84 /usr/home/LOGIN/domains/DOMAIN/scripts/task.php >> /usr/home/LOGIN/logs/cron.log 2>&1
```

Or manage via **DevilWEB panel → Zadania Cron**.

## Caching

Configure via `devil www options DOMAIN cache [mode]`:

| Mode | Behavior |
|------|----------|
| `off` | No caching |
| `control` | Respects `Cache-Control`, `Expires`, `Pragma` headers |
| `short` | 1-second microcaching |
| `long` | 120-second caching |
| `purge` | Clears cache |

**Cookie modes:** `any` (ignore cookies), `none` (no-cookie requests only), `name COOKIE_NAME`

**Debug headers:**
```bash
devil www options DOMAIN cache_debug on
# Response headers: X-DCache-KEY, X-DCache-Status (MISS/EXPIRED/HIT)
```

**For PHP sessions** use `short` or `long` + `cache_cookie name PHPSESSID`.

## Logs

| Log | Path |
|-----|------|
| Web server errors | `/usr/home/LOGIN/domains/DOMAIN/logs/error.log` |
| Access logs (90 days) | `/usr/home/LOGIN/domains/DOMAIN/logs/access/` |
| PHP errors | Configure `error_log` in `.user.ini` (not automatic) |

**Do not delete** the `logs/` directory — it breaks Node.js/Ruby/Python sites.

## Built-in PHP Server (for testing)

```bash
php -S 0.0.0.0:PORT -t public/
```

Use **DevilWEB → Rezerwacja portów** to reserve a port first.

## .htaccess Tips

```apacheconf
# Clean URLs (PHP router)
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]

# Force UTF-8
AddDefaultCharset utf-8

# Avoid BOM in .htaccess — causes 500 errors
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong MySQL host | Use `mysqlX.small.pl` matching your server number |
| PHP errors not logged | Add `error_log` path to `.user.ini` |
| Cron fails silently | Add full binary path or set `PATH` in crontab |
| Cache serving stale PHP | Set `cache off` or send proper `Cache-Control: no-store` headers |
| 500 error from .htaccess | Check for BOM encoding — save as UTF-8 without BOM |
| `exec()` blocked | Enable via `devil www options DOMAIN php_exec on` |