ChatGPT said:
Yes — setting up your development environment to mirror production (as closely as possible) is a very good idea, and in your case, installing NGINX locally with self-signed SSL certificates is a simple and smart solution.

Here’s why and how to do it effectively:

🔍 Why You're Having This Problem
Safari (and sometimes Chrome) may automatically upgrade http:// links to https://, especially for domains that "look like real domains" (like local.mars) — this is called HSTS preloading or smart HTTPS upgrades.
When your Node app doesn't support HTTPS, and the browser upgrades to HTTPS, you'll see errors like "can't reach the server" or "connection refused."
✅ Best Practice: Use NGINX Locally with SSL
This gives you a near-production setup:

Serve your site over https://local.mars
Proxy HTTPS to your Node.js app on http://localhost:3000
Helps you catch HTTPS-related bugs early (cookies, redirects, etc.)
🛠️ Steps to Set Up NGINX with SSL Locally

1. Install NGINX (if not already installed)

brew install nginx

2. Create Self-Signed SSL Certificates

mkdir -p ~/ssl/local.mars
cd ~/ssl/local.mars

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
 -keyout local.mars.key -out local.mars.crt \
 -subj "/C=US/ST=Local/L=Local/O=Dev/OU=Dev/CN=local.mars"

3.  Configure NGINX

Edit your local NGINX config (often at /opt/homebrew/etc/nginx/nginx.conf or /usr/local/etc/nginx/nginx.conf, depending on your system).

Add a server block like:

server {
listen 443 ssl;
server_name local.mars;

    ssl_certificate     /Users/youruser/ssl/local.mars/local.mars.crt;
    ssl_certificate_key /Users/youruser/ssl/local.mars/local.mars.key;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

Also, optionally add a redirect from HTTP to HTTPS:

server {
listen 80;
server_name local.mars;

    return 301 https://$host$request_uri;

}

4. Update /etc/hosts

sudo nano /etc/hosts
Add:

127.0.0.1 local.mars 5. Trust the Self-Signed Cert (optional, for Chrome/Safari)

To avoid security warnings, you can import the .crt file into macOS Keychain Access:

Open Keychain Access
Import local.mars.crt
Set trust level to "Always Trust"

6. Restart NGINX

brew services restart nginx
Now you can go to:

https://local.mars
And your dev app should load properly, served via HTTPS.

✅ Summary
Option Pros Cons
Use NGINX + SSL locally Mirrors production, solves HTTPS redirect, no browser errors Slightly more setup
Skip SSL in dev Simpler at first Causes redirect errors, not production-realistic
Let me know if you'd like a full working example config or a script to automate this.

Attach

Search

Voice

ChatGPT can make mistakes. Check important info.
Thanks for trying ChatGPT
Log in or sign up to get smarter responses, upload files and images, and more.

Log in

Sign up for free
Stay logged out
