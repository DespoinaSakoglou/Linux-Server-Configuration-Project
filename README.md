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
6. Create a new database user named `catalog` and a new PostgreSql database
   - Create a new user: run `sudo adduser catalog`, enter password, and fill out name `catalog` (rest of information is optional)
   - Give sudo permissions:
     - Run `sudo visudo`
     - Add `catalog ALL=(ALL:ALL) ALL` below lines `root ALL=(ALL:ALL) ALL` and `grader ALL=(ALL:ALL) ALL` and save and close the file.
     - Run `sudo su - catalog` to swich to catalog user.
     - Confirm `catalog` user has sudo permissions by running `sudo -l`. The following should appear:
       ```
          Matching Defaults entries for catalog on ip-172-26-14-7.ec2.internal:
          env_reset, mail_badpass, 
          secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

          User catalog may run the following commands on ip-172-26-14-7.ec2.internal:
          (ALL : ALL) ALL
       ```
   - Run `createdb catalog` to create a new database
   - Run `psql` and `\l` to confirm the database was created.
   - Run `exit` to switch back to `ubuntu` user.
7. Install git
   - Run `sudo apt-get install git`
   
#### Step 6: Deploy the Item Catalog Project
1. Clone the item catalog project and configure the database
   - Create a new itemCatalog directory in /var/www/ by running `cd /var/www/` and then `sudo mkdir itemCatalog`. 
   - Change to the itemCatalog directory (run `cd itemCatalog`) and clone the item catalog project:
     `sudo git clone https://github.com/DespoinaSakoglou/Item-Catalog-Project.git itemCatalog`
   - Change back to /var/www/ (run `cd ..`) and run `sudo chown -R ubuntu:ubuntu itemCatalog/` to change the ownership of the itemCatalog directory to `ubuntu`.
   - Change to the  /var/www/itemCatalog/itemCatalog directory
   - Run `mv application.py __init__.py` to rename the application.py file
   - Run `nano __init__.py` to go into the `__init__.py` file.
   - In `__init__.py`, find line 385: `app.run(host='0.0.0.0', port=8000)` and change it to: `app.run()`. Save and close the file.
   - Run `nano database_setup.py` to go into the database_setup file.
   - Find line 77: `engine = create_engine('sqlite:///HikingCatalog.db')` and change it to:
     `engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')`
     This way you can switch the database in the database_setup.py file from SQLite to PostgreSQL. (make sure to use the password you set for the PostgreSql catalog user, not the database catalog user - in case two different passwords were set)
2. Add client_secrets.json to authenticate login with Google
   - Create a new project on the [Google APIs Console](https://console.developers.google.com/apis)
   - Choose **Credentials** on the left and create an **OAuth Client ID**. Configure the **consent screen** with an application name and support email, save and select **Web application** from the list of application types.
   - Add http://XX.XX.XX.XX and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com as authorized JavaScript origins
   - Google will provide a client ID and client secret for the project, accecible via a downloadable JSON file. Download the JSON file, and copy the contents in order to update the client_secrets.json file
   - Run `nano client_secrets.json` in the /var/www/itemCatalog/itemCatalog/ directory to open the client_secrets.json file
   - Delete the existing contents and paste the new contents that you copied from the downloaded JSON file. Save and close client_secrets.json file.
   - Still in the /var/www/itemCatalog/itemCatalog project directory, run `cd templates` and then `nano login.html` to open the login.html file. Find line 21: `data-clientid="DISPLAYED-CURRENT-CLIENT-ID"` and replace the existing client id with the new one. Save and close the file.
   - Back to the /var/www/itemCatalog/itemCatalog project directory, run `nano __init__.py` and change the path for the client_secrets.json file in lines 40 and 83 from `'client_secrets.json'` to `'/var/www/itemCatalog/itemCatalog/client_secrets.json'`.
3. Set up the virtual environment:
   - Go back to the home directory (run `cd ~/`)
   - Install pip: run `sudo apt-get install python-pip`
   - Install virtualenv: run `sudo apt-get install python-virtualenv`
   - Change to the /var/www/itemCatalog/itemCatalog/ directory and create a temporary environment (choose a name for the environment; 'venv' is used in this configuration) by running `virtualenv venv` (_do not use `sudo` here_)
   - Activate the new environment: run: `. venv/bin/activate`
   - With the virtual environment active, install the following dependencies:
     - `pip install httplib2`
     - `pip install requests`
     - `pip install --upgrade oauth2client`
     - `pip install sqlalchemy`
     - `pip install flask`
     - `sudo apt-get install libpq-dev`
     - `pip install psycopg2`
     Note that sudo was used only when installing libpq-dev, because we want libpq-dev to be installed globally. All the other dependencies should be installed only in the virtual environment, not globally.
   - Confirm that everything was installed correctly by running `python __init__.py`. You should see the application running: 
     `* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`
   - Deactivate the virtual environment: run `deactivate`
4. Set up a virtual host:
   - Go back to the home directory (run `cd ~/`)
   - Create a file in /etc/apache2/sites-available/ called itemCatalog.conf:
     - Run `cd /etc/apache2/sites-available/`
     - Run `sudo touch itemCatalog.conf`
   - Open the file by running `sudo nano itemCatalog.conf` and add the following:
     ```
        <VirtualHost *:80>
		                 ServerName XX.XX.XX.XX
		                 ServerAdmin d.sakoglou@gmail.com
		                 WSGIScriptAlias / /var/www/itemCatalog/itemCatalog.wsgi
		                 <Directory /var/www/itemCatalog/itemCatalog/>
			                      Order allow,deny
			                      Allow from all
			                      Options -Indexes
		                 </Directory>
		                 Alias /static /var/www/itemCatalog/itemCatalog/static
		                 <Directory /var/www/itemCatalog/itemCatalog/static/>
			                      Order allow,deny
			                      Allow from all
			                      Options -Indexes
		                 </Directory>
		                 ErrorLog ${APACHE_LOG_DIR}/error.log
		                 LogLevel warn
		                 CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
     ```
     Save and close the file.
   - Enable the virtual host: run `sudo a2ensite itemCatalog`. You should see the following:
     ```
        Enabling site itemCatalog.	
        To activate the new configuration, you need to run:
        service apache2 reload
     ```
   - Run `sudo service apache2 reload`
5. Write a .wsgi file (Apache serves Flask applications by using a .wsgi file)
   - Apache serves Flask applications by using a .wsgi file; create a file called nuevoMexico.wsgi in /var/www/nuevoMexico
   - Go back to the home directory (run `cd ~/`)
   - Create a file in /var/www/itemCatalog called itemCatalog.wsgi:
     - Run `cd /var/www/itemCatalog/`
     - Run `sudo touch itemCatalog.wsgi`
   - Open the file by running `sudo nano itemCatalog.wsgi` and add the following:
     ```
        activate_this = '/var/www/itemCatalog/itemCatalog/venv/bin/activate_this.py'
        execfile(activate_this, dict(__file__=activate_this))

        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/itemCatalog/")

        from itemCatalog import app as application
        application.secret_key = '12345'
     ```
   - Resart Apache: run `sudo service apache2 restart`
6. Switch the database in the application from SQLite to PostgreSQL
   - Change to the /var/www/itemCatalog/itemCatalog directory.
   - Run `nano __init__.py` to go into the `__init__.py` file.
   - In `__init__.py`, find line 46: `engine = create_engine('sqlite:///HikingCatalog.db')` and change it to: 
     `engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')`. Save and close the file.
   - Run `nano hikingitems.py` to go into the hikingitems.py file.
   - Find line 10: `engine = create_engine('sqlite:///HikingCatalog.db')` and change it to:
     `engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')`. Save and close the file.
     Note: make sure to use the password you set for the PostgreSql catalog user, not the database catalog user.
7. Disable the default Apache site
   - Run: `sudo a2dissite 000-default.conf` . You should see the following:
     ```
        Site 000-default disabled.
        To activate the new configuration, you need to run:
           service apache2 reload
     ```
   - Run `sudo service apache2 reload`
8. 
    
    

