# AFS Cloud Deployment Guide

## Overview
This document explains how to keep your Node.js backend server running persistently on the college AFS cloud, eliminating the need to manually run `node server.js` every time someone accesses your website.

---

## Problem Statement
When deploying a Node.js application to a cloud environment, the server process (`node server.js`) needs to run continuously to handle incoming requests. Without a process manager, the server stops when:
- You close your terminal/SSH session
- The server crashes
- The system reboots

---

## Solutions for Persistent Server Deployment

### Option 1: Using PM2 (Recommended)
**PM2** is a production-grade process manager for Node.js applications that keeps your server running indefinitely.

#### Installation
```bash
npm install -g pm2
```

#### Starting Your Server with PM2
```bash
# Navigate to your Backend_VM directory
cd Backend_VM

# Start the server with PM2
pm2 start server.js --name vibebuy-backend

# Save the PM2 process list (ensures it restarts after reboot)
pm2 save

# Setup PM2 to start on system boot
pm2 startup
```

#### Useful PM2 Commands
```bash
# View running processes
pm2 list

# View logs
pm2 logs vibebuy-backend

# Restart the server
pm2 restart vibebuy-backend

# Stop the server
pm2 stop vibebuy-backend

# Delete the process
pm2 delete vibebuy-backend

# Monitor CPU/Memory usage
pm2 monit
```

#### Benefits
- ✅ Auto-restart on crashes
- ✅ Survives system reboots
- ✅ Built-in logging
- ✅ Zero-downtime reloads
- ✅ Load balancing (cluster mode)

---

### Option 2: Using Forever
**Forever** is a simpler alternative to PM2 for keeping Node.js processes running.

#### Installation
```bash
npm install -g forever
```

#### Starting Your Server with Forever
```bash
# Navigate to your Backend_VM directory
cd Backend_VM

# Start the server
forever start server.js

# Start with custom log file
forever start -l vibebuy.log -a server.js
```

#### Useful Forever Commands
```bash
# List running processes
forever list

# Stop the server
forever stop server.js

# Restart the server
forever restart server.js

# Stop all forever processes
forever stopall
```

---

### Option 3: Using nohup (Basic Solution)
**nohup** runs a command immune to hangups, allowing it to continue after logout.

#### Starting Your Server with nohup
```bash
# Navigate to your Backend_VM directory
cd Backend_VM

# Start the server in background
nohup node server.js > server.log 2>&1 &

# Note the process ID (PID) displayed
```

#### Managing nohup Processes
```bash
# Find the process ID
ps aux | grep "node server.js"

# Stop the server (replace PID with actual process ID)
kill <PID>

# View logs
tail -f server.log
```

#### Limitations
- ❌ No auto-restart on crashes
- ❌ Manual process management
- ❌ Doesn't survive reboots automatically

---

### Option 4: Using systemd (Linux Systems)
If your AFS cloud runs on a Linux system with systemd, you can create a service.

#### Create Service File
```bash
sudo nano /etc/systemd/system/vibebuy-backend.service
```

#### Service Configuration
```ini
[Unit]
Description=VibeBuy Backend Server
After=network.target

[Service]
Type=simple
User=<your-username>
WorkingDirectory=/path/to/Backend_VM
ExecStart=/usr/bin/node server.js
Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=vibebuy-backend

Environment=NODE_ENV=production
Environment=PORT=5000

[Install]
WantedBy=multi-user.target
```

#### Managing the Service
```bash
# Reload systemd
sudo systemctl daemon-reload

# Start the service
sudo systemctl start vibebuy-backend

# Enable auto-start on boot
sudo systemctl enable vibebuy-backend

# Check status
sudo systemctl status vibebuy-backend

# View logs
sudo journalctl -u vibebuy-backend -f

# Restart service
sudo systemctl restart vibebuy-backend

# Stop service
sudo systemctl stop vibebuy-backend
```

---

## Recommended Setup for AFS Cloud

### Step-by-Step Deployment

1. **SSH into your AFS cloud server**
   ```bash
   ssh <your-username>@<afs-server-address>
   ```

2. **Navigate to your project directory**
   ```bash
   cd /path/to/IT340-Project-VibeBuy_Redundancy1/Backend_VM
   ```

3. **Install dependencies (if not already done)**
   ```bash
   npm install
   ```

4. **Install PM2 globally**
   ```bash
   npm install -g pm2
   ```

5. **Start your server with PM2**
   ```bash
   pm2 start server.js --name vibebuy-backend
   pm2 save
   pm2 startup
   ```

6. **Verify it's running**
   ```bash
   pm2 list
   pm2 logs vibebuy-backend
   ```

7. **Test your deployment**
   - Access your website URL
   - The backend should respond without needing to manually run `node server.js`

---

## Environment Variables

Ensure your `.env` file is properly configured for production:

```env
PORT=5000
MONGODB_URI=mongodb://<database-vm-ip>:27017/vibebuy
NODE_ENV=production
```

> **Note**: Never commit `.env` files to version control. Keep sensitive credentials secure.

---

## Monitoring and Maintenance

### Check Server Status
```bash
# With PM2
pm2 status

# Check if port is listening
netstat -tuln | grep 5000
```

### View Real-time Logs
```bash
# With PM2
pm2 logs vibebuy-backend --lines 100

# With tail (if using nohup)
tail -f server.log
```

### Restart After Code Changes
```bash
# With PM2
pm2 restart vibebuy-backend

# Or reload with zero downtime
pm2 reload vibebuy-backend
```

---

## Troubleshooting

### Server Not Starting
1. Check if port 5000 is already in use:
   ```bash
   lsof -i :5000
   ```

2. Check MongoDB connection:
   ```bash
   # Verify MongoDB is accessible
   nc -zv <database-vm-ip> 27017
   ```

3. Review error logs:
   ```bash
   pm2 logs vibebuy-backend --err
   ```

### Server Stops Unexpectedly
1. Check PM2 status:
   ```bash
   pm2 list
   ```

2. Enable auto-restart:
   ```bash
   pm2 startup
   pm2 save
   ```

3. Check system resources:
   ```bash
   free -h  # Memory
   df -h    # Disk space
   ```

### Cannot Access Website
1. Verify server is running:
   ```bash
   pm2 status
   curl http://localhost:5000
   ```

2. Check firewall rules (if applicable):
   ```bash
   sudo ufw status
   ```

3. Verify DNS/domain configuration pointing to correct IP

---

## Security Best Practices

1. **Use Environment Variables**: Never hardcode sensitive data
2. **Enable CORS Properly**: Configure allowed origins in production
3. **Use HTTPS**: Set up SSL/TLS certificates
4. **Keep Dependencies Updated**: Regularly run `npm audit fix`
5. **Limit Access**: Use firewall rules to restrict access to necessary ports only
6. **Monitor Logs**: Regularly check for suspicious activity

---

## Quick Reference

| Task | Command |
|------|---------|
| Start server | `pm2 start server.js --name vibebuy-backend` |
| Stop server | `pm2 stop vibebuy-backend` |
| Restart server | `pm2 restart vibebuy-backend` |
| View logs | `pm2 logs vibebuy-backend` |
| Check status | `pm2 status` |
| Save process list | `pm2 save` |
| Auto-start on boot | `pm2 startup` |

---

## Summary

By using a process manager like **PM2** (recommended), your Node.js backend server will:
- ✅ Run continuously without manual intervention
- ✅ Automatically restart on crashes
- ✅ Survive system reboots
- ✅ Be accessible whenever someone clicks your website link
- ✅ Provide easy monitoring and logging

**No more need to run `node server.js` manually!** 🎉
