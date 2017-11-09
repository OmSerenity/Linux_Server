# Launching Northern Nevada Desert Plants website on a Linux Server

# About

This project details the process of launching a previous Item Catalog project ("Northern Nevada Desert Plants") onto a Linux server.  I will be launching the Item Catalog project I previously made, which is a Python Flask app.  It's called: Northern Nevada Desert Plants.  First I will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host my web application.  I will secure it from a number of attack vectors, and installing/configuring a database server, and deploy my existing Flask application onto it.  

This is what the Northern Nevada Desert Plants website is about:

Plants which grow and thrive in the desert climate of Northern Nevada are cataloged here.  This application provides a list of items, in this case, plants, within a variety of categories as well as a user registration and authentication system. Registered users will have the ability to post, edit and delete their own items.

# Inner Workings of this Project
  - Linux Server (as part of an Amazon Lightsail instance)
  - Amazon Lightsail 
  - Python
  - Flask
  - SQLAlchemy
  - OAuth
  - Google Login
  - Facebook Login
  - HTML5
  - CSS3
  - Bootstrap

# You will need these to run the program:

  - Gitbash (PC) or command line (Mac)
  - grader's private key and password
  - patience (just kiddling!)
  
# How to run the Linux Server project

# Linux Server Information
## Public IP address: 52.26.231.94
## Accessible SSH port: 2200
## View the project live at: http://52.26.231.94

# How to get the journey going:
##1. Start a new Ubuntu Linux server instance on Amazon Lightsail.

   - Log in and create an instance
   - Choose Ubuntu (OS only) as instance image.
   - Select an instance plan (basic is okay)
   - Name your instance, if desired, or stick with the default name
   - Create a static IP, if desired, and attach it to the instance you created.  The advantage is that if you shut down the instance and have to restart it, you retain the same IP address and don't have to change it again in all your files.  Otherwise, by default, instances are assigned dynamic IP addresses, which Lightsail warns, will change if you shut down the instance.  

##2. After instance starts up, SSH into your server on the web from Gitbash or your command line terminal on your local machine.  Note that it is possible to click "Connect using SSH" on your Lightsail instance webpage and this will take you to your cloud-based Lightsail terminal.  However, it can only be used with Port 22, which is limiting, and we are using Port 2200 for this project.  Also, Port 22 is a common target for hackers, so that is why we are using a different port as well.  
```
ssh ubuntu@52.26.231.94
```
When asked: Are you sure you want to continue connecting (yes/no)?, type: yes
You will get an alarming message saying: Warning: Permanently added '52.26.231.94' (ECDSA) to the list of known hosts.
Permission denied (publickey).
Don't worry, you can still go on from here. 

Test that you can log back into the server with (and know you can log back in with this command after you log out with):
```
ssh ubuntu@52.26.231.94 -i ~/desktop/SSLinuxServer/Key1.pem 
```
After -i in the above command, you list the path to where you stored the private key you downloaded from Amazon Lightsail under Accounts/SSH keys.  In this case, I used the default keys and it is stored in a .pem file, which I named Key1.pem)

I also copied and pasted my public IP address into Authorized JavaScript origins in my Google OAuth for my Client ID for Web Applications: http://52.26.231.94.  See Google Developers APIs site: https://console.developers.google.com  

Some extra steps for the Google Oauth above (may not have been necessary):

- I found the DNS address for my IP address on the internet (search IP address to DNS) and pasted that in as well: http://ec2-52-26-231-94.us-west-2.compute.amazonaws.com 
- I also put in Authorized redirect URIs: http://ec2-52-26-231-94.us-west-2.compute.amazonaws.com/gconnect, http://ec2-52-26-231-94.us-west-2.compute.amazonaws.com/login, and http://ec2-52-26-231-94.us-west-2.compute.amazonaws.com/oauth2callback

##3. Now, secure your server to automatically list, update all currently installed packages, get further updates, install finger to check users' status, and automatically remove no longer required packages.
```
sudo apt-get update
sudo apt-get upgrade    
sudo apt-get dist-upgrade
sudo apt-get install finger
sudo apt autoremove
```
Select "yes" after upgrade, then I pressed enter to select the package maintainer's version and got the message
"Now your virtual machine's packages are all installed and updated.""

I also got this message "The following package was automatically installed and is no longer required:
  snap-confine
Use 'sudo apt autoremove' to remove it."

 I used that command and  it was removed.

##4. Add a new user called *grader* and give grader sudo permissions:
 
```
 sudo adduser graderd
 ```
  Instead of editing the /etc/sudouser file, use this command instead, which may work better with Lightsail: 
```
sudo usermod -a -G sudo grader
```

##5. Set up key-based authentification for grader user:
On Gitbash terminal (PC) or command line on your LOCAL MACHINE, generate an encryption key:
```
ssh-keygen -f ~/.ssh/grader.rsa
```
Then, using another Gitbash or command line terminal, log into Lightsail's remote virtual machine as ubuntu user through ssh using 
```
ssh ubuntu@52.26.231.94
```
and create a new authorized keys file:
```
touch ~/.ssh/authorized_keys
```
Using your Windows File Explorer or Finder on Mac, find the grader.pub file and copy its contents from your local machine to the remote VM using:
```
sudo nano ~/.ssh/authorized_keys
```
Use nano commands "Ctrl-O", then press "Enter" to save, then "Ctrl-X" to exit the file.  

**Note where you found the grader.pub file because that you will need to give the (Udacity) reviewer/grader the private key that was generated/paired with this public key.  In my machine, it was under C:/Users/(what my computer's user was names)/.ssh 

Then, change the permissions and change the owner from *root* to *grader*:
```
sudo chmod 700 ~/.ssh
sudo chmod 644 ~/.ssh/authorized_keys
sudo chown -R grader:grader ~/.ssh
```
Important: from now on, we can log into the remote VM through ssh with the following command (this logs us in on the default port 22 but after we change the port in #7 below, we will add the new port # to this command)
``` 
ssh -i ~/.ssh/grader.rsa grader@52.26.231.94
```

##6.  Remove password access to server and change to key-based authentication instead:
```
sudo nano /etc/ssh/sshd_config    (In this file, edit *PasswordAuthentication* line to *no*)
sudo service ssh restart
```

##7. Change SSH port from 22 to 2200 and Configure Lightsail firewall settings to allow this change. 

In Lightsail, first go under your instance: Networking/Firewall and add TCP 2200, and TCP 80 and UDP 123.  You may have to select "custom" under Application and then indicate TCP or UDP.  It is important to add these additional ports since we'll be using port 2200 for this project.  
```
sudo nano /etc/ssh/sshd_config  (Edit the port line to 2200)
sudo service ssh restart
```
**Important:  After this, you can always log into the Lightsail VM from Gitbash or your command line (instead of using the initial ssh command we used in step 2) with: ssh -i ~/.ssh/grader.rsa -p 2200 grader@52.26.231.94 

##8.  Disable ssh login for root user
```
sudo nano /etc/ssh/sshd_config (Find the PermitRootLogin line and edit to "no")
sudo service ssh restart    (This will deny the access of root user)
```

##9. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200/tcp), HTTP (port 80/tcp), and NTP (port 123/udp).
```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
```
* Make sure when changing the SSH port that the Lightsail firewall is open for port 2200 first(following the directions in Step 7 above), so you won't get locked out of the server.*

##10. Install Apache, install mod_wsgi, and restart the server
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
sudo service apache2 start
```
See mod_wsgi documentation and pay careful attention to all of it through the end: http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/

##11. Install and Configure Git with Username and Email
```
sudo apt-get install git
git config --global user.name your_username
git config --global user.email your_email
```

##12. Clone your Catalog App from Github, change owner for Catalog folder and create catalog.wsgi file
```
cd /var/www
sudo mkdir catalog
sudo chown -R grader:grader catalog
cd /catalog
git clone https://github.com/OmSerenity/Item-Catalog.git catalog
```
Create a *catalog.wsgi* file to serve your application with *mod_wsgi* using: `touch catalog.wsgi` and then `sudo nano catalog.wsgi`
Paste this into the file (see mod_wsgi documentation mentioned in #10)

```
#!/usr/bin/python
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
Note:  If you make changes to your files in Github, you can do: `$ git pull (catalog file name)` to pull your changes to the remote server's VM.  Alternatively, you can choose to edit the files locally on the remote server, to customize it for your app, by doing `$ sudo nano file_you_want_to_edit` and making changes there.  This is faster and more immediate but changes won't be reflected in your Github.

##13. Install Pip, Virtual Environment, Flask, and Project Dependencies, also change permissions
```
sudo apt-get install python-pip
sudo pip install virtualenv
cd /var/www/catalog
sudo virtualenv venv
source venv/bin/activate
sudo chmod -R 777 venv
pip install Flask
pip install bleach httplib2 request oauth2client sqlalchemy psycopg2
``` 
To check the virtual host file is enabled:
```
ls -alh /etc/apache2/sites-enabled/
```

##14. Create, Configure, and Enable a new Virtual Host
```
sudo nano /etc/apache2/sites-available/catalog.conf
```
See the mod_wsgi(Apache)-Flask documentation for advice on what to include in this file:
```
<VirtualHost *:80>
    ServerName  (such as example.com (this is unnecessary in this project as we only have one server))
    ServerAlias (also unnecessary)
    ServerAdmin (an email address: also unnecessary)
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
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
Be careful with with your typing, spaces are important.  The mod_wsgi documentation also mentions that it is very important to have the absolute path to where your virtual environment is located.  Note that Windows systems run without the WSGI Daemon Process (see documentation).

 Enable the new virtual host:
```
 sudo a2ensite catalog
```

##15. Install and configure PostgreSQL, change user to catalog, create catalog databse owned by catalog user, connect to database
```
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
```
 Postgres automatically creates new user during its installation, whose name is 'postgres', user who can access the database software.
 Change the user with: `$ sudo su - postgres`, then connect to the database system with `$ psql`.

 Create a new user called 'catalog' with this password: `# CREATE USER catalog WITH PASSWORD 'postgres';`.
 Give *catalog* user the CREATEDB capability: `# ALTER USER catalog CREATEDB;`.
Create the 'catalog' database owned by *catalog* user: `# CREATE DATABASE catalog WITH OWNER catalog;`.

 Connect to the database: `# \c catalog`.
 Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`.
Lock down the permissions to only let *catalog* role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`.
 Log out from PostgreSQL: `# \q`. Then return to the *grader* user: `$ exit`.

 Inside the Flask application, the database connection is now performed with: 
```python
engine = create_engine('postgresql://catalog:postgres@localhost/catalog')
```
Setup the database with: `$ python /var/www/catalog/catalog/generate_dbinfo.py` or cd into the directory where your generate_dbinfo.py file is and use this command `sudo python generate_dbinfo.py`

To prevent potential attacks, double check that no remote connections to the database are allowed. Open file: `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf` and edit to make it look like this: 
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

##16. Install system monitor tools
```
sudo apt-get update
sudo apt-get install glances
glances (this starts the system monitor tool)
glances -h  (more about this program's options)
```

##17. Restart Apache to launch the app
```
sudo service apache2 restart
```

##18. Enjoy a working app on a publicly viewable server!

## Troubleshooting Tips:
1.  To find errors to aid in troubleshooting:
```
sudo tail /var/log/apache2/error.log
```
Errors I ran into and had to fix:  missing modules, absolute paths not written accurately or missing in files, virtual environment not activated.

To remove a directory (if you have one you installed by accident)
https://askubuntu.com/questions/217893/how-to-delete-a-non-empty-directory-in-terminal

## More useful tips:

Make sure you have edited your client_secrets.json file to include your IP address, and that you have included the absolute paths to your client_secrets.json file in generate_databaseinfo.py, also that the path to your virtual environment is the absolute path in the /var/www/catalog/catalog.wsgi file.  If you have problems with your pictures showing up, look in your Static folder and make sure your .jpg files are lower case (NOT .JPG).  This is because UNIX is sensitive to upper and lower case and .JPG won't be recognized.  

After you fix your errors, run:
```
sudo service apache2 restart
```
and refresh your browser set at your IP address to see if your project is now working. 

##  Issues/Known Bugs:
None.   
_If you find any, please let me know :)_

##  Frequently Asked Questions:

1.   How can I log in?
*After you've pasted your developer info / client IDs for Google in the login.html file, and run the program in your browser, click on the Google login icon and you should be able to login*

 2.  What can I do after I log in?
 *After you are authorized as a user, you will be able to create, add, edit, or delete the different categories and entries.*

#### References

DigitalOcean: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps

https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/

https://discussions.udacity.com/t/problems-with-the-digital-ocean-tutorial/336376/2

http://docs.sqlalchemy.org/en/latest/core/engines.html

https://en.wikipedia.org/wiki/Path_(computing)

http://www.ehowstuff.com/how-to-install-and-use-glances-system-monitor-in-ubuntu/

#### Attributions

Plant info from: https://www.unce.unr.edu/publications/files/ho/2007/sp0712.pdf, http://www.onlinenevada.org/articles/nevada-vegetation-overview, and https://www.desertusa.com/

Many thanks to Udacity mentors Steve Wooding and Trish Whetzel for tips, advice, and troubleshooting assistance.

License
---- 
MIT - open source!
*Please see the License.txt file for details.*

