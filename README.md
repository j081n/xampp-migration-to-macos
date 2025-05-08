# PHP-FPM + Apache Setup on macOS (Homebrew)

This guide outlines how to set up multiple PHP projects (like in XAMPP) using Apache and PHP-FPM on macOS with Homebrew. It includes socket and TCP configurations, virtual hosts, common error fixes, and system configurations.

---

## ✅ Prerequisites

- macOS with Homebrew installed
- PHP installed via Homebrew (`brew install php`)
- Apache installed via Homebrew (`brew install httpd`)

---

## 🧩 Configuration Steps

### 1. Start PHP-FPM and Apache via Homebrew

```bash
brew services start php
brew services start httpd
```

### 2. Apache Configuration

Edit the Apache config:

```bash
sudo nano /opt/homebrew/etc/httpd/httpd.conf
```

#### ✅ Ensure the following modules are loaded:

```apache
LoadModule proxy_module lib/httpd/modules/mod_proxy.so
LoadModule proxy_fcgi_module lib/httpd/modules/mod_proxy_fcgi.so
```

#### ✅ Enable PHP via FPM socket:

```apache
<FilesMatch \.php$>
    SetHandler "proxy:unix:/opt/homebrew/var/run/php/php-fpm.sock|fcgi://localhost/"
</FilesMatch>
```

🔁 **Alternative (TCP instead of socket):**

```apache
<FilesMatch \.php$>
    SetHandler "proxy:fcgi://127.0.0.1:9000"
</FilesMatch>
```

#### ✅ Add index.php to DirectoryIndex:

```apache
DirectoryIndex index.php index.html
```

### 3. Virtual Hosts

Edit (or create) the virtual host config:

```bash
sudo nano /opt/homebrew/etc/httpd/extra/httpd-vhosts.conf
```

Example:

```apache
<VirtualHost *:80>
    DocumentRoot "/Users/YOUR_USER/<folder>/<project>/public"
    ServerName <project-name>.local

    <Directory "/Users/YOUR_USER/<folder>/<project>/public">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Repeat for other projects:

### 4. Update /etc/hosts

```bash
sudo nano /etc/hosts
```

Add:

```text
127.0.0.1 <project>.local localhost
```

### 5. Restart Services

```bash
brew services restart php
brew services restart httpd
```

---

## 🛠 Common Issues & Fixes

### ❌ PHP-FPM Socket Binding Error

**Error:**

```
ERROR: unable to bind listening socket for address '/opt/homebrew/var/run/php/php-fpm.sock': No such file or directory
```

#### ✅ Fix Steps:

1. **Edit the socket path:**

   ```bash
   sudo nano /opt/homebrew/etc/php/8.4/php-fpm.d/www.conf
   ```

   Find the line:

   ```ini
   listen = 127.0.0.1:9000
   ```

   And change to:

   ```ini
   listen = /opt/homebrew/var/run/php/php-fpm.sock
   ```

2. **Ensure directory exists and has correct permissions:**

   ```bash
   sudo mkdir -p /opt/homebrew/var/run/php
   sudo chown $(whoami) /opt/homebrew/var/run/php
   ```

3. **Restart PHP-FPM:**

   ```bash
   brew services restart php
   ```

### ❌ PHP Error Code 78 (Interpolated Plist Path)

**Issue:** Plist at:

```
~/Library/LaunchAgents/homebrew.mxcl.php.plist
```

Has interpolated or incorrect variable usage.

#### ✅ Fix Steps:

1. Stop the PHP service:

   ```bash
   brew services stop php
   ```

2. Delete or inspect the plist file:

   ```bash
   rm ~/Library/LaunchAgents/homebrew.mxcl.php.plist
   # Or edit if needed
   nano ~/Library/LaunchAgents/homebrew.mxcl.php.plist
   ```

3. Restart PHP cleanly:

   ```bash
   brew services start php
   ```

---

## ✅ PHP-FPM Test Command

To test PHP-FPM config:

```bash
php-fpm -y /opt/homebrew/etc/php/8.4/php-fpm.conf -t
```

You should see:

```
NOTICE: configuration file ... test is successful
```

To run manually:

```bash
php-fpm -y /opt/homebrew/etc/php/8.4/php-fpm.conf
```

---

## 📌 Notes

- Use `;` to comment lines in `.ini` or `.conf` files under PHP.
- Use `#` to comment in Apache `.conf` files.
- `:80` is the default HTTP port. `:8080` is often used for testing or alternative services. They are not interchangeable without explicitly configuring Apache to listen on both.

---

> This guide serves as a migration reference from XAMPP (Windows) to a native macOS Apache + PHP-FPM setup with Homebrew.
