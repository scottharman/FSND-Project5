# Udacity Project 5
- Linux server

## Base Configuration of Server  

Firstly copy the private key generated on the development environment site to ~/.ssh/authorized_keys and set the appropriate permissions (700 for .ssh and 600 for authorized_keys) As root, create a new sudoers file for the grader user, to enable grader to perform all sudo functions Alter the ssh daemon to run on port 2200, disable port 22, disable root access to ssh, and enable the grader user to login to ssh with the private key. Next step is to enable UFW, set the default deny rules, and enable outbound traffic. Next step is to allow incoming traffic on the new SSH port of 2200, all http and https traffic, allowing NTP traffic for time synchronisation, then rate limiting SSH connection attempts.

## Specific Installation Guide
Create user grader, with relatively secure password 6r4d*r
Add user grader to sudoers.d file
touch /etc/sudoers.d/grader
edit the file to add the directive to allow grader to sudo:
    grader    ALL=(ALL:ALL) ALL

Edit /etc/ssh/sshd_config
    Allow AuthorizedKeysFile directive
    Comment out PermitRootLogin without-password
    Add line PertmitRootLogin no
    Add port 2200
    Comment out
Restart SSH service

cat the authorized_keys file for root
    su to grader
    mkdir .ssh
    touch .ssh/authorized_keys
copy the authorized_keys to grader's authorized_keys

    chmod 700 .ssh/
    chmod 600 .ssh/authorized_keys

Login to ssh as grader user
Disconnect root ssh session
Validate that you can no longer login as root over ssh

Check UFW Status

    sudo ufw status

Disable all inbound connections

    sudo ufw default deny incoming
Enable all outbound connections

    sudo ufw default allow outgoing
Add rate limited connection for TCP 2200

    sudo ufw allow 2200/tcp
    sudo ufw limit 2200/tcp
Add HTTP incoming on TCP 80 and TCP 443 for SSL

    sudo ufw allow http
    sudo ufw allow https
Add incoming NTP connections

    sudo ufw allow ntp
Enable UFW

    sudo ufw enable

### Auto Update scripts

Create a new file (e.g. autoupdate) containing the following lines:

```
#!/bin/bash
apt-get update
apt-get upgrade -y
apt-get autoclean
```

Make the file executable for all users (chmod 755), then copy it to /etc/cron.daily/ to trigger automatic software updates.

Execute it initially with sudo ./autoupdate

## Installation of key software  

Install packages with apt-get apache2 libapache2-mod-wsgi postgresql git python-pip build-essential python-dev python-flask python-sqlalchemy python-psycopg2

## Create Catalog postgres user
To bypass an issue with peer authentication, I've added a password protected catalog user, and allowed this to access the database over localhost.
I've specified that the user should connect to localhost, using a password 'i3hu3an'
I've loaded the default catalog database as the catalog database user.

## Install Python modules

sudo pip install bleach
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install wtforms wtforms-alchemy --upgrade
sudo pip install --upgrade flask-login==0.2.9 flask-sqlalchemy
sudo pip install flask-whooshalchemy
sudo pip install --upgrade flask
sudo pip install --upgrade flask-login
sudo pip install --upgrade flask-openid
sudo pip install --upgrade flask-mail
sudo pip install --upgrade flask-sqlalchemy
sudo pip install --upgrade sqlalchemy-migrate
sudo pip install --upgrade flask-seasurf
sudo pip install --upgrade flask-whooshalchemy
sudo pip install --upgrade flask-wtf
sudo pip install --upgrade wtforms-alchemy
sudo pip install --upgrade flask-babel
sudo pip install --upgrade guess_language
sudo pip install --upgrade flipflop
sudo pip install --upgrade coverage

## Clone catalog repository from git
git clone https://github.com/scottharman/Udacity-FSND-VM-Vagrant.git

## Configure Apache2 to server wsgi apps
Edit /etc/apache2/sites-enabled/000-default.conf
The file should contain the following to allow you to serve the app and static folders

    # Set the daemon process username and group names for the app, catalog
    WSGIDaemonProcess catalog user=www-data group=www-data
    # Set the scriptalias for the root location to the catalog app
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    # define the catalog location, I've placed this under /var/www by convention, but the wsgi script calls the app under the grader home dir.
    <Directory /var/www/catalog>
            WSGIProcessGroup catalog
            WSGIApplicationGroup %{GLOBAL}
            #Order deny,allow
            #Allow from all
            # Modern apache notation
            Options All
            AllowOverride All
            Require all granted
    </Directory>
    #Explicitly set static alias.
    Alias /static /home/grader/Udacity-FSND-VM-Vagrant/vagrant/catalog/app/static/
    <Directory /home/grader/Udacity-FSND-VM-Vagrant/vagrant/catalog/app/static>
            #Order allow,deny
            #Allow from all
            Options All
            AllowOverride All
            Require all granted
    </Directory>



This will allow you to serve the catalog app from whereever you have defined it

I've put mine into a directory called 'catalog' under /var/www/
Contents are as follows:

    import sys
    sys.path.insert(0, '/home/grader/Udacity-FSND-VM-Vagrant/vagrant/catalog')

    from application import app as application


As I have cloned the repo into grader's home directory, I'll add the grader user to apache's group, then use chgrp to set the group for the catalog app.

## App Changes
Minor changes were made to paths to reflect changes in the operating system paths for upload locations, and loading the CSRF tokens.
A minor change was made to the Google Developers Console to add a new allowed origins and redirect URLs

Changes were made in a couple of different areas:

1. Change the path of client_secrets.json for client_id
2. Change the path of client_secrets.json for oauth flow (would not pull from the catalog folder, only the app folder)
3. Explicitly set the path of the upload folder to '/home/grader/Udacity-FSND-VM-Vagrant/vagrant/catalog/app/static/uploads'
4. Alter the database path to use the database catalog user, rather than the vagrant user.

## Additional software
Rate limiting connections is potentially not enough, and attackers may also target the app itself, so have added Fail2Ban to block repeated attempts to login.
Have also added glances for server monitoring, it would have been nice to have glances running as a wsgi app inside Apache, but can only be run via Bottle

### Developer Console
Logged into Developer console - added new Origin as:
http://ec2-52-36-104-250.us-west-2.compute.amazonaws.com/
Added new redirect URLs
http://ec2-52-36-104-250.us-west-2.compute.amazonaws.com/gconnect
http://ec2-52-36-104-250.us-west-2.compute.amazonaws.com/login

## Access server
Catalog application server is accessible at:
http://ec2-52-36-104-250.us-west-2.compute.amazonaws.com/

SSH is available on 52.36.104.250 port 2200
Username is grader, and both the password and private key have been supplied with the submission.
