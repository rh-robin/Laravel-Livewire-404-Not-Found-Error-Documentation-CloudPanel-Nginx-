# 🛠️ CloudPanel / Nginx Fix for Laravel Livewire 404 Error

This guide provides **two reliable solutions** to fix the common `404 Not Found` error for `/livewire/livewire.js` when deploying a **Laravel + Livewire** application on **CloudPanel** (or any Nginx-based server with aggressive static asset handling).

---

## 🚨 The Problem: `/livewire/livewire.js` Not Found

After deployment, you see this error in the browser console:

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

---

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