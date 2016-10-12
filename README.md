Development
-----------
```

# make a development machine
dma create -d virtualbox dev1
deval dev1
dma ip dev1  # give the ip address

# stop all container
dallstop
dallrm

# rebuild and bring back up
dc build
dc up  # to show possible errors

cd web
pip install -r requirements.txt
./manage.py runserver_plus
```

Deployment
-----------
```
deval prod1
dc build
dc up -d

```


# Initial server setup


Step . Spin up both servers
----------------------------
```
docker-machine create \
-d digitalocean \
--digitalocean-access-token=DO_ACCESS_TOKEN \
--digitalocean-size=1gb \
prod1

docker-machine create \
-d digitalocean \
--digitalocean-access-token=DO_ACCESS_TOKEN \
--digitalocean-size=1gb \
dbprod

* Note ip address of db server
* Note ip address of web server
```

Step . Provision database server
-------------------------------
```
deval dbprod
cd db_machine
dc build
dc up -d
```


Step . Provision web server
----------------------------
```

Pre-provision checklist
* add database server ip to  .env file
* BRAINTREE_ENVIRONMENT = braintree.Environment.Production
* For social authentication, set your callback URL on Twitter/Facebook webapp configuration to be dashaccounting.com/complete/twitter or /complete/facebook
* DEBUG=False in your .env file
* make sure config.py is in django_social_app directory 
* make sure recaptcha site key for in templates/recaptcha/widget.html points to right domain

cd web
dc -f prod.yml build
dc -f prod.yml up -d 
dc -f prod.yml run --rm web sh create_superuser.sh
dc run --rm web /usr/local/bin/python manage.py collectstatic

Deployment/security checklist - Make sure that:
* Change admin password
* Uninstall werkzeug: dc run web pip uninstall -y werkzeug
* django settings: ALLOWED_HOSTS = ['*']
* The webhook on braintree is pointed to http://<domain>/aeotunhistEEhietqtbxEO/
* WORRY ABOUT LATER: in django settings: uncomment CACHE setting


```

Step . Harden db server using iptables and fail2ban
---------------------------------------------------
```
iptables -L --linenumbers

iptables -I DOCKER 1 -p tcp ! -s <ip_address> --dport 5432 -j DROP
apb -i hosts -e "box=<dbmachine> okhost=<db_ip>" harden.yml

# from web server
# zero is success, 1 is failure
nc -z -w5 <dbmach_ip> 5432; echo $?

# from your laptop
nc -z -w5 <dbmach_ip> 5432; echo $?
```

Step . Harden web server using fail2ban
--------------------------------------
```
apb -i hosts -e "box=<prodmachine> okhost=<prod1_ip>" harden.yml
```

Backup data
------------
```
deval <db_machine>
dma ssh <db_machine>
docker exec postgrescont pg_dump -U postgres -d postgres -f /tmp/backup.sql

TODO: clarify what these commands do
optional: from the host machine:
psql -h <postgrescont_ip> -p 5432 -U postgres --password

optional, another command:
docker run -it --name pgdumpcont -v /tmp/pgdumpcont:/tmp --volumes-from postgrescont postgres:latest bash
```


Configure logging
-----------------
```
dma ssh mybox

sudo su
vim /etc/rsyslog.d/10-docker.conf

# Docker logging
daemon.* {
 /var/log/docker.log
 stop
}

vim /etc/logrotate.d/docker

/var/log/docker.log {
    size 100M
    rotate 2
    missingok
    compress
}

service rsyslog restart


tail -f /var/log/docker.log
```


WEB
---


INSTALLATION
--------------
$ pip install -r requirements.txt
$ brew install redis

# start redis
$ redis-server

# test redis
$ redis-cli ping


# test celery
$ celery -A myproj beat -l info 


RUNNING TASKS
-------------
NOTE: In production, you will want to run thes command on supervisor.
See here for supervisor setup instructions:
https://realpython.com/blog/python/asynchronous-tasks-with-django-and-celery/

# run periodic tasks
$ celery -A myproj beat -l info 
or
$ celery -A myproj -B -l info


# run normal task
$ celery -A myproj worker -l info 


CELERY CRONTAB DOCS:
http://celery.readthedocs.org/en/latest/userguide/periodic-tasks.html#crontab-schedules



Installation
---------

Step 1. Install node package manager (npm) by going to `https://nodejs.org/` and click INSTALL

Step 2. Check that `npm` is installed:

```bash
npm -v
```

Step 3. Install gulp globally

```bash
npm install -g gulp

```

Step 4. Install requirements

```bash
cd myproject
npm install # this will create node_modules/ subdirectory in your directory
```

Run gulp + browsersync + django
------------------------------

Step 5. Run gulp
```bash
gulp # this will run django and open a browser
```

IMPORTANT: WHEN USING GULP+BROWSERSYNC, all your STDOUT+STDERR is in Chrome Console


Troubleshooting
------------
If you have trouble connecting, make sure the port is set to the correct port.
Trying closing your browser.
Also, you might have to goto the BrowserSync control panel (localhost:3001) and click 'Reload Browser' to refresh it.

