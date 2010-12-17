.. _newdevmachine:

New Dev Machine Setup
=====================

.. highlight:: sh

General config
**************

* Run Software Update

* Start downloading `XCode <http://developer.apple.com/iphone>`_

* Setup MobileMe syncing

* Clean up dock

* Change default Finder "Arrange by" to "Name"

* Increase the keyboard repeat rate

Install apps
************

* `MagicPrefs <http://magicprefs.com/>`_

* `Dropbox <https://www.dropbox.com/downloading?os=mac>`_

* Change Dropbox menubar color

* `1Password <http://agilewebsolutions.com/downloads/1Password3>`_

* `Chrome <http://www.google.com/chrome/intl/en/eula_dev.html?dl=mac>`_ and make default browser

* Set up Chrome sync

* `JSON extension <https://chrome.google.com/extensions/detail/ddngkjbldiejbheifcmnfmmfiniimbbg>`_

* `RSS extension <https://chrome.google.com/extensions/detail/nlbjncdgjeocebhnmkbbbdekmmmcbfjd>`_

* `1Password extension <http://forum.agile.ws/index.php?/topic/56-setup-instructions/>`_

* `Sparrow Mail <http://www.sparrowmailapp.com/>`_

* `TextMate <http://macromates.com>`_

* `Billings <http://www.marketcircle.com/billings/downloads/>`_

* `Adium <http://adium.im/>`_

* `Firefox <http://www.mozilla.com/en-US/firefox/firefox.html>`_

* `FireBug <http://getfirebug.com/>`_

* `Growl <http://growl.info>`_

* `HTTP Client <http://ditchnet.org/httpclient/>`_

* iLife

* iWork

* Xcode (hopefully it's done downloading by now)

Configure development environment
*********************************

* Configure Terminal

* Install Homebrew::

	$ ruby -e "$(curl -fsSL https://gist.github.com/raw/323731/install_homebrew.rb)"

* Install Git::

	$ brew install git

* Install Hub::

	$ brew install hub

* Install Git-Flow::

    $ brew install git-flow

* Install dotfiles::

	$ cd ~
	$ git clone http://github.com/bkonkle/dotfiles.git .dotfiles
	$ cd .dotfiles
	$ echo myreallylongapikeykeyfromgit > gitapikey
	$ rake install
	$ . ~/.bashrc

* Install Python and Pip::

	$ brew install python pip

* Clone TextMate themes::

	$ git clone bkonkle/TextMate-Themes ~/Library/Application\ Support/TextMate/Themes

Install TextMate plugins
************************

* `MissingSidebar <http://github.com/jezdez/textmate-missingdrawer/>`_

* `AckMate <http://github.com/protocool/AckMate>`_

* GetBundles::

	$ mkdir -p ~/Library/Application\ Support/TextMate/Bundles
	$ cd !$
	$ svn co http://svn.textmate.org/trunk/Review/Bundles/GetBundles.tmbundle/
	$ osascript -e 'tell app "TextMate" to reload bundles'

* Install Python Django (by adamv)

* Install Django Templates (by adamv)

Finishing up
************

* Run Software Update again
