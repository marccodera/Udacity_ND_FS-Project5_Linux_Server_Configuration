# Udacity Nanodegree Full Stack - Project 5 - Linux Server Configuration

This is the repository for the readme file for the last project at Udacity's Fullstack Web Developer Nanodegree Program.

It only conatins the information related on what ahs been done to make an app created on a previous project (you can find it on my [Github Repository](https://github.com/marccodera/Udacity_ND_FS-Project3_Backend))

## Amazon Lightsail IP address and SSH port

  - IP Address: 35.180.36.206
  - SSH port: 2200

## Complete URL for hosted application

* URL: http://catalog.codals.com/

## Software installed and configuration changes made

### SSH
#### SSH using Private Key
Downloaded Key pem file from Amazon Lightsail copied in ~/.ssh folder and applied permissions: 
```sh
chmod 0400 ~/.ssh/LightsailDefaultPrivateKey-eu-west-3.pem
ssh ubuntu@35.180.36.206 -i ~/.ssh/LightsailDefaultPrivateKey-eu-west-3.pem
```
#### SSH on different port
Added "TCP/2200" connections to Lightsail FW rules
Added "TCP/2200" port to /etc/ssh/sshd_config file: Port 2200
Restarted SSH service: sudo /etc/init.d/ssh restart

Tried connection to port:
```sh
ssh ubuntu@52.47.143.3 -i ~/.ssh/LightsailDefaultPrivateKey-eu-west-3.pem -p 2200
```
When connected, deleted line containing "Port 22" at /etc/ssh/sshd_config file.

Deleted port 22 connections at Amazon Lightsail Firewall

#### Disable root login on SSH
Changed PermitRootLogin property to /etc/ssh/sshd_config file
```sh
PermitRootLogin no 
```
### FW rules

#### Local Firewall (Uncomplicated Firewall)
To see if the UFW is started:
```sh
sudo ufw status
```
If the Uncomplicated Firewall is started, stop it because rules are going to be applied.
Apply new rules:
```sh
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
```
Check if all applied correctly and enable FW:
```sh
sudo ufw enable
```
To see FW rules to delete some, execute 
```sh
sudo ufw status numbered
```
To delete a Rule execute:
```sh
sudo ufw delete <number>
```
Be careful because after deleting a rule the numbers are changing!

### Grader user
Add grader user:
```sh
sudo adduser grader 
```
Make grader sudoer:
```sh
sudo touch /etc/sudoers.d/grader
sudo chmod 440 /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader
    grader ALL=(ALL) NOPASSWD:ALL
```
Loged in as grader user.

Key pair created for grader stored at /home/grader/.ssh/grader-key and /home/grader/.ssh/graderkey.pub

Permission for .ssh folder changed
```sh
/home/grader/
chmod 700 .ssh
```
Created file authorized_keys and copied there the content of grader-hey.pub
Permission for authorized_keys changed:
```sh
cd /home/grader/.ssh
chmod 644 authorized_keys 
```
To connect to the server, use private key:
```sh
ssh grader@35.180.36.206 -i ~/.ssh/grader-key -p 2200
```

### Install apache and Python support
```sh
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```
#### Configure Apache WSGI to accept requests
```sh
sudo nano /etc/apache2/sites-enabled/000-default.conf
```

For now, add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> line: 
```sh
WSGIScriptAlias / /var/www/html/Catalog/application.wsgi
```

#### Installing required Python modules
Esported a list of Python modules from the Catalog project Vagran VM to requirements.txt
Used python pip to export the list and to install them:
```sh
sudo pip install -r requirements.txt
```

### Clone Github repository to local folder
Cloned project repository to local folder
```ssh
git clone https://github.com/marccodera/Udacity_ND_FS-Project3_Backend.git /var/www/Catalog/Catalog
```
### Configure apache to use flask app
And followed instructions from [this site](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) to configure apache. Had to change "application.py" file name:
```sh
sudo mv application.py __init__.py
```
### Change code from the project python files
Some changes in the code has been made in order the application could work. Lines pointing to the database had been changed in "application.py" and "databasesetup.py" from`('sqlite:///catalog.db')` to `('sqlite:////var/www/Catalog/Catalog/catalog.db')` .
As for the json file lines in "application.py" from `open('client_secrets.json', 'r')` to `open('/var/www/Catalgo/Catalog/client_secrets.json', 'r')`.
More changes in "application.py" and "databasesetup.py", changed `engine = create_engine('sqlite:////var/www/Catalog/Catalog/catalog.db')` to `engine = create_engine('sqlite:////var/www/Catalog/Catalog/catalog.db', connect_args={'check_same_thread': False})`
All these changes have been made because apache was not able to locate the files before.
### Added server to codals.com DNS name
Created an A DNS record to codals.com DNS called catalog.codals.com pointing to the server IP address.
### Add new domain name to Google APIs credentials
Google authenticator didn't work, so had to add new domain name to Google API credentials part for the app.

## List of any third-party resources used to complete this project

A very useful post at Digital Ocean explaining how to configure a Flask App with Apache and Ubuntu server: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

To provide ssh access:
https://www.digitalocean.com/community/questions/ubuntu-16-04-creating-new-user-and-adding-ssh-keys
