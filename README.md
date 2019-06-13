# Linux Server Configuration

This is the fifth and final project for Udacity's Full Stack Web Developer Nanodegree.

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.
- The Linux distribution is Ubuntu 16.04 LTS.
- The virtual private server is Amazon Lightsail.
- The web application is the [Item Catalog project](https://github.com/gowriprathap/ItemCatalog) which I created earlier in the Full Stack Developer Nanodegree (Project four).
- The database server is PostgreSQL.

You can visit http://54.254.213.105/ or http://ec2-54-254-213-105.ap-southeast-1.compute.amazonaws.com/ to access this website.

## Get a server

### Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Create an account if you do not already have one. Log in to the site, and then click `Create instance`.
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Choose an instance plan (preferably the cheapest)
- You can retain the default name provided by AWS or rename your instance.
- Click the `Create` button to create the instance, and wait for the instance to start up.

### Step 2: SSH into the server

- From the `Account` menu on Amazon Lightsail, click on the `SSH keys` tab and download the Default Private Key into your machine.
- Move this private key file from downloads, into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.
- In your terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
- To connect to the instance via the terminal, type the command `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@54.254.213.105`,
  where `54.254.213.105` is the public IP address of the instance created on lightsail.

## Secure the server

### Step 3: Update and upgrade installed packages

Type the following commands to update and upgrade the installed packages.

```
sudo apt-get update
sudo apt-get upgrade
```

### Step 4: Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file using the command: `sudo nano /etc/ssh/sshd_config`.
- Change the port number from `22` to `2200`.
- Save the file using CTRL+X and confirm with Y, then press enter.
- Restart SSH using the command: `sudo service ssh restart`.

### Step 5: Configure the Uncomplicated Firewall (UFW)

- We have to configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) and deny port 22 (which is the default)
  ```
  sudo ufw status                  # The UFW will show 'inactive'
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in (Port 80)
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny port 22.
  ```

- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)?
  ```
  Enter y and you will see the following:
  ```
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should look like so:

  ```
  Status: active

  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- Exit the SSH connection using the command `exit`.

- Go to the Amazon Lightsail Instance, and click on the `Manage` option. Then click on the `Networking` tab, and change the firewall configuration to match the settings above.

- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

- From your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@54.254.213.105`, where `54.254.213.105` is the public IP address of the instance, and lightsail_key contains the private key to log in.

An output like this would be displayed on the terminal:
`Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-1084-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.

New release '18.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


*** System restart required ***
Last login: Thu Jun 13 12:50:48 2019 from 137.97.165.251
ubuntu@ip-172-26-2-165:~$
`

## Give `grader` access

### Step 6: Create a new user account named `grader`

- To create a new user grader, add user: `sudo adduser grader` (while logged in as `ubuntu`)
- Enter a password (twice) and fill out information for this new user (or leave blank)


### Step 7: Give `grader` the permission to sudo

- Type in the following command: `sudo visudo`.
- Search for this line:
  ```
  root    ALL=(ALL:ALL) ALL
  ```

- Below the above line, add a new line to give the sudo privileges to `grader` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Save the file using CTRL+X, confirm with Y and click enter.
- After saving and exiting, verify that `grader` has sudo permissions. Run `su - grader`, enter the password entered earlier, run `sudo -l` and enter the password again. The output should be similar to this:

  ```
  Matching Defaults entries for grader on ip-172-26-2-165.ap-southeast-1.compute.internal:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User grader may run the following commands on ip-172-26-2-165.ap-southeast-1.compute.internal:
      (ALL : ALL) ALL
  ```

### Step 8: Create an SSH key pair for `grader` using the `ssh-keygen` tool

- On the local machine:
  - Run the command `ssh-keygen`
  - Enter file in which to save the key (I named it `grader_key`) in the local directory `~/.ssh`
  - Enter in a passphrase twice. Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
  - Run the command `cat ~/.ssh/grader_key.pub` and copy the contents of this file.
  - Log in to the grader's virtual machine
- On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (using the command `mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content (which we copied earlier) into this file, save and exit.
  - Give the permissions using the following commands: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Check in `/etc/ssh/sshd_config` file using `sudo nano /etc/ssh/sshd_config` to see if `PasswordAuthentication` is set to `no`
  - Restart SSH using the command: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@54.254.213.105` to log into the grader's virtual machine.


## Deploying the project

### Step 9: Configure the local timezone

- While logged in as `grader`, configure the time zone: `sudo dpkg-reconfigure tzdata`. Configure it to your timezone.

### Step 10: Install and configure Apache to serve a Python mod_wsgi application

- While logged in as `grader`, install Apache using the command: `sudo apt-get install apache2`.
- Enter public IP of the Amazon Lightsail instance (in this case, 54.254.213.105) into browser. If Apache is working, you should see a welcome page.

- My project is built with Python 2.7. So, I ran this:
 `sudo apt-get install libapache2-mod-wsgi python-dev`.
- Enable `mod_wsgi` using: `sudo a2enmod wsgi`.
- Start the web server with `sudo service apache2 start`


### Step 11: Install and configure PostgreSQL

- While logged in as `grader`, install PostgreSQL using the command:
 `sudo apt-get install postgresql`.
- PostgreSQL should not allow remote connections. In the  `/etc/postgresql/9.5/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```

- Switch to the `postgres` user using the command: `sudo su - postgres`. The new user will be postgres.
- Open PostgreSQL interactive terminal with the command `psql`.
- Create the `catalog` user with a password and give them the ability to create databases using the command:

  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- List the existing roles using the command: `\du`. The output should be like this:
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

- Exit psql using the command: `\q`.
- Switch back to the `grader` user: `exit`.
- Create a new Linux user called `catalog`: `sudo adduser catalog`. Enter password and fill out information.
- Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
- Search for the lines that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `catalog` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  catalog  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y and click enter.
- Verify that `catalog` has sudo permissions. Run `su - catalog`, enter the password, run `sudo -l` and enter the password again. The output should be like this:

  ```
  Matching Defaults entries for catalog on ip-172-26-2-165.ap-southeast-1.compute.internal:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User catalog may run the following commands on ip-172-26-2-165.ap-southeast-1.compute.internal:
      (ALL : ALL) ALL
  ```

- While logged in as `catalog`, create a database: `createdb catalog`.
- Run `psql` and then run `\l` to see that the new database has been created. The output should be like this:
  ```
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
  -----------+----------+----------+-------------+-------------+-----------------------
   catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
  (4 rows)
  ```
- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.


### Step 12: Install git

- While logged in as `grader`, install `git`: `sudo apt-get install git`.

## Deploy the Item Catalog project

### Step 13.1: Clone and setup the Item Catalog project from the GitHub repository

- While logged in as `grader`, create `/var/www/catalog/` directory.
- Change to that directory and clone the catalog project:<br>
`sudo git clone https://github.com/gowriprathap/ItemCatalog catalog`.
- From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` using: `sudo chown -R grader:grader catalog/`.
- Change to the `/var/www/catalog/catalog` directory.
- Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`.

- In `__init__.py`, replace line 541:
  ```
  # app.run(host="0.0.0.0", port=5000)
  app.run()
  ```

- In `database_setup.py`, `__init__.py` and `lotsofitems.py`, replace:
   ```
   # engine = create_engine("sqlite:///itemcatalog.db")
   engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
   ```


### Step 14.1: Install the virtual environment and dependencies

- While logged in as `grader`, install pip: `sudo apt-get install python-pip`.
- Install the virtual environment: `sudo pip install virtualenv`
- Change to the `/var/www/catalog/catalog/` directory.
- Create the virtual environment: `sudo virtualenv venv`.
- Activate the new environment: `source venv/bin/activate`.
- Change permission: `sudo chmod -R 777 venv`
- Install the following dependencies:
  ```
  sudo pip install httplib2
  sudo pip install requests
  sudo pip install --upgrade oauth2client
  sudo pip install sqlalchemy
  sudo pip install flask
  sudo apt-get install libpq-dev
  sudo pip install psycopg2
  ```

- Run `python __init__.py` and you should see:
  ```
  * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
  ```

### Step 14.2: Set up and enable a virtual host

- Create `/etc/apache2/sites-available/catalog.conf` and add the
following lines to configure the virtual host:

  ```
  <VirtualHost *:80>
	  ServerName 54.254.213.105
    ServerAlias ec2-54-254-213-105.ap-southeast-1.compute.amazonaws.com
	  WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	  <Directory /var/www/catalog/catalog/>
	  	Order allow,deny
		  Allow from all
	  </Directory>
	  Alias /static /var/www/catalog/catalog/static
	  <Directory /var/www/catalog/catalog/static/>
		  Order allow,deny
		  Allow from all
	  </Directory>
	  ErrorLog ${APACHE_LOG_DIR}/error.log
	  LogLevel warn
	  CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- Enable virtual host: `sudo a2ensite catalog`. The following prompt will be returned:
  ```
  Enabling site catalog.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Reload Apache: `sudo service apache2 reload`.



### Step 14.3: Set up the Flask application

- Create `/var/www/catalog/catalog.wsgi` file add the following lines:

  ```
  activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
  with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/catalog/")
  sys.path.insert(1, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = "..."
  ```

- Restart Apache: `sudo service apache2 restart`.


### Step 14.6: Launch the Web Application

- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.
- Open your browser to http://54.254.213.105 or http://ec2-54-254-213-105.ap-southeast-1.compute.amazonaws.com/.


## Useful commands

 - To get log messages from Apache server: `sudo tail /var/log/apache2/error.log`.
 - To restart Apache: `sudo service apache2 restart`.
