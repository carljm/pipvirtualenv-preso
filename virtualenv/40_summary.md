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

!SLIDE small

## Sometimes you can't use the virtualenv's binary. ##

## Faking it: ##

    @@@ python
    import sys
    sys.path.insert(0,
        "/path/to/venv/lib/python2.6/site-packages")

    import site
    site.addsitedir(
        "/path/to/venv/lib/python2.6/site-packages")

    execfile("/path/to/venv/bin/activate_this.py")

!SLIDE incremental smbullets

# Forked `site.py`. #

* Distributors modify `site.py` (e.g. Debian `dist-packages`).
* New Python versions modify `site.py`.
* Virtualenv has to play catchup.

!SLIDE

# Other approaches #

!SLIDE incremental smbullets

## virtualenv-small-sitepy ##

### `https://bitbucket.org/brandon/virtualenv-small-sitepy` ###

* Brandon Craig Rhodes' experiment.
* Make virtualenv's `site.py` a light wrapper around the system `site.py`.
* Extend, rather than copy-paste-modify.
* Very close, but not 100%; currently dormant.

!SLIDE incremental smbullets

## rvirtualenv ##

### `https://github.com/kvbik/rvirtualenv` ###

* Jakub Vysoky.
* Completely different approach.
* Uses PYTHONPATH.
* No copied Python binary.
* Relocatable, inheritable.
* Doesn't support --no-site-packages.

!SLIDE incremental smbullets

## Just use `PYTHONHOME`! ##

* Sets `sys.prefix` with no copied binary.
* Still requires copied stdlib or `site.py` hacks.
* Harder to pin a "virtualenv" to a Python version.
* Ian: "I don't like environment variables."

!SLIDE incremental smbullets

## pythonv ##

* Larry Hastings.
* C wrapper that calls system Python binary.
* Most promising direction for integration into Python.
* Just needs a PEP and a patch?
