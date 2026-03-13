# Day 22: Configuring Instances with User Data

As a member of the Nautilus DevOps Team, your task is to create a VM with the following specifications:

**Instance Name**: The VM must be named `xfusion-vm`.

**Image**: Use any available Ubuntu image to create this VM.

**Custom Script Extension/User Data**: Configure the VM to run a custom script during its launch. This script should:

* Install the `Nginx` package.
* Start the `Nginx` service. \*\* Network Security Group (NSG)\*\*: Ensure that the VM allows HTTP traffic on port 80 from the internet.

### Setup Azure Virtual machine with user script

1.  Create VM name `xfusion-vm` in `EAST US` region.

    ![image](https://github.com/user-attachments/assets/52a20daf-66da-4e20-963f-808f566a1400)
2.  Select `Ubuntu` image for the virtual machine.

    ![image](https://github.com/user-attachments/assets/7bfd4fcd-272a-4f89-a039-ea45de2a835b)
3.  Create `SSH` public key and choose `RSA format`.

    ![image](https://github.com/user-attachments/assets/e626ecb2-82d5-422f-8c2d-bf7c8a074900)
4.  For `size`, select `Standard_B1s`.

    ![image](https://github.com/user-attachments/assets/4c6e1936-131b-4400-871e-9d198c925990)
5.  On `Disk` tab, select `Standard HDD`

    ![image](https://github.com/user-attachments/assets/5d60885f-75c7-46e0-9af2-d22a4d0a7f4f)
6.  Now, go to `Advance` tab and click on `Enable user data` under `User data` section.

    ![image](https://github.com/user-attachments/assets/5f906262-ada5-454d-9a33-df996546ef59)

    **Enter the following script for `nginx` installation under `User data`**.

    ```bash
    #!/bin/bash
    apt update -y
    apt install -y nginx
    systemctl start nginx
    systemctl enable nginx
    ```
7. Create the VM by clicking `Create` on `Review + create` tab.
8.  After creating VM, go to resource and choose `Network settings` from navigation pane.

    ![image](https://github.com/user-attachments/assets/81d5d71d-9c2d-4e99-a0c9-1d21fa6c395b)
9.  From `Network security group`, click on `Create port rule` and choose `Inbound port rule`.

    ![image](https://github.com/user-attachments/assets/cbeff78c-6c7b-41be-922a-36232d6fb1e4)
10. Add rule for `HTTP` port.

    ![image](https://github.com/user-attachments/assets/9f1cf713-0d20-4149-809d-582903b3ac4a)
11. Enter port rule name and description(optional) and click on `Add`.

    ![image](https://github.com/user-attachments/assets/2135f806-5937-4ea9-86b1-0ec58a408788)
12. Verify new rule added to the `Inbound port rule` dashboard.

    ![image](https://github.com/user-attachments/assets/6879bf7e-9deb-4be6-9848-c02006b0c5bf)
13. Now copy the VM public IP from `Overview` section and enter it on a browser to see `Nginx` welcome page.

    ![image](https://github.com/user-attachments/assets/06dbb8f8-867e-43d5-91c2-0d9e0a9b6f4f)
14. Also, you can ssh into the Azure VM using ssh key you generated and check Nginx status.

    ```bash
    sudo systemctl status nginx
    ```

    ![image](https://github.com/user-attachments/assets/070970d3-4e5e-4b37-a8d2-2f2edc9fb7f8)

    ```bash
    curl http://<xfusion-vm-ip>
    ```

    ![image](https://github.com/user-attachments/assets/c705c70d-76bb-4ac1-8777-57aa8a5feb31)
