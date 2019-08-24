# Linux Server Configuration

> A baseline installation of a Linux server and prepare it to host web applications, secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

* IP Address: 54.165.201.125
* Accessible SSH port: 2200
* Application URL: [http://54.165.201.125.xip.io/](http://54.165.201.125.xip.io/)

## Instruction to configure the server

1. **SSH into server**

```
$ ssh -i item-cat.pem ubuntu@54.165.201.125
```

2. **Update all packages**

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

3. **Change timezone to UTC**

```
$ sudo dpkg-reconfigure tzdata
```

4. **Create new user named grader**

```
$ sudo adduser grader
```

* To give grader sudo access

```
$ sudo nano /etc/sudoers.d/grader
```

* Add the following text to this file `grader ALL=(ALL:ALL) ALL`

5. **Setup SSH keys for grader**

```
$ sudo su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ sudo chmod 700 .ssh
$ sudo chmod 600 .ssh/authorized_keys
$ nano .ssh/authorized_keys
```

* Paste the content of Public key (aws_key.pub) created on local machine.

6. **Change SSH port from 22 to 2200 | Enforce key-based authentication | Disable login for root user**

```
$ sudo nano /etc/ssh/sshd_config
```

* Find the line `Port 22` and chnge it to `Port 2200`

* Find the `PasswordAuthentication` line and edit it to `PasswordAuthentication no`.

* Find the `PermitRootLogin` line and edit it to `PermitRootLogin no`.

* Save the file and run:

```
$ sudo service ssh restart
```

7. **To Configure the Uncomplicated Firewall (UFW) run following commands**

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow 123/udp
$ sudo ufw enable
```

8. **Install Apache**

```
$ sudo apt-get install apache2
```

9. **Install mod_wsgi | Enable mod_wsg | Start the web server**

```
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo a2enmod wsgi
$ sudo service apache2 start
```

10. **Clone the Item Catalog app from Github**

```
$ sudo apt-get install git
$ cd /var/www
$ sudo mkdir FlaskApp
$ sudo chown -R grader:grader FlaskApp
$ cd FlaskApp
$ git clone https://github.com/theashishmalik/item-catalog.git FlaskApp
```

11. **Create a flaskapp.wsgi**

```
$ cd /var/www/FlaskApp
sudo nano flaskapp.wsgi
```

* Add following lines

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```

12. **Rename `app.py` to `__init__.py`**

```
$ mv app.py __init__.py
```

13. **Install virtual environment**

```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
```

14. **Install Flask and other required dependencies**

```
$ sudo apt-get install python-pip
$ pip install Flask
$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
```

15. **Update path for `client_secrets.json` file in `__init__.py`**

* From `client_secrets.json` to `/var/www/FlaskApp/FlaskApp/client_secrets.json`

16. **Create Apache configuration file**

```
$ sudo nano /etc/apache2/sites-available/FlaskApp.conf
```

* Paste this code

```apache
<VirtualHost *:80>
		ServerName 54.165.201.125.xip.io
		ServerAdmin admin@aashish.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Enable the virtual host sudo `sudo a2ensite FlaskApp`

17. **Install and configure PostgreSQL**

```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres
$ psql
$ CREATE USER catalog WITH PASSWORD 'password';
$ ALTER USER catalog CREATEDB;
$ CREATE DATABASE catalog WITH OWNER catalog;
$ \c catalog
$ REVOKE ALL ON SCHEMA public FROM public;
$ GRANT ALL ON SCHEMA public TO catalog;
$ \q
$ exit
```

18. **Change create engine line in `__init__.py`, `database_setup.py`, `populate_database.py` to:** `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

19. **Run `database_setup.py` and `populate_database.py`**

```
$ python /var/www/FlaskApp/FlaskApp/database_setup.py
$ python /var/www/FlaskApp/FlaskApp/populate_database.py
```

  * **Restart Apache**

  ```
  $ sudo service apache2 restart
  ```

## Oauth Client Login

To setup it up. Update the path in the application.py program. Update the client id and oauth_flow

```
CLIENT_ID = json.loads(
    open('/var/www/FlaskApp/FlaskApp/client_secrets.json', 'r').read())['web']['client_id']

oauth_flow = flow_from_clientsecrets('/var/www/FlaskApp/FlaskApp//client_secrets.json', scope='')
```

### Now open your brouser and visit to [http://54.165.201.125.xip.io/](http://54.165.201.125.xip.io/)

## SCREENSHOTS

> ssh as grader :

![Create](https://github.com/theashishmalik/linux-server-configuration/blob/master/static/login.png)

> Application:

![Create](https://github.com/theashishmalik/linux-server-configuration/blob/master/static/host.png)

## References
[Flask document](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)

[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)


