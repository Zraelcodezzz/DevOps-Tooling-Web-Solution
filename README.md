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

- Spin up a new ec2 instance with RHEL Linux operating system 

![image](https://github.com/user-attachments/assets/302d2b80-5d3e-4437-bff4-4e022cc47516)


- Create and attach new EBS volume to your instance

![image](https://github.com/user-attachments/assets/3df59b8d-ff14-4a69-81e8-9e46fa912eec)

![image](https://github.com/user-attachments/assets/456f5647-654e-497d-86bd-64c9f7faf19e)

![image](https://github.com/user-attachments/assets/5388c281-785e-4568-b91b-4c2cc7fa31f5)

- COnfigure LVMs in the server

![image](https://github.com/user-attachments/assets/24d15598-782f-4391-b20d-95c55704acb0)

- Create physical volumes PVs

![image](https://github.com/user-attachments/assets/59e763eb-930d-4948-9d70-676282a68198)





-  Next we'll create three logical volumes namely

`lv-ops`

`lv-apps`

`lv-logs`

![image](https://github.com/user-attachments/assets/9b4cc05c-99f1-4ee7-9a6e-6f50d963f0a1)

![image](https://github.com/user-attachments/assets/e2b5e4d8-d3e8-4d7f-beb2-bd2959bae496)




- We'll be formatting to `xfs` and not `ext4`

![image](https://github.com/user-attachments/assets/f3252308-3446-455d-a5e3-353f2b6b4869)


Create mount points on /mnt directoy for the logical volumes as volumes:

Mount-lv-apps on /mnt/apps

Mount-lv-logs on /mnnt/logs


                   sudo mkdir -p /mnt/apps /mnt/logs /mnt/opt
                   sudo mount /dev/webdata-vg/lv-apps /mnt/apps
                    sudo mount /dev/webdata-vg/lv-logs /mnt/logs
                    sudo mount /dev/webdata-vg/lv-opt /mnt/opt


![image](https://github.com/user-attachments/assets/508c8018-7f4c-4ace-a4e9-e6bae3fbfe46)


Now install NFS server and configure it to start on reboot



                sudo yum -y update
                sudo yum install nfs-utils -y
                 sudo systemctl start nfs-server.service
                 sudo systemctl enable nfs-server.service
                 sudo systemctl status nfs-server.service


![image](https://github.com/user-attachments/assets/70697b9f-a79f-4b0e-8112-74931497ddea)

# CONFIGURE NFS EXPORTS
  Export the mounts for web servers' subnet CIDR to connect as clients.
  
  # Set permissions
  
          sudo chown -R nobody: /mnt/apps /mnt/logs /mnt/opt
         sudo chmod -R 777 /mnt/apps /mnt/logs /mnt/opt
         
# Configure access

         sudo vi /etc/exports

   ![image](https://github.com/user-attachments/assets/5935f364-14f1-469a-be99-5556802b7d76)

   save and exit the file

              sudo exportfs -arv

 ### Open ports for NFS

 Open TCP 111, UDP 111, and UDP 2049 in the Security Groups for NFS Server. (Note you'll only allow access from the Web Server)

 ![image](https://github.com/user-attachments/assets/1c6233cd-3ba3-4c92-a4ad-721746913ba6)



 # STEP 2: CONFIGURE THE DATABASE SERVER


Launch a new EC2 instance for our database server

Update system

![image](https://github.com/user-attachments/assets/d16f94be-a734-4ff0-962f-3140c8816bc6)

  Install MySQL server

     sudo apt install mysql-server -y

  ![image](https://github.com/user-attachments/assets/020cf55b-6a70-4e5a-a6f9-5bdb45188a8f)


  Configure MySQL database

-  Create new database and user

     CREATE DATABASE tooling;
     CREATE USER 'webaccess'@'%' IDENTIFIED BY 'mypass';
     GRANT ALL PRIVILEGES ON tooling. * TO 'webaccess'@'%' WITH GRANT OPTION;
     FLUSH PRIVILEGES;


 -  Update the bind-address in the /etc/mysql/mysql.conf.d/mysqld.cnf file to allow remote connections:
   
        bind-address = 0.0.0.0


-    Restart server

         sudo systemctl restart mysql
    

   ![image](https://github.com/user-attachments/assets/186f1ac0-0834-4db7-bad8-ea97790138c9)

   ![image](https://github.com/user-attachments/assets/aac693e8-db7c-4208-a109-3be22dbf1b22)

-  Now  Ports for MySQL
  
     Open TCP 3306 in the Edit Inbound rules section of our Security Groups for MySQL Server.

   ![image](https://github.com/user-attachments/assets/7d81fd6c-c615-4ec5-a9ec-45eda62f632a)


   # STEP  3: PREPARE THE WEB SERVERS


- We'll launch a new  RedHat EC2 instance


   ![image](https://github.com/user-attachments/assets/f043ce5f-dd45-41e1-8d10-280e7957a5cf)

-  Next we'll configure NFS client on the webserver

                 sudo yum install nfs-utils nfs4-acl-tools -y
                 sudo mkdir /var/www
                 sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

![image](https://github.com/user-attachments/assets/a933e38e-bf94-4609-ac8e-8067fc727e93)
            
- Next we verify if it was mounted successfully

             df -h

![image](https://github.com/user-attachments/assets/cb22e878-67e0-454d-be30-1a093e4b59db)

persist the mount on reboot

     sudo vi /etc/fstab

![image](https://github.com/user-attachments/assets/17db87d2-77b4-4899-9d90-6257dbf549b3)

Install Apache and php on the web server

![image](https://github.com/user-attachments/assets/b3ec8cae-816a-41f1-a588-6e39b72bf476)


     sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

# Mount Apache logs to NFS server

      sudo mkdir /var/log/httpd
      sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd

  ![image](https://github.com/user-attachments/assets/34f6ba9b-10af-4a8e-ad6b-ba92ddba44f4)

  #   Deploy Tooling Website

- Firstly, Install git 

             sudo yum install git -y

- We'll be forking the tooling solution from steghub
  
- Next we'll be cloning to our web server

  ![image](https://github.com/user-attachments/assets/2d28c212-f8d0-4424-9930-ec1146ccd93c)

 ![image](https://github.com/user-attachments/assets/0dda3944-b1dd-434b-aa4e-4d8e17aee374)

 ![image](https://github.com/user-attachments/assets/db826ddb-f2ea-450b-84c0-29fcf9f58ea7)

  # Update website configuration

 -  We'll be updating the website configuration in the in /var/www/html/funv=ctions.php

        $servername = "<Database-Private-IP>";
  
        $username = "webaccess";
  
        $password = "mypass";
  
        $dbname = "tooling";

  
![image](https://github.com/user-attachments/assets/a335560c-5633-49e2-b88e-64289ef14030)



![image](https://github.com/user-attachments/assets/82a74af1-bdd5-48e6-b021-b5c70b09c0a7)

  
![image](https://github.com/user-attachments/assets/eed42d78-7c85-4760-a8f4-a4297a6cf37c)


![image](https://github.com/user-attachments/assets/948c5c3c-2ead-4007-a086-5fc55de8851f)

![image](https://github.com/user-attachments/assets/0e414a21-02ef-4d2c-9ea6-1c888dd289d4)

- We update the website configuration to connect to the database by adding the tooling.db.sql script to our database using the command

      mysql -h <Database-Private-IP> -u webaccess -p tooling < tooling-db.sql

  ![image](https://github.com/user-attachments/assets/8cd0ddb1-3b7d-4e9f-80d9-44cea93fa3f3)


- Now let's go to our browser and access our site

- http://PublicIPforWebServer/login.php
  
![image](https://github.com/user-attachments/assets/d35fa138-3497-4a1c-a140-e449891a8374)


![image](https://github.com/user-attachments/assets/e5d0608b-ee58-41c9-9356-3f55b7783e8f)

Cheers!
       

             
    

          

   

   



  
   





   





           

       
           










   
