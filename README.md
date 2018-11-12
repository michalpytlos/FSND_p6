# Linux Server Configuration
Project 6 of the Full Stack Nanodegree <br>
Michal Pytlos, November 2018

## 1. Access details

* IP address of the server: `18.185.86.131`
* SSH port: `2200`
* URL to the hosted application: `bgc.18.185.86.131.xip.io`

## 2. Installed software and configuration changes made

1. Disable SSH login for the root user:
  * set the value of `PemitRootLogin` in **/etc/ssh/sshd_config** to `no`
2. Update all system packages to the most recent versions:
  * resynchronize the package index files from their sources: `apt-get update`
  * install the newest versions of all packages currently installed on the system: `apt-get upgrade`
3. Install python 2.7 and pip:
  * `apt-get install python python-pip`
4. Allow connections on ports 2200 and 123 (Lightsail firewall):
  * Add firewall rules for ports 2200 and 123 using the Lightsail management console
5. Change SSH port to 2200:
  * set the value of `Port` in **/etc/ssh/sshd_config** to `2200`
6. Delete allow connections on port 22 (Lightsail firewall):
  * Remove the firewall rule for port 22 using the Lightsail management console
7. Configure and enable Uncomplicated Firewall (ufw):
  * Allow incoming tcp on ports 80, 2200 and 123:
  `ufw allow 80/tcp`
  `ufw allow 2200/tcp`
  `ufw allow 123/tcp`
  * Outgoing traffic is allowed on all ports by default
  * Enable ufw: `ufw enable`
8. Create user grader:
  * `adduser grader`
9. Grant grader `sudo` access:
  * Add grader to the sudo group: `usermod -aG sudo grader`
  * Sudo group has `sudo` access as per **/etc/sudoers**
10. Provide SSH key-based authentication to grader:
  * Generate rsa key pair on a local machine: `ssh-keygen -t rsa`
  * Create **/home/grader/.ssh** directory on the server
  * Create **authorized_keys** file in **/home/grader/.ssh**
  * Give ownership of **/home/grader/.ssh** and **authorized_keys** to grader using `chown`
  * Set file permissions for **/home/grader/.ssh** and **authorized_keys** to `700` and `600` respectively using `chmod`
  * Copy the content of **~/.ssh/id_rsa.pub** on the local machine to **/home/grader/.ssh/authorized_keys** on the server using `scp`. This is the public key.
  * User grader can now log in to the server with the private key (**~/.ssh/id_rsa** on the local machine).
11. Install virtualenv:
  * `pip install virtualenv`
12. Create virtual environment for the boardgameclub application:
  * Create container for virtual environments: `mkdir /home/ubuntu/virtualenvs`
  * Navigate to the container
  * Create the virtual environment: `virtualenv bgc_virtualenv`
13. Install the boardgameclub application
  * create the source distribution archive for boardgameclub as per its [readme file](https://github.com/pollux-pw/FSND_p4/blob/master/README.md)
  * copy the archive to the server with `scp`
  * activate the virtual environment: `source <path/to/bgc_virtualenv>/bin/activate`
  * install boardgameclub: `pip install <path/to/the/archive>`
  * the above also installs two command line tools: **bgc_init_db** and **bgc_add_admin**
14. Restrict access to **bgc_init_db** and **bgc_add_admin**:
  * Set file permissions for **<path/to/bgc_virtualenv>/bin/bgc_init_db** and **<path/to/bgc_virtualenv>/bin/bgc_add_admin** to `700` using `chmod`
15. Install Apache HTTP Server and mod_wsgi:
  * `apt-get install apache2 libapache2-mod-wsgi`
16. Create a .wsgi file for boardgameclub:
  * Create **/var/www/boargameclub/** directory
  * Inside this directory create **boardgameclub.wsgi** file with the following content: `from boardgameclub import app as application`
  * After [flask.pocoo.org](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#creating-a-wsgi-file): *This file contains the code mod_wsgi is executing on startup to get the application object. The object called application in that file is then used as application.*
17. Create an Apache configuration file for boardgameclub:
  * create **boardgameclub.conf** file in **/etc/apache2/sites-available** with the following content:
  ```
  <VirtualHost *:80>
        # Specify hostname of the virtual host
        ServerName bgc.18.185.86.131.xip.io
        # Configure log level in the error logs and define paths to log files
        LogLevel info
        ErrorLog ${APACHE_LOG_DIR}/bgc_error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        # Daemon process and process group
        WSGIDaemonProcess boardgameclub python-home=/home/ubuntu/virtualenvs/bgc_virtualenv
        WSGIPRocessGroup boardgameclub
        # Mount the application at root of site
        WSGIScriptAlias / /var/www/boardgameclub/boardgameclub.wsgi
  </VirtualHost>
  ```
  * The usage of **ServerName** directive allows for other virtual hosts to share the same IP address; this means that configuring Apache to serve a second application would be as simple as creating another configuration file and then enabling it. See
  [httpd.apache.org](https://httpd.apache.org/docs/2.4/vhosts/name-based.html) for more information on name-based virtual hosting.
  * For more information on log level see [httpd.apache.org](http://httpd.apache.org/docs/2.4/mod/core.html#loglevel)
  * For more information on daemon process and process group see [modwsgi.readthedocs.io](https://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIDaemonProcess.html) and [modwsgi.readthedocs.io](https://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIProcessGroup.html) respectively.
18. Enable Apache virtual host for boardgameclub:
  * `a2ensite <path/to/boardgameclub.conf>`
19. Install PostgreSQL:
  * `apt-get install postgresql`
  * PostgreSQL is listening on port 5432 by default as per **postgresql.conf**. The firewall does not allow connections on port 5432, therefore remote connections to PostgreSQL are not possible as per project requirement.
20. Install psycopg2 into **bgc_virtualenv**
  * psycopg2 is a PostgreSQL database adapter for Python
  * install python-psycopg2: `apt-get install python-psycopg2`
  * activate the virtual environment: `source <path/to/bgc_virtualenv/bin/activate>`
  * install psycopg2 into the virtual environment: `pip install psycopg2`
21. Create PostgreSQL database and user for boardgameclub
  * log into PostgreSQL as postgres (default PostgreSQL superuser): `sudo -u postgres psql`
  * create user catalog: `CREATE USER catalog WITH PASSWORD '<password>';`
  * create database bgclub: `CREATE DATABASE bgclub;`
22. Create instance folder for boardgameclub with config and client secret files as per its [readme file](https://github.com/pollux-pw/FSND_p4/blob/master/README.md)
  * database name and user used by the app are set in the config file: `DB_URL = 'postgresql://catalog:<password>@localhost:5432/bgclub'`
23. Initialize database for boardgameclub:
  * activate the virtual environment: `source <path/to/bgc_virtualenv/bin/activate>`
  * initialize the bgclub database: `bgc_init_db`
24. Application will be up and running at `bgc.18.185.86.131.xip.io` after Apache restart

## 3. Bibliography
1. [flask.pocoo.org](http://flask.pocoo.org/)
2. [httpd.apache.org](https://httpd.apache.org/)
3. [virtualenv.pypa.io](https://virtualenv.pypa.io/)
4. [modwsgi.readthedocs.io](https://modwsgi.readthedocs.io/)
5. [lightsail.aws.amazon.com](https://lightsail.aws.amazon.com/ls/docs/en/overview)
6. [docs.aws.amazon.com](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
7. [www.postgresqltutorial.com](http://www.postgresqltutorial.com/)
8. [stackoverflow.com](https://stackoverflow.com/)
9. [serverfault.com](https://serverfault.com/)
10. [askubuntu.com](https://askubuntu.com/)
11. [security.stackexchange.com](https://security.stackexchange.com/)
12. Manuals of the packages used: `man <package name>`
