# Ultimate Free AI Dev Stack — Elite Enterprise Installation Guide (v2025.06)

## Executive Summary

Deploy a secure, offline-capable, and extensible AI development environment with zero SaaS dependencies. This guide enables IT teams and non-expert users to build, test, and operate modern AI workflows using only free, open-source tools. Now enhanced with observability and self-healing for true enterprise resilience.

---

## Full Visual System Architecture — Ultimate Free AI Dev Stack (2025)

### Visual Diagram (Markdown/ASCII)

```
+-----------------------------------------------+
|              Windows 11 Host (v23H2+)         |
|   [User Desktop, Browser, Firewall]           |
+---------------------+-------------------------+
                      |
                      v
+-----------------------------------------------+
|          WSL2 Virtualization Layer            |
|         (Ubuntu 22.04 LTS - Isolated)         |
+---------------------+-------------------------+
                      |
                      v
+---------------------------------------------------------------+
|                  Core Infrastructure (WSL2)                   |
|                                                               |
|   +-------------+      +---------------+     +-------------+  |
|   |  Docker     |      |  Node.js      |     |  Python     |  |
|   |  Compose    |      |  (nvm, npm)   |     |  (venv)     |  |
|   +-------------+      +---------------+     +-------------+  |
|                                                               |
|   +-------------------+    +-------------------+              |
|   |     Fail2ban      |    |      UFW          |              |
|   | (Intrusion Prev.) |    | (Firewall)        |              |
|   +-------------------+    +-------------------+              |
+---------------------+-------------------------+----------------+
                      |
                      v
+--------------------------------------------------------------------------------------+
|                       AI Agent/Dev Layer (MCP Framework)                            |
|                                                                                      |
|  +----------------+   +----------------+   +--------------------+                   |
|  | Continue.dev   |   | AnythingLLM    |   |    Ollama (LLMs)   |                   |
|  | (VS Code,      |   | (Memory/RAG)   |   |  (Mistral, Phi-2)  |                   |
|  |  running       |   |                |   |                    |                   |
|  |  inside WSL)   |   +----------------+   +--------------------+                   |
|  +----------------+           |                    |                               |
|         |                     |                    |                               |
|         +---------+-----------+----------+---------+                               |
|                   |                      |                                         |
|        [MCP Layer: Context/Agent Orchestration, API, CLI]                           |
|                   |                      |                                         |
+-------------------+----------------------+------------------------------------------+
                      |
                      v
+--------------------------------------------------------+
|           Observability & Self-Healing Layer            |
|                                                        |
|   +---------------+     +-------------+   +----------+ |
|   |  Prometheus   |<--->| Node Export |<->| cAdvisor | |
|   | (Collector)   |     |  (Metrics)  |   | (Docker) | |
|   +---------------+     +-------------+   +----------+ |
|           |                            |               |
|           v                            v               |
|       Grafana (Dashboards, Alerts)  Alertmanager       |
+--------------------------------------------------------+
                      |
                      v
+-------------------+
|  End User / IT    |
|  Dashboards &     |
|  Notifications    |
+-------------------+
```

---

## Step 1: Prepare Windows 11 Host

- **Enable Virtualization in BIOS/UEFI:**

  - Restart PC, access BIOS (Del/F2/Esc at boot)
  - Enable “Virtualization Technology”/“Intel VT-x/AMD-V”
  - Save, reboot

- **Install WSL2:**

  - Open PowerShell as Admin
  - Run: wsl --install Ubuntu-24.04

- **Update Windows and WSL2:**

  - `wsl --update`

---

## Step 2: Set Up Ubuntu 22.04 in WSL2
A
- ** `:**
  ```bash
  sudo usermod -aG sudo $USER
  echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER
  sudo chmod 0440 /etc/sudoers.d/$USER
  ```
B
- **Launch Ubuntu from Start Menu**
- **Update Linux system:**
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```
- **Set your username and password when prompted**

---

## Step 3: Install Core Development Tools

```bash
sudo apt install -y build-essential curl git wget unzip python3-pip python3-venv
python3 --version
python3 -m venv .venv
source .venv/bin/activate
```

**Install Node.js (v20.x):**

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
node -v && npm -v
```

---

## Step 4: Install Docker & Docker Compose

```bash
sudo apt install docker.io docker-compose -y
sudo usermod -aG docker $USER
newgrp docker
sudo systemctl enable docker
sudo systemctl start docker
docker --version
docker-compose --version
```

---

## Step 5: Secure Your System

**Install Firewall (UFW):**

```bash
sudo apt install ufw -y
sudo ufw enable
sudo ufw allow OpenSSH
sudo ufw allow 9898   # AnythingLLM
sudo ufw allow 11434  # Ollama
sudo ufw allow 9000   # (If using SMB/9P file sharing)
sudo ufw status
```

**Install Fail2ban:**

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status
```

---

## Step 6: Set Up GitHub SSH Access (Optional)

**Generate SSH key:**

```bash
ssh-keygen -t ed25519 -C "your@email.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

**Add your public key to GitHub:**\
Go to GitHub → Settings → SSH & GPG Keys → New SSH Key

**Test SSH connection:**

```bash
ssh -T git@github.com
```

---

## Step 7: Install AI Agents (MCP Layer)

### a. Ollama (Local LLM Runner)

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama run mistral
# To try other models:
ollama run phi
ollama run mixtral
```

### b. AnythingLLM (Memory/RAG Agent)

```bash
git clone https://github.com/Mintplex-Labs/anything-llm.git
cd anything-llm
cp .env.example .env
nano .env  # Set HOST, PORT, MODEL values
cd server
npm install
HOST=127.0.0.1 PORT=9898 npm run dev
# AnythingLLM now running locally on port 9898
```

### c. Continue.dev (VS Code Copilot)

- Open VS Code
- Go to Extensions → Search for “Continue - AI Assistant” → Install
- Press `Ctrl+Shift+P` → “Continue: Start New Chat”

---

## Step 8: Validate, Test, and Use

- **Check each service:**
  - Docker: `docker ps`
  - Ollama: `ollama list`, open [http://localhost:11434](http://localhost:11434) if enabled
  - AnythingLLM: open [http://localhost:9898](http://localhost:9898)
  - Continue.dev: open a chat in VS Code
- **Review logs for errors and resolve using troubleshooting guide**

---

## Step 9: Backup & Ongoing Maintenance

**Automate encrypted backups (recommended):**

```bash
# Example using restic
sudo apt install restic -y
restic init -r /mnt/backup/llm-backups
restic backup --repo /mnt/backup/llm-backups ~/projects/freedom-stack-ai
# Schedule with cron for daily/weekly backups
```

**Update regularly:**

- Ubuntu: `sudo apt update && sudo apt upgrade -y`
- Docker: `sudo apt install docker.io -y`
- VS Code, Node, Python, NPM: as per official docs

---

## Step 10: Troubleshooting (Quick Reference)

| Issue                                 | Solution                                               |
| ------------------------------------- | ------------------------------------------------------ |
| WSL2 does not start                   | Update WSL2, reboot, check virtualization              |
| Docker "permission denied"            | `sudo usermod -aG docker $USER && newgrp docker`       |
| Ollama model won’t download           | Check internet, retry, check disk space                |
| AnythingLLM port error                | Verify port availability, adjust firewall              |
| Continue.dev not appearing in VS Code | Reinstall extension, restart VS Code, check agent logs |
| Fail2ban not starting                 | Check `/var/log/fail2ban.log` for errors               |
| Security update failures              | Run `sudo apt update && sudo apt upgrade`              |

---

## Step 11: FAQ

- **Can this stack run fully offline?** Yes, after initial model/tool downloads.
- **Can I use this for multiple users?** Yes—enable RBAC and configure per-user project directories.
- **How do I ensure backups are encrypted?** Use tools like `restic` with password-protected repos.
- **How do I expand the agent layer?** Integrate new REST/CLI tools or VS Code extensions using the MCP integration model.

---

## Step 12: Clone & Initialize Your Project (Optional)

After all tools and security controls are in place, you can clone or initialize your project repository for development.

```bash
mkdir -p ~/projects/freedom-stack-ai
cd ~/projects/freedom-stack-ai
git init
touch README.md
echo "# Freedom Stack AI" >> README.md
```

**(If using GitHub) Connect your remote repo:**

```bash
git remote add origin git@github.com:your-username/freedom-stack-ai.git
git branch -M main
git push -u origin main
```

---

## MCP/AI Agent Layer — Architecture, Extensibility, and Workflow

**Overview:**\
The Multi-Context Protocol (MCP) orchestrates agent interactions, automating secure data flows and context exchange across Continue.dev, Ollama, and AnythingLLM. This pattern is designed for extensibility, privacy, and enterprise readiness.

**MCP Layer Diagram:**

```
[User]
  |
  v
[VS Code (Continue.dev)] <—> [AnythingLLM]
        |                          |
        v                          v
    [Ollama (LLMs)] <———————> [MCP Orchestration Layer]
             |                    |
     [Secure Output/Logs/Actions] |
                   |______________|
```

- Add new agents via REST/CLI
- Chain complex workflows for code review, QA, compliance, etc.
- Keep all context and inference local for privacy

---

## Security Reference — Best Practices & Compliance

- Patch OS/Docker/agent dependencies regularly; automate checks
- Run agents under dedicated Unix users, restrict .env/keys
- Encrypted backups (restic/duplicity), schedule cron jobs
- Firewall: open only agent/SSH ports; block outbound where possible
- Retain and review agent logs for audits; map controls to CIS/NIST/ISO

---

# Observability & Self-Healing — Non-Expert, Enterprise Edition

## Purpose & Business Value

Observability and self-healing means your stack is “always-on,” self-monitoring, and easy to manage—even for non-expert users:

- Instantly see if anything is wrong—before it impacts your work.
- Get automatic notifications (email, Slack) if agents/services fail or resources spike.
- Services restart automatically, minimizing downtime and manual troubleshooting.
- All performance data is stored and visualized for proactive support and compliance.

---

## System Diagram

```
[AI Dev Stack: Ollama | AnythingLLM | Continue.dev]
        |
  [Prometheus Exporters: Node Exporter, cAdvisor]
        |
    [Prometheus Collector]
        |
    [Grafana Dashboard]
        |
 [User & Operator Alerts (Email/Slack)]
```

---

## Step-by-Step Guide

### 1. Install Prometheus (Metrics Collector)

```bash
sudo apt update && sudo apt install prometheus -y
```

### 2. Install Node Exporter (System Health)

```bash
sudo apt install prometheus-node-exporter -y
sudo systemctl enable prometheus-node-exporter
sudo systemctl start prometheus-node-exporter
```

### 3. (Optional) Monitor Docker Containers

```bash
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  gcr.io/cadvisor/cadvisor:latest
```

### 4. Install Grafana (Visualization Dashboard)

```bash
sudo apt install -y apt-transport-https software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana -y
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

- **Access Grafana:** [http://localhost:3000](http://localhost:3000)\
  (default login: admin / admin)

---

### 5. Connect Prometheus to Grafana

- In Grafana, add Prometheus as a data source (default URL: `http://localhost:9090`).
- Import dashboards from Grafana.com or create panels for CPU, memory, disk, and container health.

---

### 6. Enable Automated Alerts

- **Install Alertmanager:**
  ```bash
  sudo apt install prometheus-alertmanager -y
  sudo systemctl enable alertmanager
  sudo systemctl start alertmanager
  ```
- **Sample alert rule:** (Edit `/etc/prometheus/alert.rules`)
  ```yaml
  groups:
  - name: agent-alerts
    rules:
    - alert: HighCPUUsage
      expr: 100 * (1 - avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m]))) > 90
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU Usage (>90%)"
        description: "CPU usage above 90% for 5 minutes."
    - alert: AgentDown
      expr: up{job="ollama"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Ollama agent is down"
        description: "Ollama agent not responding."
  ```
- **Restart Prometheus:**
  ```bash
  sudo systemctl restart prometheus
  ```

---

### 7. Self-Healing — Auto-Restart on Failure

- **systemd services:**\
  Ensure each agent and exporter uses `Restart=always` in its systemd unit file.
  ```ini
  [Service]
  ExecStart=/usr/local/bin/ollama run mistral
  Restart=always
  RestartSec=5
  ```
- **Docker containers:**\
  Always use `--restart=always` with `docker run` or in `docker-compose.yml`.

---

## Best Practices for Non-Expert Users

- **Check Grafana regularly** to see health dashboards (browser, user-friendly).
- **Respond to alerts** sent by email or Slack.
- **Most problems heal automatically**—if a service crashes, it restarts on its own.
- **If issues repeat**, contact your technical lead with screenshots/logs.
- **Back up configs** for Prometheus and Grafana periodically.

---

## Integration Note

- This section should follow “Security Reference — Best Practices & Compliance.”
- Place extended guides and sample dashboards in `/docs/observability.md`.

---

**With this, even non-experts can keep your AI platform running like a Fortune 500 enterprise—24/7, visible, resilient, and future-proof.**

