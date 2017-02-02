# Memo: Configuration Details


### User setting

- Create a new user, `grader`

Run `adduser` command and input the password twice when prompted.

```bash
root# sudo adduser grader
...
...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for grader
Enter the new value, or press ENTER for the default
	Full Name []: Grader
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] 
```

- Give `grader` a permission to sudo

Create a new file `/etc/sudoers.d/grader` and add one line as in below:

```bash
root# sudo vim /etc/sudoers.d/grader
```
```
grader ALL=(ALL) ALL
```

- Make `sudo` ask password at some interval

Open `/etc/sudoers` using the command below and add `timestamp_timeout`.

```bash
root# sudo visudo
```
```
Defaults        env_reset,timestamp_timeout=5
```

By these changes, a user `grader` got a super user access.
From now, all setup are done by the `grader` instead of `root`.


- User change

```bash
# su - grader
```


### SSH setting

- SSH port change

Open the file `/etc/ssh/sshd_config` and change `Port` parameter to 2200

```bash
grader@ sudo vim /etc/ssh/sshd_config
```

On this Ubuntu, the setting is around line 5.

```
# What ports, IPs and protocols we listen for
Port 2200
```

- Disable password authentication

On the same file, `/etc/ssh/sshd_config`, change `PasswordAuthentication` to no.
On this Ubuntu, the setting is aroung line 50.


```
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
```


- Disable the remote login by a user `root`.

After disabling the `root` user login, at least, one user should have
ssh login access to the server. The steps below is to give `grader` ssh login
from a remote machine.

Create `.ssh/authorized_keys` file and copy&paste a public key
from `/root/.ssh/authorized_keys`.

```bash
grader@ vim ~/.ssh/authorized_keys
```
Add the public key:
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAA....(snip)...
...(snip)...
...(snip)...4pqklO8yan/z3dzOyy1dcWltD Udacity Student
```

Edit `/etc/ssh/sshd_config`

```bash
PermitRootLogin no
```


- Restart the ssh process

```bash
grader@ sudo service ssh restart
```


### Firewall setting

This will set the Uncomplicated Firewall (UFW) to allow SSH(port 2200),
HTTP (port 80), and NTP(port 123) incoming connections.

```
grader@ sudo ufw default deny incoming
grader@ sudo ufw default allow outgoing
grader@ sudo ufw allow 2200/tcp
grader@ sudo ufw allow www
grader@ sudo ufw allow ntp
grader@ sudo ufw enable
grader@ sudo ufw status

Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123                        ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123 (v6)                   ALLOW       Anywhere (v6)

```

### Timezone setting

This step is to change timezone to UTC if it is not. First check the current
timezone by `date` command.

```
grader@ date
Wed Feb  1 03:58:57 UTC 2017
```

In this case, the timezone is already set to UTC. If not, use the
command:

```bash
grader@ sudo dpkg-reconfigure tzdata
```

Choose `None of the above` by hitting a return key, then choose `UTC`.
Move the cursor by hitting a tab key, then choose `Ok`.

```bash
Current default time zone: 'Etc/UTC'
Local time is now:      Wed Feb  1 04:04:09 UTC 2017.
Universal Time is now:  Wed Feb  1 04:04:09 UTC 2017.
```

### Update packages

Before installing new software packages, update all packages already installed.

```bash
grader@ sudo apt-get update
grader@ sudo apt-get upgrade
```

### PostgreSQL

- PostgreSQL installation

```bash
grader@ sudo apt-get install postgresql
```

- Disallow remote connections

This is a default setting, probably, there's any to make this available.
The file, `/etc/postgresql/9.3/main/pg_hba.conf`, has entries as in below:

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```

- Create a user `catalog`

First, create a Unix domain user without home directory and nologin policy.
Then, create a user on PostgreSQL, no password

```bash
grader@ sudo adduser --no-create-home --shell /usr/sbin/nologin catalog 
grader@ sudo -u postgres createuser catalog
```

To see the user is added to PostgreSQL:

```sql
postgres=# select * from pg_user;
 usename  | usesysid | usecreatedb | usesuper | usecatupd | userepl |  passwd  | valuntil | useconfig 
----------+----------+-------------+----------+-----------+---------+----------+----------+-----------
 postgres |       10 | t           | t        | t         | t       | ******** |          | 
 catalog  |    16431 | f           | f        | f         | f       | ******** |          | 
(2 rows)
```

- Give all access to a database `catalog`

```bash
grader@ sudo -u postgres psql
psql (9.3.15)
Type "help" for help.

postgres=# create database catalog;
CREATE DATABASE

postgres=# grant all privileges on database catalog to catalog;
GRANT

postgres=# \list
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 catalog   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres+
           |          |          |             |             | catalog=CTc/postgres
 ...(snip)...
```

- Access database by a user `catalog`

Once `catalog` user and database are created, the database can be
access by:

```bash
grader@ sudo -u catalog psql
psql (9.3.15)
Type "help" for help.

catalog=> 
```

### Clone out Catalog app from Github repository

- Install git and setup

```bash
grader@ sudo apt-get install git
grader@ git config --global user.name "John Doe"
grader@ git config --global user.email johndoe@example.com
```

- Create a key-pair to set github repository

```bash
grader@ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/grader/.ssh/id_rsa): 
Created directory '/home/grader/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/grader/.ssh/id_rsa.
Your public key has been saved in /home/grader/.ssh/id_rsa.pub.
The key fingerprint is:
...(snip)...
...(snip)...
```

- Add a public key to Github account

After login to the Github account, go to `https://github.com/settings/keys`.
Click `New SSH key` and add public key (`.ssh/id_rsa.pub`).

- Clone the repo

```bash
grader@ git clone [catalog app github url]
```


### Catalog app setting

- Install Linux and Python packages

When the server is created, it doesn't have all packages to run the app.
The PostgreSQL has been installed at previous step already,
here're the rest of packages.


```bash
grader@ sudo apt-get install python-psycopg2
grader@ sudo apt-get install python-flask python-sqlalchemy
grader@ sudo apt-get install python-pip
grader@ sudo pip install Flask
grader@ sudo pip install Flask-Restless
grader@ pip install oauth2client
grader@ pip install httplib2
```

- Create Schema and Seed data

The cloned catalog app repo has schema definition and seed data.
Run below before running the app.

```bash
grader@ sudo -u catalog python database_setup.py
grader@ sudo -u catalog python seeds.py
```

- OAuth setting update

Go to Facebook Developer console and Google OAuth API website,
and add this server's URL. As for Google OAuth, download a json file
and replace it.


- Changes on main Python code

In this app, the main Python code to start Flask app was `project.py`.
When the app is hooked up from Apache server, it should be a Python module.
There are a couple ways to do this. Among those, renaming the file name to
`__init__.py` is the one taken here.

```bash
grader@ move project.py __init__.py
```

Additionally, relative file paths to the current doesn't work.
There are a couple of lines to make those to absolute paths.


On the local env, it wasn't the issue. However, the secret key
definition should be done outside of `__main__` definition.
So, the line was moved up to the line right next to the `app` definition.


Lastly, the application's port is defined on the Apache side,
Flask app doesn't set any.

```python
app.run()
```




### Apache HTTP server

- Apache HTTP server(`apache2` package) installation

```bash
grader@ sudo apt-get install apache2
```

When the installation finishes, Apache HTTP server starts running.


- Find the server URL

External IP address is the one used to ssh this instance.
However, the IP can be check inside the EC2 instance.
The document,
[Amazon EC2 Instance IP Addressing](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html)
explains how.
Make a request to the exactly the same url in the document.

```
grader@ curl http://169.254.169.254/latest/meta-data/public-ipv4
52.34.19.54
```

Then, hit the `host` command to know a fully qualified domain name of the IP.

```
grader@ host 52.34.19.54
54.19.34.52.in-addr.arpa domain name pointer ec2-52-34-19-54.us-west-2.compute.amazonaws.com.
```

Apache HTTP server is accessible by the URL from anywhere:
```
http://ec2-52-34-19-54.us-west-2.compute.amazonaws.com/
```


- Lynx (Command line browser)

The command, `curl localhost`, works to check the website on the remote server. 
However, `curl` output is not human friendly. For this reason,
install `lynx`.

```
grader@ sudo apt-get install lynx
```

The command, `lynx localhost`, shows the website much better.


### Apache to Flask app setting

- Install WSGI package

Reference: [mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)

```bash
grader@ sudo apt-get install libapache2-mod-wsgi
```

- Create app home directory

```bash
grader@ sudo mkdir /var/www/catalog
```

- Create wsgi file to hook up the app

The file name can be any. The name is referenced from Apache configuration.
In this case, the filename is `catalog.wsgi`.
The catalog app's `__init__.py` file exists in the directory,
`/home/grader/serene-cliffs/vagrant`.


```python
import sys
sys.path.insert(0, '/home/grader/serene-cliffs/vagrant')
from catalog import app as application
```

- Apache site definition

The file to edit is `/etc/apache2/sites-enabled/000-default.conf`, which
looks like below to run the app.

What need to be changed were:

1. ServerName
2. DocumentRoot
3. WSGIDaemonProcess
4. WSGIScriptAlias
5. Directory block

The user catalog is the access to the catalog database on PostgreSQL.
This way, the access to database catalog from Apache is permitted.

```xml
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com
	ServerName 127.0.0.1

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/catalog

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
	WSGIDaemonProcess catalog user=catalog group=catalog home=/var/www/catalog
	WSGIScriptAlias / /var/www/catalog/catalog.wsgi

        <Directory /var/www/catalog>
            WSGIProcessGroup catalog
            WSGIApplicationGroup %{GLOBAL}
            Require all granted
        </Directory>
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

- Changes Apache process user/group

Lastly, the Apache process should run as the user `catalog`'s process.
By default, the user is www-data. To change the user,
the file `/etc/apache2/envvars` should be changed accordingly.

```bash
#export APACHE_RUN_USER=www-data
export APACHE_RUN_USER=catalog
#export APACHE_RUN_GROUP=www-data
export APACHE_RUN_GROUP=catalog
```

- Restart Apache

Once all setups are done, restart the Apache process:

```bash
grader@ sudo service apache2 restart
```


# Useful Resources

- (book)[Linux in a Nutshell](https://www.amazon.com/Linux-Nutshell-Ellen-Siever/dp/0596154488)
- (blog) [nixCraft](https://www.cyberciti.biz/)
- (article) [77 Linux commands and utilities you'll actually use](http://searchenterpriselinux.techtarget.com/tutorial/77-useful-Linux-commands-and-utilities)
- (doc) [An A-Z index of the Bash command line for Linux](http://ss64.com/bash/)

# Useful Tools to be considered

- `fail2ban`: monitoring and IP panning tool, detects malicious signs
    - [official site](http://www.fail2ban.org/wiki/index.php/Main_Page),
    - [tutorial](https://www.digitalocean.com/community/tutorials/how-to-protect-an-apache-server-with-fail2ban-on-ubuntu-14-04)
- `Glances`: System monitoring tool, web based interface available
    - [web site](https://pypi.python.org/pypi/Glances)