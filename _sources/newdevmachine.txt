.. _newdevmachine:

New Dev Machine Setup
=====================

.. highlight:: sh

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

* Start with Pro as the base

* Change font to Menlo Regular 11pt

* Uncheck "Use bold fonts"

* Check "Use bright colors"

* On the *Shell* tab, select "Close if the shell exited cleanly"

* On the *Keyboard* tab, check "Use option as meta key"

* Save as default

Install Homebrew::

	$ ruby -e "$(curl -fsSL https://gist.github.com/raw/323731/install_homebrew.rb)"

Install some packages::

	$ brew install git hub git-flow bash-completion

Install dotfiles::

	$ cd ~
	$ git clone http://github.com/bkonkle/dotfiles.git .dotfiles
	$ cd .dotfiles
	$ echo myreallylongapikeykeyfromgit > gitapikey
	$ rake install
	$ . ~/.bashrc

Install some dev requirements::

    $ brew install python --universal --framework
	$ brew install postgresql nginx memcached
        
        $ . ~/.bashrc
	$ easy_install pip

Clone TextMate themes::

	$ git clone bkonkle/TextMate-Themes ~/Library/Application\ Support/TextMate/Themes

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
