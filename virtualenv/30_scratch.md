!SLIDE

# Let's try it. #

    $ mkdir scratch; cd scratch

!SLIDE commandline incremental

# Copy the Python binary. #

    $ mkdir bin

    $ cp /usr/bin/python bin/

    $ tree
    .
    `-- bin
        `-- python

    $ ./bin/python -c "import sys; print(sys.prefix)"
    /usr

!SLIDE incremental smbullets

# `sys.prefix` is still `/usr`. #

* No `PYTHONHOME` set.
* Binary is in `/home/carljm/scratch/bin/`.
* No `/home/carljm/scratch/bin/lib/python2.6/os.py`
* No `/home/carljm/scratch/lib/python2.6/os.py`
* No `/home/carljm/lib/python2.6/os.py`
* No `/home/lib/python2.6/os.py`
* `/usr` is the fallback build prefix.

!SLIDE commandline incremental

# Create the landmark. #

    $ mkdir -p lib/python2.6

    $ touch lib/python2.6/os.py

    $ tree
    .
    |-- bin
    |   `-- python
    `-- lib
        `-- python2.6
            `-- os.py

!SLIDE incremental commandline

# Success! #

    $ ./bin/python -c "import sys; print(sys.prefix)"
    'import site' failed; use -v for traceback
    /home/carljm/scratch

    $ ./bin/python -c "import sys; print(sys.path)"
    'import site' failed; use -v for traceback
    ['',
     '/home/carljm/scratch/lib/python2.6/',
     '/home/carljm/scratch/lib/python2.6/plat-linux2',
     '/home/carljm/scratch/lib/python2.6/lib-tk',
     '/home/carljm/scratch/lib/python2.6/lib-old',
     '/usr/lib/python2.6/lib-dynload']

!SLIDE commandline

# Fixing `sys.exec_prefix` #

    $ mkdir lib/python2.6/lib-dynload

    $ ./bin/python -c "import sys; print(sys.exec_prefix)"
    'import site' failed; use -v for traceback
    /home/carljm/scratch

    $ ./bin/python -c "import sys; print(sys.path)"
    'import site' failed; use -v for traceback
    ['',
     '/home/carljm/scratch/lib/python2.6/',
     '/home/carljm/scratch/lib/python2.6/plat-linux2',
     '/home/carljm/scratch/lib/python2.6/lib-tk',
     '/home/carljm/scratch/lib/python2.6/lib-old',
     '/home/carljm/scratch/lib/python2.6/lib-dynload']

!SLIDE incremental bullets

# We have an isolated virtualenv! #

* It's broken.
* No standard library.
* No `site.py`, thus no `site-packages` at all.

!SLIDE incremental bullets

# How could we fix it? #

* Copy the entire stdlib (including `site.py`) into our `lib/python2.6`.
* Works, but heavy.
* We're not going to modify the standard library; could we just use the system
  one?

!SLIDE incremental bullets

# Bootstrap like a `virtualenv`. #

* Write `orig-prefix.txt` to `lib/python2.6`.
* Contains the system Python's `sys.prefix`.
* Place our own, modified `site.py` in `lib/python2.6`.
* Our `site.py` reads `orig-prefix.txt` and adds system stdlib paths to
  `sys.path`.

!SLIDE commandline

    $ tree
    .
    |-- bin
    |   `-- python
    `-- lib
        `-- python2.6
            |-- lib-dynload
            |-- orig-prefix.txt
            |-- os.py
            `-- site.py


!SLIDE small

# From virtualenv's modified `site.py` #

    @@@ python
    def virtual_install_main_packages():
        f = open(os.path.join(os.path.dirname(__file__),
                              'orig-prefix.txt'))
        sys.real_prefix = f.read().strip()
        # ...
        if sys.platform == 'win32':
            paths = [os.path.join(sys.real_prefix, 'Lib'),
                     os.path.join(sys.real_prefix, 'DLLs')]
        # ...
        else:
            paths = [os.path.join(sys.real_prefix,
                                  'lib',
                                  'python'+sys.version[:3])]
        # ...
        sys.path.extend(paths)

!SLIDE incremental bullets

## But `site.py` depends on the standard library! ##

* We have a bootstrapping problem.
* Solution: symlink (or copy) just the stdlib bits that we need
  to bootstrap `site.py`.
* After `site.py` runs, we'll have access to the full stdlib thanks to its
  `sys.path` additions.

!SLIDE smaller

# From `virtualenv.py` #

    @@@ python
    REQUIRED_MODULES = [
        'os', 'posix', 'posixpath', 'nt', 'ntpath', 'genericpath',
        'fnmatch', 'locale', 'encodings', 'codecs',
        'stat', 'UserDict', 'readline', 'copy_reg', 'types',
        're', 'sre', 'sre_parse', 'sre_constants', 'sre_compile',
        'zlib']

    def copy_required_modules(dst_prefix):
        import imp
        for modname in REQUIRED_MODULES:
            # ...
            try:
                f, filename, _ = imp.find_module(modname)
            except ImportError:
                logger.info("Cannot import bootstrap module: %s" % modname)
            else:
                if f is not None:
                    f.close()
                dst_filename = change_prefix(filename, dst_prefix)
                copyfile(filename, dst_filename)
                # ...

!SLIDE

## We can also add the global `site-packages`, if we want it. ##

!SLIDE smaller

# `site.py` #

    @@@ python
    def virtual_addsitepackages(known_paths):
        # ...
        return addsitepackages(known_paths,
                               sys_prefix=sys.real_prefix)

    # ...

    def main():
        # ...
        if GLOBAL_SITE_PACKAGES:
            paths_in_sys = virtual_addsitepackages(paths_in_sys)
        # ...
