# Linux Server Configuration Project

This README explains how to access a remote Ubuntu server and
how the server was setup.


## Remote Server

- IP address: 52.34.19.54
- SSH port: 2200
- Web Application: http://ec2-52-34-19-54.us-west-2.compute.amazonaws.com/


## Summaries

### Installed Software

- PostgreSQL
- Apache HTTP Server
- mod_wsgi
- git
- Python related
    - (Linux package) python-psycopg2
    - (Linux package) python-flask
    - (Linux package) python-sqlalchemy
    - (Linux package) python-pip
    - (Python package) Flask-Restless
    - (Python package) oauth2client
    - (Python package) httplib2
  
### Configuration Change

#### Making a new user grader sudoer

- A new file, `/etc/sudoers.d/grader`
- Changed password requesting interval in `/etc/sudoers`

#### Making SSH secure

All changes are done in the file `/etc/ssh/sshd_config`

- Port change: `Port 2200`
- Disabling password authentication: `PasswordAuthentication no`
- Disabling root login: `PermitRootLogin no`

#### Firewall setting

Incoming connections are limited to:

- SSH (Port 2200)
- HTTP (Port 80),
- NTP (Port 123)

#### PostgreSQL setting

By default, a remote access is disabled. No change was made on
PostgreSQL itself.


Other than that, a database access user, `catalog` was created.
The user `catalog` has all permissions to the database `catalog`.
The user is also an Apache process owner.

#### Flask app

The app was cloned out from Github repository. It resides
in `/home/grader/serene-cliffs/vagrant/catalog` directory.


There were a few relative paths references in flask app. Those
were replaced by absolute paths.

Explicit port number setting was removed, and it is now:

```python
app.run()
```


OAuth settings at Facebook and Google were updated to allow
this server as the redirect endpoint.

#### Apache HTTP server setting

- Home directory: `/var/www/catalog`
- WSGI script: `/var/www/catalog/catalog.wsgi`
- Apache configuration: `/etc/apache2/sites-enabled/000-default.conf`
- (edited) Apache Envvars: `/etc/apache2/envvars`


## References

Multiple sources from:
- [ask ubuntu](http://askubuntu.com/)
- [DigitalOcean](https://www.digitalocean.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/9.3/static/index.html)
- Stackoverflow questions

