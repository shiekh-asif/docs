---
last_modified_at: 2025-06-13
last_modified_by: Mohammad_Asif
title: "How to enable HTTP/2 Protocol on Apache (Ubuntu)"
---
# Enabling HTTP/2 Protocol on Apache (Ubuntu)

## Overview

This guide walks through enabling the **HTTP/2 protocol** on Apache web server running on Ubuntu. HTTP/2 improves performance with features like multiplexing, header compression, and reduced latency.

HTTP/2 requires:
- An **SSL-enabled** (HTTPS) virtual host
- Apache compiled with or using **`mpm_event`** (not `mpm_prefork`)

---

## Prerequisites

- Ubuntu system with Apache installed
- A valid SSL certificate
- Root or sudo privileges

---

## Step 1: Enable the HTTP/2 Module

Enable the `http2` module:

```bash
sudo a2enmod http2
```

> This loads Apache’s `mod_http2`, which is required to serve HTTP/2 traffic.

---

## Step 2: Update Virtual Host Configuration

Edit your SSL virtual host configuration file (typically in `/etc/apache2/sites-available/`), and ensure the following is present inside your `<VirtualHost *:443>` block:

```apache
ServerName your.domain.com
Protocols h2 http/1.1
```

> The `Protocols` directive enables HTTP/2 (`h2`) and falls back to HTTP/1.1 when needed. This must be set **within the SSL-enabled virtual host block**.

---

## Step 3: Restart Apache

Apply changes by restarting Apache:

```bash
sudo systemctl restart apache2
```

---

## Step 4: Verify HTTP/2 Is Enabled

Use `curl` to verify HTTP/2 support:

```bash
curl -I --http2 -k https://your.domain.com
```

Expected output should start with `HTTP/2`, e.g.:

```http
HTTP/2 200
```

> If the response shows `HTTP/1.1`, Apache may still be using the `prefork` MPM, which is incompatible with HTTP/2.

---

## Step 5: Check Current MPM Mode

To determine which Multi-Processing Module (MPM) Apache is using:

```bash
apachectl -V | grep -i mpm
```

If the output is:

```
Server MPM: prefork
```

Then proceed to the next step to switch to the `event` MPM.

---

## Step 6: Switch to `mpm_event` and Enable PHP-FPM

Run the following commands to replace `prefork` with `event` MPM and enable PHP via FPM:

```bash
sudo a2dismod php8.4
sudo a2dismod mpm_prefork
sudo a2enmod mpm_event
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php8.4-fpm
```

**Explanation:**
- `mpm_event` is required for HTTP/2 and better performance.
- `mod_php` (used with `mpm_prefork`) must be disabled.
- PHP-FPM handles PHP processing via FastCGI, which is compatible with `mpm_event`.

---

## Step 7: Restart Apache Again

```bash
sudo systemctl restart apache2
```

---

## Step 8: Re-Verify HTTP/2

```bash
curl -I --http2 -k https://your.domain.com
```

You should now see an `HTTP/2` status in the output.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **`HTTP/1.1` shown instead of `HTTP/2`** | Ensure you’ve switched to `mpm_event` and added `Protocols h2 http/1.1` in the SSL virtual host block. |
| **Apache fails to start after module changes** | Check logs via `journalctl -xe` or `/var/log/apache2/error.log`. |
| **PHP pages not loading after switching MPM** | Ensure PHP-FPM is installed and running: `sudo systemctl status php8.2-fpm`. |

---

## Summary

| Step | Command/Setting |
|------|------------------|
| Enable HTTP/2 Module | `sudo a2enmod http2` |
| Configure Virtual Host | `Protocols h2 http/1.1` |
| Restart Apache | `sudo systemctl restart apache2` |
| Check MPM | `apachectl -V | grep -i mpm` |
| Enable PHP-FPM & Event MPM | See full list above |
| Verify HTTP/2 | `curl -I --http2 -k https://your.domain.com` |
