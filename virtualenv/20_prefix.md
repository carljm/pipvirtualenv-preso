!SLIDE incremental bullets

## Where does `sys.prefix` come from? ##

* `Python/sysmodule.c`
* `Modules/getpath.c`

!SLIDE incremental bullets

# The hunt for `sys.prefix` #

* If `PYTHONHOME` is set, that's `sys.prefix`.
* Otherwise...

!SLIDE incremental bullets

* Starting from the location of the Python binary, search upwards.
* At each step, look for `lib/pythonX.X/os.py`.
* If it exists, we've found `sys.prefix`.
* If we never find it, fallback on hardcoded `configure --prefix` from build.

!SLIDE

## Yes, `os.py` is the hardcoded landmark. ##

## `Modules/getpath.c` ##

    @@@ c
    #ifndef LANDMARK
    #define LANDMARK "os.py"
    #endif
