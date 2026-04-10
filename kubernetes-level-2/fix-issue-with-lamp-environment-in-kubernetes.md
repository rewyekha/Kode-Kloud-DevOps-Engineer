# Fix issue with LAMP Environment in Kubernetes

One of the DevOps team members was trying to install a WordPress website on a LAMP stack, which is deployed on a Kubernetes cluster. It was working well, and we could see the installation page a few hours ago. However, something seems to have gone wrong with the stack after the website went down. Please look into the issue and fix it:

FYI, the deployment name is `lamp-wp` and it is using a service named `lamp-service`. Apache is using the default HTTP port, and the NodePort is `30008`. From the application logs, it has been identified that the application is facing some issues connecting to the database, in addition to other problems. Additionally, there are some environment variables associated with the pods, such as `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, and `MYSQL_HOST`

Also, do not attempt to delete or modify any other existing components, such as deployment names, service names, types, labels, secrets and so on.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



***

## Troubleshooting WordPress LAMP Stack Deployment on Kubernetes

### Document Purpose

This document describes the complete troubleshooting and remediation process performed to restore a WordPress website running on a LAMP stack deployed in a Kubernetes cluster. The website was previously accessible but later became unavailable due to service misconfiguration and application-level database connectivity issues.

The actions performed, commands used, configuration changes made, and validation steps are documented exactly as executed during the incident investigation.

***

### Environment Information

| Component         | Value                                 |
| ----------------- | ------------------------------------- |
| Deployment Name   | `lamp-wp`                             |
| Service Name      | `lamp-service`                        |
| Service Type      | NodePort                              |
| Apache Port       | 80                                    |
| Expected NodePort | 30008                                 |
| Namespace         | default                               |
| Kubernetes Access | Configured via `kubectl` on jump-host |

***

### Initial Issue Description

* WordPress installation page was accessible earlier.
* Website became unreachable after a period of time.
* Application logs indicated database connectivity issues.
* Provided constraints stated that existing components (deployment names, service names, types, labels, secrets) must not be deleted or modified.

***

### Step 1: Inspect Service Configuration

Retrieve the current service configuration:

thor@jump-host \~$ kubectl get svc lamp-service -o yaml > lamp-svc.yml

Contents of the service definition before modification:

spec:  ports:  - nodePort: 30009    port: 80    protocol: TCP    targetPort: 80  selector:    app: lamp    tier: frontend  type: NodePort

#### Observation

The NodePort was set to `30009`, while the required NodePort according to the problem statement was `30008`.

***

### Step 2: Correct NodePort Configuration

Edit the service YAML file:

thor@jump-host \~$ vi lamp-svc.yml

Updated configuration:

spec:  ports:  - nodePort: 30008    port: 80    protocol: TCP    targetPort: 80  selector:    app: lamp    tier: frontend  type: NodePort

Apply the updated service configuration:

thor@jump-host \~$ kubectl apply -f lamp-svc.ymlservice/lamp-service configured

Verify service status:

thor@jump-host \~$ kubectl get svc lamp-service -o wide

Output:

NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTORlamp-service   NodePort   10.43.175.12   \<none>        80:30008/TCP   7m23s   app=lamp,tier=frontend

***

### Step 3: Identify Application Pod

Retrieve the pod name associated with the LAMP stack:

thor@jump-host \~$ POD=$(kubectl get pod -l app=lamp -o jsonpath="{.items\[0].metadata.name}")

***

### Step 4: Access Application Container

Enter the Apache/PHP container:

thor@jump-host \~$ kubectl exec -it $POD -c httpd-php-container -- sh

Navigate to application directory:

/app # cd /app/app # lsindex.php

***

### Step 5: Inspect Application Code

Examine the `index.php` file:

/app # cat index.php

Original content:

\<?php$dbname = $\_ENV\['MYSQL\_DATABASE'];$dbuser = $\_ENV\['MYSQL\_USER'];$dbpass = $\_ENV\[''MYSQL\_PASSWORD""];$dbhost = $\_ENV\['MYSQL-HOST'];\
$connect = mysqli\_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");\
$test\_query = "SHOW TABLES FROM $dbname";$result = mysqli\_query($test\_query);\
if ($result->connect\_error) {   die("Connection failed: " . $conn->connect\_error);}  echo "Connected successfully";

#### Issues Identified

* Invalid environment variable reference: `$_ENV[''MYSQL_PASSWORD""]`
* Incorrect environment variable name: `MYSQL-HOST` instead of `MYSQL_HOST`
* Incorrect `mysqli_query()` usage (missing connection parameter)

***

### Step 6: Verify Environment Variables

Check runtime environment variables within the container:

/app # env | grep -E 'MYSQL\_'

Output:

MYSQL\_ROOT\_PASSWORD=R00tMYSQL\_DATABASE=kodekloud\_db2MYSQL\_USER=kodekloud\_gemMYSQL\_PASSWORD=Rc5C9EyvbUMYSQL\_HOST=127.0.0.1MYSQL\_SERVICE\_SERVICE\_HOST=10.43.127.37MYSQL\_SERVICE\_SERVICE\_PORT=3306MYSQL\_SERVICE\_PORT\_3306\_TCP\_ADDR=10.43.127.37

***

### Step 7: Modify Application Code

Edit the PHP file:

/app # vi index.php

Updated content:

\<?php$dbuser = $\_ENV\['MYSQL\_USER'];$dbpass = $\_ENV\['MYSQL\_PASSWORD'];$dbname = $\_ENV\['MYSQL\_DATABASE'];$dbhost = $\_ENV\['MYSQL\_HOST'];\
$connect = mysqli\_connect($dbhost, $dbuser, $dbpass)  or die("Unable to Connect to '$dbhost'");\
$test\_query = "SHOW TABLES FROM $dbname";$result = mysqli\_query($connect, $test\_query);\
if (!$result) {   die("Connection failed: " . mysqli\_error($connect));}\
echo "Connected successfully";?>

***

### Step 8: Restart PHP-FPM

Restart PHP-FPM service to apply changes:

/app # service php-fpm restartphp-fpm:php-fpmd: stoppedphp-fpm:php-fpmd: started

Exit the container:

/app # exit

***

### Step 9: Validate Application Logs

Retrieve the latest application logs:

thor@jump-host \~$ kubectl logs $POD -c httpd-php-container --tail=20

Sample output:

\[php-fpm:access] 127.0.0.1 - 10/Apr/2026:04:56:58 +0000 "GET /index.php" 200\[httpd:access] 30008-port-san57o4znceews6y.labs.kodekloud.com:80

No database or PHP errors observed.

***

### Step 10: Application Validation

Access the application via browser using NodePort `30008`.

Observed output:

Connected successfully\`\`

***

### Final Outcome

* NodePort mismatch corrected
* PHP syntax and database connectivity issues resolved
* Application restored and accessible via browser
* No Kubernetes components were deleted or renamed

***

### Summary of Root Causes

1. Incorrect NodePort configuration on the Kubernetes service
2. Invalid PHP environment variable references
3. Improper MySQL connection logic in the application code

***

### Conclusion

The outage was caused by both infrastructure-level misconfiguration and application-level errors. Correcting the Kubernetes service configuration restored external access, while addressing PHP and database issues resolved backend connectivity problems. After the fixes, the WordPress LAMP stack resumed normal operation.

***



<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

