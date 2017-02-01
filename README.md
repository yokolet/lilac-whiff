# Linux Server Configuration Project

This README explains how to access a remote Ubuntu server and
how the server was setup.


## Remote Server

- IP address: 52.34.19.54
- SSH port: 2200
- Web Application: http://ec2-52-34-19-54.us-west-2.compute.amazonaws.com/


## Configurations

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

```bash
grader@ sudo -u postgres createuser -P catalog
Enter password for new role: 
Enter it again: 
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
```

- Access database by a user `catalog`

Once `catalog` user and database are created, the database can be
access by:

```bash
grader@ psql -U catalog -d catalog -h localhost
Password for user catalog: 
psql (9.3.15)
SSL connection (cipher: DHE-RSA-AES256-GCM-SHA384, bits: 256)
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
.....
.....
```

- Add a public key to Github account

After login to the Github account, go to `https://github.com/settings/keys`.
Click `New SSH key` and add public key (`.ssh/id_rsa.pub`).

- Clone the repo

```bash
grader@ git clone [catalog app github url]
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



