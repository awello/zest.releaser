Uploading to pypi (or custom servers)
=======================================

When the (full)release command tries to upload your package to a pypi server,
zest.releaser basically just executes the command ``python setup.py register
sdist --formats=zip upload``.

For safety reasons zest.releaser will *only* offer to upload your package to
https://pypi.python.org when the package is already registered there.  If this
is not the case yet, you get a confirmation question whether you want to
create a new package.

If the ``setup.py register ...`` command fails, you probably need to configure
your PyPI configuration file. And of course you need to have
setuptools installed.


PyPI configuration file (``~/.pypirc``)
---------------------------------------

For uploads to PyPI to work you will need a ``.pypirc`` file in your home directory that
has your pypi login credentials, like this::

  [server-login]
  username:maurits
  password:secret


Uploading to other servers
--------------------------

Since python 2.6 you can specify multiple indexes for uploading your
package in your ``.pypirc``::

  [distutils]
  index-servers =
    pypi
    local

  [pypi]
  #pypi.python.org
  username:maurits
  password:secret

  [local]
  repository:http://localhost:8080/test/products/
  username:maurits
  password:secret
  # You may need to specify the realm, which is the domain the
  # server sends back when you do a challenge:
  #realm:Zope

See the `Python Packaging User Guide`_ for more info.

.. _`Python Packaging User Guide`: https://packaging.python.org/en/latest/distributing.html#uploading-your-project-to-pypi for more info.

When all this is configured correctly, zest.releaser will first register and
upload at the official PyPI (if the package is registered there already).
Then it will offer to upload to the other index servers that you have
specified in ``.pypirc``.

Note that since version 3.15, zest.releaser also looks for this information in
the setup.cfg if your package has that file.  One way to use this, is to
restrict the servers that zest.releaser will ask you upload to.  If you have
defined 40 index-servers in your pypirc but you have the following in your
setup.cfg, you will not be asked to upload to any server::

  [distutils]
  index-servers =

Note that after creating the tag we still ask you if you want to checkout that
tag for tweaks or pypi/distutils server upload.  We could add some extra
checks to see if that is really needed, but someone who does not have
index-servers listed, may still want to use an entry point like
`gocept.zestreleaser.customupload
<http://pypi.python.org/pypi/gocept.zestreleaser.customupload>`_ to do
uploading, or do some manual steps first before uploading.

Some people will hardly ever want to do a release on PyPI but in 99 out of 100
cases only want to create a tag.  They won't like the default answer of 'yes'
to that question of whether to create a checkout of the tag.  So since version
3.16 you can influence this default answer.  You can add some lines to the
``.pypirc`` file in your home directory to change the default answer for all
packages, or change it for individual packages in their ``setup.cfg`` file.
The lines are this::

  [zest.releaser]
  release = no

You can use no/false/off/0 or yes/true/on/1 as answers; upper, lower or mixed
case are all fine.


Uploading with twine
--------------------

Since version 4.0, we prefer twine_ for uploading to the Python
Package Index, because it is safer: it uses ``https`` for uploading.
If the ``twine`` command is available, it is used for uploading to
PyPI.  It is installed automatically if you use the
``zest.releaser[recommended]`` extra.  Note that if the ``twine``
command is not available, you may need to change your system ``PATH``
or need to install ``twine`` explicitly.  This seems more needed when
using ``zc.buildout`` than when using ``pip``.

Note that if your package is not yet on PyPI you must first register
it manually_.  We could use the ``python setup.py register`` command,
but that may use cleartext passwords, which defeats the purpose of
``twine``.  That may have been fixed meanwhile in `twine issue 21`_.

.. _twine: https://pypi.python.org/pypi/twine
.. _manually: https://pypi.python.org/pypi?%3Aaction=submit_form
.. _`twine issue 21`: https://github.com/pypa/twine/issues/21


Uploading wheels
----------------

First, you should install the ``zest.releaser[recommended]`` extra, or
run ``pip install wheel`` yourself next to ``zest.releaser``.  Then
create or edit ``setup.cfg`` in your project (or globally in your
``~/.pypirc``) and add this to create and upload a wheel to upload to
PyPI::

  [zest.releaser]
  create-wheel = yes

See http://pythonwheels.com for deciding whether this is a good idea
for your package.  Briefly, if it is a pure Python 2 *or* pure Python
3 package: just do it.


Adding extra text to a commit message
-------------------------------------

``zest.releaser`` makes commits in the prerelease and postrelease
phase.  Something like ``Preparing release 1.0`` and ``Back to
development: 1.1``.  You can add extra text to these messages by
configuration in your ``setup.cfg`` or global ``~/.pypirc``.  One use
case for this is telling Travis to skip Continuous Integration builds::

  [zest.releaser]
  extra-message = [ci skip]


Including all files in your release
-----------------------------------

By default, only the Python files and a ``README.txt`` are included (by
setuptools) when you make a release. So you miss out on your changelog, json
files, stylesheets and so on. There are two strategies to include those other
files:

- Add a ``MANIFEST.in`` file in the same directory as your ``setup.py`` that
  lists the files you want to include. Don't worry, wildcards are
  allowed. Actually, zest.releaser will suggest a sample ``MANIFEST.in`` for
  you if you don't already have it. The default is often good enough.

- Setuptools *can* detect which files are included in your version control
  system (svn, git, etc.) which it'll then automatically include.

The last approch has a problem: not every version control system is supported
out of the box. So you might need to install extra packages to get it to
work. So: use a ``MANIFEST.in`` file to spare you the trouble. If not, here
are some extra packages:

- setuptools-git (Setuptools plugin for finding files under Git
  version control)

- setuptools_hg (Setuptools plugin for finding files under Mercurial
  version control)

- setuptools_bzr (Setuptools plugin for finding files under Bazaar
  version control)

- setuptools_subversion (Setuptools plugin for finding files under
  Subversion version control.)  You probably need this when you
  upgrade to the recent subversion 1.7.  If you suddenly start missing
  files in the sdists you upload to PyPI you definitely need it.
  Alternatively: set up a proper MANIFEST.in as that method works with
  any version control system.

In general, if you are missing files in the uploaded package, the best
is to put a proper ``MANIFEST.in`` file next to your ``setup.py``.
See `zest.pocompile`_ for an example.

.. _`zest.pocompile`: http://pypi.python.org/pypi/zest.pocompile


Running automatically without input
-----------------------------------

Sometimes you want to run zest.releaser without hitting ``<enter>`` all the
time. You might want to run zest.releaser from your automatic test
environment, for instance. For that, there's the ``--no-input`` commandline
option. Pass that and all defaults will be accepted automatically.

This means your version number and so must be OK. If you want to have a
different version number from the one in your ``setup.py``, you'll need to
change it yourself by hand. And the next version number will be chosen
automatically, too. So ``1.2`` will become ``1.3``. This won't detect that you
might want to do a ``1.3`` after a ``1.2.1`` bugfix release, but we cannot
perform feats of magic in zest.releaser :-)

In case you always want to accept the defaults, a setting in your
``setup.cfg`` is available::

    [zest.releaser]
    no-input = yes

An important reminder: if you want to make sure you never upload anything
automatically to the python package index, include the ``release = no``
setting in ``setup.cfg``::

    [zest.releaser]
    no-input = yes
    release = no
