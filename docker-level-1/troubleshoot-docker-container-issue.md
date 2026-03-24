# Troubleshoot Docker Container Issue

## 📘 GitBook Documentation

## Fixing Static Website Container Issue on App Server 1

***

### 📌 Environment Details

| Server  | Hostname                        | IP            | Purpose               |
| ------- | ------------------------------- | ------------- | --------------------- |
| stapp01 | stapp01.stratos.xfusioncorp.com | 172.16.238.10 | Nautilus App Server 1 |

Container Name: **nautilus**\
Image Used: **httpd**\
Expected Access: `http://localhost:8080`

***

## 🔐 Step 1: Connect to App Server 1

Login from jump host:

```bash
ssh tony@stapp01
```

Accept host authenticity:

```bash
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

Switch to root:

```bash
sudo su -
```

***

## 🔎 Step 2: Verify Container Status

Check running containers:

```bash
docker ps
```

No running containers found.

Check all containers:

```bash
docker ps -a
```

Output:

```
CONTAINER ID   IMAGE     COMMAND              STATUS
e8c0892856f9   httpd     "httpd-foreground"   Exited (0)   nautilus
```

🔎 Observation:\
Container `nautilus` exists but is stopped.

***

## 🔄 Step 3: Restart the Container

```bash
docker restart nautilus
```

Verify again:

```bash
docker ps
```

Output:

```
CONTAINER ID   IMAGE     COMMAND              STATUS       PORTS                  NAMES
e8c0892856f9   httpd     "httpd-foreground"   Up 6 seconds 0.0.0.0:8080->80/tcp   nautilus
```

✅ Container is now running\
✅ Port mapping is correct (8080 → 80)

***

## 🌐 Step 4: Verify Website Accessibility

Run:

```bash
curl http://localhost:8080
```

Output:

```
Welcome to xFusionCorp Industries!
```

✅ Website is accessible\
✅ Content is being served successfully\
✅ Port 8080 working correctly

***

## 🔍 Validation Checklist

| Check                | Command                      | Status |
| -------------------- | ---------------------------- | ------ |
| Container exists     | `docker ps -a`               | ✅      |
| Container running    | `docker ps`                  | ✅      |
| Port mapping correct | `docker ps`                  | ✅      |
| Website accessible   | `curl http://localhost:8080` | ✅      |

***

## 🎯 Root Cause

The container `nautilus` was in **Exited state**.\
Restarting the container resolved the issue.

***

## 🏁 Final Result

✔ Static website is running successfully on App Server 1\
✔ Accessible via `http://localhost:8080`\
✔ Container `nautilus` running properly

***

**Status: Issue Resolved Successfully** 🚀
