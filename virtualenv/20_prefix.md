!SLIDE smaller

# `Python/sysmodule.c` #

    @@@ c
     /* System module */

    /*
    Various bits of information used by the interpreter are
    collected in module 'sys'.
    Function member:
    - exit(sts): raise SystemExit
    Data members:
    - stdin, stdout, stderr: standard file objects
    - modules: the table of modules (dictionary)
    - path: module search path (list of strings)
    - argv: script arguments (list of strings)
    - ps1, ps2: optional primary and secondary prompts (strings)
    */

!SLIDE smaller

# `Python/sysmodule.c` #

    @@@ c
    SET_SYS_FROM_STRING("prefix",
                        PyString_FromString(Py_GetPrefix()));
    SET_SYS_FROM_STRING("exec_prefix",
                        PyString_FromString(Py_GetExecPrefix()));

!SLIDE

# Down the rabbit hole! #

!SLIDE smallest

# `Modules/getpath.c` #

    @@@ c
    /* Search in some common locations for the associated Python libraries.
     *
     * Two directories must be found, the platform independent directory
     * (prefix), containing the common .py and .pyc files, and the platform
     * dependent directory (exec_prefix), containing the shared library
     * modules.  Note that prefix and exec_prefix can be the same directory,
     * but for some installations, they are different.
     *
     ...

!SLIDE incremental bullets

# The hunt for `sys.prefix` #

* If `PYTHONHOME` is set, that's the prefix. No questions asked.
* Otherwise...

!SLIDE incremental bullets

# The hunt for `sys.prefix` #

* Starting from the location of the Python binary, search upwards.
* At each step, look for `lib/pythonX.X/os.py`.
* If it exists, we've found `sys.prefix`.
* If we never find it, fallback on hardcoded `configure --prefix` from build.

!SLIDE

## Yes, `os.py` is the hardcoded landmark. ##

!SLIDE smaller

# `Modules/getpath.c` #

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
