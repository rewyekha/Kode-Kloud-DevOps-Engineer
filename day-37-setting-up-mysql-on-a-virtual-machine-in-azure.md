# Day 37: Setting Up MySQL on a Virtual Machine in Azure

The Nautilus DevOps team is tasked with integrating a PHP application hosted on an Azure VM with a MySQL database hosted on another Azure VM. This will validate the application's ability to connect to the database in the cloud.

1. **Create the MySQL VM**:
   * Create a VM named `devops-mysql-vm` using the **MySQL Jetware** image from the Azure Marketplace.
   * Configure the VM in the **Central US** region.
   * Use `Password` as the authentication type.
   * Set the **username** as `devops_admin` and the **password** as `Namin@123456`.
   * Allow inbound traffic on port `3306` to enable MySQL access.
2. **Setup the MySQL Database**:
   * SSH into the `devops-mysql-vm`.
   * Use the `sudo /jet/enter mysql` command to access the MySQL shell.
   * Create a database named `devops_db`.
   * Create a MySQL user named `devops_user` with password `password123`.
   * Grant all privileges on the `devops_db` database to this user.
3. **PHP VM Setup**:
   * A VM named `devops-php-vm` already exists in the **East US** region.
   * This VM is hosting a PHP application and contains a pre-existing `db_test.php` file in the `/var/www/html/` directory.
4. **Database Connection Configuration**:
   * Retrieve the **public IP address** of the `devops-mysql-vm`.
   * Update the database connection settings in the `db_test.php` file to use the MySQL credentials and public IP address of the `devops-mysql-vm`.
5. **Validation**:
   * Access the `db_test.php` file from the `devops-php-vm` using its public IP address.
   * Ensure the file displays the message `Connected successfully`, confirming the connection between the PHP application and the MySQL database.

Use the Azure Portal URL and login credentials below:

\
`Notes:`

* Ensure the MySQL database allows inbound traffic on port `3306`.
* Verify that the PHP application on the `devops-php-vm` successfully connects to the MySQL database on the `devops-mysql-vm`.



***

## 📘 Day 37: Setting Up MySQL on a Virtual Machine in Azure

***

### 🧾 **Task**

The Nautilus DevOps team is tasked with integrating a PHP application hosted on an Azure VM with a MySQL database hosted on another Azure VM.

#### **Objectives**

1. Create a MySQL VM in Azure
2. Configure MySQL database and user
3. Update PHP application to connect to MySQL
4. Validate connection via browser

***

## 🖥️ **GUI Solution (Azure Portal)**

### **Step 1: Create MySQL VM**

1. Go to Azure Portal → **Virtual Machines → Create**
2. Configure:

| Setting        | Value             |
| -------------- | ----------------- |
| VM Name        | `devops-mysql-vm` |
| Region         | `Central US`      |
| Image          | MySQL Jetware     |
| Authentication | Password          |
| Username       | `devops_admin`    |
| Password       | `Namin@123456`    |

3. Create VM

***

### **Step 2: Allow MySQL Port (3306)**

1. Go to:
   * **devops-mysql-vm → Networking**
2. Add inbound rule:

| Field    | Value |
| -------- | ----- |
| Port     | 3306  |
| Protocol | TCP   |
| Action   | Allow |

***

### **Step 3: Configure MySQL**

1. SSH into MySQL VM from portal
2. Enter MySQL shell:

```bash
sudo /jet/enter mysql
```

3. Run:

```sql
CREATE DATABASE devops_db;
CREATE USER 'devops_user'@'%' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON devops_db.* TO 'devops_user'@'%';
FLUSH PRIVILEGES;
```

***

### **Step 4: Update PHP VM**

1. Go to **devops-php-vm**
2. Connect via SSH
3. Edit file:

```bash
sudo nano /var/www/html/db_test.php
```

4. Update:

```php
<?php
$servername = "<MYSQL_PUBLIC_IP>";
$username = "devops_user";
$password = "password123";
$dbname = "devops_db";

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>
```

***

### **Step 5: Allow HTTP (Port 80)**

1. Go to:
   * **devops-php-vm → Networking**
2. Add rule:

| Field    | Value |
| -------- | ----- |
| Port     | 80    |
| Protocol | TCP   |
| Action   | Allow |

***

### **Step 6: Validate**

Open browser:

```
http://<PHP_VM_PUBLIC_IP>/db_test.php
```

✅ Expected:

```
Connected successfully
```

***

## 💻 **CLI Solution**

### **Step 1: SSH into MySQL VM**

```bash
ssh devops_admin@<mysql-vm-public-ip>
```

***

### **Step 2: Configure Database**

```bash
sudo /jet/enter mysql
```

```sql
CREATE DATABASE devops_db;
CREATE USER 'devops_user'@'%' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON devops_db.* TO 'devops_user'@'%';
FLUSH PRIVILEGES;
EXIT;
```

***

### **Step 3: Verify MySQL Port**

```bash
sudo netstat -tulnp | grep 3306
```

Expected:

```
0.0.0.0:3306
```

***

### **Step 4: SSH into PHP VM**

```bash
ssh azureuser@<php-vm-public-ip>
```

***

### **Step 5: Update PHP File**

```bash
sudo nano /var/www/html/db_test.php
```

***

### **Step 6: Test Connectivity**

```bash
nc -zv <mysql-vm-public-ip> 3306
```

Expected:

```
succeeded
```

***

### **Step 7: Test Locally**

```bash
curl http://localhost/db_test.php
```

Expected:

```
Connected successfully
```

***

### **Step 8: Final Validation**

```bash
http://<php-vm-public-ip>/db_test.php
```

***

## 🧠 **Key Learnings**

* NSG rules control VM access (ports like 3306, 80)
* MySQL must allow remote connections (`'%'`)
* PHP connects using MySQLi
* Always test:
  * Network (`nc`)
  * App (`curl`)
  * Browser (final validation)

***

## ✅ **Conclusion**

Successfully connected a PHP application running on one Azure VM to a MySQL database hosted on another VM and validated the connection via browser.

***







<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>
