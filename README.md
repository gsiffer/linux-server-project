## Linux Server Configuration Project
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.
### Server info
IP address: 54.213.28.90  
SSH port: 2200  
URL: http://54.213.28.90.xip.io/
### Get your server.
Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Flightsail.aws.amazon.com%2Fls%2Fwebapp%3Fstate%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fparksidewebapp&forceMobileApp=0).
* **Log in!** First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
* **Create an instance.** Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
* **Choose an instance image: Ubuntu.** Lightsail supports a lot of different instance types. An instance image is a particular software setup, including an operating system and optionally built-in applications.
For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.
* **Choose your instance plan.** The instance plan controls how powerful of a server you get. It also controls how much money they want to charge you. For this project, the lowest tier of instance is just fine.
* **Give your instance a hostname.** Every instance needs a unique hostname. You can use any name you like, as long as it doesn't have spaces or unusual characters in it. Your instance's name will be visible to you and to the project reviewer.
* **Wait for it to start up.** It may take a few minutes for your instance to start up.
* **It's running; let's use it!** Once your instance has started up, you can log into it with SSH from your browser. The public IP address of the instance is displayed along with its name. For example `54.84.49.254`.

    Note: When you set up OAuth for your application, you will need a DNS name that refers to your instance's IP address. You can use the [xip.io](http://xip.io/) service to get one; this is a public service offered for free by Basecamp. For instance, the DNS name `54.84.49.254.xip.io` refers to the server above.  

### Secure your server.
Update all currently installed packages.  
* The first step to upgrading your installed software is to update your package source list. Run the following commands on your terminal:  
`sudo apt-get update`  
`sudo apt-get upgrade`  

Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
* Run the following command: `sudo nano /etc/ssh/sshd_config`.
* Change the port number from **22** to **2200** in the _Port_ line.
* Run the following command: `sudo service ssh restart`.  

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
* Run the following command:  
```
sudo ufw default deny incoming  
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw deny 22
sudo ufw enable
sudo ufw status
```

* Go to your Amazon Lightsail Instances and allows ports 2200 (TCP), 80 (TCP), and 123 (UDP). Deny the default port 22.

### Give grader access.  
In order for your project to be reviewed, the grader needs to be able to log in to your server.

Create a new user account named `grader`.
* Install _finger_: `apt-get install finger`.
* Create a new user _grader_: `sudo adduser grader`

Give `grader` the permission to `sudo`.
* First copy the _90-cloud-init-users_ file and name it _grader_.  
  * `sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader`
* Make a small edit to this file.
    * `sudo nano /etc/sudoers.d/grader`
    * Change the word ubuntu to grader.
    * **grader ALL=(ALL) NOPASSWD:ALL**

Create an SSH key pair for `grader` using the `ssh-keygen` tool.
* Always generate key pairs locally. We will generate our key pair using an application called: `ssh-keygen`
* You will first be asked to give a file name for the key pair. Name it `grader`. The default directory that key pairs should exist in. Once done you’ll see that `ssh-keygen` has generated two files, _grader_ and _grader.pub_.
* Log into your server as the grader. First create a directory called .ssh using the mkdir command: `mkdir .ssh`
* Then create a new file within this directory called _authorized_keys_.
  * `touch .ssh/authorized_keys`
* Back on your local machine and read out the contents of grader.pub.
    * `cat .ssh/grader.pub`
    * Copy that then back on your server as the grader user to edit this authorized key file.
    * `nano .ssh/authorized_keys`
    * Paste in that content and save it.

Set up some specific file permission on the authorized key file and the SSH directory.   
* As the grader run the following commands:
```
sudo chmod 700 /home/grader/.ssh
sudo chmod 644 /home/grader/.ssh/authorized_keys
sudo chown -R grader:grader /home/grader/.ssh
sudo service ssh restart
```
* Now you can log in as a grader by the following command:
  * `ssh -i ~/.ssh/grader -p 2200 grader@54.213.28.90`

### Prepare to deploy your project.
Configure the local timezone to UTC.
* Execute `sudo dpkg-reconfigure tzdata`, scroll to the bottom of the Continents list and select `Etc` or `None of the above`, in the second list select `UTC`.

Install and configure Apache to serve a Python mod_wsgi application.
* Install Apache using your package manager with the following command:
  * `sudo apt-get install apache2`
  * in your browser you should see the Apache2 Ubuntu Default Page.
* Run the following commands:
```
sudo apt-get install libapache2-mod-wsgi
sudo a2enmod wsgi
sudo service apache2 start
```

Install GIT: `sudo apt-get install git`.

Clone the catalog project.
* Go into the `cd var/www` directory.
* Then: `sudo mkdir catalog`.
* Change owner for _catalog_ directory use:
    * `sudo chown -R grader:grader catalog`
* Run this command inside the _catalog_ directory:
  * `git clone https://github.com/gsiffer/catalog.git catalog`
* Run this command inside `cd /var/www/catalog/catalog`:
  * `mv project.py __init__.py`
* Create a _catalog.wsgi_ file inside the `var/www/catalog` directory:  
    * `sudo touch catalog.wsgi`
* Use the following command:
  * `sudo nano catalog.wsgi`
  * Copy these lines into it:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
```  

Install virtual environment.
* Install pip: `sudo apt-get install python-pip`.
* Install virtualenv: `sudo pip install virtualenv`.
* Create a new virtual environment inside the `var/www/catalog` directory:
  * `sudo virtualenv venv`
* Activate the virtual environment: `source venv/bin/activate`.
* Change permissions to the virtual environment folder:
  * `sudo chmod -R 777 venv`.
* Install Flask: `pip install Flask`.  
* Install all these project's dependencies:
```
pip install httplib2
pip install requests
pip install oauth2client
pip install sqlalchemy
pip install psycopg2
```

Configure and enable a new virtual host.
* Create a virtual host conifg file:
  * `sudo nano /etc/apache2/sites-available/catalog.conf`.
* Copy the following lines:
```
<VirtualHost *:80>
    ServerName 54.213.28.90.xip.io
    ServerAlias ec2-54.213.28.90.us-west-2.compute.amazonaws.com
    ServerAdmin admin@54.213.28.90
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/$
    WSGIProcessGroup catalog
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
* Enable the new virtual host: `sudo a2ensite catalog`.

### Install and configure PostgreSQL
Install PostgreSQL: `sudo apt-get install postgresql postgresql-contrib`.  
Install some Python packages: `sudo apt-get install libpq-dev python-dev`.  
To create a PostgreSQL database:
* At the command line, type the following command as the root user:
  * `su - postgres`<br/><br/>
* Run this command:
    * `createuser –interactive --pwprompt`
    * At the Enter name of role to add: prompt, type _catalog_.
    * At the Enter password for new role: prompt, type _catalog_.
    * At the Enter it again: prompt, retype the password _catalog_.
    * At the Shall the new role be a superuser? prompt, type n.
    * At the Shall the new role be allowed to create databases? prompt, type y.
    * At the Shall the new role be allowed to create more new roles? prompt, type n.
    * PostgreSQL creates the user with the settings you specified.<br/><br/>
* Connect to the database: `psql`.    
* Create the database:  
  * `CREATE DATABASE catalog WITH OWNER catalog;`.<br/><br/>
* Connect to the database: `\c catalog`.
* Revoke all rights:
  * `REVOKE ALL ON SCHEMA public FROM public;`.<br/><br/>
* To let only catalog role create tables:
  * `GRANT ALL ON SCHEMA public TO catalog;`.<br/><br/>
* Log out from PostgreSQL: `\q`.  
* Then: `exit`.

### Deploy the Item Catalog project.
Log in as a `grader` and go `cd var/www/catalog/catalog`.
* Run: `nano __init__.py`
* At the bottom under `if __name__ == '__main__':` line delete all lines and add `app.run()`.
* Close to the top change line `engine = create_engine('sqlite:///catalog.db', connect_args={'check_same_thread': False}, echo=True)` to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
* Save and exit.
* Run: `nano database_setup.py`.
* At the bottom change line `engine = create_engine('sqlite:///catalog.db')` to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
* Save and exit.

Open the following file: `sudo nano /etc/postgresql/9.3/main/pg_hba.conf` and edit it like this:
```
local   all             postgres                                peer
local   all             all                                     md5
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Save and exit.  
Run the following command: `sudo service apache2 restart`

Sources: [Udacity](https://classroom.udacity.com/courses/ud299), [liliketomatoes](https://github.com/iliketomatoes/linux_server_configuration), [A2hosting](https://www.a2hosting.com/kb/developer-corner/postgresql/managing-postgresql-databases-and-users-from-the-command-line#Creating-PostgreSQL-users)

### LICENCE
https://choosealicense.com/
