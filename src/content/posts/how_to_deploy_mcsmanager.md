---
title: "How to deploy MCSManager Panel?"
published: 2024-02-21
description: 'A tutorial for deploy a modern and easy manager panel.'
image: ''
tags: [
  dev,
  mcsm,
  en
]
category: 'Tutorial'
draft: false 
lang: 'en'
---

> [Official Website](https://mcsmanager.com) & [Official Docs](https://docs.mcsmanager.com)

## What is MCSManager Panel?

> MCSManager Panel（abbr: MCSM Panel）is a multilingual, lightweight, out-of-the-box, and multi-instance Minecraft server control panel with Docker support.

MCSManager Panel is a modern and a panel that support any excutable files. Includes: Steamapps, Minecraft, Terraria, Python files, Java files, etc.

## Installation:

### Installation with one command (Recommand)
> x86 Ubuntu/CentOS/Debian/Archlinux **ONLY**!
``` bash
wget -qO- https://gitee.com/mcsmanager/script/raw/master/setup_cn.sh | bash # Root User
wget -qO- https://gitee.com/mcsmanager/script/raw/master/setup_cn.sh | sudo bash # non-Root User
```
- You are able to use `systemctl start mcsm-{web,daemon}` or `sudo systemctl start mcsm-{web,daemon}` for non-Root user to start the panel.
- The default installation directory is `/opt/mcsmanager`
- After you finished this step, you are able to visit your Panel at `http://yourdomain:23333` or `http://yourip:23333`.

### Manual installation
``` bash
# Switch to the installation directory. If not existed, create with 'mkdir /opt/'
cd /opt/

# If not done already, install Node.js runtime (14+ required).
# Download node-v14.17.6, you can also use another version. The minimum requirement is v14.
wget https://nodejs.org/dist/v14.17.6/node-v14.17.6-linux-x64.tar.gz

# Extract required files.
tar -zxvf node-v14.17.6-linux-x64.tar.gz

# Add Node.js to system PATH
ln -s /opt/node-v14.17.6-linux-x64/bin/node /usr/bin/node
ln -s /opt/node-v14.17.6-linux-x64/bin/npm /usr/bin/npm

# Create and switch to installation directory
mkdir /opt/mcsmanager/
cd /opt/mcsmanager/

# Download the web project. (Skip this if you do not plan to run web panel on this machine)
git clone https://github.com/MCSManager/MCSManager-Web-Production.git web
cd web
# Install dependencies
npm install --production
cd /opt/mcsmanager/

# Download the Daemon (Skip this if you do not plan to run daemon service on this machine.)
git clone https://github.com/MCSManager/MCSManager-Daemon-Production.git daemon
cd daemon
# Install dependencies
npm install --production

# You need two terinals or Screens for the following step.
# Run the daemon first (Skip this if you do not plan to run daemon service on this machine.)
cd /opt/mcsmanager/daemon
# Start the daemon
node app.js

# Run the web project (in another screen) (Skip this if you do not plan to run web panel on this machine)
cd /opt/mcsmanager/web
# start the application
node app.js

# Access http://localhost:23333/ for web interface
```

## Confirgure Proxy (Recommand)

> Nginx is required

- This step is required for those who wish to make the panel public. It is recommand to proxy both web and daemon.

``` bash
# /etc/nginx/nginx.conf
http {
    
    # limit upload size
    client_max_body_size 100g;

    server {
        # Web to public
        listen 80;
        listen 443 ssl;
        ssl_certificate /path/to/file;
        ssl_certificate_key /path/to/file;
    
        location / {
            # Web 
            proxy_pass http://localhost:23333/;
            root   html;
            index  index.html index.htm;
            proxy_set_header Host localhost;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header REMOTE-HOST $remote_addr;
            # Websocket
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            add_header X-Cache $upstream_cache_status;
            add_header Cache-Control no-cache;
            expires -1;
        }
    }

    server {
        # Daemon to public
        listen 8443 ssl;
        ssl_certificate /path/to/file;
        ssl_certificate_key /path/to/file;

        location / {
            # daemon
            proxy_pass http://localhost:24444/;
            root   html;
            index  index.html index.htm;
            proxy_set_header Host localhost;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header REMOTE-HOST $remote_addr;
            # websocket
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            add_header X-Cache $upstream_cache_status;
            add_header Cache-Control no-cache;
            expires -1;
        }
    }
}
```

- Replace `/path/to/file` to your SSL certificate and Key file.
- After you finished confirgure proxy, you are able to visit your Panel at `https://yourdomain.com`.

p.s. You have to change daemons connections address to proxied by using this format `wss://yourdomain` or `wss://yourip`, also change port to proxied port.

If you have a question or problem, please join [the Official Discord community](https://discord.gg/RyTnUjW7WB) to ask for help.