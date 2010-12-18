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
