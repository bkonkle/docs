.. _serversetup:

Manual Server Setup
===================

.. highlight:: sh

This document details how to manually provision a new server.  It uses a web
stack based on nginx and uwsgi, hosted on Rackspace Cloud.

Initial Setup
-------------

Create an Ubuntu 10.10 (Maverick) server instance.  Once the build is complete,
log in and install emacs.  Configure it as the default::

	$ apt-get install emacs23-nox
	$ update-alternatives --config editor

Set up sudo access::

    $ visudo
    
Under this line:

.. code-block:: none

    Defaults        env_reset

Add the following lines to preserve editor and Django environment settings:

.. code-block:: none

    Defaults        env_keep += "EDITOR VISUAL"
    Defaults        env_keep += "DJANGO_SETTINGS_MODULE PYTHONPATH"
    
Then, under this line:

.. code-block:: none

    %sudo   ALL=(ALL) ALL

Add:

.. code-block:: none

    %admin  ALL=(ALL) NOPASSWD:ALL

Now, add a new user and the admin group::

	$ adduser myuser
	$ addgroup admin
	$ adduser myuser admin

Switch to the new user, and add your public key to authorized_keys::

	$ su - myuser
	$ mkdir .ssh
	$ cd !$
	$ wget http://mydomain.com/path/to/my/pubkey/id_rsa.pub
	$ mv id_rsa.pub authorized_keys

Log out of both users, and then make sure you can SSH in using the new user
and invoke sudo with no problems.  You can now lock down the root account::

	$ sudo passwd -l root

Upgrade anything that's out of date::

    $ sudo apt-get update
    $ sudo apt-get dist-upgrade

Install various packages that are often useful::

	$ sudo apt-get install bash-completion python-software-properties

Database Servers
----------------

.. _postfix:

Postfix
*******

We want the servers to be able to send mail, so install postfix and mailx::

    $ sudo apt-get install postfix bsd-mailx

Postgres
********

First, set your locale since Rackspace doesn't set a default::

	$ sudo update-locale LANG=en_US.UTF-8 && export LANG=en_US.UTF-8

For a standard install::

    $ sudo apt-get install postgresql postgresql-server-dev-8.4 postgresql-contrib-8.4

If you want to use streaming replication::

    $ sudo apt-get install python-software-properties
    $ sudo add-apt-repository ppa:pitti/postgresql
    $ sudo apt-get update
    $ sudo apt-get install postgresql postgresql-server-dev-9.0 postgresql-contrib-9.0

Set a password for the *postgresql* user::

	$ sudo passwd postgres
	$ sudo -u postgres psql

	postgres=# \password postgres
	postgres=# \q

Set up a new user for the site being deployed on this instance::

	$ sudo -u postgres createuser myproject
	$ sudo -u postgres psql

	postgres=# \password myproject
	postgres=# \q

Change the access settings::

    $ sudo emacs /etc/postgresql/8.4/main/pg_hba.conf
    
Add the line below.  This restricts access to only allow other instances on the
public Rackspace Cloud.  This will likely need to be edited for each project.

.. code-block:: cfg

    host    all         all         184.106.0.0/16        md5

Edit the configuration file::
    
    $ sudo emacs /etc/postgresql/8.4/main/postgresql.conf

Consider changing the following settings:

.. code-block:: cfg

    listen_addresses = '<server IP>'
    
    max_connections = 250
    
    ssl = false
    
    shared_buffers = <Divide the amount of RAM by 4 and use the value here>
    
    work_mem = 8MB
    
    checkpoint_segments = 24
    
    effective_cache_size = <Divide the amount of RAM by 2>

You will likely need to raise the kernel's SHMMAX parameter.  To find out how
much it needs to be changed to, try to restart *postgres*.

::

    $ sudo /etc/init.d/postgresql restart

You should see something like this:

.. code-block:: none

     * Restarting PostgreSQL 8.4 database server
     * The PostgreSQL server failed to start. Please check the log output:
    2010-11-07 23:15:24 UTC FATAL:  could not create shared memory segment: Invalid argument
    2010-11-07 23:15:24 UTC DETAIL:  Failed system call was shmget(key=5432001, size=282058752, 03600).
    2010-11-07 23:15:24 UTC HINT:  This error usually means that PostgreSQL's request for a shared memory segment exceeded your kernel's SHMMAX parameter.  You can either reduce the request size or reconfigure the kernel with larger SHMMAX.  To reduce the request size (currently 282058752 bytes), reduce PostgreSQL's shared_buffers parameter (currently 32768) and/or its max_connections parameter (currently 253).

The request size is what you are looking for.  Above, it shows a request size
of 282058752.  Edit sysctl.conf::

    $ sudo emacs /etc/sysctl.conf

Add:

.. code-block:: cfg

    kernel.shmmax=282058752

Then reboot the server::

    $ sudo reboot

.. _geodjango:

GeoDjango
*********

Follow the steps at the `Django Docs`_ site to build from source.  Things don't
typically work well when the requirements are installed via Ubuntu packages.

It's a good idea to use ``checkinstall`` install of ``make install`` so that
packages are created for each custom-compiled requirement.  These can then be
easily shared between identical instances or easily removed later.

Place a *dpkg* hold on the custom packages so they're not overwritten::

    echo "geos hold" | sudo dpkg --set-selections
    echo "proj hold" | sudo dpkg --set-selections
    echo "postgis hold" | sudo dpkg --set-selections
    echo "gdal hold" | sudo dpkg --set-selections

.. warning:: Make sure to run *ldconfig*, or bad things will happen and it will be
	difficult to track down::

        $ sudo ldconfig

.. _Django Docs: http://docs.djangoproject.com/en/1.2/ref/contrib/gis/install/#building-from-source

Postgis Template
****************

Create a template for *postgis* databases.

::

    POSTGIS_SQL_PATH=`pg_config --sharedir`/contrib/postgis-1.5
    sudo -u postgres createdb -E UTF8 template_postgis
    sudo -u postgres createlang -d template_postgis plpgsql
    sudo -u postgres psql -d postgres -c "UPDATE pg_database SET datistemplate='true' WHERE datname='template_postgis';"
    sudo -u postgres psql -d template_postgis -f $POSTGIS_SQL_PATH/postgis.sql
    sudo -u postgres psql -d template_postgis -f $POSTGIS_SQL_PATH/spatial_ref_sys.sql
    sudo -u postgres psql -d template_postgis -c "GRANT ALL ON geometry_columns TO PUBLIC;"
    sudo -u postgres psql -d template_postgis -c "GRANT ALL ON spatial_ref_sys TO PUBLIC;"

Also, copy the utils from the *postgis* tar package to a subdirectory of
*/usr/local/share*::

    # If you installed GeoDjango's requirements from packages, you'll need to
    # download the tar archive.
    $ wget http://postgis.refractions.net/download/postgis-1.5.2.tar.gz
    $ tar -xzf postgis-1.5.2.tar.gz
    $ cd postgis-1.5.2
    
    $ sudo mkdir /usr/local/share/postgresql-8.4-postgis/
    $ sudo cp -r utils /usr/local/share/postgresql-8.4-postgis/

NFS Sharing
***********

If you would like to use streaming replication, you will need to set up NFS
sharing between database servers.  Otherwise, this section can be skipped.

Standby
~~~~~~~

Find out the user and group IDs for *postgres*. You'll use them in the next
step::

    $ id -u postgres
    104
    
    $ id -g postgres
    108

Install NFS::

    $ sudo apt-get install nfs-kernel-server nfs-common portmap

If you're on Rackspace, the init.d script will need to be modified because
of Rackspace's custom kernels::

    $ sudo emacs /etc/init.d/nfs-kernel-server

Comment out these lines::

    # See if our running kernel supports the NFS kernel server
    if [ -f /proc/kallsyms ] && ! grep -qE ' nfsd_serv     ' /proc/kallsyms; then      
           log_warning_msg "Not starting $DESC: no support in current kernel."         
           exit 0                                                                      
    fi

Continue setting up NFS::

    $ sudo dpkg-reconfigure portmap

    # Select 'no'

    $ sudo restart portmap

    $ sudo emacs /etc/exports

Add a line like this:

.. code-block:: none

    /var/backups/postgres/mysite    184.106.0.0/16(rw,async,no_subtree_check,all_squash,anonuid=104,anongid=108)

Restart and export::

    $ sudo /etc/init.d/nfs-kernel-server restart

    $ sudo exportfs -a

Make sure to create the /var/backups/postgres/mysite directory and give it
the correct permissions so that the NFS client can access it.

Primary
~~~~~~~

The primary needs to mount the NFS share so that WAL archives can be saved
there::

    $ sudo apt-get install nfs-common portmap
    $ sudo emacs /etc/fstab
    
Add a line like this:

.. code-block:: none

    standby-server:/var/backups/postgres/mysite /var/backups/postgres/mysite nfs rsize=8192,wsize=8192,timeo=14,intr 0 0

Then mount it::

    $ sudo mount /var/backups/postgres/mysite

Make sure to create the /var/backups/postgres/mysite directory so that the NFS
share can be mounted there.

Test the NFS connection by touching a new file on the primary machine, and
making sure it can be removed on the standby machine.

Replication
***********

This section can be skipped if you are not setting up streaming replication.

Primary
~~~~~~~

Edit *postgresql.conf* to enable WAL archiving::

    wal_level = hot_standby
    
    archive_mode = on
    
    archive_command = 'cp -i %p /var/backups/postgres/mysite/%f </dev/null'
    
    max_wal_senders = 1

Create a superuser for streaming replication::

	$ sudo -u postgres createuser -s mysitestandby
	$ sudo -u postgres psql

	postgres=# \password mysitestandby
	postgres=# \q

Modify *pg_hba.conf* to allow the replication user to connect::

    host    replication     mysitestandby     184.106.0.0/16        md5

Base Backup
~~~~~~~~~~~

A base backup needs to be created.  This must be a filesystem-level backup,
not a logical backup like the *pg_dump* command creates.  On the primary
server, run the *pg_start_backup* command:

.. code-block:: sql

    SELECT pg_start_backup('base_backup');

Then, tar and bzip the data directory::

    cd /var/lib/postgresql/9.0/
    sudo tar -cjf postgres-data.tar.bz2 main

This will take awhile.  Once it's done, run the *pg_stop_backup* command:

.. code-block:: sql

    SELECT pg_stop_backup();

Then, copy the archive to the standby server.  Make sure postgresql is not
running on the standby, and unzip the archive::

    sudo /etc/init.d/postgres stop
    cd /var/lib/postgresql/9.0/
    # Make SURE you're on the standby server here
    sudo rm -rf main
    sudo tar -xjf ~/postgres-data.tar.bz2
    sudo chown -R postgres:postgres main

Standby
~~~~~~~

Edit *postgresql.conf* to enable hot standby::

    hot_standby = on
    max_standby_archive_delay = 30s
    max_standby_streaming_delay = 30s

Adjust the delay settings to a value appropriate for your project.

Create a *recovery.conf* in */var/lib/postgresql/9.0/main/* similar to this::

    standby_mode = 'on'
    primary_conninfo = 'host=184.106.207.82 port=5432 user=mysitestandby password=mypassword'
    restore_command = 'cp /var/backups/postgres/mysite/%f %p'
    trigger_file = '/var/backups/postgres/mysite/trigger_file'
    archive_cleanup_command = 'pg_archivecleanup /var/backups/postgres/mysite %r'

The postgresql-contrib-9.0 package doesn't link pg_archivecleanup into the
path.  Do so manually::

    sudo ln -s /usr/lib/postgresql/9.0/bin/pg_archivecleanup /usr/bin/

Now you can start up postgres::

    sudo /etc/init.d/postgresql start

Smoke Test
~~~~~~~~~~

To make sure everything is working properly, first check and make sure wal
sender and receiver processes are live on the primary and standby servers.  On
the primary, run ``ps -ef | grep "wal sender"`` and make sure
you see a wal sender process.  On the standby run ``ps -ef | grep "wal
receiver"``.

Next, create a test table on the primary:

.. code-block:: sql

    CREATE TABLE test (test varchar(30));
    INSERT INTO test VALUES ('Testing 1 2 3');

On the standby, query the table:

.. code-block:: sql

    SELECT * FROM test;
    
You should see the following output:

.. code-block:: none

         test      
    ---------------
     Testing 1 2 3
    (1 row)

You can then delete your test table:

.. code-block:: sql

    DROP TABLE test;

Web Servers
-----------

Postfix and GeoDjango
*********************

Follow the instructions for :ref:`postfix` and :ref:`geodjango` above.

Supervisord
***********

Install supervisord with apt-get::

    $ sudo apt-get install supervisor

uWSGI
*****

uWSGI needs to be compiled from source.  Install some requirements first,
though::

    sudo apt-get install build-essential python-dev libxml2-dev
    wget http://projects.unbit.it/downloads/uwsgi-0.9.6.5.tar.gz
    tar -xzf uwsgi-0.9.6.5.tar.gz
    cd uwsgi-0.9.6.5
    make -f Makefile.Py26
    
Copy the executable to the local sbin directory::

    $ sudo cp uwsgi /usr/local/sbin

Also, copy the default uwsgi settings to the /etc/nginx directory::

    sudo mkdir /etc/nginx
    sudo cp nginx/uwsgi_params /etc/nginx
    cd ..

Nginx
*****

First, install build dependencies and grab the source::

    $ sudo apt-get install libssl-dev
    $ sudo apt-get build-dep nginx
    $ apt-get source nginx
    $ cd nginx-0.7*

Define the ``CONFIGURE_OPTS`` environment variable when building the package to
add *uwsgi* and *ssl*::
    
    $ CONFIGURE_OPTS="--add-module=../uwsgi-0.9.6.5/nginx --with-http_ssl_module" dpkg-buildpackage
    $ cd ..
    $ sudo dpkg -i nginx*.deb
    $ echo "nginx hold" | sudo dpkg --set-selections
    $ echo "nginx-dbg hold" | sudo dpkg --set-selections
    
Starting with nginx-0.8.41, you can add
--http-uwsgi-temp-path=/var/lib/nginx/uwsgi to the ``CONFIGURE_OPTS`` so that
the temp files are kept in the right place. Until then, you'll need to create a
*/usr/local/nginx/uwsgi_temp* folder to use for the temp files::

    $ sudo mkdir -p /usr/local/nginx/uwsgi_temp

Git
***

Install the most recent version of git from the PPA::

    $ sudo add-apt-repository ppa:git-core/ppa
    $ sudo apt-get update
    $ sudo apt-get install git-core

Site Setup
----------

Basic Setup
***********

Set up the directory structure::

	$ sudo mkdir /sites
	$ sudo chown myuser:admin /sites
	$ cd /sites
	$ mkdir -p mydomain.com/{code,public,logs,backup}
	$ cd mydomain.com
	$ sudo chown :www-data logs public
	$ sudo chmod g+w logs public

Check out the site::

	$ git clone user@mygithost.com:/path/to/repository.git code/

Link media into the public directory, if needed::

	$ cd public
	$ ln -s ../code/projectname/media
	$ sudo chown :www-data media
	$ sudo chmod g+w media

Virtualenvs
***********

Setup virtualenvs and create a directory to hold them::

	$ sudo apt-get install build-essential python-dev libpq-dev
	$ sudo apt-get install python-setuptools
	$ sudo easy_install -U pip
	$ sudo pip install virtualenv
	$ sudo pip install virtualenvwrapper
	$ mkdir /sites/virtualenvs

Edit .bashrc::

	$ emacs ~/.bashrc
	
	# Virtualenvwrapper
	export WORKON_HOME=/sites/.virtualenvs
	source /usr/local/bin/virtualenvwrapper.sh

Then, back in the shell::

	$ . ~/.bashrc
	$ mkvirtualenv --no-site-packages projectname
	$ workon projectname
	$ pip install -r /sites/mydomain.com/code/requirements.txt
	$ deactivate

Install the Ubuntu PIL package, and symlink it into the environment::

	$ sudo apt-get install python-imaging
	$ workon projectname
	$ cdsitepackages
	$ ln -s /usr/lib/python2.6/dist-packages/PIL
	$ ln -s /usr/lib/python2.6/dist-packages/PIL.pth

Utilities
*********

Install ntpdate to keep the server's clock in sync::

	$ sudo dpkg-reconfigure tzdata
	$ sudo apt-get install ntpdate
	$ sudo crontab -e

	30 23 * * * /usr/sbin/ntpdate ntp.ubuntu.com > /dev/null

Nginx Configuration
*******************

Add an nginx.conf to your source control::

	$ emacs /sites/mydomain.com/code/deploy/nginx.conf

.. code-block:: nginx

	upstream myapp {
	    server 10.1.2.3:10000;
	    server 10.1.2.4:10000;
	    server 10.1.2.5:10000;
	}
	
	server {
	  listen 80;
	  server_name www.mydomain.com;
	  rewrite ^/(.*) http://mydomain.com/$1 permanent;
	}

	server {
	  listen 80;
	  server_name mydomain.com;

	  access_log /sites/mydomain.com/logs/access.log;
	  error_log /sites/mydomain.com/logs/error.log;

	  location /media {
	    root /sites/mydomain.com/public;
	  }

	  location / {
	    uwsgi_pass myapp;
		include uwsgi_params;
	  }
	}

Symlink it into */etc/nginx/sites-available/* and *sites-enabled/*::

	$ sudo ln -s /sites/mydomain.com/code/deploy/nginx.conf /etc/nginx/sites-available/mydomain.com
	$ sudo ln -s /etc/nginx/sites-available/mydomain.com /etc/nginx/sites-enabled/mydomain.com
	$ sudo /etc/init.d/nginx restart

Supervisor & uWSGI
******************

Add a file to */etc/supervisor/conf.d*::

	$ sudo emacs /etc/supervisor/conf.d/mysite.conf

.. code-block:: conf

	[program:myapp]
	command=/usr/local/sbin/uwsgi
	  --home /sites/virtualenvs/myapp/
	  --module myapp.deploy.wsgi
	  --socket 10.1.2.3:10000
	  --pythonpath /sites/myapp.com/code/myapp
	  --processes 5
	  --master
	directory=/sites/myapp.com/code/myapp
	environment=DJANGO_SETTINGS_MODULE='myapp.settings'
	user=www-data
	autostart=true
	autorestart=true
	stdout_logfile=/sites/myapp.com/logs/uwsgi.log
	redirect_stderr=true
	stopsignal=QUIT

The module specified in the --module option simply contains:

.. code-block:: python

	from django.core.handlers.wsgi import WSGIHandler

	application = WSGIHandler()

If you're just using uWSGI on localhost, you can use something like ``--socket
/sites/myapp.com/var/run/myapp.sock`` instead of an IP address to avoid the
overhead of the full TCP stack.

Reload the uWSGI config and update processes by running::

	$ sudo supervisorctl update
