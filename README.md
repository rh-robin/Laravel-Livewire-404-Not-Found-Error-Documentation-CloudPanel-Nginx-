# 🛠️ CloudPanel / Nginx Fix for Laravel Livewire 404 Error

This guide provides **two reliable solutions** to fix the common `404 Not Found` error for `/livewire/livewire.js` when deploying a **Laravel + Livewire** application on **CloudPanel** (or any Nginx-based server with aggressive static asset handling).

---

## 🚨 The Problem: `/livewire/livewire.js` Not Found

After deployment, you see this error in the browser console:
`GET https://yourdomain.app/livewire/livewire.js 404 (Not Found)`

### Root Cause

Livewire serves its JavaScript asset (`livewire.js`) and update endpoint (`/livewire/update`) via **dynamic Laravel routes**, not physical files.

However, **Nginx** is configured (especially in CloudPanel) to treat all `.js` requests as **static files**. Since no physical file exists at that path, Nginx returns a **404** before Laravel can handle the request.

---

## ⚙️ Solution A: Modify Nginx Vhost Configuration (**Recommended**)

This is the **cleanest and most maintainable** fix. It tells Nginx to route Livewire’s dynamic paths to Laravel instead of treating them as static assets.

### Step 1: Access the Vhost Editor

1. Log in to your **CloudPanel dashboard**.
2. Go to your **Site settings**.
3. Open the **Vhost** tab.

> You’ll see two server blocks:  
> - Main block (ports **80 & 443**) – handles public traffic  
> - Backend block (port **8080**) – proxies to PHP-FPM

---

### Step 2: Apply Fixes in **Both** Server Blocks

> **Important**: Add these `location` blocks **before** the generic static file handler (usually `location ~* \.(css|js|...)$`).


#### A. Main Server Block (Ports 80 & 443)

```nginx
# --- LIVEWIRE FIX START: Route Dynamic Assets to Backend ---
location ^~ /livewire/livewire.js {
    proxy_pass http://127.0.0.1:8080;
}

location ^~ /livewire/update {
    proxy_pass http://127.0.0.1:8080;
}
# --- LIVEWIRE FIX END ---
```

#### B. Backend Server Block (Port 8080)

```nginx
# --- LIVEWIRE FIX START: Pass Dynamic Assets to index.php ---
location = /livewire/livewire.js {
    expires off;
    try_files $uri /index.php?$query_string;
}
# --- LIVEWIRE FIX END ---
```

This ensures the request reaches Laravel’s `index.php` where Livewire can generate the JS dynamically.

---

### Step 3: Save & Reload

1. Save the Vhost configuration in CloudPanel.
2. Reload Nginx (CloudPanel does this automatically).
3. Clear Laravel caches via SSH:
```php
php artisan optimize:clear
```

---

## 📂 Solution B: Publish Livewire Assets (Fallback)
If you **cannot modify Vhost settings** (e.g., shared hosting, frequent overwrites), force Livewire to publish a **real static file**.

### Step 1: Publish Assets

SSH into your server and run:
```phh
php artisan livewire:publish --assets
```
This creates:
> public/vendor/livewire/livewire.js

Now Nginx can serve it directly as a static file.

#### Warning
> **Re-run this command** after every `composer update` or Livewire upgrade.
