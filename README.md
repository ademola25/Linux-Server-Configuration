# Linux-Server-Configuration
Full-stack Nano degree web development Program udacity Fifth Project

## Project Detail:
* A baseline installation of a Linux server, prepared  to host my web applications. Then i secured the server from a number of attack vectors, then i installed and configured a database server, and i finally deployed my Spa Catalog web applications onto it.



## Server Details
    ip: 18.184.157.0.xip.io/
    url: http://ec2-18-184-157-0.eu-central-1.compute.amazonaws.com/
    ssh 2200
    
## Configuration Steps

### Create a new user named grader
   
* Add a new user grader : sudo useradd -m -s /bin/bash grader
* Set a Password for the new user: sudo passwd grader
* Give Sudo Permissions to Grader: sudo usermod -aG sudo grader

### Update and Upgrade all Packages
Run sudo apt-get update command to update package indexes.
Run sudo apt-get upgrade command to upgrade and install packages.
If at login the message System restart required is display, run the following command to reboot the machine: reboot.

### Fix sudo resolve host error
When the grader user issues a sudo command, got the following warning: sudo: unable to resolve host ip-172-26-15-250

 fix the error by adding the hostname  to the loopback address in the /etc/hosts file so that the first line now look like this: 127.0.0.1 localhost ip-172-26-15-250
 
### Configure the key-based authentication for grader user
* Generate an encryption key on your local machine with:  ssh-keygen 
* Print out the generated key with: cat ~/.ssh/id_rsa.pub
* Log into the newly created user grader with: su - grader
* Create a new directory called .ssh with: mkdir ~/.ssh
* Now open a file in .ssh called authorized_keys with a text editor: nano ~/.ssh/authorized_keys
* And restrict its permissions with the following commands:chmod 700 ~/.ssh
* Now insert your public key (which should be in your clipboard) by pasting it into the editor
* Restrict the permissions of the authorized_keys file with this command: chmod 644 ~/.ssh/authorized_keys
*Return to root user: exit
* Now log in with : ssh -i ~/.ssh/authorized_keys.rsa grader@18.184.157.0
  
 ###  Change the SSH port from 22 to 2200 and configure SSH access
 *  Open the config file: $ sudo nano /etc/ssh/sshd_config
 *  Change from port 22 to Port 2200.
 *  Change PermitRootLogin from without-password to no
 *  Now log in with : ssh -i ~/.ssh/authorized_keys.rsa grader@18.184.157.0 -p2200
 
 ###  Change timezone to UTC
 Open time configuration dialog and set it to UTC with: $ sudo dpkg-reconfigure tzdata
 
 ###  Configuration Uncomplicated Firewall (UFW)
 
 *  By default, block all incoming connections on all ports: sudo ufw default deny incoming
 *  Allow outgoing connection on all ports: sudo ufw default allow outgoing
 *  Allow incoming connection for SSH on port 2200: sudo ufw allow 2200/tcp
 *  Allow incoming connections for HTTP on port 80: sudo ufw allow www
 *  Allow incoming connection for NTP on port 123: sudo ufw allow ntp
 *  To check the rules that have been added before enabling the firewall: sudo ufw show added
 *  To enable the firewall: sudo ufw enable
 *  To check the status of the firewall: sudo ufw status
 
 ###  Install Apache to serve a Python mod_wsgi application
 *  Install Apache: sudo apt-get install apache2
 *  Install the libapache2-mod-wsgi package: sudo apt-get install libapache2-mod-wsgi python-dev
 *  To enable mod_wsgi, run the following command: sudo a2enmod wsgi 
 
 ###  Install Git
 *  $ sudo apt-get install git.
 *  Configure your username: $ git config --global user.name <username>.
 *  Configure your email: $ git config --global user.email <email-address>.
  
 ### Clone the Catalog app from Github
 
 *  Move to the www directory: $ cd /var/www
 *  create a directory for the application. catalog: $ sudo mkdir catalogApp
 *  Move inside that newly created folder: $ cd /catalog and 
 *  clone the catalog repository from Github: $ sudo git clone https://github.com/ademola25/catalog.git
 *  To make github repository inaccessible make a .htaccess file: $ cd /var/www/catalogApp/ and : $ sudo nano .htaccess
  Paste in the following: RedirectMatch 404 /\.git
 ###  Install virtual environment, Flask and the project's dependencies
 STEP 1
 *  Install pip, the tool for installing Python packages: $ sudo apt-get install python-pip
 *  Then use pip to install Virtualenv: $ sudo pip install virtualenv
 *  Move to the catalog folder: $ cd /var/www/catalogApp/catalog and Then create the virtual environment: $ sudo virtualenv venv
 *  Activate virtual environment: $ source venv/bin/activate
 *  Change permissions to the virtual environment folder: $ sudo chmod -R 777 venv
 *  Install Flask: $ pip install Flask.
 *  Install httplib2 module in venv: $ pip install httplib2
 *  Install requests module in venv: $ pip install requests
 *  Install oauth2client.client: $ sudo pip install oauth2client
 *  Install SQLAlchemy: $ sudo pip install sqlalchemy
 *  Install the Python PostgreSQL adapter psycopg: $ sudo pip install psycopg2

STEP 2:
* A virtual host configuration file needs to be created in the directory /etc/apache2/sites-available/, it is called catalog-app.conf. 
* $ sudo nano /etc/apache2/sites-available/catalog.conf
  
  <VirtualHost *:80>
                ServerName 18.184.157.0.xip.io
                ServerAdmin admin@18.184.157.0

                WSGIDaemonProcess catalog user=www-data group=www-data threads=5
                WSGIProcessGroup catalog
                WSGIApplicationGroup %{GLOBAL}
                WSGIScriptAlias / /var/www/catalogApp/catalog.wsgi
                <Directory /var/www/catalogApp/catalog/>
                        Require all granted
                </Directory>
                Alias /static /var/www/catalogApp/catalog/static
                <Directory /var/www/catalogApp/catalog/static/>
                        Require all granted
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
 *  Disable the default virtual host with:sudo a2dissite 000-default.conf
    If you deactivate the default conf, the catalog one will automatically become the default.
 *  Enable the new virtual host: $ sudo a2ensite catalog

STEP 3:

 *  Make a catalog.wsgi file to serve the application over the mod_wsgi. Below is the content
 *  Create the wsgi file using: cd /var/www/catalogApp
 
    activate_this = '/var/www/catalogApp/catalog/venv/bin/activate_this.py'
    execfile(activate_this, dict(__file__=activate_this))
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalogApp/")

    from catalog import app as application
    application.secret_key = 'add your secret key'
    
  * Restart Apache : sudo service apache2 restart
 
 
 ###  Install and configure PostgreSQL
 
 *  Install PostgreSQL: $ sudo apt-get install postgresql postgresql-contrib.
 *  A postgres username will be automatically ctreated during its installation and this user name can access the database software.
    so we switch to the postgres user by:$ sudo su - postgres and then connect to the database with: $ psql
 *  Create a new user called 'catalog' with his password: # CREATE USER catalog WITH PASSWORD 'postgres';.
 *  Grant catalog user the CREATEDB: # ALTER USER catalog CREATEDB;.
 *  Create the 'catalog' database owned by catalog user: # CREATE DATABASE catalog WITH OWNER catalog;.
 *  Connect to the database: # \c catalog.
 *  Revoke all rights: # REVOKE ALL ON SCHEMA public FROM public;.
 *  Lock down the permissions to only let catalog role create tables: # GRANT ALL ON SCHEMA public TO catalog;.
 *  Log out from PostgreSQL: # \q. 
 *  Return to the grader user: $ exit.
 Now set the database connection inside the application to
  engine = create_engine('postgresql://catalog:postgres@localhost/catalog')
 *  Setup the database with: $ python /var/www/catalogApp/catalog/database_setup.py.
 *  Setup the database with: $ python /var/www/catalogApp/catalog/lotsofspaitem.py.
 *  Setup the database with: $ python /var/www/catalogApp/catalog/__init__.py.
 
 ###  Run the application:
 *  move to  /var/www/catalogApp/catalog directory
 *  Create the database schema: python database_setup.py python lotsofspaitem.py
 *  Restart Apache : sudo service apache2 restart
 *  execute - python __init__.py

### Update OAuth authorized JavaScript origins
* Go to xip.io and get the host name of public IP address 18.184.157.0.xip.io
* sudo nano /etc/apache2/sites-available/catalog.conf and 
* Add the hostname below ServerAlias: paste ServerAlias ec2-18-184-157-0.eu-central-1.compute.amazonaws.com
* Enable the virtual host : sudo a2ensite catalog


### Restart Apache to launch the application
To make these Apache2 configuration changes, reload Apache: sudo service apache2 restart


# License
Project is released under the [MIT License](http://opensource.org/licenses/MIT).

### Resource:
[Initial Server Setup with Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)

[How To Deploy a Flask Application on an Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

[How To Install and Use PostgreSQL on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)

[virtual environment installation](http://flask.pocoo.org/docs/0.12/installation/#installation)

### References:
* [amunategui blog](http://amunategui.github.io/idea-to-pitch/#installing-flask)

* Udacity Mentor
