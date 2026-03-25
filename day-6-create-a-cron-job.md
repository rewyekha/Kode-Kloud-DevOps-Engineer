# Day 6: Create a Cron Job

The `Nautilus` system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in `Stratos DC` on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:\
\
a. Install `cronie` package on all `Nautilus` app servers and start `crond` service.\
b. Add a cron `*/5 * * * * echo hello > /tmp/cron_text` for `root` user.

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

add  `*/5 * * * * echo hello > /tmp/cron_text`

![](<.gitbook/assets/image (2) (1) (1) (1).png>) &#x20;

You’re starting from the **jumphost**, and this task must be done on **ALL Nautilus App Servers**:

* **stapp01** (user: tony)
* **stapp02** (user: steve)
* **stapp03** (user: banner)

### ✅ Task Breakdown

#### a) Install `cronie` and start `crond`

#### b) Add cron job for **root user**:

```
*/5 * * * * echo hello > /tmp/cron_text
```

***

### 🔁 You must repeat the steps on ALL app servers

***

## 🔹 App Server 1 (stapp01)

#### 1️⃣ SSH from jumphost

```bash
ssh tony@stapp01
```

Password:

```
Ir0nM@n
```

Confirm:

```bash
hostname
```

***

#### 2️⃣ Install cronie

```bash
sudo yum install -y cronie
```

***

#### 3️⃣ Start and enable crond

```bash
sudo systemctl start crond
sudo systemctl enable crond
```

***

#### 4️⃣ Add cron job for root

```bash
sudo crontab -e
```

Add **exactly this line**:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

Save and exit.

***

#### 5️⃣ Exit server

```bash
exit
```

***

## 🔹 App Server 2 (stapp02)

#### 1️⃣ SSH

```bash
ssh steve@stapp02
```

Password:

```
Am3ric@
```

#### 2️⃣ Install cronie

```bash
sudo yum install -y cronie
```

#### 3️⃣ Start & enable crond

```bash
sudo systemctl start crond
sudo systemctl enable crond
```

#### 4️⃣ Add root cron

```bash
sudo crontab -e
```

Add:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

Save and exit.

#### 5️⃣ Exit

```bash
exit
```

***

## 🔹 App Server 3 (stapp03)

#### 1️⃣ SSH

```bash
ssh banner@stapp03
```

Password:

```
BigGr33n
```

#### 2️⃣ Install cronie

```bash
sudo yum install -y cronie
```

#### 3️⃣ Start & enable crond

```bash
sudo systemctl start crond
sudo systemctl enable crond
```

#### 4️⃣ Add root cron

```bash
sudo crontab -e
```

Add:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

Save and exit.

***

### ✅ What the Evaluator Checks

On **stapp01, stapp02, stapp03**:

✔ `cronie` package installed\
✔ `crond` service running\
✔ Root cron exists with correct schedule and command

***

### 🧠 Common Mistakes to Avoid

* ❌ Doing this on jumphost
* ❌ Adding cron as normal user instead of root
* ❌ Forgetting one app server
* ❌ Typo in cron timing

###
