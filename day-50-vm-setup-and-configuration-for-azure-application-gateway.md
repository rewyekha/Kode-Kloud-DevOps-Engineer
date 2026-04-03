# Day 50: VM Setup and Configuration for Azure Application Gateway

The Nautilus DevOps team needs to set up an Azure Application Gateway to manage traffic for a backend pool of virtual machines. The gateway will serve as a load balancer, distributing traffic across the VMs.

#### Task: <a href="#task" id="task"></a>

1\) **Azure Virtual Network and Subnet**:

* Create a Virtual Network (VNet) named `datacenter-vnet` in the **East US** region.
* Create a Subnet named `datacenter-subnet` within the VNet for the VMs.
* Create a Subnet named `datacenter-apgw-subnet` within the VNet for the Application Gateway.

2\) **Azure Virtual Machines**:

* Create two VMs named `datacenter-vm1` and `datacenter-vm2` in the **East US** region.
* Install Nginx on both VMs.
* Configure `index.html` on VM1 to display "Welcome to KKE Labs:Version 1".
* Configure `index.html` on VM2 to display "Welcome to KKE Labs:Version 2".

3\) **Azure Application Gateway**:

* Create an Application Gateway named `datacenter-apgw` in the **East US** region.
* Assign the `datacenter-apgw-subnet` to the Application Gateway.
* Create a frontend IP configuration named `datacenter-apgw-ip`.
* Add the VMs `datacenter-vm1` and `datacenter-vm2` to the backend pool.
* Configure a basic routing rule to distribute traffic between the VMs.

4\) **Validation**:

* Verify that the Application Gateway distributes traffic to both VMs.
* Ensure that accessing the Application Gateway URL displays either "Welcome to KKE Labs:Version 1" or "Welcome to KKE Labs:Version 2" depending on the load balancing.\
  `Notes:`
* Create all resources in the `East US` region.
* Use the Azure Portal or Azure CLI for resource creation.
* Ensure proper routing and traffic distribution through the Application Gateway.



***

### Overview

This guide details the process of setting up an Azure Application Gateway to manage traffic for a backend pool consisting of two Virtual Machines (VMs). The Application Gateway will serve as a load balancer distributing traffic across the VMs running Nginx web servers with distinct content.

***

### Prerequisites

* Azure subscription with appropriate permissions.
* Azure CLI installed and configured.
* Basic knowledge of Azure networking and virtual machines.

***

### 1. Create Azure Virtual Network and Subnets

1. **Create a Virtual Network (VNet):**
   * Name: `nautilus-vnet`
   * Region: East US
2. **Create Subnets within the VNet:**
   * `nautilus-subnet` for the backend VMs.
   * `nautilus-apgw-subnet` for the Application Gateway.

***

### 2. Create Azure Virtual Machines

1.  Create two VMs:

    * `nautilus-vm1`
    * `nautilus-vm2`

    Both should be in the East US region and deployed in the `nautilus-subnet`.
2.  Install Nginx on both VMs:

    ```bash
    sudo apt update
    sudo apt install nginx -y
    ```
3. Configure `index.html` on each VM:
   *   On `nautilus-vm1`:

       ```bash
       echo "Welcome to KKE Labs:Version 1" | sudo tee /var/www/html/index.html
       sudo systemctl restart nginx
       ```
   *   On `nautilus-vm2`:

       ```bash
       echo "Welcome to KKE Labs:Version 2" | sudo tee /var/www/html/index.html
       sudo systemctl restart nginx
       ```

***

### 3. Create Azure Application Gateway

1. Create an Application Gateway:
   * Name: `nautilus-apgw`
   * Region: East US
   * SKU: Basic (to comply with policy)
   * Assign the subnet `nautilus-apgw-subnet`.
2. Create a frontend IP configuration:
   * Name: `nautilus-apgw-ip`
   * Assignment: Static Public IPv4
3. Create a backend pool and add the VMs:
   * Name: `nautilus-backend`
   * Add `nautilus-vm1` and `nautilus-vm2` to the backend pool by their private IPs.
4. Configure a basic routing rule to distribute HTTP traffic between backend VMs.

***

### 4. Validation and Verification

#### Verify Application Gateway Deployment

Use Azure CLI:

```bash
az network application-gateway show \
  --name nautilus-apgw \
  --resource-group <resource-group-name> \
  --query "{Name:name, State:operationalState, SKU:sku.name}" \
  -o table
```

Expected output should show the Application Gateway in a `Running` state with SKU `Basic`.

#### Check Backend Pool

```bash
az network application-gateway address-pool list \
  --gateway-name nautilus-apgw \
  --resource-group <resource-group-name> \
  -o table
```

Ensure backend pool provisioning state is `Succeeded`.

#### Obtain Application Gateway Public IP

```bash
az network public-ip list \
  --resource-group <resource-group-name> \
  --query "[?contains(name,'apgw')].ipAddress" \
  -o tsv
```

#### Test Load Balancing

Run the following to test traffic distribution:

```bash
IP=<Application-Gateway-Public-IP>

for i in {1..8}; do
  curl -s http://$IP
done
```

You should see alternating responses:

* "Welcome to KKE Labs:Version 1"
* "Welcome to KKE Labs:Version 2"

If the responses are all identical, verify that the `index.html` files on each VM contain the correct version text.

***

### Troubleshooting Tips

* If the Application Gateway deployment fails due to SKU policy, ensure the SKU is set to `Basic` as some subscriptions enforce this.
* If the load balancing is not reflected in responses, verify backend VM configurations and ensure the Application Gateway backend pool includes both VMs.
* Confirm that the VMs are reachable and Nginx is running.

***

### Summary

This guide demonstrated how to:

* Set up an Azure Virtual Network with appropriate subnets.
* Deploy two backend VMs running Nginx with unique content.
* Configure an Azure Application Gateway to load balance HTTP traffic across the VMs.
* Validate the setup and verify traffic distribution.

***

If further enhancements are desired, consider adding health probes and SSL termination to the Application Gateway configuration.

***



1\) **Azure Virtual Network and Subnet**:

<figure><img src=".gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

2\) **Azure Virtual Machines**:

<figure><img src=".gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

VM1:

<figure><img src=".gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

VM2:

<figure><img src=".gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>



3\) **Azure Application Gateway**:

<figure><img src=".gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>



Final :thumbsup:<br>

<figure><img src=".gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>
