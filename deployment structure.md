Great question! Let me explain the **execution flow**:

## Execution Timeline:

```
┌─────────────────────────────────────────────────────────────┐
│ YOU: Push code to GitHub (git push)                         │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ JENKINS SERVER: Jenkinsfile executes                        │
│ (4 Stages run here)                                         │
│                                                             │
│ Stage 1: Clone Repository ← On Jenkins server              │
│ Stage 2: Upload Project to VPS ← On Jenkins server         │
│ Stage 3: Build & Deploy on VPS ← CALLS vps-deploy.sh      │
│ Stage 4: Verification ← On Jenkins server                  │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ VPS SERVER: vps-deploy.sh executes                          │
│ (when Stage 3 triggers it)                                 │
│                                                             │
│ 1. Build JAR (mvn clean package)                           │
│ 2. Build Docker image (docker build)                       │
│ 3. Stop old container                                      │
│ 4. Start new container                                     │
│ 5. Verify and show logs                                    │
└─────────────────────────────────────────────────────────────┘
```

---

## **WHEN does it execute?**

### **Option 1: Automatically (Recommended)**

**When you push code to GitHub:**

```bash
git add .
git commit -m "Update application"
git push origin main
```

**Then automatically:**
1. GitHub notifies Jenkins (via webhook)
2. Jenkins clones your repo
3. Jenkinsfile executes automatically
4. vps-deploy.sh runs on VPS

---

### **Option 2: Manually trigger in Jenkins UI**

1. Open Jenkins dashboard
2. Click your project name
3. Click "Build Now"
4. Jenkins runs Jenkinsfile → vps-deploy.sh

---

### **Option 3: Scheduled (Cron)**

You can schedule it to run at specific times. Add this to Jenkinsfile:

```groovy
triggers {
    cron('0 2 * * *')  // Runs daily at 2 AM
}
```

---

## **Detailed Execution Breakdown:**

**Jenkinsfile runs on Jenkins Server (Your CI/CD machine):**

```bash
# Stage 1 - Runs on Jenkins
git clone https://github.com/saddamhdev/icsQuizUserService

# Stage 2 - Runs on Jenkins
scp -r . user@vps-ip:/www/wwwroot/CITSNVN/icsQuizUserService

# Stage 3 - Triggers script on VPS
ssh user@vps-ip "cd /www/wwwroot/CITSNVN/icsQuizUserService && bash vps-deploy.sh"
```

**vps-deploy.sh runs on VPS Server:**

```bash
# Now on VPS...
mvn clean package -DskipTests      # Build JAR
docker build -t icsquiz-user-service:latest .  # Build image
docker run -d -p 3090:3090 icsquiz-user-service:latest  # Run container
```

---

## **Practical Example:**

**Day 1: Setup**
```bash
# On your local machine
git clone https://github.com/saddamhdev/icsQuizUserService
cd icsQuizUserService

# Add Jenkinsfile and vps-deploy.sh
# Commit and push
git add .
git commit -m "Add CI/CD setup"
git push origin main
```

**Day 2: Make changes to code**
```bash
# Edit your Java files
# Commit and push
git add src/
git commit -m "Fixed bug in UserService"
git push origin main
```

**What happens automatically:**
1. ✅ GitHub webhook notifies Jenkins
2. ✅ Jenkins clones your updated repo
3. ✅ Stage 1: Clone (Jenkinsfile on Jenkins)
4. ✅ Stage 2: Upload (Jenkinsfile on Jenkins)
5. ✅ Stage 3: Executes vps-deploy.sh (on VPS)
   - VPS builds new JAR
   - VPS builds new Docker image
   - VPS stops old container
   - VPS starts new container with latest code
6. ✅ Stage 4: Verification (Jenkinsfile checks if running)
7. ✅ **Your updated application is live!**

---

## **Jenkins Web Hook Setup** (for automatic trigger):

1. Go to Jenkins → Your Project → Configure
2. Check: **"GitHub hook trigger for GITScm polling"**
3. Go to GitHub → Settings → Webhooks → Add webhook
4. Payload URL: `http://your-jenkins-ip:8080/github-webhook/`
5. Click Add

Now every `git push` triggers Jenkins automatically!

---

## **Where are they stored?**

| File | Location | When executes |
|------|----------|---------------|
| **Jenkinsfile** | GitHub repo root | On Jenkins server (automatic) |
| **vps-deploy.sh** | GitHub repo root | On VPS (called by Jenkinsfile) |

Does this clear things up? Want help setting up the GitHub webhook?
