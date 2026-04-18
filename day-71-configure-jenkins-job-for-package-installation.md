# Day 71: Configure Jenkins Job for Package Installation

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

Some new requirements have come up to install and configure some packages on the Nautilus infrastructure under Stratos Datacenter. The Nautilus DevOps team installed and configured a new Jenkins server so they wanted to create a Jenkins job to automate this task. Find below more details and complete the task accordingly:<br>

1\. Access the Jenkins UI by clicking on the `Jenkins` button in the top bar. Log in using the credentials: username `admin` and password `Adm!n321`.\
2\. Create a new Jenkins job named `install-packages` and configure it with the following specifications:

* Add a string parameter named `PACKAGE`.
* Configure the job to install a package specified in the `$PACKAGE` parameter on the `storage server` (Stratos Datacenter).
* Build the job at least once (e.g. with parameter `PACKAGE=vim-enhanced`) so the package is installed on the Storage server and can be verified.

`Note`:

1\. Ensure to install any required plugins and restart the Jenkins service if necessary. Opt for `Restart Jenkins when installation is complete and no jobs are running` on the plugin installation/update page. Refresh the UI page if needed after restarting the service.

2\. Verify that the Jenkins job runs successfully on repeated executions to ensure reliability.

3\. Capture screenshots of your configuration for documentation and review purposes. Alternatively, use screen recording software like `loom.com` for comprehensive documentation and sharing.


***

## Lab: Automating Package Installation on a Storage Server Using Jenkins

### Objective

The Nautilus DevOps team required an automated solution to install software packages on the storage server in the `Stratos Datacenter`. To address this requirement, a Jenkins server was provisioned.

The goal of this lab is to create a **parameterized Jenkins Freestyle job** named **`install-packages`** that installs a user‑specified package on the storage server **`ststor01`**.

***

### Infrastructure Details

* **Jenkins Server**: `jenkins` (accessible via the lab interface)
* **Storage Server**: `ststor01.stratos.xfusioncorp.com`
  * **Username**: `natasha`
  * **Password**: `Bl@kW`
* **Purpose**: Automate RPM package installation using `yum` via Jenkins

***

### Prerequisites

* Access to the Nautilus lab environment
* Jenkins UI access with administrative privileges

***

### Step‑by‑Step Implementation

#### 1. Access Jenkins

1. Click the **Jenkins** button in the top navigation bar of the lab interface.
2. Log in using the following credentials:
   * **Username**: `admin`
   * **Password**: `Adm!n321`

***

#### 2. Install the SSH Plugin

1. Navigate to **Manage Jenkins** → **Manage Plugins**.
2. Open the **Available** tab and search for **SSH**.
3. Select **SSH Plugin – Version 158.ve2a\_e90fb\_7319**\
   _(Provides the “Execute shell script on remote host using SSH” functionality.)_
4. Click **Install without restart** or **Download now and install after restart**.
5. On the installation page, select **Restart Jenkins when installation is complete and no jobs are running**.
6. Wait for Jenkins to restart and refresh the browser if required.

***

#### 3. Add Credentials for the Storage Server

1. Navigate to **Manage Jenkins** → **Manage Credentials**.
2. Under **Stores scoped to Jenkins**, select **(global)**.
3. Click **Add Credentials** and configure the following:
   * **Kind**: Username with password
   * **Scope**: Global
   * **Username**: `natasha`
   * **Password**: `Bl@kW`
   * **ID**: `storage`
   * **Description**: Storage server credentials
4. Click **OK** to save the credentials.

***

#### 4. Configure the SSH Remote Host

1. Go to **Manage Jenkins** → **Configure System**.
2. Scroll down to the **SSH Remote Hosts** section.
3. Click **Add** and enter:
   * **Hostname**: `ststor01.stratos.xfusioncorp.com`
   * **Port**: `22`
   * **Credentials**: Select the `storage` credential
4. Click **Check Connection** and verify that it returns **Success**.
5. Click **Save**.

***

#### 5. Create the Jenkins Job

1. From the Jenkins dashboard, click **New Item**.
2. Enter the job name: **`install-packages`**
3. Select **Freestyle project**, then click **OK**.

***

**Job Configuration**

**General Settings**

* Enable **This project is parameterized**
* Add a **String Parameter**:
  * **Name**: `PACKAGE`
  * **Default Value**: (optional, e.g., `vim-enhanced`)
  * **Description**: Name of the package to install on the storage server

**Build Configuration**

1. Click **Add build step** → **Execute shell script on remote host using SSH**
2. Select the **SSH site**:\
   `natasha@ststor01.stratos.xfusioncorp.com:22`
3. Enter the following command:

    echo 'Bl@kW' | sudo -S yum install -y $PACKAGE
4. Click **Apply**, then **Save**.

***

#### 6. Execute and Verify the Job

1. Open the **install-packages** job.
2. Click **Build with Parameters**.
3. Provide a package name (for example, `vim-enhanced` or `net-tools`).
4. Click **Build**.
5. Verify that the build completes successfully (blue indicator).
6. Review the **Console Output** to confirm the package installation.
7. Run the job multiple times with different packages to ensure reliability.

***

#### 7. Optional Verification on the Storage Server

You may manually verify the installation by logging into the storage server:

ssh [natasha@ststor01.stratos.xfusioncorp.com](mailto:natasha@ststor01.stratos.xfusioncorp.com)sudo yum list installed | grep \<package-name>

***

### Screenshots / Documentation (Recommended)

For lab submission or review, capture the following:

* SSH plugin installation screen
* SSH Remote Hosts configuration
* Jenkins job configuration (parameterized build and SSH build step)
* Successful build console output

Alternatively, a short screen recording (e.g., using Loom) may be submitted.

***

### Conclusion

The **`install-packages`** Jenkins job successfully automates RPM package installation on the Nautilus storage server. The solution is reusable, reliable, and easily extensible through parameterization.

This implementation leverages the Jenkins SSH plugin to securely execute remote commands and supports repeated executions without manual intervention.


<figure><img src=".gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>


```bash
Started by user admin

Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/install-packages
[SSH] script:
PACKAGE="vim-enhanced"

echo 'Bl@kW' | sudo -S yum install -y $PACKAGE

[SSH] executing...
Last metadata expiration check: 0:26:51 ago on Thu Apr  9 03:49:18 2026.
Dependencies resolved.
================================================================================
 Package             Arch        Version                   Repository      Size
================================================================================
Installing:
 vim-enhanced        x86_64      2:8.2.2637-27.el9         appstream      1.7 M
Installing dependencies:
 gpm-libs            x86_64      1.20.7-29.el9             appstream       21 k
 vim-common          x86_64      2:8.2.2637-27.el9         appstream      7.0 M
 vim-filesystem      noarch      2:8.2.2637-27.el9         baseos          13 k

Transaction Summary
================================================================================
Install  4 Packages

Total download size: 8.8 M
Installed size: 34 M
Downloading Packages:
(1/4): gpm-libs-1.20.7-29.el9.x86_64.rpm        219 kB/s |  21 kB     00:00    
(2/4): vim-filesystem-8.2.2637-27.el9.noarch.rp  57 kB/s |  13 kB     00:00    
(3/4): vim-enhanced-8.2.2637-27.el9.x86_64.rpm  9.3 MB/s | 1.7 MB     00:00    
(4/4): vim-common-8.2.2637-27.el9.x86_64.rpm     16 MB/s | 7.0 MB     00:00    
--------------------------------------------------------------------------------
Total                                            13 MB/s | 8.8 MB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : gpm-libs-1.20.7-29.el9.x86_64                          1/4 
  Installing       : vim-filesystem-2:8.2.2637-27.el9.noarch                2/4 
  Installing       : vim-common-2:8.2.2637-27.el9.x86_64                    3/4 
  Installing       : vim-enhanced-2:8.2.2637-27.el9.x86_64                  4/4 

 
  Verifying        : vim-filesystem-2:8.2.2637-27.el9.noarch                1/4 
  Verifying        : gpm-libs-1.20.7-29.el9.x86_64                          2/4 
  Verifying        : vim-common-2:8.2.2637-27.el9.x86_64                    3/4 
  Verifying        : vim-enhanced-2:8.2.2637-27.el9.x86_64                  4/4 

Installed:
  gpm-libs-1.20.7-29.el9.x86_64         vim-common-2:8.2.2637-27.el9.x86_64    
  vim-enhanced-2:8.2.2637-27.el9.x86_64 vim-filesystem-2:8.2.2637-27.el9.noarch

Complete!

[SSH] completed
[SSH] exit-status: 0

Finished: SUCCESS

```

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
