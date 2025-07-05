# 🚀 Ubuntu Server Stack Installation Script

This repository contains a bash script to fully provision a **Node.js backend environment** on an Ubuntu-based server.  
It installs and configures all necessary tools for running modern web applications, including **MongoDB, Redis, Node.js (via FNM), PM2, PNPM**, and **Nginx**.

> ✅ Tested on: **Ubuntu 22.04 (Jammy Jellyfish)**  
> 🛠️ Use for: Fresh VM/server setup on cloud (e.g., GCP, AWS, DigitalOcean)

---

## 📂 Script: `setup-server-stack.sh`

```bash
#!/bin/bash
set -e

# ====== Utility function ======
log() {
  echo -e "\033[1;34m[INFO]\033[0m $1"
}

# ====== Update & install dependencies ======
log "Updating system..."
sudo apt-get update -y && sudo apt-get upgrade -y

log "Installing dependencies (curl, unzip, gnupg, etc.)..."
sudo apt-get install -y curl unzip gnupg lsb-release gpg ca-certificates software-properties-common build-essential

# ====== Install FNM ======
log "Installing FNM..."
mkdir -p ~/.local/share/fnm # ensure directory exists

curl -fsSL https://github.com/Schniz/fnm/releases/latest/download/fnm-linux.zip -o fnm.zip
unzip -q fnm.zip -d ~/.local/share/fnm && rm fnm.zip
chmod +x ~/.local/share/fnm/fnm

# Add FNM to shell
if ! grep -q 'fnm env' ~/.bashrc; then
  cat <<'EOF' >> ~/.bashrc

# FNM Setup
export PATH="$HOME/.local/share/fnm:$PATH"
eval "$(fnm env)"
EOF
fi

# Apply FNM in this script
export PATH="$HOME/.local/share/fnm:$PATH"
eval "$($HOME/.local/share/fnm/fnm env)"

log "FNM installed. Version: $(fnm --version)"

log "Installing latest LTS Node.js via FNM..."
fnm install --lts
fnm use lts-latest
log "Node version: $(node -v)"
log "NPM version: $(npm -v)"

# ====== Install PNPM ======
log "Installing PNPM..."
curl -fsSL https://get.pnpm.io/install.sh | sh -
export PATH="$HOME/.local/share/pnpm:$PATH"
if ! grep -q 'pnpm' ~/.bashrc; then
  echo 'export PATH="$HOME/.local/share/pnpm:$PATH"' >> ~/.bashrc
fi
log "PNPM version: $(pnpm -v)"

# ====== Install PM2 ======
log "Installing PM2 globally..."
npm install -g pm2
pm2 update
log "PM2 version: $(pm2 -v)"

# ====== Install MongoDB 7.0 ======
log "Setting up MongoDB 7.0 repository..."
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

log "Installing MongoDB..."
sudo apt-get update
sudo apt-get install -y mongodb-org

log "Enabling and starting MongoDB..."
sudo systemctl enable mongod
sudo systemctl start mongod
log "MongoDB status:"
sudo systemctl status mongod --no-pager

# ====== Install Redis ======
log "Adding Redis repository..."
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

log "Installing Redis..."
sudo apt-get update
sudo apt-get install -y redis

log "Enabling and starting Redis..."
sudo systemctl enable redis-server
sudo systemctl start redis-server
log "Redis status:"
sudo systemctl status redis-server --no-pager

# ====== Install Nginx ======
log "Installing Nginx..."
sudo apt install -y nginx

log "Enabling and starting Nginx..."
sudo systemctl enable nginx
sudo systemctl start nginx
log "Nginx status:"
sudo systemctl status nginx --no-pager

# ====== Final message ======
log "✅ Server stack installed successfully."
log "To use fnm and pnpm in new sessions, run: source ~/.bashrc"
log "You can now:"
echo "  - Clone your project repo"
echo "  - Run 'pnpm install'"
echo "  - Start with PM2 using: pm2 start server.js --name backend"
echo "  - Serve frontend with: pm2 serve build 3000 --spa --name frontend"

```

### 🔧 What it installs and configures:

- **System updates & common build tools**: `curl`, `unzip`, `gnupg`, `ca-certificates`, etc.
- **Node.js via FNM**: Fast Node Manager (supports multiple Node versions)
- **PNPM**: Fast, disk-efficient package manager
- **PM2**: Process manager to keep your Node app alive
- **MongoDB 7.0**: Installs and enables the service
- **Redis**: Installs and enables the service
- **Nginx**: Web server and reverse proxy

---

## 📦 How to Use

1. **Make the script executable**
   ```bash
   chmod +x setup-server-stack.sh
   ```

## 📗 After Install – What You Can Do

```bash
source ~/.bashrc           # Load fnm and pnpm paths in current shell
pnpm install               # Install dependencies from your cloned repo
pm2 start server.js        # Start your backend with PM2
pm2 serve build 3000 --spa --name frontend   # Serve frontend (SPA)

```
