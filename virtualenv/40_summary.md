!SLIDE

## What makes a virtualenv work is the location of its binary, and its hacked `site.py`. ##

### Corollary: if you aren't using the virtualenv's binary, you aren't really using the virtualenv. ###

!SLIDE bullets incremental

# What about `bin/activate`? #

* A shell convenience only.
* Adds the virtualenv's `bin/` to the front of your shell `PATH`.
* Bad mental model.

!SLIDE

# Problems #

!SLIDE

## Sometimes you can't use the virtualenv's binary. ##

### `mod_wsgi` ###

!SLIDE small

    @@@ python
    import sys
    sys.path.insert(0,
        "/path/to/venv/lib/python2.6/site-packages")

!SLIDE incremental bullets

# `sys.path.insert(0, ...)` #

* Doesn't process `.pth` files.
* Fine if your virtualenv only includes flat installs.
* Won't work with eggs or develop/editable installs.

!SLIDE small

    @@@ python
    import site
    site.addsitedir(
        "/path/to/venv/lib/python2.6/site-packages")

!SLIDE incremental bullets

# `site.addsitedir` #

* Processes `.pth` files.
* Appends to end of `sys.path`, global site-packages will take precedence.

!SLIDE small

    @@@ python
    execfile("/path/to/venv/bin/activate_this.py")

!SLIDE incremental smbullets

# `activate_this.py` #

* Uses `site.addsitedir`.
* Rearranges sys.path to put venv site-packages first.
* No support for `--no-site-packages`.
* Changes `sys.prefix`.
* This makes Graham sad.

!SLIDE incremental smbullets

# Forked `site.py`. #

* Distributors modify `site.py` (e.g. Debian `dist-packages`).
* New Python versions modify `site.py`.
* Virtualenv has to play catchup.

!SLIDE

# Other approaches #

!SLIDE incremental smbullets

### `https://bitbucket.org/brandon/virtualenv-small-sitepy` ###

* Brandon Craig Rhodes' experiment.
* Refactor virtualenv's `site.py` to be a light wrapper around the system `site.py`.
* Extend, rather than copy-paste-modify.
* Close, but not 100%; not actively developed.

!SLIDE incremental smbullets

### `https://github.com/kvbik/rvirtualenv` ###

* Jakub Vysoky.
* Completely different approach.
* Uses PYTHONPATH.
* No copied Python binary.
* Doesn't support --no-site-packages.

!SLIDE incremental smbullets

## Just use `PYTHONHOME` env var ##

* Sets `sys.prefix` without need for relocated binary.
* Still requires copied stdlib or `site.py` hacks.
* Quite similar to virtualenv, largely a matter of preference.
