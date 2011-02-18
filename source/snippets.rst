Snippets
========

.. highlight:: sh

Git Hooks
*********

Pre-Commit: Build & Add Sphinx Docs
-----------------------------------

The pre-commit hook used for this site::

    #!/bin/sh

    # Build Sphinx docs
    make html
    if [ $? = 0 ]; then
        # If the build is successful, add everything except for the source
        # directory, which should be managed manually
        git add *
        git add _sources
        git add _static
        git add doctrees
    else
        exit $?
    fi

Convert Git-Svn Tags to Actual Tags
***********************************

Modify this bash script to suit your needs, and run it inside the repo::

    #!/bin/bash

    for branch in `git branch -r`; do
        if [ `echo $branch | egrep "tags/.+$"` ]; then
            version=`basename $branch`
            subject=`git log -1 --pretty=format:"%s" $branch`
            export GIT_COMMITTER_DATE=`git log -1 --pretty=format:"%ci" $branch`

            echo "Tag $version [Y/n]?"
            read yesno
            if [ -z $yesno ] || [ $yesno = "Y" ]; then
                git tag -f -m "$subject" "$version" "$branch"
                git branch -d -r $branch
            fi
        fi
    done

Install the PostgreSQL Gem on an x86_64 Intel Mac
*************************************************

Using `Homebrew <https://github.com/mxcl/homebrew>`_, install PostgreSQL::

    $ brew install postgresql

Then, run the gem install with ARCHFLAGS set::

    $ sudo env ARCHFLAGS='-arch x86_64' gem install pg

Installing Mediatomb on Ubuntu 10.10 with JS Support
****************************************************

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

    $ sudo apt-get build-dep mediatomb
    $ apt-get source mediatomb
    $ cd mediatomb*/
    $ emacs debian/rules
    
    # Change --disable-libjs to --enable-libjs in MEDIATOMB_CONFIG_OPTIONS
    
    $ dpkg-buildpackage -rfakeroot -us -uc
    $ cd ..
    $ sudo dpkg -i mediatomb*.deb
    
    $ echo "libjs hold" | sudo dpkg --set-selections
    $ echo "mediatomb-common hold" | sudo dpkg --set-selections
    $ echo "mediatomb-daemon hold" | sudo dpkg --set-selections
    $ echo "mediatomb hold" | sudo dpkg --set-selections

Now you can create custom virtual containers using JavaScript, to keep your 
content much more organized.

Snow Leopard NFS Automount
**************************

Disk Utility --> File --> NFS Mounts...

**Remote NFS URL**: nfs://[server]/[path]

**Mount location**: [path to local mount folder]

**Advanced Mount Parameters**: -i,-s,-w=32768,-r=32768

SSH Port Forwarding
*******************

Standard
--------

::

    # ssh -N -L <LOCAL PORT>:<HOST>:<REMOTE PORT> <REMOTE HOST>
    $ ssh -N -L 8000:web03ilo:80 web03

Reverse
-------

::

    # ssh -N -R <REMOTE PORT>:<HOST>:<LOCAL PORT> <REMOTE HOST>
    $ ssh -N -R 2022:localhost:22 user@server
