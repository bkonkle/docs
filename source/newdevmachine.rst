.. _newdevmachine:

New Dev Machine Setup
=====================

.. highlight:: sh

I like to keep my machines squeaky-clean, so I tend to format my system
partition every 6 months or so.  I keep this doc around so to remind me how to
reinstall everything.

General config
**************

Run Software Update

Setup iCloud syncing

Clean up dock

Change default Finder "Arrange by" to "Name"

Increase the keyboard repeat rate

Install apps
************

Begin the installation of App Store apps in the background.

`Dropbox <https://www.dropbox.com/downloading?os=mac>`_

* Change Dropbox menubar color

`1Password <http://agilewebsolutions.com/downloads/1Password3>`_

`Chrome <http://www.google.com/chrome/intl/en/eula_dev.html?dl=mac>`_

`Firefox <http://www.mozilla.com/en-US/firefox/firefox.html>`_

* `FireBug <http://getfirebug.com/>`_

`TextMate <http://macromates.com>`_

iLife

iWork

Configure development environment
*********************************

Configure Terminal

* Install the `Tomorrow Theme <https://github.com/ChrisKempson/Tomorrow-Theme>`_

* Change font to Menlo Regular 12pt

* On the *Shell* tab, select "Close if the shell exited cleanly"

* On the *Keyboard* tab, check "Use option as meta key"

* Give the background color 95% transparency

* Change the window size to 100 x 50 for iMac, or 100 x 40 for Macbook.

* Save as default

Rsync .ssh config::

    $ rsync -avz myserver:~/.ssh/ ~/.ssh/

Install Homebrew::

	$ /usr/bin/ruby -e "$(curl -fsSL https://raw.github.com/gist/323731)"

Install some packages::

	$ brew install git hub git-flow wget

Install oh-my-zsh::

	$ wget --no-check-certificate https://github.com/bkonkle/oh-my-zsh/raw/master/tools/install.sh -O - | sh

Install dotfiles::

	$ cd ~
	$ git clone http://github.com/bkonkle/dotfiles.git .dotfiles
	$ cd .dotfiles
	$ scp myserver:~/path/to/apikeys.rb ./
	$ rake install

Restart terminal.

Install some dev requirements::

    $ brew install python postgresql nginx memcached redis
    
Install the launch agents for each, changing RunAtLoad to OnDemand.  Then
install some Python packages.
        
	$ easy_install pip
	
	$ PIP_REQUIRE_VIRTUALENV= pip install virtualenv virtualenvwrapper

GeoDjango
*********

If you're using GeoDjango, install the requirements with Homebrew::

    $ brew install geos proj postgis gdal

Set up a template for new postgis databases::

    POSTGIS_SQL_PATH=`pg_config --sharedir`/contrib/postgis-1.5
    createdb -E UTF8 template_postgis
    createlang -d template_postgis plpgsql
    psql -d postgres -c "UPDATE pg_database SET datistemplate='true' WHERE datname='template_postgis';"
    psql -d template_postgis -f $POSTGIS_SQL_PATH/postgis.sql # Loading the PostGIS SQL routines
    psql -d template_postgis -f $POSTGIS_SQL_PATH/spatial_ref_sys.sql
    psql -d template_postgis -c "GRANT ALL ON geometry_columns TO PUBLIC;" # Enabling users to alter spatial tables.
    psql -d template_postgis -c "GRANT ALL ON spatial_ref_sys TO PUBLIC;"

Install TextMate plugins
************************

`MissingSidebar <http://github.com/jezdez/textmate-missingdrawer/>`_

`AckMate <http://github.com/protocool/AckMate>`_

GetBundles::

	$ mkdir -p ~/Library/Application\ Support/TextMate/Bundles
	$ cd !$
	$ svn co http://svn.textmate.org/trunk/Review/Bundles/GetBundles.tmbundle/
	$ osascript -e 'tell app "TextMate" to reload bundles'

Install Python Django (by adamv)

Install Django Templates (by adamv)

Install reStructuredText

Finishing up
************

Run Software Update again
