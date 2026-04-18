# Day 68: Set Up Jenkins Server

The DevOps team at xFusionCorp Industries is initiating the setup of CI/CD pipelines and has decided to utilize Jenkins as their server. Execute the task according to the provided requirements:

1\. Install `Jenkins` on the jenkins server using the `apt` utility only, and start it using the `service` command.

* If you face a timeout issue while starting the Jenkins service, first check the service status with `service jenkins status`
* Then review the logs in `/var/log/jenkins/jenkins.log` to identify the cause.

2\. Jenkin's admin user name should be `theadmin`, password should be `Adm!n321`, full name should be `Siva` and email should be `siva@jenkins.stratos.xfusioncorp.com`.

`Note:`

1\. To access the `jenkins` server, connect from the jump host using the `root` user with the password `S3curePass`.

2\. After Jenkins server installation, click the `Jenkins` button on the top bar to access the Jenkins UI and follow on-screen instructions to create an admin user.



### Overview

This document outlines the steps to install and configure Jenkins on the designated server using the `apt` package manager. It also includes the process to create an administrative user through the Jenkins web interface.

***

### Objective

* Install Jenkins using `apt`
* Start and verify Jenkins service
* Troubleshoot startup issues if any
* Configure initial admin user via Jenkins UI

***

### Prerequisites

* Access to the **jump host**
* Root access to the **jenkins server**
* Network connectivity between systems

***

### Step 1: Connect to Jenkins Server

From the jump host, connect to the Jenkins server:

```bash
thor@jump-host ~$ ssh root@jenkins-server
```

Enter the password when prompted:

```bash
S3curePass
```

***

### Step 2: Install Jenkins Using APT

Update package index:

```bash
root@jenkins-server ~# apt update
```

Install Jenkins:

```bash
root@jenkins-server ~# apt install -y jenkins
```

***

### Step 3: Start Jenkins Service

Start the Jenkins service:

```bash
root@jenkins-server ~# service jenkins start
```

***

### Step 4: Verify Service Status

Check if Jenkins is running:

```bash
root@jenkins-server ~# service jenkins status
```

#### Expected Output (Sample)

```bash
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/lib/systemd/system/jenkins.service; enabled)
   Active: active (running)
```

***

### Step 5: Troubleshooting (If Service Fails)

If Jenkins fails to start or times out:

#### Check Service Status

```bash
root@jenkins-server ~# service jenkins status
```

#### Check Logs

```bash
root@jenkins-server ~# cat /var/log/jenkins/jenkins.log
```

Look for errors such as:

* Java not installed
* Port conflicts
* Permission issues

***

### Step 6: Access Jenkins Web UI

* Open the Jenkins interface using the provided **Jenkins button** in the environment.

***

### Step 7: Unlock Jenkins

Retrieve the initial admin password:

```bash
root@jenkins-server ~# cat /var/lib/jenkins/secrets/initialAdminPassword
```

#### Sample Output

```bash
f3a9c8b2e1d74a6c9fexamplepassword
```

Use this password to unlock Jenkins in the browser.

***

### Step 8: Install Suggested Plugins

* Select **Install suggested plugins** when prompted.
* Wait for installation to complete.

***

### Step 9: Create Admin User

Provide the following details:

* **Username**: theadmin
* **Password**: Adm!n321
* **Confirm Password**: Adm!n321
* **Full Name**: Siva
* **Email**: [siva@jenkins.stratos.xfusioncorp.com](mailto:siva@jenkins.stratos.xfusioncorp.com)

***

### Step 10: Finalize Setup

* Complete the setup wizard
* Confirm Jenkins URL (default is acceptable)
* Access Jenkins dashboard

***

### Verification

* Jenkins service is running
* Web UI is accessible
* Admin user is successfully created

***

### Conclusion

Jenkins has been successfully installed using the `apt` package manager, the service is running, and the administrative user has been configured. The system is now ready for CI/CD pipeline setup.

<figure><img src=".gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>
