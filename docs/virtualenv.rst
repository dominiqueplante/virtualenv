Introduction
------------

``virtualenv`` is a tool to create isolated Python environments.

The basic problem being addressed is one of dependencies and versions,
and indirectly permissions. Imagine you have an application that
needs version 1 of LibFoo, but another application requires version
2. How can you use both these applications?  If you install
everything into ``/usr/lib/python2.7/site-packages`` (or whatever your
platform's standard location is), it's easy to end up in a situation
where you unintentionally upgrade an application that shouldn't be
upgraded.

Or more generally, what if you want to install an application *and
leave it be*?  If an application works, any change in its libraries or
the versions of those libraries can break the application.

Also, what if you can't install packages into the global
``site-packages`` directory?  For instance, on a shared host.

In all these cases, ``virtualenv`` can help you. It creates an
environment that has its own installation directories, that doesn't
share libraries with other virtualenv environments (and optionally
doesn't access the globally installed libraries either).


Installation
------------

.. warning::

    We advise installing virtualenv-1.9 or greater. Prior to version 1.9, the
    pip included in virtualenv did not not download from PyPI over SSL.

.. warning::

    When using pip to install virtualenv, we advise using pip 1.3 or greater.
    Prior to version 1.3, pip did not download from PyPI over SSL.

.. warning::

    We advise against using easy_install to install virtualenv when using
    setuptools < 0.9.7, because easy_install didn't download from PyPI over SSL
    and was broken in some subtle ways.

To install globally with `pip` (if you have pip 1.3 or greater installed globally):

::

 $ [sudo] pip install virtualenv

Or to get the latest unreleased dev version:

::

 $ [sudo] pip install https://github.com/pypa/virtualenv/tarball/develop


To install globally from source:

::

 $ curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-X.X.tar.gz
 $ tar xvfz virtualenv-X.X.tar.gz
 $ cd virtualenv-X.X
 $ [sudo] python setup.py install


To *use* locally from source:

::

 $ curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-X.X.tar.gz
 $ tar xvfz virtualenv-X.X.tar.gz
 $ cd virtualenv-X.X
 $ python virtualenv.py myVE

.. note::

    The ``virtualenv.py`` script is *not* supported if run without the
    necessary pip/setuptools/virtualenv distributions available locally. All
    of the installation methods above include a ``virtualenv_support``
    directory alongside ``virtualenv.py`` which contains a complete set of
    pip and setuptools distributions, and so are fully supported.

Usage
-----

The basic usage is::

    $ virtualenv ENV

This creates ``ENV/lib/pythonX.X/site-packages``, where any libraries you
install will go. It also creates ``ENV/bin/python``, which is a Python
interpreter that uses this environment. Anytime you use that interpreter
(including when a script has ``#!/path/to/ENV/bin/python`` in it) the libraries
in that environment will be used.

It also installs `Setuptools
<http://peak.telecommunity.com/DevCenter/setuptools>`_ into the environment.

.. note::

  Virtualenv (<1.10) used to provide a ``--distribute`` option to use the
  setuptools fork Distribute_. Since Distribute has been merged back into
  setuptools this option is now no-op, it will always use the improved
  setuptools releases.

A new virtualenv also includes the `pip <http://pypi.python.org/pypi/pip>`_
installer, so you can use ``ENV/bin/pip`` to install additional packages into
the environment.

.. _Distribute: https://pypi.python.org/pypi/distribute

activate script
~~~~~~~~~~~~~~~

In a newly created virtualenv there will be a ``bin/activate`` shell
script. For Windows systems, activation scripts are provided for CMD.exe
and Powershell.

On Posix systems you can do::

    $ source bin/activate

This will change your ``$PATH`` so its first entry is the virtualenv's
``bin/`` directory. (You have to use ``source`` because it changes your
shell environment in-place.) This is all it does; it's purely a
convenience. If you directly run a script or the python interpreter
from the virtualenv's ``bin/`` directory (e.g. ``path/to/env/bin/pip``
or ``/path/to/env/bin/python script.py``) there's no need for
activation.

In some shells (notably the original Bourne shell) the ``source`` command does
not exist, but the ``.`` command (which does the same thing) can be used as an
alternative::

    $ . bin/activate

After activating an environment you can use the function ``deactivate`` to
undo the changes to your ``$PATH``.

The ``activate`` script will also modify your shell prompt to indicate
which environment is currently active. You can disable this behavior,
which can be useful if you have your own custom prompt that already
displays the active environment name. To do so, set the
``VIRTUAL_ENV_DISABLE_PROMPT`` environment variable to any non-empty
value before running the ``activate`` script.

On Windows you just do::

    > \path\to\env\Scripts\activate

And type `deactivate` to undo the changes.

Based on your active shell (CMD.exe or Powershell.exe), Windows will use
either activate.bat or activate.ps1 (as appropriate) to activate the
virtual environment. If using Powershell, see the notes about code signing
below.

.. note::

    If using Powershell, the ``activate`` script is subject to the
    `execution policies`_ on the system. By default on Windows 7, the system's
    excution policy is set to ``Restricted``, meaning no scripts like the
    ``activate`` script are allowed to be executed. But that can't stop us
    from changing that slightly to allow it to be executed.

    In order to use the script, you have to relax your system's execution
    policy to ``AllSigned``, meaning all scripts on the system must be
    digitally signed to be executed. Since the virtualenv activation
    script is signed by one of the authors (Jannis Leidel) this level of
    the execution policy suffices. As an administrator run::

        PS C:\> Set-ExecutionPolicy AllSigned

    Then you'll be asked to trust the signer, when executing the script.
    You will be prompted with the following::

        PS C:\> virtualenv .\foo
        New python executable in C:\foo\Scripts\python.exe
        Installing setuptools................done.
        Installing pip...................done.
        PS C:\> .\foo\scripts\activate

        Do you want to run software from this untrusted publisher?
        File C:\foo\scripts\activate.ps1 is published by E=jannis@leidel.info,
        CN=Jannis Leidel, L=Berlin, S=Berlin, C=DE, Description=581796-Gh7xfJxkxQSIO4E0
        and is not trusted on your system. Only run scripts from trusted publishers.
        [V] Never run  [D] Do not run  [R] Run once  [A] Always run  [?] Help
        (default is "D"):A
        (foo) PS C:\>

    If you select ``[A] Always Run``, the certificate will be added to the
    Trusted Publishers of your user account, and will be trusted in this
    user's context henceforth. If you select ``[R] Run Once``, the script will
    be run, but you will be prometed on a subsequent invocation. Advanced users
    can add the signer's certificate to the Trusted Publishers of the Computer
    account to apply to all users (though this technique is out of scope of this
    document).

    Alternatively, you may relax the system execution policy to allow running
    of local scripts without verifying the code signature using the following::

        PS C:\> Set-ExecutionPolicy RemoteSigned

    Since the ``activate.ps1`` script is generated locally for each virtualenv,
    it is not considered a remote script and can then be executed.

.. _`execution policies`: http://technet.microsoft.com/en-us/library/dd347641.aspx

The ``--system-site-packages`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you build with ``virtualenv --system-site-packages ENV``, your virtual
environment will inherit packages from ``/usr/lib/python2.7/site-packages``
(or wherever your global site-packages directory is).

This can be used if you have control over the global site-packages directory,
and you want to depend on the packages there. If you want isolation from the
global system, do not use this flag.


Environment variables and configuration files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

virtualenv can not only be configured by passing command line options such as
``--python`` but also by two other means:

- Environment variables

  Each command line option is automatically used to look for environment
  variables with the name format ``VIRTUALENV_<UPPER_NAME>``. That means
  the name of the command line options are capitalized and have dashes
  (``'-'``) replaced with underscores (``'_'``).

  For example, to automatically use a custom Python binary instead of the
  one virtualenv is run with you can also set an environment variable::

      $ export VIRTUALENV_PYTHON=/opt/python-3.3/bin/python
      $ virtualenv ENV

  It's the same as passing the option to virtualenv directly::

      $ virtualenv --python=/opt/python-3.3/bin/python ENV

  This also works for appending command line options, like ``--find-links``.
  Just leave an empty space between the passsed values, e.g.::

      $ export VIRTUALENV_EXTRA_SEARCH_DIR="/path/to/dists /path/to/other/dists"
      $ virtualenv ENV

  is the same as calling::

      $ virtualenv --extra-search-dir=/path/to/dists --extra-search-dir=/path/to/other/dists ENV

- Config files

  virtualenv also looks for a standard ini config file. On Unix and Mac OS X
  that's ``$HOME/.virtualenv/virtualenv.ini`` and on Windows, it's
  ``%APPDATA%\virtualenv\virtualenv.ini``.

  The names of the settings are derived from the long command line option,
  e.g. the option ``--python`` would look like this::

      [virtualenv]
      python = /opt/python-3.3/bin/python

  Appending options like ``--extra-search-dir`` can be written on multiple
  lines::

      [virtualenv]
      extra-search-dir =
          /path/to/dists
          /path/to/other/dists

Please have a look at the output of ``virtualenv --help`` for a full list
of supported options.

Windows Notes
~~~~~~~~~~~~~

Some paths within the virtualenv are slightly different on Windows: scripts and
executables on Windows go in ``ENV\Scripts\`` instead of ``ENV/bin/`` and
libraries go in ``ENV\Lib\`` rather than ``ENV/lib/``.

To create a virtualenv under a path with spaces in it on Windows, you'll need
the `win32api <http://sourceforge.net/projects/pywin32/>`_ library installed.

PyPy Support
~~~~~~~~~~~~

Beginning with virtualenv version 1.5 `PyPy <http://pypy.org>`_ is
supported. To use PyPy 1.4 or 1.4.1, you need a version of virtualenv >= 1.5.
To use PyPy 1.5, you need a version of virtualenv >= 1.6.1.

Creating Your Own Bootstrap Scripts
-----------------------------------

While this creates an environment, it doesn't put anything into the
environment. Developers may find it useful to distribute a script
that sets up a particular environment, for example a script that
installs a particular web application.

To create a script like this, call
``virtualenv.create_bootstrap_script(extra_text)``, and write the
result to your new bootstrapping script. Here's the documentation
from the docstring:

Creates a bootstrap script, which is like this script but with
extend_parser, adjust_options, and after_install hooks.

This returns a string that (written to disk of course) can be used
as a bootstrap script with your own customizations. The script
will be the standard virtualenv.py script, with your extra text
added (your extra text should be Python code).

If you include these functions, they will be called:

``extend_parser(optparse_parser)``:
    You can add or remove options from the parser here.

``adjust_options(options, args)``:
    You can change options here, or change the args (if you accept
    different kinds of arguments, be sure you modify ``args`` so it is
    only ``[DEST_DIR]``).

``after_install(options, home_dir)``:

    After everything is installed, this function is called. This
    is probably the function you are most likely to use. An
    example would be::

        def after_install(options, home_dir):
            if sys.platform == 'win32':
                bin = 'Scripts'
            else:
                bin = 'bin'
            subprocess.call([join(home_dir, bin, 'easy_install'),
                             'MyPackage'])
            subprocess.call([join(home_dir, bin, 'my-package-script'),
                             'setup', home_dir])

    This example immediately installs a package, and runs a setup
    script from that package.

Bootstrap Example
~~~~~~~~~~~~~~~~~

Here's a more concrete example of how you could use this::

    import virtualenv, textwrap
    output = virtualenv.create_bootstrap_script(textwrap.dedent("""
    import os, subprocess
    def after_install(options, home_dir):
        etc = join(home_dir, 'etc')
        if not os.path.exists(etc):
            os.makedirs(etc)
        subprocess.call([join(home_dir, 'bin', 'easy_install'),
                         'BlogApplication'])
        subprocess.call([join(home_dir, 'bin', 'paster'),
                         'make-config', 'BlogApplication',
                         join(etc, 'blog.ini')])
        subprocess.call([join(home_dir, 'bin', 'paster'),
                         'setup-app', join(etc, 'blog.ini')])
    """))
    f = open('blog-bootstrap.py', 'w').write(output)

Another example is available `here
<https://github.com/socialplanning/fassembler/blob/master/fassembler/create-venv-script.py>`_.


Using Virtualenv without ``bin/python``
---------------------------------------

Sometimes you can't or don't want to use the Python interpreter
created by the virtualenv. For instance, in a `mod_python
<http://www.modpython.org/>`_ or `mod_wsgi <http://www.modwsgi.org/>`_
environment, there is only one interpreter.

Luckily, it's easy. You must use the custom Python interpreter to
*install* libraries. But to *use* libraries, you just have to be sure
the path is correct. A script is available to correct the path. You
can setup the environment like::

    activate_this = '/path/to/env/bin/activate_this.py'
    execfile(activate_this, dict(__file__=activate_this))

This will change ``sys.path`` and even change ``sys.prefix``, but also allow
you to use an existing interpreter. Items in your environment will show up
first on ``sys.path``, before global items. However, global items will
always be accessible (as if the ``--system-site-packages`` flag had been used
in creating the environment, whether it was or not). Also, this cannot undo
the activation of other environments, or modules that have been imported.
You shouldn't try to, for instance, activate an environment before a web
request; you should activate *one* environment as early as possible, and not
do it again in that process.

Making Environments Relocatable
-------------------------------

Note: this option is somewhat experimental, and there are probably
caveats that have not yet been identified.

.. warning::

    The ``--relocatable`` option currently has a number of issues,
    and is not guaranteed to work in all circumstances. It is possible
    that the option will be deprecated in a future version of ``virtualenv``.

Normally environments are tied to a specific path. That means that
you cannot move an environment around or copy it to another computer.
You can fix up an environment to make it relocatable with the
command::

    $ virtualenv --relocatable ENV

This will make some of the files created by setuptools use relative paths,
and will change all the scripts to use ``activate_this.py`` instead of using
the location of the Python interpreter to select the environment.

**Note:** scripts which have been made relocatable will only work if
the virtualenv is activated, specifically the python executable from
the virtualenv must be the first one on the system PATH. Also note that
the activate scripts are not currently made relocatable by
``virtualenv --relocatable``.

**Note:** you must run this after you've installed *any* packages into
the environment. If you make an environment relocatable, then
install a new package, you must run ``virtualenv --relocatable``
again.

Also, this **does not make your packages cross-platform**. You can
move the directory around, but it can only be used on other similar
computers. Some known environmental differences that can cause
incompatibilities: a different version of Python, when one platform
uses UCS2 for its internal unicode representation and another uses
UCS4 (a compile-time option), obvious platform changes like Windows
vs. Linux, or Intel vs. ARM, and if you have libraries that bind to C
libraries on the system, if those C libraries are located somewhere
different (either different versions, or a different filesystem
layout).

If you use this flag to create an environment, currently, the
``--system-site-packages`` option will be implied.

The ``--extra-search-dir`` option
---------------------------------

This option allows you to provide your own versions of setuptools and/or
pip to use instead of the embedded versions that come with virtualenv.

To use this feature, pass one or more ``--extra-search-dir`` options to
virtualenv like this::

    $ virtualenv --extra-search-dir=/path/to/distributions ENV

The ``/path/to/distributions`` path should point to a directory that contains
setuptools and/or pip wheels.

virtualenv will look for wheels in the specified directories, but will use
pip's standard algorithm for selecting the wheel to install, which looks for
the latest compatible wheel.

As well as the extra directories, the search order includes:

#. The ``virtualenv_support`` directory relative to virtualenv.py
#. The directory where virtualenv.py is located.
#. The current directory.

If no satisfactory local distributions are found, virtualenv will
fail. Virtualenv will never download packages.


Compare & Contrast with Alternatives
------------------------------------

There are several alternatives that create isolated environments:

* ``workingenv`` (which I do not suggest you use anymore) is the
  predecessor to this library. It used the main Python interpreter,
  but relied on setting ``$PYTHONPATH`` to activate the environment.
  This causes problems when running Python scripts that aren't part of
  the environment (e.g., a globally installed ``hg`` or ``bzr``). It
  also conflicted a lot with Setuptools.

* `virtual-python
  <http://peak.telecommunity.com/DevCenter/EasyInstall#creating-a-virtual-python>`_
  is also a predecessor to this library. It uses only symlinks, so it
  couldn't work on Windows. It also symlinks over the *entire*
  standard library and global ``site-packages``. As a result, it
  won't see new additions to the global ``site-packages``.

  This script only symlinks a small portion of the standard library
  into the environment, and so on Windows it is feasible to simply
  copy these files over. Also, it creates a new/empty
  ``site-packages`` and also adds the global ``site-packages`` to the
  path, so updates are tracked separately. This script also installs
  Setuptools automatically, saving a step and avoiding the need for
  network access.

* `zc.buildout <http://pypi.python.org/pypi/zc.buildout>`_ doesn't
  create an isolated Python environment in the same style, but
  achieves similar results through a declarative config file that sets
  up scripts with very particular packages. As a declarative system,
  it is somewhat easier to repeat and manage, but more difficult to
  experiment with. ``zc.buildout`` includes the ability to setup
  non-Python systems (e.g., a database server or an Apache instance).

I *strongly* recommend anyone doing application development or
deployment use one of these tools.

Contributing
------------

Refer to the `pip development`_ documentation - it applies equally to
virtualenv, except that virtualenv issues should filed on the `virtualenv
repo`_ at GitHub.

Virtualenv's release schedule is tied to pip's -- each time there's a new pip
release, there will be a new virtualenv release that bundles the new version of
pip.

Files in the `virtualenv_embedded/` subdirectory are embedded into
`virtualenv.py` itself as base64-encoded strings (in order to support
single-file use of `virtualenv.py` without installing it). If your patch
changes any file in `virtualenv_embedded/`, run `bin/rebuild-script.py` to
update the embedded version of that file in `virtualenv.py`; commit that and
submit it as part of your patch / pull request.

.. _pip development: http://www.pip-installer.org/en/latest/development.html
.. _virtualenv repo: https://github.com/pypa/virtualenv/

Running the tests
~~~~~~~~~~~~~~~~~

Virtualenv's test suite is small and not yet at all comprehensive, but we aim
to grow it.

The easy way to run tests (handles test dependencies automatically)::

    $ python setup.py test

If you want to run only a selection of the tests, you'll need to run them
directly with nose instead. Create a virtualenv, and install required
packages::

    $ pip install nose mock

Run nosetests::

    $ nosetests

Or select just a single test file to run::

    $ nosetests tests.test_virtualenv


Other Documentation and Links
-----------------------------

* James Gardner has written a tutorial on using `virtualenv with
  Pylons
  <http://wiki.pylonshq.com/display/pylonscookbook/Using+a+Virtualenv+Sandbox>`_.

* `Blog announcement
  <http://blog.ianbicking.org/2007/10/10/workingenv-is-dead-long-live-virtualenv/>`_.

* Doug Hellmann wrote a description of his `command-line work flow
  using virtualenv (virtualenvwrapper)
  <http://www.doughellmann.com/articles/CompletelyDifferent-2008-05-virtualenvwrapper/index.html>`_
  including some handy scripts to make working with multiple
  environments easier. He also wrote `an example of using virtualenv
  to try IPython
  <http://www.doughellmann.com/articles/CompletelyDifferent-2008-02-ipython-and-virtualenv/index.html>`_.

* Chris Perkins created a `showmedo video including virtualenv
  <http://showmedo.com/videos/video?name=2910000&fromSeriesID=291>`_.

* `Using virtualenv with mod_wsgi
  <http://code.google.com/p/modwsgi/wiki/VirtualEnvironments>`_.

* `virtualenv commands
  <https://github.com/thisismedium/virtualenv-commands>`_ for some more
  workflow-related tools around virtualenv.

Status and License
------------------

``virtualenv`` is a successor to `workingenv
<http://cheeseshop.python.org/pypi/workingenv.py>`_, and an extension
of `virtual-python
<http://peak.telecommunity.com/DevCenter/EasyInstall#creating-a-virtual-python>`_.

It was written by Ian Bicking, sponsored by the `Open Planning
Project <http://openplans.org>`_ and is now maintained by a
`group of developers <https://github.com/pypa/virtualenv/raw/master/AUTHORS.txt>`_.
It is licensed under an
`MIT-style permissive license <https://github.com/pypa/virtualenv/raw/master/LICENSE.txt>`_.
