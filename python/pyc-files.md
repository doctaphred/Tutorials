# On Compiled Python Files


## Background

When a Python module is loaded from a .py file, the interpreter compiles the source to its choice of bytecode, which remains in memory and is executed. CPython writes this compiled bytecode out to .pyc files for imported modules (but not for scripts that are directly executed), in order to skip the compilation step and load them faster the next time.

From [the docs](https://docs.python.org/2/tutorial/modules.html#compiled-python-files):

> As an important speed-up of the start-up time for short programs that use a lot of standard modules, if a file called `spam.pyc` exists in the directory where `spam.py` is found, this is assumed to contain an already-“byte-compiled” version of the module `spam`. The modification time of the version of `spam.py` used to create `spam.pyc` is recorded in `spam.pyc`, and the .pyc file is ignored if these don’t match.

> A program doesn’t run any faster when it is read from a .pyc or .pyo file than when it is read from a .py file; the only thing that’s faster about .pyc or .pyo files is the speed with which they are loaded.

## Problem

> It is possible to have a file called spam.pyc (or spam.pyo when -O is used) without a file spam.py for the same module.

This means that, if a module has imported and compiled and later moved, its corresponding .pyc file may remain **as a viable, identically-named module of its own**.

With Python 2's relative import semantics, this creates a problem, since this "ghost" module may shadow the name of the moved, "real" module and render it inaccessible. Consider the following project structure:

    $ tree ap
    ap
    ├── ap
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   ├── wsgi.py
    ├── db.sqlite3
    ├── manage.py
    └── organization
        ├── admin.py
        ├── __init__.py
        ├── models.py
        ├── tests.py
        ├── utils.py
        └── views.py

After running the project, the file tree might look like this:

    ap
    ├── ap
    │   ├── __init__.py
    │   ├── __init__.pyc
    │   ├── settings.py
    │   ├── settings.pyc
    │   ├── urls.py
    │   ├── urls.pyc
    │   ├── wsgi.py
    │   └── wsgi.pyc
    ├── db.sqlite3
    ├── manage.py
    └── organization
        ├── admin.py
        ├── admin.pyc
        ├── __init__.py
        ├── __init__.pyc
        ├── models.py
        ├── models.pyc
        ├── tests.py
        ├── utils.py
        ├── utils.pyc
        ├── views.py
        └── views.pyc

(Side note: Certain .py files do not have corresponding .pyc files because they are executed as scripts rather than imported. If desired, .pyc files may be [manually generated](https://docs.python.org/2/library/compileall.html#module-compileall) for them via `python -m compileall`. Remember, though, that this will only affect the scripts' load time, and must be repeated after every change to the scripts, or else Python will detect that the .pyc files are outdated and ignore them.)

Now suppose we move utils.py out of the organization app, to become a base-level app-agnostic module. To do this properly, the version control system must be made aware of the move, and any `from organization import utils` or `from . import utils` statements must be refactored to `import utils`. (This process is as simple as dragging and dropping the file with PyCharm and other similar tools, but is also doable manually.) The file tree will now look like this:

    ap
    ├── ap
    │   ├── __init__.py
    │   ├── __init__.pyc
    │   ├── settings.py
    │   ├── settings.pyc
    │   ├── urls.py
    │   ├── urls.pyc
    │   ├── wsgi.py
    │   └── wsgi.pyc
    ├── db.sqlite3
    ├── manage.py
    ├── organization
    │   ├── admin.py
    │   ├── admin.pyc
    │   ├── __init__.py
    │   ├── __init__.pyc
    │   ├── models.py
    │   ├── models.pyc
    │   ├── tests.py
    │   ├── utils.pyc
    │   ├── views.py
    │   └── views.pyc
    └── utils.py

Note the file ap/organization/utils.pyc, which should be ignored by the VCS and thus remains in the `organization` package. This .pyc file constitutes a viable module, but as explained above, no imports should be specifically targeting it anymore.

The problem is that, even if the imports have been changed as described, within the package `organization` the statement `import utils` still first looks in the local directory and finds `utils.pyc`, yielding import errors when the code attempts to access new attributes defined in ap/utils.py.


https://docs.python.org/2/tutorial/modules.html#the-module-search-path
> When a module named `spam` is imported, the interpreter first searches for a built-in module with that name. If not found, it then searches for a file named `spam.py` in a list of directories given by the variable `sys.path`.

https://docs.python.org/2/tutorial/modules.html#intra-package-references
> the import statement first looks in the containing package before looking in the standard module search path.


## Solution

A quick fix for this problem is to manually delete the rogue .pyc files, although they will return the next time the module is imported. For a more permanent fix, add a post-checkout hook to Git, or configure your environment to never write out .pyc files in the first place.


### Manual Removal

- bash 4: `find -name '*.pyc' -delete`

- bash 3 (OS X default): `find . -name '*.pyc' -delete` (also works in bash 4)

- PyCharm: right-click the project directory; select `Clean Python Compiled Files`


### Git Post-Checkout Hook

[David Winterbottom](http://codeinthehole.com/writing/a-useful-git-post-checkout-hook-for-python-repos/) provides a script to clean up .pyc files and empty directories after a checkout. Save this code as `.git/hooks/post-checkout` and make it executable:

```bash
#!/usr/bin/env bash

# Delete .pyc files and empty directories from root of project
cd ./$(git rev-parse --show-cdup)

# Clean-up (OS X-specific)
find . -name ".DS_Store" -delete

NUM_PYC_FILES=$( find . -name "*.pyc" | wc -l | tr -d ' ' )
if [ $NUM_PYC_FILES -gt 0 ]; then
    find . -name "*.pyc" -delete
    printf "\e[00;31mDeleted $NUM_PYC_FILES .pyc files\e[00m\n"
fi

NUM_EMPTY_DIRS=$( find . -type d -empty | wc -l | tr -d ' ' )
if [ $NUM_EMPTY_DIRS -gt 0 ]; then
    find . -type d -empty -delete
    printf "\e[00;31mDeleted $NUM_EMPTY_DIRS empty directories\e[00m\n"
fi
```


### Prevention

It can make sense to prevent .pyc files entirely on a development machine: some modules may load slightly slower, but you might also avoid a lot of problems. To do so, set the environment variable `PYTHONDONTWRITEBYTECODE` (to any value).

From [the docs](https://docs.python.org/2/using/cmdline.html#envvar-PYTHONDONTWRITEBYTECODE):

> If this is set, Python won’t try to write .pyc or .pyo files on the import of source modules. This is equivalent to specifying the -B option.

However, the -B option does not appear to work with django (TODO: find out why). Setting `PYTHONDONTWRITEBYTECODE` does, though:

    $ find . -name '*.pyc' -delete
    $ export PYTHONDONTWRITEBYTECODE=1
    $ ./manage.py runserver
    $ find . -name '*.pyc'  # No .pyc files found.

This process may be simplified by adding `PYTHONDONTWRITEBYTECODE=1` to the environment settings of your run configuration:

- In PyCharm, select `Run -> Edit Configurations...`, select the configuration you use and the `Configuration` tab, click the `...` button next to `Environment variables:`, and add a Name `PYTHONDONTWRITEBYTECODE` with Value `1`. (Click `OK` on both modal windows to save the changes.)

- For a more general solution, add the line [`export PYTHONDONTWRITEBYTECODE=1` to your `~/.bashrc` file](http://unix.stackexchange.com/questions/107851/using-export-in-bashrc). Note that this will affect your Python system-wide, though, and might not be desirable.

- For a more targeted solution, add that line instead to your virtualenv's `bin/activate` file. This will disable writing .pyc files in that shell after you source the file, but will not unset the variable when you call `deactivate`; other shells will remain unaffected, though. If you really want your virtualenv to set and unset that variable transparently like it does for your `PATH`, make the following changes to its `bin/activate`:

    - Add this code anywhere:

            # Tell Python not to write out .pyc files.
            _OLD_PYTHONDONTWRITEBYTECODE="$PYTHONDONTWRITEBYTECODE"
            PYTHONDONTWRITEBYTECODE=1
            export PYTHONDONTWRITEBYTECODE

    - Add this code to the `deactivate` function:

            PYTHONDONTWRITEBYTECODE="$_OLD_PYTHONDONTWRITEBYTECODE"
            export PYTHONDONTWRITEBYTECODE
            unset _OLD_PYTHONDONTWRITEBYTECODE

        (Note: I do not know why the other similar actions in `deactivate` are conditional on `$_OLD_X` being nonempty; maybe there's a good reason, but it results in the variable remaining set after running `deactivate` if it was not set before sourcing `activate`, which seems like incorrect behavior.)


### Future Hope

These problems have been addressed in Python 3, which uses absolute import semantics by default.

- TODO: what happens in the case `from . import module` where `module.py` has been moved to another package but `__pycache__/module.[magic].pyc` persists?


## Further Reading

- [Alex Martelli's take on the "interpreted" (vs "compiled") nature of Python](http://stackoverflow.com/questions/2998215/if-python-is-interpreted-what-are-pyc-files/2998544#2998544)
- [PEP 328 -- Imports: Multi-Line and Absolute/Relative](http://legacy.python.org/dev/peps/pep-0328/)
- [PEP 3147 -- PYC Repository Directories](http://legacy.python.org/dev/peps/pep-3147/)
