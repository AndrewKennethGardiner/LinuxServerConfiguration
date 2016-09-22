# Readme File

**Project** :Linux Server Configuration

**Version** :1.0

**Release** :2016

**Python:**        Version 2.7

**Pricing**:       Free to All!

**Developed by**:        Andrew Gardiner

# Background:

This program has been written in response to the Udacity – Full Stack Web Developer Nanodegree – Project: Linux Server Configuration. Specifically, the web application developed in Project 3: Item Catalog, is to be deployed. A virtual server supplied by Udacity and Amazon in Amazon's Elastic Compute Cloud (EC2) has been provided for use in this project.

# Basic Stuff 

1.  IP Address: 52.43.201.174
2.  URL:        ec2-52-43-201-174.us-west-2.compute.amazonaws.com

# Perform Basic Configuration

**1. Launch the Virtual Machine**

    1.  Create new development environment
    2.  Downloand the Private Key (Udacity)
    3.  Move the private key file into the folder ~/.ssh 
        (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
        $ mv ~/Downloads/udacity_key.rsa ~/.ssh/
    4.  Open Git Bash as your terminal and type in
        $ chmod 600 ~/.ssh/udacity_key.rsa
    5.  SSH (connect) to the instance:
        $ ssh -i ~/.ssh/udacity_key.rsa root@52.43.201.174


**2. Create a new User named 'grader' and grant 'grader' sudo permissions**
    (Udacity)

    1.  $ sudo adduser grader
    2.  Orinially looked at $ usermod -a -G sudo grader
    3.  Better approach:
        a.  Create a new file under the sudoers directory
            $ sudo nano /etc/sudoers.d/grader
        b.  Place the following command within that file
            "grader ALL=(ALL:ALL) ALL"
        c.  Save the file
    4.  To check keep the current command box open (with root)
    5.  Use a new command box to check grader works
    6.  $ ssh grader@52.43.201.174 -p 22 (note this is port 22 for time being)
    7.  Starting and stopping SSH (just in case)
        a.  $ stop ssh
        b.  $ start ssh

    (Thank you iliketomatoes! - see acknowledgements)
    There was an issue with "sudo:unable to resolve host"
    4.  $ sudo nano /etc/hosts
    5.  add the host 127.0.1.1 ip-10-20-0-90 (whatever comes up on the prompt)

**3. Update all currently installed packages**
    (Udacity)

    1.  $ sudo apt-get update
    2.  $ sudo apt-get upgrade
    3.  When asked I installed the package maintainers version
    4.  I also installed finger - to look at users
    5.  $ sudo apt-get install finger

**4. Configure the local timezone to UTC**
    
    1.  $ sudo dpkg-reconfigure tzdata
    2.  Note - need to choose other and then scroll to bottom to get UTC

# Secure your Server

**5. Change the SSH port from 22 to 2200**
(Udacity and http://linuxlookup.com/howto/change_default_ssh_port)
    
    1.  The file that needs to be amended is: 
    2.  $ sudo nano /etc/ssh/sshd_config
    3.  Change the port from 22 (was our default) to 2200
    4.  NEED TO BE VERY CAREFUL HERE AS CAN LOCK SELF OUT IN THESE STEPS
    5.  $ sudo restart ssh
    6.  New login is (Bash Terminal):
    7.  $ ssh grader@52.43.201.174 -p 2200
    8.  We can use new login at moment as we have not set firewall.
    9.  Firewall must allow 2200 before we log out or we are in trouble.
    10. Useful: to show what ports are openand being used
    11. $ sudo netstat - peanut

**6. Configure the Uncomplicated Firewall (UFW)**
(Udacity)
    
    1.  To find out where the firewall currently stands:
    2.  $ sudo ufw status
    3.  Block general incoming - hence why caution is needed.
    4.  $ sudo ufw default deny incoming
    5.  Allo general outgoing.
    6.  $ sudo ufw default allow outgoing
    7.  Set out SSH port. 
    8.  Note just $sudo ufw allow ssh works but sets port 22 as the default.
    9.  We want port 2200 for ssh. **Has to match config file set above.**
    10. $ sudo ufw allow 2200/tcp
    11. Set the web port. As default is 80 anyway.
    12. $ sudo ufw allow www
    13. Set the ntp. Again default is 123
    14. $ sudo ufw allow 123/udp
    15. Would suggest that there be 2 bash terminals logged in if possible.
    16. Any mistake may be able to be rectified.
    17. Otherwise likely to have to start again.
    18. $ sudo ufw enable

**7. Autmatically manage updates**
(Udacity and iliketomatoes)
    
    1.  Install the package   
    2.  $ sudo apt-get install unattended-upgrades
    3.  Enable
    4.  $ sudo dpkg-reconfigure --priority=low unattended-upgrades

# Install the Catalog Application

**8. Install git & clone**
(https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)
    Note: While I tried to follow the order of the project outline this is, with respect, a better order.
    By using git and cloning at this stage the directory structure is known.

    1.  Install git.
    2.  $ sudo apt-get install git
    3.  Configure your username: 
    4.  $ git config --global user.name <username>
    5.  Configure your email: 
    6.  $ git config --global user.email <email>
    7.  Now we need to clone from github
    8.  $ cd /var/www
    9.  $ sudo mkdir catalog
    10. Change the owner of the catalog directory
    11. $ sudo chown -R grader:grader catalog
    12. Move to catalog directory
    13. $ cd /catalog
    14. Clone the catalog repository
    15. $ sudo git clone https://github.com/AndrewKennethGardiner/CatalogApp.git catalog
    16. We now need a wsgi file. I put it in the /var/www/catalog/catalog directory
    17. $ cd catalog
    18. Create a catalog.wsgi file as follows:
    19. $ sudo nano catalog.wsgi
    20. That file should contain:
        
        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/catalog/")

        from catalog import app as application
        application.secret_key = 'super_secret_key'

**9. Install Flask**

    1.  Install the pip installer
    2.  $ sudo apt-get install python-pip
    3.  Install the virual environment
    4.  $ sudo pip install virtualenv
    5.  Give the virtual environment a name venv
    6.  $ sudo virtualenv venv
    7.  Enable permissions
    8.  $ sudo chmod -R 777 venv
    9.  Activate the virtual environment
    10. $ source venv/bin/activate 
    11. Install Flask inside the virtual environment
    12. $ sudo pip install Flask
    13. To deactivate the environment, give the following command:
    14. $ sudo deactivate

**10. Install and configure Apache to serve a Python mod_wsgi application**
(IlikeTomatoes)    
    
    1.  $ sudo apt-get install apache2
    2.  Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install as follows:
    3.  $ sudo apt-get install libapache2-mod-wsgi python-dev python-setuptools
    4.  Enable
    5.  $ sudo a2enmod wsgi
    6.  Then start Apache:
    7.  $ sudo service apache2 start
    8.  Note restarting and/or reloading apache recommended as things may get stuck while you are working it out.
    9.  Confirm it is working by going to the ip address given to us:
    10. 52.43.201.174 (In this case.)
    11. Should see Apache2 Default Page

Now do some nifty configurations. 

    12. You then need to configure Apache to handle requests using the WSGI module.
    13. Be careful as when going through some of the sites and documents the examples play around with editing: /etc/apache2/sites-enabled/000-default.conf (Do not edit this file)
    15. Instead create a virtual host config file of our own:
    16. sudo nano /etc/apache2/sites-enabled/catalog.conf

        <VirtualHost *:80>
            ServerName 52.43.201.174
            ServerAdmin admin@52.43.201.174
            ServerAlias ec2-52-43-201-174.us-west-2.compute.amazonaws.com
            WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
            WSGIProcessGroup catalog
            WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
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

    17. Be very careful with this file. Note the python version and that the directories for our web site are set. Make sure the path to our newly created catalog.wsgi is correctly set!
    18. The ServerAlias was to sensure the OAuth stuff worked with out virtual environment.
    19. Enable the virtual host.
    20. $ sudo a2ensite catalog

**11. Install and configure PostgreSQL**

    1.  Install PostgreSQL
    2.  $ sudo apt-get install postgresql
    2a. If issues then you may need more functionality
    2b. $ sudo apt-get install postgresql-contrib
    3.  Do not allow remote connections. Change the pb_hba.conf file to only allow local.
    4.  & sudo nano /var/postgresql/9.3/main/pb_hba.conf

        local   all             postgres                                peer
        local   all             all                                     peer
        host    all             all             127.0.0.1/32            md5
        host    all             all             ::1/128                 md5

    5.  Postgres creates a new user during installation called 'postgres'. Connect to that user.
    6.  $ sudo su - postgres
    7.  Connect to the database
    8.  $ psql
    9.  Create a new user named catalog that has limited permissions.
    10. # CREATE USER catalog WITH PASSWORD 'catlog_password'
    11. User catlog needs to create a database.
    12. # ALTER USER catalog CREATEDB
    13. Connect to the Database
    14. # \c catalog
    15. Revoke rights
    16. # REVOKE ALL ON SCHEMA public FROM public
    17. Limit permissions to only our user catalog.
    18. # GRANT ALL ON SCHEMA public TO catalog
    19. Log out and return
    20. # \q
    21. $ exit

**12. Getting our Catalog App up and running**

**Changing to PostgreSQL and Installing Modules**

    1.  The catalog application from the 3rd assignment was what was cloned into a catalog folder: /var/www/catalog/catalog/.
    2.  That catalog application:
        a.  imported modules that it relied on; and
        b.  used sqlite
    3.  The import modules in the python files need to be installed along with some necessary python packages:
    4.  $ sudo pip install httplib2 requests oauth2client sqlalchemy python-psycopg2
    5.  If problems - check the log file (see below) as may well be missing modules that need to be installed.
    6.  Change the Catalog Application from sqlite to postgreSQL **VERY IMPORTANT**
    7.  The .py files used in my Catalog application were:
        a.  application.py
        b.  database_setup.py; and
        c.  populate_database.py
    8. Each of these files had the following line:
        engine = create_engine('sqlite:///catalog.db')
    9. Each of these files needed to change the above with:
         engine = create_engine('postgresql://catalog:MyPasswordGoesHere@localhost/catalog')
    10. To edit, say, application.py
    11. $sudo nano /var/www/catalog/catalog/application.py
 
**Setting up the databases and populating them**

    12. The application.py file runs the application. It does not set up the databases each time it runs.
    13. Run the database_setup file.
    14. $ python database_setup.py
    15. I also populated the file.
    16. $ python populate_database.py

**Getting the Oauth logins working**
(stueken - well well done)
    
    17. Get the host name for our public IP
    18.  http://www.hcidata.info/host2ip.cgi
    19. This, in this instance was:
        ServerAlias ec2-52-43-201-174.us-west-2.compute.amazonaws.com
    20. That line was the line in the catalog.conf file created earlier. Was a little backward and forward here.
    21. Enable the virtual host again if you change the file:
    22. $ sudo a2ensite catalog

    23. To get the Google+ authorization working:
    24. Go to the project on the Developer Console: https://console.developers.google.com/project
    25. Navigate to APIs & auth > Credentials > Edit Settings
    26. add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-52-43-201-174.us-west-2.compute.amazonaws.com/oauth2callback

    27. To get the Facebook authorization working:
    28. Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
    29. Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
    30. To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'

# General Error Checking

    1.  The following command helps with general error checking. It shows the last few entries of the log file.
    2.  $ sudo tail -20 /var/log/apache2/error.log
    3.  Also the reload of apache often brings up errors:
    4.  $ sudo service apache2 reload

# Acknowledgements

Udacity - I relied on the course a lot. Where it failed me was really the implementation from sqlite to postgresql and what changes I had to make to my files and the configuration files.
iliketomatoes
https://github.com/iliketomatoes/linux_server_configuration/blob/master/README.md
stueken
https://github.com/stueken/FSND-P5_Linux-Server-Configuration
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://discussions.udacity.com/t/running-apache-problem/35719/6
Stackoverflow!
AskUbuntu!
and finally for .md preview http://dillinger.io/