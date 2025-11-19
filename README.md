# **AWS Lightsail Frontend Hosting — Single Script Setup Documentation**

## **1. Overview**

This guide explains how to set up and host a static frontend application (React, Vite, Vue, HTML, etc.) on AWS Lightsail using a **single automated shell script**:

```
setup-site.sh
```

The script configures:

* Application folder structure
* Nginx site configuration
* Automatic SSL via Let’s Encrypt
* HTTP → HTTPS redirection
* Domain-based hosting

After this setup, you can deploy your frontend build anytime by simply uploading it into the server’s `/dist` directory.

---

## **2. Prerequisites**

### **2.1 Server Requirements**

Ensure your Lightsail Ubuntu instance has:

* **Ubuntu 22.04 LTS**
* **Static IP**
* **Nginx installed**
* **Certbot installed**
* **Firewall allowing**

  * 22 (SSH)
  * 80 (HTTP)
  * 443 (HTTPS)

### **2.2 Script Required**

Place the script on your server:

```
setup-site.sh
```

Make it executable:

```
chmod +x setup-site.sh
```

---

## **3. DNS Preparation**

Before running the script, DNS **must** point to your Lightsail instance.

### **3.1 Add DNS A-Record**

| Type | Name             | Value                   |
| ---- | ---------------- | ----------------------- |
| A    | `@`              | `<Lightsail Static IP>` |
| A    | `www` (optional) | `<Lightsail Static IP>` |

### **3.2 Verify DNS**

Run this from your local machine:

```
ping <domain>
```

If it resolves to your Lightsail IP, DNS is ready.

---

## **4. Directory Structure Created**

Running the script creates:

```
/var/www/<app>/
│── dist/            → Your production build goes here
│── deploy-temp/     → Temporary upload folder (optional)
```

---

## **5. Script Usage**

### **Command**

```
./setup-site.sh <app-name> <domain>
```

### **Example**

```
./setup-site.sh boom-fit boom-fit.chatweave.space
```

---

## **6. What the Script Does**

### **1. Creates Folders**

```
/var/www/<app>/dist
/var/www/<app>/deploy-temp
```

### **2. Installs Temporary HTTP Nginx Config**

Used for domain validation.

### **3. Requests SSL Certificate (Let’s Encrypt)**

Uses Certbot `--webroot` mode.

### **4. Installs Final HTTPS Nginx Config**

Includes:

* Automatic redirect to HTTPS
* Static file serving
* SPA routing (`index.html`)
* 30-day cache headers

### **5. Reloads Nginx**

Your site becomes available at:

```
https://<domain>
```

---

## **7. Deploying the Frontend Build**

After setup, deployment is simple.

### **Step 1 — Upload build files**

Copy your built project (React/Vite/etc.) into:

```
/var/www/<app>/dist
```

### **Step 2 — Reload Nginx (optional)**

```
sudo systemctl reload nginx
```

That’s it___your site is live.

---

## **8. Verification Checklist**

### Redirect check:

```
curl -I http://<domain>
```

Expected → `301`

### HTTPS check:

```
curl -I https://<domain>
```

Expected → `200`

### SSL status:

```
sudo certbot certificates
```

---

## **9. Troubleshooting**

### Certbot fails

→ DNS not propagated
→ Wrong A-Record

### Nginx fails reload

Run:

```
sudo nginx -t
```

### SPA routing fails

Ensure:

```
try_files $uri $uri/ /index.html;
```

---

## **10. GitHub Actions Deployment (Short Guidelines)**

To automate deployments from GitHub, follow these **quick instructions**:

### **10.1 Create Workflow File**

Inside your repo:

```
.github/workflows/deploy.yml
```

### **10.2 Add Required Secrets**

In GitHub → Settings → Secrets → Actions:

| Secret Name       | Value                   |
| ----------------- | ----------------------- |
| `LIGHTSAIL_HOST`  | Your instance public IP |
| `LIGHTSAIL_USER`  | `ubuntu`                |
| `SSH_PRIVATE_KEY` | Your Lightsail PEM key  |

### **10.3 What GitHub Actions Should Do (High-Level Points)**

1. **Checkout code**
2. **Install Node**
3. **Install dependencies**
4. **Build project** (`npm run build`)
5. **Upload build files to the server**
   → Use SCP to copy `dist/*` into

   ```
   /var/www/<app>/dist
   ```
6. **SSH into server and reload Nginx**

   ```
   sudo systemctl reload nginx
   ```

### **10.4 Folder to Upload**

```
dist/*
```

### **10.5 Final Steps**

* Workflow runs on every push to `main`
* Site updates instantly
* No downtime

This is all you need for clean GitHub-to-Lightsail deployments.

---

## **11. Summary**

| Task             | Action                           | Frequency    |
| ---------------- | -------------------------------- | ------------ |
| Prepare DNS      | Add A-Record                     | Once         |
| Run setup script | `./setup-site.sh <app> <domain>` | Once         |
| Deploy build     | Copy to `/dist`                  | Every update |
| Automate deploy  | GitHub Actions                   | Optional     |

You now have a **clean, repeatable, automated hosting system** for any number of frontend websites.
