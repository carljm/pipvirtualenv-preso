!SLIDE smbullets

# Pip! #

* Pip installs (Python) Packages

!SLIDE

# How? #

!SLIDE incremental bullets

## Pip's dirty little secret ##

* Pip doesn't install packages at all...
* Pip <3 setuptools!

!SLIDE small

# `pip/req.py` #

    @@@ python
    def install(self, install_options, global_options=()):
        # ...
        install_args = [
            sys.executable, '-c',
            "import setuptools;__file__=%r;"\
            "execfile(__file__)" % self.setup_py] +\
            list(global_options) + [
            'install',
            '--single-version-externally-managed',
            '--record', record_filename]

        # ...

        call_subprocess(install_args + install_options,
            cwd=self.source_dir,
            filter_stdout=self._filter_install,
            show_stdout=False)

!SLIDE bullets incremental

# Translation: #

* Ugliest.
* Hack.
* Evar.
* (Sorry, Ian.)
* (Like a disturbing number of Ian's ugly hacks, it works quite well.)

!SLIDE smaller

# Install like pip: #

    python -c
      "import setuptools;__file__=/p/to/setup.py;execfile(__file__)"
      install
      --single-version-externally-managed
      --record /tmp/pip-ZgGVWG-record/install-record.txt

!SLIDE smbullets incremental

### `"import setuptools;__file__=/p/to/setup.py;execfile(__file__)"` ###

* Setuptools works by monkeypatching the builtin distutils.
* Thus, importing setuptools is enough to activate its features.
* This hack allows pip to do setuptools-only things (generate .egg-info
  metadata, develop/editable installs) on all projects, even those that stick
  to pure distutils in their setup.py.

!SLIDE smbullets incremental

## `install` ##

* Just like a normal `python setup.py install`.

!SLIDE smbullets incremental

## `--single-version-externally-managed` ##

* Tells setuptools to do a "flat" (distutils-style) install and put the
  metadata next to it in a `.egg-info` directory, rather than a zipped egg or
  egg-directory.

* A setuptools feature.

!SLIDE commandline

## Setuptools normally installs like this: ##

    $ easy_install yolk

    # cd to site-packages
    $ ls | egrep "(yolk|easy)"
    easy-install.pth
    yolk-0.4.1-py2.6.egg

    $ cat easy-install.pth
    ./setuptools-0.6c11-py2.6.egg
    ./pip-0.8.1-py2.6.egg
    ./yolk-0.4.1-py2.6.egg
    import sys; new=sys.path[sys.__plen:]; del sys.path[sys.__plen:]; p=getattr(sys,'__egginsert',0); sys.path[p:p]=new; sys.__egginsert = p+len(new)

!SLIDE commandline

## With `--single-version-externally-managed`: ##

    $ pip install yolk

    $ ls | grep yolk
    yolk/
    yolk-0.4.1-py2.6.egg-info/

!SLIDE smbullets incremental

### `--record /tmp/pip-ZgGVWG-record/install-record.txt` ###

* Generates a record of all files installed.
* distutils and setuptools: Y U NO DO THIS BY DEFAULT?!?!??!
* (Fortunately PEP 376 mandates it, and distutils2 does it.)
* Pip generates it in a temporary location, then moves it to
  `installed-files.txt` in the `.egg-info` metadata directory.
