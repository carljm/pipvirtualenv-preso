!SLIDE incremental bullets

## Where does `sys.prefix` come from? ##

* `Python/sysmodule.c`
* Down the rabbit hole...
* `Modules/getpath.c`

!SLIDE smaller

## `Modules/getpath.c` ##

    @@@ c
    #ifndef LANDMARK
    #define LANDMARK "os.py"
    #endif

    static int
    search_for_prefix(char *argv0_path, char *home)
    {
    /* ... */

        /* Search from argv0_path, until root is found */
        copy_absolute(prefix, argv0_path);
        do {
            n = strlen(prefix);
            joinpath(prefix, lib_python);
            joinpath(prefix, LANDMARK);
            if (ismodule(prefix))
                return 1;
            prefix[n] = '\0';
            reduce(prefix);
        } while (prefix[0]);

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
