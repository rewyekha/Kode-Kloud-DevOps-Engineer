# Day 22: Clone Git Repository on Storage Server

The DevOps team established a new Git repository last week, which remains unused at present. However, the Nautilus application development team now requires a copy of this repository on the `Storage Server` in the Stratos DC. Follow the provided details to clone the repository:

1. The repository to be cloned is located at `/opt/news.git`
2. Clone this Git repository to the `/usr/src/kodekloudrepos` directory. Perform this task using the natasha user, and ensure that no modifications are made to the repository or existing directories, such as changing permissions or making unauthorized alterations.

## Clone Git Repository on Storage Server

### 📌 Task Description

The DevOps team created a new Git repository that needs to be cloned on the **Storage Server** in the Stratos DC.

#### Requirements

* Repository location:\
  `/opt/games.git`
* Destination directory:\
  `/usr/src/kodekloudrepos`
* User:\
  `natasha`
* Do **not** modify:
  * Repository permissions
  * Existing directory permissions
  * Any system configuration

***

### 🖥️ Infrastructure Details

| Server   | Hostname                         | User    | Purpose        |
| -------- | -------------------------------- | ------- | -------------- |
| ststor01 | ststor01.stratos.xfusioncorp.com | natasha | Storage Server |

***

## ✅ Solution Steps

***

### Step 1: Connect to Storage Server

From the jump host:

```bash
ssh natasha@ststor01
```

***

### Step 2: Verify Repository Exists

```bash
ls -ld /opt/games.git
```

#### Expected Output

```bash
drwxr-xr-x 7 natasha natasha 4096 Feb 11 04:15 /opt/games.git
```

***

### Step 3: Clone the Repository

Clone the repository **inside** `/usr/src/kodekloudrepos`:

```bash
git clone /opt/games.git /usr/src/kodekloudrepos/games
```

#### Expected Output

```bash
Cloning into '/usr/src/kodekloudrepos/games'...
warning: You appear to have cloned an empty repository.
done.
```

> ⚠️ The warning is normal because the repository is empty.

***

### Step 4: Verify Clone

#### Check Parent Directory

```bash
ls -la /usr/src/kodekloudrepos
```

Expected:

```bash
games
```

#### Check Repository Directory

```bash
ls -la /usr/src/kodekloudrepos/games
```

Expected:

```bash
.git
```

***

## 📂 Final Directory Structure

```
/usr/src/kodekloudrepos/
└── games/
    └── .git/
```

***

## 🎯 Validation Checklist

* ✅ Logged in as `natasha`
* ✅ Repository `/opt/games.git` exists
* ✅ Cloned under `/usr/src/kodekloudrepos/games`
* ✅ No permission changes made
* ✅ No existing directories modified
