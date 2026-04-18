# Day 72:

A new DevOps Engineer has joined the team and he will be assigned some Jenkins related tasks. Before that, the team wanted to test a simple parameterized job to understand basic functionality of parameterized builds. He is given a simple parameterized job to build in Jenkins. Please find more details below:

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.\
\
1\. Create a `parameterized` job which should be named as `parameterized-job`\
\
2\. Add a `string` parameter named `Stage`; its default value should be `Build`.\
\
3\. Add a `choice` parameter named `env`; its choices should be `Development`, `Staging` and `Production`.\
\
4\. Configure job to execute a shell command, which should echo both parameter values (you are passing in the job).\
\
5\. Build the Jenkins job at least once with choice parameter value `Development` to make sure it passes.\
\
`Note:`\
\
1\. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.\
\
2\. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.





````markdown
# Jenkins: Create a Parameterized Job

**Platform:** KodeKloud | **Topic:** Jenkins, Parameterized Builds, Freestyle Project  
**Difficulty:** Beginner | **Lab Type:** Configuration  

## Table of Contents

1. [Lab Overview](#lab-overview)
2. [Lab Objectives](#lab-objectives)
3. [Prerequisites](#prerequisites)
4. [Solution Steps](#solution-steps)
5. [Build Execution & Verification](#build-execution--verification)
6. [Lab Complete](#lab-complete)
7. [Key Learnings](#key-learnings)
8. [Common Mistakes & Tips](#common-mistakes--tips)

## Lab Overview

A new DevOps engineer joined the Nautilus team. Before assigning real tasks, the team wanted to test his understanding of **Parameterized Builds** in Jenkins. The task was to create a simple parameterized Jenkins job that accepts two parameters and prints their values during the build.

## Lab Objectives

1. Create a Freestyle job named **`parameterized-job`**.
2. Add a **String Parameter** named `Stage` with default value `Build`.
3. Add a **Choice Parameter** named `env` with choices: `Development`, `Staging`, and `Production`.
4. Configure the job to execute a shell command that echoes both parameter values.
5. Build the job at least once with `env = Development` and ensure it completes successfully.

## Prerequisites

- Jenkins UI accessible via the **Jenkins** button on the top bar of KodeKloud lab.
- Login credentials:
  - **Username**: `admin`
  - **Password**: `Adm!n321`

## Solution Steps

### Step 1: Create the Job

1. Click **New Item** on the Jenkins dashboard.
2. Enter job name: `parameterized-job`
3. Select **Freestyle project**
4. Click **OK**

### Step 2: Configure Parameters

1. Check the box **This project is parameterized**
2. Add **String Parameter**:
   - Name: `Stage`
   - Default Value: `Build`
3. Add **Choice Parameter**:
   - Name: `env`
   - Choices (one per line):
     ```
     Development
     Staging
     Production
     ```

### Step 3: Add Build Step

1. Scroll to **Build** section.
2. Click **Add build step** → **Execute shell**
3. Enter the following command:

```bash
echo "Stage: $Stage"
echo "Environment: $env"
````

4. Click **Save**

#### Step 4: Build the Job with Parameters

1. On the job page, click **Build with Parameters** (left sidebar).
2. Keep `Stage` as default (`Build`)
3. Select `env` = **`Development`**
4. Click **Build**

### Build Execution & Verification

**Console Output of Build #1:**

```bash
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/parameterized-job
[parameterized-job] $ /bin/sh -xe /tmp/jenkins1024274920151805213.sh
+ echo Stage: Build
Stage: Build
+ echo Environment: Development
Environment: Development
Finished: SUCCESS
```

The build completed successfully and correctly displayed both parameter values.

### Lab Complete

**Requirement Status**

| Requirement                                               | Status  |
| --------------------------------------------------------- | ------- |
| Job created with name `parameterized-job`                 | ✅ Done  |
| String Parameter `Stage` (default: Build)                 | ✅ Done  |
| Choice Parameter `env` (Development, Staging, Production) | ✅ Done  |
| Shell script echoes both parameters                       | ✅ Done  |
| Built once with `env = Development`                       | ✅ Done  |
| Build status                                              | SUCCESS |

### Key Learnings

* **Parameterized Builds** allow passing values at build time instead of hardcoding them.
* **String Parameter** is used for free-text input with an optional default value.
* **Choice Parameter** provides a dropdown list to restrict user input.
* Environment variables in shell scripts are accessed using `$PARAMETER_NAME`.
* "Build with Parameters" is used instead of normal "Build" when parameters are configured.
* Jenkins automatically exposes parameters as environment variables during the build.

### Common Mistakes & Tips

* Forgetting to check **"This project is parameterized"** box.
* Using wrong parameter names (`stage` instead of `Stage`, `ENV` instead of `env`).
* Not selecting **Freestyle project**.
* Typing choices in a single line instead of one per line in Choice Parameter.
* Using normal **Build** button instead of **Build with Parameters**.
* Jenkins UI getting stuck after restart — always **refresh the page** multiple times.

**Best Practice**: Always take screenshots of:

* Job configuration (parameters section)
* Build with Parameters screen
* Successful Console Output

***

**Lab Completed on:** April 10, 2026\
**Platform:** KodeKloud Jenkins Lab\
**Status:** Completed Successfully



<figure><img src=".gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>
