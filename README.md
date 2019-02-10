# Linux-Server-Configuration-Project
This project describes a Linux server configuration process as part of the Udacity Full Stack Nanodegree Program.

### Description
This is a baseline installation of a Linux server configured as a web and database server to host a web application. The instructions below explain how to configure the server to host the [Item Catalog app](https://github.com/DespoinaSakoglou/Item-Catalog-Project) on an Amazon Lightsail instance.

### Server Specific Details
- IP address: 

- SSH port: 2200

- URL to the hosted website: 

### Software installed during configuration

### Configuration process

#### Step 1: Create an Amazon Lightsail instance
1. Create an Amazon Web Services account and sign in to [Amazon Lightsail](https://lightsail.aws.amazon.com/).
2. Create an instance (a Linux server running on a virtual machine inside an Amazon datacenter) by following the **_Create an instance_** link.
3. Choose the **_OS Only_** instance and the **_Ubuntu 16.04 LTS_** option.
4. Choose a payment plan. The instance plan controls how powerful of a server you get. For this project, the lowest tier of instance is just fine. 
5. Give the instance a unique hostname and click **_Create_**.
6. Wait for the instance to start up. It may take a few minutes until the instance is running.

#### Step 2: Connect using SSH
1. Download the instance's private key by navigating to the Amazon Lightsail **_Account page_**.
2. Click on **_Download default key_**.
3. With a text editor, open the downloaded file (called LightsailDefaultPrivateKey.pem or LightsailDefaultPrivateKey-YOUR-AWS-REGION.pem).
4. Click on the  AWS instance, go to the **_Connect_** tab and click on **_Connect using SSH_**
5. Once you connect to the instance from the browser, navigate to the local directory `~/.ssh/`, create a file called lightsail_key.rsa and copy the text from the downloaded .pem file into the .rsa file in the .ssh directory.
6. Run `chmod 600 ~/.ssh/lightsail_key.rsa`.
7. Log in by running `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance.
8. When you SSH in, you'll be logged as the ubuntu user. When you want to execute commands as root, you'll need to use the sudo command to do it.

#### Step 3: Secure the server
1. Update all currently installed packages by running `sudo apt-get update` and then `sudo apt-get upgrade`.
2. Configure the firewall:
   - Change the SSH port from 22 to 2200: `sudo nano /etc/ssh/sshd_config` and change the port number on line 5 from 22 to 2200.
   - Restart SSH by running `sudo service ssh restart`.
   - Run `sudo ufw status` to check if the ufw (uncomplicated firewall) is active.
   - Run `sudo ufw default deny incoming` to set the ufw to block everything coming in.
   - Run `sudo ufw default allow outgoing` to set the ufw to allow everything outgoing.
   - Run `sudo ufw allow ssh` to set the ufw to allow SSH.
   - Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port 2200 so that SSH will work.
   - Run `sudo ufw allow www` to set the ufw to allow a basic HTTP server.
   - Run `sudo ufw allow 123/udp` to set the ufw to allow NTP.
   - Run `sudo ufw deny 22` to deny port 22. We run this command because port 22 is not being used for anything anymore. It is the default port for SSH, but this virtual machine has now been configured so that SSH uses port 2200.
   - Run `sudo ufw enable` to enable the ufw firewall.
   - Run `sudo ufw status` to check which ports are open and to see if the ufw is active.
   - Back to the AWS website, update the external firewall by clicking on the **_Networking_** tab, and then changing the firewall configuration to match the internal firewall settings just set on the browser terminal (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed, default port 22 should be denied).
3. Now that the SSH port is changed, the Lightsail instance will no longer be accessible through the web app **_Connect using SSH_** button, because the button assumes the default port 22 is being used.
4. To login (on Windows), save the downloaded .pem file to a directory (e.g. Desktop) as **_lightsail_key.pem_**, open the command line, and run `ssh -i ~/Desktop/lightrail_key.pem -p 2200 ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance.

#### Step 4: Give `grader` access
1. Create a new user account named `grader` by running `sudo adduser grader`, and providing a UNIX password (twice) and the rest information for the `grader` user.
2. Give `grader` sudo permissions:
   - Run `sudo visudo`.
   - Find line `root ALL=(ALL:ALL) ALL` and in a new line below it add `grader	ALL=(ALL:ALL) ALL`.
   - Save and close the visudo file
   - You can verify that `grader` has sudo permissions by running `su - grader`, entering the password, then running `sudo -l` and entering the password again. If grader has sudo permissions, the following will appear: 
   
   ```
      Matching Defaults entries for grader on ip-XX-XX-XX-XX.ec2.internal: 
      env_reset, mail_badpass, 
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
      
      User grader may run the following commands on ip-XX-XX-XX-XX.ec2.internal:
      (ALL : ALL) ALL
   ```
3. Create an SSH key pair for `grader` to log in to the virtual machine:
   - Run `ssh-keygen` on the local machine, not on the server. (Always generate ke pairs locally, to ensure the private key is private.
   - Choose a file name for the key pair, such as _grader-key_: `DEFAULT-DIRECTORY-KEY-EXISTS-IN/grader-key`
   - Enter in a passphrase twice. `ssh-keygen` will generate two files, one ending in .pub
4. Place the public key on the remote server so ssh can use it to log in
   - Log in to the server, switch to the `grader`'s home directory, and create a new directory .ssh by running `mkdir .ssh`.
   - Run `touch .ssh/authorized_keys`.
   - Back on the local machine, run `cat ~/.ssh/insert-name-of-file.pub` (if using a Windows command prompt, run: `cd .ssh` and `type insert-name-of-file.pub`, since `cat` is not a recognized command).
   - Copy the contents of the .pub file, go back to the server as the grader user and run `nano .ssh/authorized_keys` to paste the copied content in the file on the virtual machine.
   - Run `chmod 700 .ssh` on the virtual machine as grader user.
   - Run `chmod 644 .ssh/authorized_keys` on the virtual machine as grader user.
5. Force key based authorization by disabling password based log in
   - On the virtual machine as grader user, run `sudo nano /etc/ssh/sshd_config`.
   - Find the line with the comment that says: _# Change to no to disable tunnelled clear text passwords_. If the next line is set to 'yes': _PasswordAuthentication yes_, then set 'yes' to 'no' and save and exit the file.
   - Run `sudo service ssh restart`.
   - Log in as a grader user using `ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`.
   
#### Step 5: Prepare to deploy the project
1. Configure local timezone to UTC
   - Run `sudo dpkg-reconfigure tzdata` and select `None of the above` and `UTC` when prompted.
   - Confirm that timezone is correctly set to UTC by running `date`.
2. Install Apache HTTP Server
   - Run `sudo apt-get install apache2` to install Apache.
   - Confirm that Apache was installed successfully by using the public IP of the Amazon Lightsail instance as as a URL in a browser. If Apache is working, you should see the Apache2 Ubuntu Default Page.
3. Install mod_wsgi
   - Run `sudo apt-get install libapache2-mod-wsgi python-dev` to install mod-wsgi package (that allows Apache to serve Flask applications) and python-dev package (required for python extensions).
   - If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server: `sudo apt-get install libapache2-mod-wsgi-py3`.
   - Confirm that mod-wsgi is enabled by running `sudo a2enmod wsgi`.
4. Install PostgreSQL
   - Run `sudo apt-get install postgresql` to install PostgreSQL that will allow persistent data storage using a database server.
   - Confirm that remote connections are not allowed by checking that the `/etc/postgresql/9.5/main/pg_hba.conf` file has the following lines:
   ```
      local   all             postgres                                peer
      local   all             all                                     peer
      host    all             all             127.0.0.1/32            md5
      host    all             all             ::1/128                 md5
   ```
5. Create a new PostgreSql user named `catalog` with limited permissions to the catalog application database
   - During instalation, PostgreSQL created a Linux user with the name `postgres`. Run `sudo su - postgres` to switch to this user.
   - Run `psql` to connect to the terminal to interact with PostgreSQL.
   - Run `CREATE ROLE catalog WITH LOGIN;` to create the `catalog` user.
   - Run `ALTER ROLE catalog CREATEDB;` to give the `catalog` user the ability to create databases.
   - Run `\password catalog` to give the user a password.
   - Confirm the user was created by running `\du;`. The table returned should look like the following:
     ```
                                          List of roles
       Role name |                         Attributes                         | Member of 
      -----------+------------------------------------------------------------+-----------
       catalog   | Create DB                                                  | {}
       postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
     ```
   - Run `\q` to exit psql
   - Run `exit` to switch back to the `ubuntu` user
6. Create a new database user named `catalog` and a new PostgreSql datapase
    
    

