
# udacity-linux-server-configuration

### Description
Deploying Flask wep-app on linux server.

- IP address: 3.8.78.162
- Accessible SSH port: 2200


### Packages and Software installed
- Flask==1.0.2
- Flask-Script==2.0.6
- Flask-Bootstrap==3.3.7.1
- Flask-Moment==0.6.0
- Flask-WTF==0.14.2
- FLASK-SQLAlchemy==2.3.2
- Flask-Migrate==2.3.1
- Flask-Login==0.4.1
- requests-oauthlib==1.0.0
- pyOpenSSL==18.0.0
- psycopg2
- apache2
- posgres




### Methods used

1. Create new user named grader and give it the permission to sudo
  - login to the server with ssh ` ssh -i ~/.ssh/pkey.pem ubuntu@3.8.78.162`
  - Run `$ sudo adduser grader` to create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add this`grader ALL=(ALL:ALL) NOPASSWD:ALL`
  - Run `sudo nano /etc/hosts`

2. Update all currently installed packages
  - Download package lists with `sudo apt-get update`
  - Fetch new versions of packages with `sudo apt-get upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - then run `sudo service ssh restart`
  - Confirm it

4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`

5. Change local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata`  and then none of the above and will show UTC now choose it

6. Configure key-based authentication for grader user
  - Run this command ` sudo mkdir /home/grader/.ssh`
then run this
` sudo nano /home/grader/.ssh/authorized_keys`
and paste your public key
7. Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Restart ssh with `sudo service ssh restart`
  - Now you are only able to login using `ssh -i ~/.ssh/udacity -p 2200 grader@3.8.78.162`

8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi-py3`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`


10. Clone from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd /catalog`
  - Clone your project from github `git clone https://github.com/SkySail07/item-catalog catalog`
  - Rename the run.py file to __init__.py using the following command sudo mv run.py __init__.py
  - Create a catalog.wsgi file, with this content
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from __init__ import app as application
  ```

11. Install virtual environment
  - First install pip with this command
`sudo apt-get install python3-pip`

13. Install Flask and other dependencies
  - Install Flask `pip3 install Flask`
  - Install all others project dependencies `sudo pip install -r requiremtns.txt`

14. Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`

15. Configure a new virtual host
  - Create a new file with this : `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Put this code:
  ```
  <VirtualHost *:80>
    ServerName 3.8.78.162
    ServerAlias ec2-3-8-78-162.eu-west-2.compute.amazonaws.com
    ServerAdmin admin@3.8.78.162
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog_items/static
    <Directory /var/www/catalog/catalog_items/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  - Enable the virtual host `sudo a2ensite catalog`

16. Install and setup PostgreSQL
  - `sudo apt-get install postgresql`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
  - now you need to change the sqlite in `config.py` and to:
  `SQLALCHEMY_DATABASE_URI = "postgresql://catalog:password@localhost/catalog"`
  - export DATABASE_URL="postgresql://catalog:password@localhost/catalog"

17. Restart Apache
  - `sudo service apache2 restart`

18. Visit site at [http://3.8.78.162/]

### Resources used to accomplish this project
1. **helped me a lot *[digitalocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)***
2. **Great blog *[theodo-blog](https://blog.theodo.fr/2017/03/developping-a-flask-web-app-with-a-postresql-database-making-all-the-possible-errors/)***
3. **Helpful Documentations *[flask Docs](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)***

