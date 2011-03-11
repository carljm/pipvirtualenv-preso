!SLIDE incremental bullets

# virtualenv #

* From scratch!

!SLIDE incremental bullets

# What should it do? #

* Fully isolated environments.
* Use its own `site-packages` directory
* ...for everything (imports and installation).
* Ignore the system `site-packages`.

!SLIDE incremental bullets

# `site-packages` #

* Where third-party installed modules go.
* e.g. `/usr/lib/python2.6/site-packages/`
* (Standard library in `/usr/lib/python2.6`)

!SLIDE incremental bullets

# `site-packages` #

* How does it work?
* Python imports `site.py` at startup.
* e.g. `usr/lib/python2.6/site.py`
* Or `Lib/site.py` in the Python source.

!SLIDE smallest

# `site.py` #

    @@@ python
    # Prefixes for site-packages;
    PREFIXES = [sys.prefix, sys.exec_prefix]

    def getsitepackages():
        """Returns a list containing all global site-packages
        directories (and possibly site-python).

        For each directory present in the global ``PREFIXES``,
        this function will find its `site-packages` subdirectory
        depending on the system environment, and will return a
        list of full paths.
        """
        sitepackages = []
        seen = set()

        for prefix in PREFIXES:
            if not prefix or prefix in seen:
                continue
            seen.add(prefix)

            if sys.platform in ('os2emx', 'riscos'):
                sitepackages.append(os.path.join(prefix, "Lib", "site-packages"))
            elif os.sep == '/':
                sitepackages.append(os.path.join(prefix, "lib",
                                            "python" + sys.version[:3],
                                            "site-packages"))
                sitepackages.append(os.path.join(prefix, "lib", "site-python"))
            else:
                sitepackages.append(prefix)
                sitepackages.append(os.path.join(prefix, "lib", "site-packages"))
            if sys.platform == "darwin":
                # for framework builds *only* we add the standard Apple
                # locations.
                from sysconfig import get_config_var
                framework = get_config_var("PYTHONFRAMEWORK")
                if framework and "/%s.framework/"%(framework,) in prefix:
                    sitepackages.append(
                            os.path.join("/Library", framework,
                                sys.version[:3], "site-packages"))
        return sitepackages
