Below is a **complete, clean, GitHub-ready documentation** on how to solve:

```
ERROR: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
```

This is the **exact problem you faced**, and this documentation explains the **root cause, symptoms, and permanent fix**.

---

# 🚨 How to Fix

**`ERROR: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory`**

This error appears when Docker **cannot read** your `Dockerfile` — **even if the file exists**.

This usually happens when:

* You installed **Docker via SNAP**
* The `Dockerfile` is stored in a restricted directory (e.g., `/www/wwwroot`, `/mnt`, `/opt`)
* Snap Docker is running inside a sandbox and cannot see system directories

This documentation explains:

1. **Why this happens**
2. **How to confirm the issue**
3. **Temporary workaround**
4. **PERMANENT FIX (recommended)**
5. **Verification steps**

---

# ❗ 1. Root Cause: Snap Docker Sandbox Blocking File Access

If you installed Docker using:

```bash
sudo snap install docker
```

You are using **Snap Docker**, which runs inside a sandbox with limited filesystem access.

Snap Docker cannot access:

| Path           | Access    |
| -------------- | --------- |
| `/www`         | ❌ Blocked |
| `/www/wwwroot` | ❌ Blocked |
| `/opt/*`       | ❌ Blocked |
| `/mnt/*`       | ❌ Blocked |
| `/var/www/*`   | ❌ Blocked |
| `/root/*`      | ✔ Allowed |
| `/home/*`      | ✔ Allowed |

So Docker thinks your Dockerfile “does not exist”, even though it does.

### Example:

```
Dockerfile exists:
-rw-r--r-- 1 root root 165 Dockerfile

But Docker says:
failed to read dockerfile: open Dockerfile: no such file or directory
```

This is a **Snap sandbox problem**, NOT your file.

---

# 📌 2. How to Confirm It’s Snap Docker Issue

Run:

```bash
which docker
```

If output is:

```
/snap/bin/docker
```

= You are using SNAP Docker (problematic).

Also run:

```bash
docker info | grep -i snap
```

If you see Snap directories → same issue.

---

# 🩹 3. Temporary Workaround (moves files to /root)

Move your project files to a location Snap can see:

```bash
mkdir -p /root/docker-build
cp Dockerfile /root/docker-build/
cp your-app.jar /root/docker-build/
cd /root/docker-build
docker build -t your-app .
```

This works, but it is NOT a permanent fix.

---

# 🔥 4. PERMANENT FIX (Recommended)

## Step 1 — Remove Snap Docker

```bash
sudo snap remove docker
```

## Step 2 — Install Official Docker (CE)

```bash
curl -fsSL https://get.docker.com | sh
```

If you get a GPG key error:

Add Docker's official key:

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
 | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Add Docker repo:

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update & install:

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Check Docker version:

```bash
docker --version
```

➡️ Should show something like:

```
Docker version 29.x.x
```

---

# 🎯 5. Why This Fix Works

Official Docker (from docker.com):

* Runs as a **full system service**
* Has access to **entire filesystem**
* Can read Dockerfiles **anywhere**
* Supports Jenkins, CI/CD, pipelines
* Supports volumes, networking, reverse proxies

Snap Docker:

* Runs in a **restricted sandbox**
* Cannot read files outside `/root` or `/home`
* Breaks Jenkins pipelines
* Blocks Dockerfile access
* Causes misleading errors

---

# 🎉 6. After Installing Official Docker

You can now run:

```bash
docker build -t app .
```

From any directory, including:

```
/www/wwwroot/
/var/www/
/opt/
/mnt/
```

Everything works — **no more errors**.

---

# 📘 Summary (Copy-Ready)

| Problem                            | Solution                                            |
| ---------------------------------- | --------------------------------------------------- |
| `failed to read dockerfile`        | Snap Docker sandbox cannot access project directory |
| Dockerfile exists but not detected | Docker runs inside /snap isolation                  |
| Jenkins CI/CD failing              | Snap Docker permission issues                       |
| Fix                                | Remove Snap Docker → install official Docker CE     |

---

# Need a PDF version of this documentation?

Just say:
👉 **“Give me PDF of this documentation.”**

I can also generate a **GitHub README.md** version.
