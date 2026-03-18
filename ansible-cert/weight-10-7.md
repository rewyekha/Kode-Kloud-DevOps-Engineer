# Weight: 10

The Nautilus Application development team wanted to test some applications on app servers in `Stratos Datacenter`. They shared some pre-requisites with the DevOps team, and packages need to be installed on app servers. Since they already created some playbooks, now they wanted to make some changes in inventories.

There is an inventory file `/home/thor/playbook/inventory-t3q3` on `jump host`. It has some aliases named `web1`, `web2` and `web3` for three hosts respectively. Update this inventory file to add an alias called `db1` for `server4.company.com` host.



```bash
thor@jump-host ~/playbook$ cat /home/thor/playbook/inventory-t3q3
[applications]
web1 ansible_host=stapp01 ansible_user=tony
web2 ansible_host=stapp02 ansible_user=steve
web3 ansible_host=stapp03 ansible_user=banner

thor@jump-host ~/playbook$ nano /home/thor/playbook/inventory-t3q3
# Edited inventory to add db1 alias
[applications]
web1 ansible_host=stapp01 ansible_user=tony
web2 ansible_host=stapp02 ansible_user=steve
web3 ansible_host=stapp03 ansible_user=banner
db1 ansible_host=server4.company.com ansible_user=peter

thor@jump-host ~/playbook$ cat /home/thor/playbook/inventory-t3q3
[applications]
web1 ansible_host=stapp01 ansible_user=tony
web2 ansible_host=stapp02 ansible_user=steve
web3 ansible_host=stapp03 ansible_user=banner
db1 ansible_host=server4.company.com ansible_user=peter
thor@jump-host ~/playbook$ 
```

