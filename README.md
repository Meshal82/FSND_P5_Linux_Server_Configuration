###### Udacity Full Stack Web Developer Nanodegree
# Project 5: Linux Server Configuration

A baseline installation & configuration of Ubuntu Linux on a virtual machine to host a [flask][1] web application. This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers/servieces.

**Note:** Below is a step-by-step walkthrough solution for the 5th project of the Udacity Full Stack Web Developer Nanodegree and deploys a [flask][1] application that was built on [project 4 (Item Catalog)][2] on the virtual machine.

**Note:** Project was uploaded on [Digital Ocean][3] 

### 1 - Launch Virtual Machine and SSH into the server 

1. Create new development environment.
2. Download private keys and write down your public IP address.
3. Move the private key file into the folder ~/.ssh:  
  `$ mv ~/Downloads/udacity_key.rsa ~/.ssh/`
4. Set file rights (only owner can write and read.):  
  `$ chmod 600 ~/.ssh/udacity_key.rsa`
5. SSH into the instance:  
  `$ ssh -i ~/.ssh/udacity_key.rsa root@PUPLIC-IP-ADDRESS`

### 2 - User Management: Create a new user and give user the permission to sudo
Source: [DigitalOcean][4]  

1. Create a new user:  
  `$ adduser grader`
2. Give new user the permission to sudo
  1. Open the sudo configuration:  
    `$ visudo`
  2. Add the following line below `root ALL...`:  
    `grader ALL=(ALL:ALL) ALL`
  3. *List all users (Source: [Ask Ubuntu][5]):    
    `$ cut -d: -f1 /etc/passwd`

### 3 - Update and upgrade all currently installed packages
Source: [Ask Ubuntu][6]  
    
1. Update the list of available packages and their versions:  
  `$ sudo apt-get update`
2. Install newer vesions of packages you have:  
  `$ sudo sudo apt-get upgrade`

### 4 - Change the SSH port from 22 to 2200 and configure SSH access
Source: [Ask Ubuntu][7]  

  1. Open the config file:  
    `$ nano /etc/ssh/sshd_config` 
  2. Change to Port 2200.
  3. Change `PermitRootLogin` from `without-password` to `no`.
  4. Temporalily change `PasswordAuthentication` from `no` to `yes`.
  5. Append `UseDNS no`.
  6. Append `AllowUsers grader`.  
**Note:** All options on [UNIXhelp][8]
2. Restart SSH Service:  
  `# service sshd restart` 
3. Create SSH Keys:  
 Source: [DigitalOcean][9]  

  1. Generate a SSH key pair on the local machine:  
    `$ ssh-keygen`
  2. Copy the public id to the server:  
    `$ ssh-copy-id username@remote_host -p**_PORTNUMBER_**`
  3. Login with the new user:  
    `$ ssh -v grader@PUBLIC-IP-ADDRESS -p2200`
  4. Open SSHD config:  
    `$ sudo nano /etc/ssh/sshd_config`
  5. Change `PasswordAuthentication` back from `yes` to `no`.

### 5 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
Source: [Ubuntu documentation][10]  

1. Turn UFW on with the default set of rules:  
  `$ sudo ufw enable` 
2. *Check the status of UFW:  
  `$ sudo ufw status verbose`
3. Allow incoming TCP packets on port 2200 (SSH):  
  `$ sudo ufw allow 2200/tcp` 
4. Allow incoming TCP packets on port 80 (HTTP):  
  `$ sudo ufw allow 80/tcp` 
5. Allow incoming UDP packets on port 123 (NTP):  
  `$ sudo ufw allow 123/udp`  

### 6 - Configure the local timezone to UTC
Source: [Ubuntu documentation][11]

1. Open the timezone selection dialog:  
  `$ sudo dpkg-reconfigure tzdata`
2. Then chose 'None of the above', then UTC.
3. *Setup the ntp daemon ntpd for regular and improving time sync:  
  `$ sudo apt-get install ntp`
4. *Chose closer NTP time servers:  
5. Open the NTP configuration file:  
    `$ sudo nano /etc/ntp.conf`

### 7 - Install and configure Apache to serve a Python mod_wsgi application
Source: [Udacity][12]

1. Install Apache web server:  
  `$ sudo apt-get install apache2`
2. Open a browser and open your public ip address, e.g. http://68.183.134.22/ - It should open apache dashboard on the page.
3. Install **mod_wsgi** for serving Python apps from Apache and the helper package **python-setuptools**:  
  `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
4. Restart the Apache server for mod_wsgi to load:  
  `$ sudo service apache2 restart`

### 8 - Install and configure git
Source: [GitHub][13]
        
1. Install Git:  
  `$ sudo apt-get install git`

### 9 - Setup for deploying a Flask Application on Ubuntu VPS
Source: [DigitalOcean][14]

1. Extend Python with additional packages that enable Apache to serve Flask applications:  
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi (if not already enabled):  
  `$ sudo a2enmod wsgi`
3. Create a Flask app:  
  1. Move to the www directory:  
    `$ cd /var/www`
  2. Setup a directory for the app, e.g. catalog:  
    1. `$ sudo mkdir catalog`  
    2. `$ cd catalog` and `$ sudo mkdir catalog`  
    3. `$ cd catalog` and `$ sudo mkdir static templates`  
    4. Create the file that will contain the flask application logic:  
      `$ sudo nano __init__.py`
    5. Paste in the following code:  
    ```python  
      from flask import Flask  
      app = Flask(__name__)  
      @app.route("/")  
      def hello():  
        return "Welcome"  
      if __name__ == "__main__":  
        app.run()  
    ```  
4. Install Flask
  1. Install pip installer:  
    `$ sudo apt-get install python-pip` 
  2. Install virtualenv:  
    `$ sudo pip install virtualenv`
  3. Set virtual environment to name 'venv':  
    `$ sudo virtualenv venv`
  4. Enable all permissions for the new virtual environment (no sudo should be used within):  
    Source: [Stackoverflow][15]              
    `$ sudo chmod -R 777 venv`
  5. Activate the virtual environment:  
    `$ source venv/bin/activate`
  6. Install Flask inside the virtual environment:  
    `$ pip install Flask`
  7. Run the app:  
    `$ python __init__.py`
  8. Deactivate the environment:  
    `$ deactivate`
5. Configure and Enable a New Virtual Host#
  1. Create a virtual host config file  
    `$ sudo nano /etc/apache2/sites-available/catalog.conf`
  2. Paste in the following lines of code and change names and addresses regarding your application:  
  ```
    <VirtualHost *:80>
        ServerName YOUR-SERVER-PUBLIC-IP-ADDRESS
        ServerAdmin admin@YOUR-SERVER-PUBLIC-IP-ADDRESS
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
        ErrorLog /var/log/apache2/error.log
        LogLevel warn
        CustomLog /var/log/apache2/access.log combined
    </VirtualHost>
  ```
  3. Enable the virtual host:  
    `$ sudo a2ensite catalog`
6. Create the .wsgi File and Restart Apache
  1. Create wsgi file:  
    `$ cd /var/www/catalog` and `$ sudo nano catalog.wsgi`
  2. Paste in the following lines of code:  
  ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")
    
    from catalog import app as application
    application.secret_key = 'secret'
  ```
  7. Restart Apache:  
    `$ sudo service apache2 restart`

### 10 - Clone GitHub repository and make it web inaccessible
1. Clone project 3 solution repository on GitHub:  
  `$ git clone https://github.com/Meshal82/Item-Catalog.git`
2. Move all content of created FSND-P3_Music-Catalog-Web-App directory to `/var/www/catalog/catalog/`-directory and delete the leftover empty directory.
3. Make the GitHub repository inaccessible:  
  Source: [Stackoverflow][16]
  1. Create and open .htaccess file:  
    `$ cd /var/www/catalog/` and `$ sudo nano .htaccess` 
  2. Paste in the following:  
    `RedirectMatch 404 /\.git`

### 11 - Install needed modules & packages
1. Activate virtual environment:  
  `$ source venv/bin/activate`
2. Install httplib2 module in venv:  
  `$ pip install httplib2`
3. Install requests module in venv:  
  `$ pip install requests`
4. *Install flask.ext.seasurf (only seems to work when installed globally):  
  `$ *sudo pip install flask-seasurf`
5. Install oauth2client.client:  
  `$ sudo pip install --upgrade oauth2client`
6. Install SQLAlchemy:  
  `$ sudo pip install sqlalchemy`
7. Install the Python PostgreSQL adapter psycopg:  
  `$ sudo apt-get install python-psycopg2`

### 12 - Install and configure PostgreSQL
Source: [DigitalOcean][17] (alternatively, nice short guide on [Kill The Yak][18] as well)  

1. Install PostgreSQL:  
  `$ sudo apt-get install postgresql postgresql-contrib`
2. Check that no remote connections are allowed (default):  
  `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
3. Open the database setup file:  
  `$ sudo nano database_setup.py`
4. Change the line starting with "engine" to (fill in a password):  
  ```python engine = create_engine('postgresql://catalog:secret@localhost/catalog')```  
5. Change the same line in application.py respectively
6. Rename application.py:  
  `$ mv application.py __init__.py`
7. Create needed linux user for psql:  
  `$ sudo adduser catalog` (choose a password)
8. Change to default user postgres:  
  `$ sudo su - postgre`
9. Connect to the system:  
  `$ psql`
10. Add postgre user with password:  
  Sources: [Trackets Blog][19] and [Super User][20]
  1. Create user with LOGIN role and set a password:  
    `# CREATE USER catalog WITH PASSWORD 'secret';` (# stands for the command prompt in psql)
  2. Allow the user to create database tables:  
    `# ALTER USER catalog CREATEDB;`
  3. *List current roles and their attributes:
    `# \du`
11. Create database:  
  `# CREATE DATABASE catalog WITH OWNER catalog;`
12. Connect to the database catalog
  `# \c catalog` 
13. Revoke all rights:  
  `# REVOKE ALL ON SCHEMA public FROM public;`
14. Grant only access to the catalog role:  
  `# GRANT ALL ON SCHEMA public TO catalog;`
15. Exit out of PostgreSQl and the postgres user:  
  `# \q`, then `$ exit` 
16. Create postgreSQL database schema:  
  $ python database_setup.py

### 13 - Run application 
1. Restart Apache:  
  `$ sudo service apache2 restart`
2. Open a browser and put in your public ip-address as url, e.g. 68.183.134.22 - if everything works, the application should come up
3. *If getting an internal server error, check the Apache error files:  
  Source (Thanks to Misk/Udacity Slack members for help): [A2 Hosting][21]  
  1. View the last 20 lines in the error log: 
    `$ sudo tail -20 /var/log/apache2/error.log`
  2. *If a file like 'g_client_secrets.json' couldn't been found: 
    Source: [Stackoverflow][22]  

### 14 - Get OAuth-Logins Working
  Source: [Udacity][23] and [Apache][24]  
  
1. Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for http://68.183.134.22, its http://68.183.134.22.xip.io/
2. Open the Apache configuration files for the web app:
  `$ sudo nano /etc/apache2/sites-available/catalog.conf`
3. Paste in the following line below ServerAdmin:  
  `ServerAlias HOSTNAME`, e.g. http://68.183.134.22.xip.io
4. Enable the virtual host:  
  `$ sudo a2ensite catalog`
5. To get the Google+ authorization working:  
  1. Go to the project on the Developer Console: https://console.developers.google.com/project
  2. Navigate to APIs & auth > Credentials > Edit Settings
  3. add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://68.183.134.22.xip.io

[1]: https://de.wikipedia.org/wiki/Flask "Wikipedia entry to Flask"
[2]: https://github.com/Meshal82/Item-Catalog "GitHub repository of my item catalog web app"
[3]: cloud.digitalocean.com "Digitalo Ocean for Cloud solutions"
[4]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps "How To Add and Delete Users on an Ubuntu 14.04 VPS"
[5]: http://askubuntu.com/questions/410244/a-command-to-list-all-users-and-how-to-add-delete-modify-users "How to list, add, delete and modify users"
[6]: http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade "What is the difference between apt-get update and upgrade?"
[7]: http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server "Create a new SSH user on Ubuntu Server"
[8]: http://unixhelp.ed.ac.uk/CGI/man-cgi?sshd_config "UNIX man page: SSHD_CONFIG"
[9]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server "How To Configure SSH Key-Based Authentication on a Linux Server"
[10]: https://help.ubuntu.com/community/UFW "UFW - Uncomplicated Firewall"
[11]: https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29 "Ubuntu Time Management"
[12]: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html "A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu"
[13]: https://help.github.com/articles/set-up-git/#platform-linux "Set Up Git for Linux"
[14]: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "How To Deploy a Flask Application on an Ubuntu VPS"
[15]: http://stackoverflow.com/questions/14695278/python-packages-not-installing-in-virtualenv-using-pip "python packages not installing in virtualenv using pip"
[16]: http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible "Make .git directory web inaccessible"
[17]: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps "How To Secure PostgreSQL on an Ubuntu VPS"
[18]: http://killtheyak.com/use-postgresql-with-django-flask/ "All I want to do is use PostgreSQL with Flask or Django."
[19]: http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html "PostgreSQL Basics by Example"
[20]: http://superuser.com/questions/769749/creating-user-with-password-or-changing-password-doesnt-work-in-postgresql "Creating user with password or changing password doesn't work in PostgresQL"
[21]: https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files "How to view Apache log files"
[22]: http://stackoverflow.com/questions/12201928/python-open-method-ioerror-errno-2-no-such-file-or-directory "Python: No such file or directory"
[23]: http://discussions.udacity.com/t/oauth-provider-callback-uris/20460 "OAuth Provider callback uris"
[24]: http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html "Name-based Virtual Host Support"