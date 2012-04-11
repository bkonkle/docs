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

Pre-Commit: PEP 8 Checking
--------------------------

::

    #!/usr/bin/env python
    from __future__ import with_statement
    import os
    import re
    import shutil
    import subprocess
    import sys
    import tempfile
 
 
    def system(*args, **kwargs):
        kwargs.setdefault('stdout', subprocess.PIPE)
        proc = subprocess.Popen(args, **kwargs)
        out, err = proc.communicate()
        return out
 
 
    def main():
        modified = re.compile('^[AM]+\s+(?P<name>.*\.py)', re.MULTILINE)
        files = system('git', 'status', '--porcelain')
        files = modified.findall(files)
 
        tempdir = tempfile.mkdtemp()
        for name in files:
            filename = os.path.join(tempdir, name)
            filepath = os.path.dirname(filename)
            if not os.path.exists(filepath):
                os.makedirs(filepath)
            with file(filename, 'w') as f:
                system('git', 'show', ':' + name, stdout=f)
        output = system('pep8', '.', cwd=tempdir)
        shutil.rmtree(tempdir)
        if output:
            print 'Commit aborted for pep8 issues:'
            print output,
            sys.exit(1)
 
 
    if __name__ == '__main__':
        main()

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
