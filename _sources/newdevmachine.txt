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

Increase the keyboard repeat rate and decrease the delay

Install apps
************

Begin the installation of App Store apps in the background.

`Dropbox <https://www.dropbox.com/downloading?os=mac>`_

* Change Dropbox menubar color

`1Password <http://agilewebsolutions.com/downloads/1Password3>`_

`Chrome <http://www.google.com/chrome/intl/en/eula_dev.html?dl=mac>`_

`Firefox <http://www.mozilla.com/en-US/firefox/firefox.html>`_

* `FireBug <http://getfirebug.com/>`_

`Sublime Text 2 <http://www.sublimetext.com/2>`_

* `Package Control <http://wbond.net/sublime_packages/package_control>`_
* SublimeLinter
* Djaneiro
* Nginx
* Less
* SCSS

iLife

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

    $ /usr/bin/ruby -e "$(/usr/bin/curl -fksSL https://raw.github.com/mxcl/homebrew/master/Library/Contributions/install_homebrew.rb)"

Install some packages::

    $ brew install git git-flow wget

Install oh-my-zsh::

    $ wget --no-check-certificate https://github.com/bkonkle/oh-my-zsh/raw/master/tools/install.sh -O - | sh

Install dotfiles::

    $ cd ~
    $ git clone http://github.com/bkonkle/dotfiles.git .dotfiles
    $ cd .dotfiles
    $ scp myserver:~/path/to/apikeys.rb ./
    $ rake install

Restart the terminal.

Install some dev requirements::

    $ brew install python postgresql nginx memcached redis rbenv ruby-build

Restart the terminal again.

Install some Python and Ruby requirements::

    $ easy_install pip
    $ PIP_REQUIRE_VIRTUALENV= pip install virtualenv virtualenvwrapper

    $ rbenv install -l
    $ rbenv install 1.9.3-p429  # Replace with current latest release
    $ rbenv rehash
    $ rbenv global 1.9.3-p429

Restart the terminal one last time.


GeoDjango
*********

If you're using GeoDjango, install the requirements with Homebrew::

    $ PIP_REQUIRE_VIRTUALENV= pip install numpy
    $ brew install geos proj postgis gdal

To create new spatial databases::

    $ createdb  <db name>
    $ psql <db name>
    > CREATE EXTENSION postgis;
    > CREATE EXTENSION postgis_topology;


Postgres
********

To get better performance out of your local postgres, edit
*/usr/local/var/postgres/postgresql.conf* and change the following values:

.. code-block:: cfg

    shared_buffers = 128MB

    work_mem = 8MB

    checkpoint_segments = 24

    effective_cache_size = 256MB

You'll also need to adjust your system's shmmax and shmall values so that
Postgres can allocate the memory it needs. Create the file */etc/sysctl.cconf*
and add the following lines:

.. code-block:: cfg

    kern.sysv.shmmax=144089088
    kern.sysv.shmall=144089088

Then, change the values live::

    $ sudo sysctl -w kern.sysv.shmmax=144089088
    $ sudo sysctl -w kern.sysv.shmall=144089088


Finishing up
************

Run Software Update again
