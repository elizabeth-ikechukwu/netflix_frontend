# Netflix Clone — Frontend

> Part of [netflix-3tier-aws](https://github.com/elizabeth-ikechukwu/netflix-clone-aws) — see full architecture and overview there.

A React frontend manually deployed to AWS EC2 as part of a 3-tier cloud infrastructure project. Fetches and displays movie data from a Java Spring Boot backend via Axios.

---

## Stack

| Layer | Technology |
|---|---|
| Framework | React |
| HTTP Client | Axios |
| Runtime | Node.js 20 |
| Static Server | serve |
| Server | AWS EC2 (Ubuntu 22.04, t3.micro) |
| Process Management | systemd |

---

## Configuration

Edit `src/api/axiosConfig.js`:
```javascript
export default axios.create({
  baseURL: 'http://<BACKEND_EC2_IP>:8080',
});
```

> Do not add `/api` to the baseURL. The API calls already include the full path. Adding it causes a double `/api/api/` error.

---

## Deployment

**Prerequisites:** Security group with port 3000 open.
```bash
# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node -v
npm -v

# Install serve
sudo npm install -g serve

# Swapfile — required on t3.micro before building
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Clone
git clone https://github.com/elizabeth-ikechukwu/netflix_frontend.git
cd netflix_frontend

# Install dependencies
npm install

# Build
npm run build

# Run
serve -s build -l 3000
```

### Run as a Service

Create `/etc/systemd/system/netflix-frontend.service`:
```ini
[Unit]
Description=Netflix Frontend
After=network.target

[Service]
WorkingDirectory=/home/ubuntu/netflix_frontend
ExecStart=/usr/bin/serve -s build -l 3000
Restart=always
User=ubuntu
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable netflix-frontend
sudo systemctl start netflix-frontend
sudo systemctl status netflix-frontend
```

---

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Build crashes on t3.micro | No swap memory | Create 1GB swapfile before building |
| Double `/api/api/` in requests | Extra `/api` in baseURL | Remove `/api` from axiosConfig baseURL |
| serve starts on wrong port | Missing `-l` flag | Use `serve -s build -l 3000` |
| Port 3000 unreachable | Security group | Add inbound TCP 3000, 0.0.0.0/0 |

---

## Author

**Ikechukwu Elizabeth Nkwo**
Cloud and DevOps Engineer Trainee — AWS · Linux · Infrastructure
[LinkedIn](https://linkedin.com/in/uroko-elizabeth-) · [GitHub](https://github.com/elizabeth-ikechukwu)
```

---

Commit message for both:
```
Update README — add Java 17 and Node.js 20 installation steps
