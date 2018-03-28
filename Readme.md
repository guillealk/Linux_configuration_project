# Linux Sever Configuration Udacity Project


The below step-by-step walkthrough is the solution to project 5 of the Udacity Full Stack Web Developer Nanodegree and deploys the solution of project 3 on the virtual machine.

# Configutation Steps

## 1. Create Development Environment: Launch Virtual Machine

* Create new development environment: https://lightsail.aws.amazon.com.
* Download the private key from your lightsail aws instance.

## 2. Config ssh file in your computer to access to your server
* Move the private key filo into the folder `~/.ssh`:  
`~$ mv ~/Downloads/LightsailDefaultPrivateKey-us-east-2.pem ~/.ssh`
* Set file rights (only owner can write and read.):
`$ chmod 600 ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem`
* SSH into the instance:
`$ ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem ubuntu@18.188.112.81`

## 3. Create a new User and 

* Create a new user: `$ adduser grader`

## 4. Give user the "sudo" permission

* Give new user the permission to sudo: `$ sudo visudo`
   * Add the following line below `root ALL...`: `grader ALL=(ALL:ALL) ALL`

## 5. Update and upgrade all currently installed packages

* Update the list of available packages and their versions:
`$ sudo apt-get update`
* Install newer vesions of packages you have:
`$ sudo sudo apt-get upgrade`

## 6. Change the SSH port from 22 to 2200 and configure SSH access

* Change ssh config file:
   - Open the config file: `$ sudo nano /etc/ssh/sshd_config`
   - Change `Port 22` to `Port 2200`.
   - Change PermitRootLogin from `without-password` to `no`.
   - Append AllowUsers `grader`.

* Restart SSH Service:
`$ /etc/init.d/ssh restart` or `$ service sshd restart`

* Create SSH Keys:
   - Generate a SSH key pair on the local machine: `$ ssh-keygen`
   - Copy the public id to the server:
   `$ ssh-copy-id username@remote_host -p 2200`
* Login with the new user:
`$ ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem grader@18.188.112.81 -p2200`

## 7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

* Check the firewall status: `sudo ufw status`
* Block all incoming connections on all ports: `sudo ufw default deny incoming`
* Allow outgoing connections on all ports: `sudo ufw default allow outgoing`
* Allow incoming connection for SSH(port 2200): `sudo ufw allow 2200/tcp`
* Allow incoming connection for HTTP(port 80): `sudo ufw allow 80/tcp`
* Allow incoming connection for NTP(port 123): `sudo ufw allow 123/udp`
* Check the added rules: `sudo ufw show added`
* Enable the firewall: `sudo ufw enable`
* Check whether firewall is enable or not: `sudo ufw status`

## 8. Configure the local timezone to UTC

* Open the timezone selection dialog:
`$ sudo dpkg-reconfigure tzdata`
* Then chose 'None of the above', then UTC.
* Setup the ntp daemon ntpd for regular and improving time sync:
`$ sudo apt-get install ntp`
* Chose closer NTP time servers. Open the NTP configuration file:
`$ sudo nano /etc/ntp.conf`. Open http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace the given servers with the new server list.

## 9. Install and configure Apache to serve a Python mod_wsgi application

* Install Apache web server:
`$ sudo apt-get install apache2`. Open a browser and open your public ip address, e.g. http://18.188.112.81/ - It should say 'It works!' on the top of the page.

* Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
`$ sudo apt-get install python-setuptools libapache2-mod-wsgi`

* Restart the Apache server for mod_wsgi to load:
`$ sudo service apache2 restart`
* You might get this message "Could not reliably determine the servers's fully qualified domain name" after restart 
    * Create an empty Apache config file with the hostname:
      `$ echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf`
    * Enable the new config file:
    `$ sudo a2enconf fqdn`

## 10. Install and configure git

* Install Git:
`$ sudo apt-get install git`
* Set your name, e.g. for the commits:
`$ git config --global user.name "YOUR NAME"`
* Set up your email address to connect your commits to your account:
`$ git config --global user.email "YOUR EMAIL ADDRESS"`

## 11. Setup for deploying a Flask Application on Ubuntu VPS

* Extend Python with additional packages that enable Apache to serve Flask applications:
`$ sudo apt-get install libapache2-mod-wsgi python-dev`
* Enable mod_wsgi (if not already enabled):
`$ sudo a2enmod wsgi`
* Create a Flask app:
    * Move to the www directory: `$ cd /var/www`
    * Setup a directory for the app, e.g. catalog:
    `$ sudo mkdir catalog`<br>
    `$ cd catalog` and `$ sudo mkdir catalog`<br>
    `$ cd catalog` and `$ sudo mkdir static templates`<br>
* Create the file that will contain the flask application logic:
`$ sudo nano __cess.log combined`<br>
  `</VirtualHost>`init__.py`
* Paste in the following code:
`from flask import Flask`<br>
`app = Flask(__name__)`<br>
`@app.route("/")`<br>
`def hello():` <br>
  `return "Hello!!"`<br>
 `if __name__ == "__main__":`<br>
   `app.run()`

* Install Flask
    * Install pip installer: `$ sudo apt-get install python-pip`
    * Install virtualenv: `$ sudo pip install virtualenv`
    * Set virtual environment to name 'venv': `$ sudo virtualenv venv`
    * Enable all permissions for the new virtual environment (no sudo should be used within): `$ sudo chmod -R 777 venv`
    * Activate the virtual environment: `$ source venv/bin/activate`
    * Install Flask inside the virtual environment: `$ pip install Flask`
    * Run the app: `$ python __init__.py`
    * Deactivate the environment:`$ deactivate`
    Source: Stackoverflow
* Configure and Enable a New Virtual Host#
    * Create a virtual host config file
    `$ sudo nano /etc/apache2/sites-available/catalog.conf`
    *Paste in the following lines of code and change names and addresses regarding your application:
    
    `<VirtualHost *:80>` <br>
      `ServerName PUBLIC-IP-ADDRESS` <br>
      `ServerAdmin admin@PUBLIC-IP-ADDRESS`<br>
      `WSGIScriptAlias / /var/www/catalog/catalog.wsgi`<br>
      `<Directory /var/www/catalog/catalog/>`<br>
          `Order allow,deny`<br>
          `Allow from all`<br>
      `</Directory>`<br>
      `Alias /static /var/www/catalog/catalog/static`<br>
      `<Directory /var/www/catalog/catalog/static/>`<br>
          `Order allow,deny`<br>
          `Allow from all`<br>
      `</Directory>`<br>
      `ErrorLog ${APACHE_LOG_DIR}/error.log`<br>
      `LogLevel warn`<br>
      `CustomLog ${APACHE_LOG_DIR}/access.log combined`<br>
  `</VirtualHost>`<br>
* Enable the virtual host:
`$ sudo a2ensite catalog`
* Create the .wsgi File and Restart Apache
    * Create wsgi file: `$ cd /var/www/catalog` and <br>
                        `$ sudo nano catalog.wsgi`
    * Paste in the following lines of code: `#!/usr/bin/python`<br>
                                            `import sys`<br>
                                            `import logging`<br>
                                            `logging.basicConfig(stream=sys.stderr)`<br>
                                            `sys.path.insert(0,"/var/www/catalog/")`<br>
                                            `from catalog import app as application`<br>
                                            `application.secret_key = 'Add your secret key'`
    * Restart Apache: `$ sudo service apache2 restart`

## 12. Clone GitHub repository and make it web inaccessible
* Clone project 3 solution repository on GitHub:
`$ git clone https://github.com/guillealk/item_catalog.git`
* Move all content of created Item Catalog Project directory to `/var/www/catalog/catalog/` directory and delete the leftover empty directory.
* Make the GitHub repository inaccessible:
* Create and open .htaccess file:
`$ cd /var/www/catalog/` and <br>
`$ sudo nano .htaccess`
* Paste in the following:
  `RedirectMatch 404 /\.git`

## 13. Install needed modules & packages

* Activate virtual environment: `$ source venv/bin/activate`
* Install httplib2 module in venv:
`$ pip install httplib2`
* Install requests module in venv:
`$ pip install requests`
* Install flask.ext.seasurf (only seems to work when installed globally):
`$ *sudo pip install flask-seasurf`
* Install oauth2client.client:
`$ sudo pip install --upgrade oauth2client`
* Install SQLAlchemy:
`$ sudo pip install sqlalchemy`
* Install the Python PostgreSQL adapter psycopg:
`$ sudo apt-get install python-psycopg2`

## 14. Install and configure PostgreSQL

* Install PostgreSQL:
`$ sudo apt-get install postgresql postgresql-contrib`
* Check that no remote connections are allowed (default):
`$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
* Open the database setup file:
`$ sudo nano database_setup.py`
  * Change the line starting with "engine" to (fill in a password): `engine = create_engine('postgresql://catalog:catalog-pw@localhost/catalog')`
  * Change the same line in `project.py` respectively
* Rename project.py: `$ mv project.py __init__.py`
* Change to default user postgres: `$ sudo su - postgres`
* Connect to the system: `$ psql`
* Add postgres user with password:
    * Create user with LOGIN role and set a password:
    `CREATE USER catalog WITH PASSWORD 'catalog-pw';` (# stands for the command prompt in psql)
    * Allow the user to create database tables:
    `ALTER USER catalog CREATEDB;`
    * List current roles and their attributes: `\du`
* Create database:
`CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to the database catalog: `\c catalog`
* Revoke all rights:
`REVOKE ALL ON SCHEMA public FROM public;`
* Grant only access to the catalog role:
`GRANT ALL ON SCHEMA public TO catalog;`
* Exit out of PostgreSQl and the postgres user: `\q`, then `$ exit`
* Create postgreSQL database schema:
`$ python database_setup.py`

## 15. Run application

* Restart Apache:
`$ sudo service apache2 restart`
* Open a browser and put in your public ip-address as url, e.g. 18.188.112.81 - if everything works, the application should come up
* If getting an internal server error, check the Apache error files:

    * View the last 20 lines in the error log: `$ sudo tail -20 /var/log/apache2/error.log`

## 16. OAuth Login

* Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 18.188.112.81, its ec2-18-188-112-81.us-east-2.compute.amazonaws.com
* Open the Apache configuration files for the web app: `$ sudo nano /etc/apache2/sites-available/catalog.conf`
* Paste in the following line below ServerAdmin:
`ServerAlias ec2-18-188-112-81.us-east-2.compute.amazonaws.com`
* Enable the virtual host:
`$ sudo a2ensite catalog`
* To get the Google+ authorization working:
    * Go to the project on the Developer Console: https://console.developers.google.com/project
    * Navigate to *APIs & auth > Credentials > Edit Settings*:
        - Add http://18.188.112.81 in *Authorized JavaScript origins*
        - Add http://ec2-18-188-112-81.us-east-2.compute.amazonaws.com in *Authorized JavaScript origins*
        - Add http://ec2-18-188-112-81.us-east-2.compute.amazonaws.com/oauth2callback in *Authorized redirect URIs*
        - Add http://18.188.112.81 and http://ec2-18-188-112-81.us-east-2.compute.amazonaws.com in *javacript_origins* in the `/var/www/catalog/catalog/client_secrets.json` file: `sudo nano /var/www/catalog/catalog/client_secrets.json`
        - Add http://ec2-18-188-112-81.us-east-2.compute.amazonaws.com/oauth2callback in *redirect_uris* in the `/var/www/catalog/catalog/client_secrets.json` file: `sudo nano /var/www/catalog/catalog/client_secrets.json`

* To get the Facebook authorization working:
    * Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
    * Click on your App, go to *Settings*, *Advanced*, click on *Add a Domain*, add http://18.188.112.81 and http://ec2-18-188-112-81.us-east-2.compute.amazonaws.com and click on *Save Changes*
