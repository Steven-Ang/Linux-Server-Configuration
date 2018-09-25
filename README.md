# Linux Server Configuration

Installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

# Information of the Server

| Name                   | Value                                                            |
| ---------------------- |:----------------------------------------------------------------:|
| Local IP Address       | ~~13.229.122.182~~                                                   |
| SSH Port               | ~~2200~~                                                             |
| User                   | ~~grader~~                                                           |

# Prerequisites

If you don't have the following skills, you may have trouble following the guide.

* **Comfortable using Unix based command line user interface**
* **Good understanding on how database works**
* **Intermediate Python**

# Modules and Software Installed during the Configuration

* **apache2**
* **mod_wsgi**
* **git**
* **Python**
* **pip**
* **Flask**
* **sqlalchemy**
* **psycopg2**
* **httplib2**
* **requests**
* **outh2client**
* **PostgreSQL**

# Step by Step Guide on Configuring the Web Server

### 1. Get Started on Lightsail

We will use [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) to host the web server. It is easy to use, we could easily set up an instance with just few clicks.

##### 1. Create an account

If you already have an amazon account, you can skip this part and log in. Otherwise, create one.

![](https://i.gyazo.com/0bfd6fada54ec98c69a0049181aeb244.png)

##### 2. Create an Instance

Once you're logged in, Lightsail will give a friendly message. Since we haven't any instances, let's make one! Here the "**Create an Instance!**" text and to the next step.

![](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e45a0_screen-shot-2017-02-10-at-14.58.17/screen-shot-2017-02-10-at-14.58.17.png)

##### 3. Pick an Instance Location

Not very important, but you can select your desire location to host your server. Since I live in Asia, the instance automatically select Singapore. You can change the region by clicking "**Change Region and zone**".

![](https://i.gyazo.com/2150a41d8b21268f8961d8b89aa3d0f6.png)

##### 4. Pick an Instance Image

By default, **Linux/Unix** is selected so don't change that.

![](https://i.gyazo.com/951856c233067fed999e2ffc2077302e.png)

##### 5. Pick a Blueprint for the Instance

We will use **Ubuntu** for our server so click that option.

![](https://i.gyazo.com/b8cbc554187379ace76117dd045449e6.png)

##### 6. Pick an Instance Plan

You can select all of the plans that are been shown to you. I recommend choosing the $5 month plan so I will choose that.

![](https://i.gyazo.com/6d56cb609fc4d1b1c8e85df63f0e2ffe.png)

##### 7. Name Your Instance

Choose a name for your instance, make sure it's unique! Click the create button, wait for it to start up. That's it, you have an instance up and running. Pretty simple, right?

![](https://i.gyazo.com/ceccbd55bb6342c5d57abde7efd020a8.png)

### 2. Accessing the Instance Through SSH

Before setting up the server, we need to download the private key. You can find the private key by going to the "**Account**" page and click "**SSH Keys**" section.

![](https://i.gyazo.com/35f86e791dcb3bc2b2e46e5e0ede8f0a.png)

Create a directory to store the private key, usually you will have a directory named **.ssh** where you stored all of your private keys. I will mine there, but you can put it anywhere you think it's safe.

Open your terminal, for Windows user I recommend using GitBash, and type this `chmod 400 YOUR_DIRECTORY/YOUR_KEY_NAME.pem`. This is to prevent anyone but the owner to read and write the file.

Now, type this to access to the instance `ssh -i YOUR_DIRECTORY/YOUR_KEY_NAME.pem ubuntu@IP-ADDRESS`. By default, the ssh port is set to the port 22. Later on we will change that.

If it's a success, you will get this message to notify you that you have ssh into the instance. Welcome to your new server, now let's get working.

![](https://i.gyazo.com/dca3407d6217fd542c99d18dc724a81c.png)

### 3. Create a New User

* We will be working on a different user entirely instead of the root/ubuntu user. We will disable the ability to remotely log in as the root user later on, but now let's create a new user named `grader`.

* Type the following:
`sudo adduser grader`

* Now give this user administration permission by editing the Sudoers file. We can access the file by typing this command:
`sudo visudo`

* Find this line. It's at the bottom of the file:
```
# User privilege specification
root    ALL=(ALL:ALL) ALL
```

* Below `root    ALL=(ALL:ALL) ALL` add `grader ALL=(ALL:ALL) ALL`. Now close the file by pressing command/control X, Y, and finally enter/return.

* Let's switch to the `grader` now. `sudo su - grader`

### 4. Generate SSH Keys For Grader

* Open a new tab on your terminal, make sure you're on your local machine instead of the server. We will use `ssh-keygen` to generate our key. By default, the key will be stored in the directory called **.ssh** I recommend it to not change the directory. Once it's generated, you're prompted to enter a passphrase. Enter your passphrase and then it's done.

* We will copy the content from the key and paste inside a file called **authorized_keys** inside **.ssh** directory in the `grader` home directory. We will create the file in the next step, now let's get the content. `cat YOUR_DIRECTORY/YOUR_KEY.pub`

* Now back to the server as `grader`, at your home directory, make sure you are there by typing this `cd` without argument.

* Create a new directory named **.ssh** - `sudo mkdir .ssh` and create a file named **authorized_keys** - `sudo touch .ssh/authorized_keys`. Open it with nano or vim, and paste the key inside the file - `sudo nano .ssh/authorized_keys`.

* Set file permissions to **.ssh** and **authorized_key**.
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```

* Connect to the server as `grader` - `ssh -i YOUR_DIRECTORY/YOUR_KEY grader@YOUR_IP_ADDRESS`

![](https://i.gyazo.com/1ad1cbc85abebc49d0e089f05e7e2af9.png)

### 5. Enforce Key-Based Authentication and Disable the Ability to Login as the Root User

Password authentication is okay, but key-base authentication is much more secure so we will be using it from now on.

* Edit this file by using either nano or vim - `sudo nano /etc/ssh/sshd_config`
* Find the line where it says `PasswordAuthentication yes` change it to `PasswordAuthentication no`
* Find the line where it says `PermitRootLogin yes` change it to `PermitRootLogin no`
* Exit the file and restart the service to enforce it - `sudo service ssh restart`

### 6. Update all of the Packages

For security purpose, it is important to keep our packages up-to-date. Type the following commands.

`sudo apt-get update` - updates the package lists for upgrades for packages that need upgrading
`sudo apt-get upgrade` - installs newer versions of the packages

### 7. Change the Time-Zone to UTC

You have two options on changing the time-zone:

By using this command `sudo timedatectl set-timezone UTC` or `sudo dpkg-reconfigure tzdata` then select "None of the above" and select "UTC".
I recommend using the first option because it's quicker.

### 8. Change the SSH Port to 2200

* Edit the file `/etc/ssh/sshd_config` by using either nano or vim. Find the line with the content of "Port 22", change the 22 to 2200.
* Restart the ssh service - `sudo service ssh restart`
* To confirm that it works type this `ssh -i YOUR_DIRECTORY/YOUR_KEY grader@YOUR_ID -p 2200`

### 9. Configure the Firewall

For security purpose, the server will only allow three connections - HTTP/80, SSH/2200, and NTP/123.

1. Block all the incoming connection - `sudo ufw default deny incoming`
2. Allow outgoing connection - `sudo ufw default allow outgoing`
3. Allow incoming connection for SSH on port 2200 - `sudo ufw allow 2200/tcp`
4. Allow incoming connection for HTTP on port 80 - `sudo ufw allow wwww`
5. Allow incoming connection for NTP on port 123 - `sudo ufw allow ntp`
6. Enable the firewall - `sudo ufw enable`

### 10. Install Apache2, mod_wsgi, and git

The Web Server Gateway Interface (WSGI) is a specification for simple and universal interface between web servers and web applications or frameworks for the Python programming language. Mod_wsgi is an Apache HTTP Server module that provides a WSGI compliant interface for hosting Python based web applications under Apache. It enables Apache to serve Flask applications.

* Install Apache2 `sudo apt-get install apache2`. You can put your ip address in the address bar and you will get this message "It works".
* Install Python mod_wsgi - `sudo apt-get install libapache2-mod-wsgi python-dev`
* Install git - `sudo apt-get install git`
* Configure your username and email
```
git config --global user.name "Your Name"
git config --global user.email "Your email"
```
* Start the web server - `sudo service apache2 start`

### 11. Install PostgreSQL

* Install postgresql `sudo apt-get install postgresql postgresql-contrib`
* Double check if no remote connections alowed - `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
* Upon the installation, postgresql will create a new user named postgresql. Let's switch to that user and configuring the database - `sudo -u postgres psql`
* Type `\q` or `exit` to exit the shell.

### 12. Configuring the Database

**Note: Remember to put a semi-colon at the end to end the statement.**

* Create a new user named **catalog** - `CREATE USER catalog WITH PASSWORD 'Your Password';`
* Give the user the ability to create a databse - `ALTER USER catalog CREATEDB;`
* Create a new database named **catalog** and give the ownership to the **catalog** user - `CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to the database by typing this - `\c catalog`
* Only give permission to create tables to the **catalog** user
```
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO public;
```
* Exit the shell and back as `grader` by typing - `exit`

### 13. Install All of the Python Packages

`sudo apt-get install sqlalchemy python-requests python-oauth2client python-psycopg2 python-flask python-pip python-httplib2`

### 14. Clone the Item-Catalog Project

* Create a new directory named `catalog` inside `/var/www` - `sudo mkdir /var/www/catalog`
* Inside this directory, change the owner to grader - `sudo chown -R grader:grader catalog`
* Change your current directory to `catalog` - `cd /var/www/catalog`
* Clone the project - `git clone https://github.com/Steven-Ang/Item-Catalog`
* Rename Item-Catalog to catalog - `mv Item-catalog/ catalog/`

### 15. Setting Up the Web Servers

* Change to the catalog directory and edit the file: application.py, db_data.py, and db_setup.py. Find the line like this `engine = create_engine('sqlite:///music.db')` change it to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`. We no longer use SQLite as our database anymore so I must change it.
* Inside application.py, make sure CLIENT_ID's path is an absolute path.
* Rename `application.py` to `__init__.py` - `mv application.py __init__.py`
* Setup the data for the application:
```
python db_setup.py
python db_data.py
```
* We will create a WSGI for the project, it is required. Back one directory - `cd ..`, we are back at catalog again (This is getting confusing, I know). Now create the file - `sudo nano catalog.wsgi` and paste this inside it:

```
#!/usr/bin/python

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```

### 16. Create a Configuration File

* Create the configuration file - `sudo nano /etc/apache2/sites-available/catalog.conf`
* Paste this in the file:
```
<VirtualHost *:80>
      ServerName 34.207.86.0
      ServerAlias ec2-13-229-98-127.ap-southeast-1a.compute.amazonaws.com
      ServerAdmin admin@34.207.86.0
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
* Enable it - `sudo a2ensite catalog`
* Restart it - `sudo service apache2 restart`

### 17. Visit the Server

Go to your instance's ip address to see your newly create server.

# Third-Party Resources

Here are the list of resources helped me completing the guide and the project.

* [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
* [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Deploy a Flask Application on Ubuntu 14.04](https://devops.profitbricks.com/tutorials/deploy-a-flask-application-on-ubuntu-1404/)
* [How To Install and Use PostgreSQL on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
* [How To Use Roles and Manage Grant Permissions in PostgreSQL on a VPS](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)
* [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [Changing the SSH Port for Your Linux Server](https://my.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306)
* [Setting Time-Zone from Terminal ](https://askubuntu.com/questions/3375/how-to-change-time-zone-settings-from-the-command-line)
* [How to disable SSH logins for the root account](https://www.a2hosting.com/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root)
* [How To Edit the Sudoers File on Ubuntu and CentOS](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos)
* [How To Set Up Apache Virtual Hosts on Ubuntu 14.04 LTS](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
