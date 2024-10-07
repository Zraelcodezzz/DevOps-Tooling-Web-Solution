# DevOps-Tooling-Web-Solution


In this project, we'll be implementing a solution that's going to consist of the following tools under the following terms.

1. Infrastrucure: AWS
2. Webserver Linux : RedHat Enterprise Linux 8
3. Database server: Ubuntu 24.04 +MYSQL
4. Storage server:  RedHat Enterprise Linux 8 + NFS Server
5. Programming Language: PHP
6. Code Repository: Github

The diagram below shows how several serverless servers  share a common database  and access the same files using the network files system (NFS) as a shared file storage.

![image](https://github.com/user-attachments/assets/a1d2899a-a52b-4627-ade8-e2cc122514f8)


# STEP 1: PREPARE A NFS SERVER

i. Spin up a new ec2 instance with RHEL Linux operating system 

ii. Next we configure LVM on the server

iii. We'll be formatting to `xfs` and not `ext4`

iv.   Next we'll create three logical volumes namely

`lv-ops`

`lv-apps`

`lv-logs`


Create mount points on /mnt directoy for the logical volumes as volumes:

Mount-lv-apps on /mnt/apps

Mount-lv-logs on /mnnt/logs

Now install NFS server and configure it to start on reboot










   
