.. _homeserversetup:

Home Server Setup
=================

.. highlight:: sh

This is how I set up my home server, which I use for hosting streaming video
for my PS3, file storage, backups, and more.

Initial Setup
-------------

Install Ubuntu 10.10 on your machine using a CD or USB key.  Once the install
is complete, log in and install emacs.  Configure it as the default::

	$ sudo apt-get install emacs23-nox
	$ sudo update-alternatives --config editor

Set up sudo access::

    $ sudo visudo
    
Under this line:

.. code-block:: none

    Defaults        env_reset

Add the following line to preserve editor environment settings:

.. code-block:: none

    Defaults        env_keep += "EDITOR VISUAL"
    
Then, change this line:

.. code-block:: none

    %admin  ALL=(ALL) ALL

To this:

.. code-block:: none

    %admin  ALL=(ALL) NOPASSWD:ALL

Upgrade anything that's out of date::

    $ sudo apt-get update
    $ sudo apt-get dist-upgrade

Install various packages that are often useful::

	$ sudo apt-get install bash-completion python-software-properties build-essential checkinstall

Add your public key to authorized_keys::

	$ su - myuser
	$ mkdir .ssh
	$ cd !$
	$ wget http://mydomain.com/path/to/my/pubkey/id_rsa.pub
	$ mv id_rsa.pub authorized_keys

Once you've confirmed pubkey authentication is working, disable SSH password
authentication::

    $ sudo emacs /etc/ssh/sshd_config
    
    # Make sure the following are set to no
    ChallengeResponseAuthentication no
    PasswordAuthentication no
    UsePAM no
    
Reload the SSH config::

    $ sudo /etc/init.d/ssh reload

NFS Sharing
-----------

Find out your user and group IDs. You'll use them in the next
step::

    $ id -u myuser
    1000
    
    $ id -g myuser
    1000

Install NFS::

    $ sudo apt-get install nfs-kernel-server nfs-common portmap
    
    $ sudo dpkg-reconfigure portmap

    # Select 'no'

    $ sudo restart portmap

    $ sudo emacs /etc/exports

Add a line like this:

.. code-block:: none

    /path/for/storage    10.0.1.0/24(rw,insecure,async,no_subtree_check,all_squash,anonuid=1000,anongid=1000)
    /path/for/backups    10.0.1.0/24(rw,insecure,async,no_subtree_check,all_squash,anonuid=1000,anongid=1000)

10.0.1.0/24 will allow IPs in the range of 10.0.1.0 - 10.0.1.255 to connect.
Make sure to create the backups and storage directories and give it the correct
permissions so that the NFS client can access it.

Restart and export::

    $ sudo /etc/init.d/nfs-kernel-server restart

    $ sudo exportfs -a

Installing Avahi for Zeroconf (Bonjour) Support
-----------------------------------------------

This is probably unnecessary, but I like being able to SSH to myhost.local::

    $ sudo apt-get install avahi-daemon

Installing Mediatomb with JS Support
------------------------------------

First, download and build SpiderMonkey::

    $ wget http://ftp.mozilla.org/pub/mozilla.org/js/js-1.8.0-rc1.tar.gz
    $ tar -xzf js-1.8.0-rc1.tar.gz
    $ cd js/src/
    $ make BUILD_OPT=1 -f Makefile.ref

Then create a quick Makefile::

    BLD := Linux_All_OPT.OBJ
    PREFIX := /usr

    install:
    	cp ${BLD}/libjs.so ${PREFIX}/lib
    	cp ${BLD}/js ${PREFIX}/bin
    	cp ${BLD}/jscpucfg ${PREFIX}/bin
    	cp ${BLD}/jskwgen ${PREFIX}/bin
    	mkdir -p ${PREFIX}/include/js
    	cp *.h ${PREFIX}/include/js
    	cp *.tbl ${PREFIX}/include/js
    	cp ${BLD}/*.h ${PREFIX}/include/js

Install with checkinstall so that it can be removed later if needed::

    $ sudo checkinstall --pkgname=libjs --pkgversion=1.8.0-rc1

Now, grab the mediatomb source and edit the debian rules to enable libjs::

    $ sudo apt-get install libjs-prototype
    $ sudo apt-get build-dep mediatomb
    $ apt-get source mediatomb
    $ cd mediatomb*/
    $ emacs debian/rules
    
    # Change --disable-libjs to --enable-libjs in MEDIATOMB_CONFIG_OPTIONS
    
    $ dpkg-buildpackage -rfakeroot -us -uc
    $ cd ..
    $ sudo dpkg -i mediatomb*.deb
    
    echo "libjs hold" | sudo dpkg --set-selections
    echo "mediatomb-common hold" | sudo dpkg --set-selections
    echo "mediatomb-daemon hold" | sudo dpkg --set-selections
    echo "mediatomb hold" | sudo dpkg --set-selections

Now you can create custom virtual containers using JavaScript, to keep your 
content much more organized.
