.. spelling::

  MyProfile


.. _conan_config:

conan config
============

.. code-block:: bash

    $ conan config [-h] {rm,set,get,install} ...

Manages Conan configuration. Edits the conan.conf or installs config files.

.. code-block:: text

    positional arguments:
      {rm,set,get,install}  sub-command help
        rm                  Remove an existing config element
        set                 Set a value for a configuration item
        get                 Get the value of configuration item
        install             install a full configuration from a local or remote
                            zip file

    optional arguments:
      -h, --help            show this help message and exit


**Examples**

- Change the logging level to 10:

  .. code-block:: bash

      $ conan config set log.level=10

- Get the logging level:

  .. code-block:: bash

      $ conan config get log.level
      $> 10

.. _conan_config_install:

conan config install
--------------------

The ``config install`` is intended to share the Conan client configuration. For example, in a company or organization,
is important to have common ``settings.yml``, ``profiles``, etc.

It can get its configuration files from a local or remote zip file, or a local directory. It then installs the files
in the local Conan configuration.

The configuration may contain all or a subset of the allowed configuration files. Only the files that are present will be
replaced. The only exception is the **conan.conf** file for which only the variables declared will be installed,
leaving the other variables unchanged.

This means for example that **profiles** and **plugins** files will be overwritten if already present, but no profile or
plugin file that the user has in the local machine will be deleted.

All the configuration files will be copied to the conan home directory.
These are the special files and the rules applied to merge them:

+--------------------------------+----------------------------------------------------------------------+
| File                           | How it is applied                                                    |
+================================+======================================================================+
| profiles/MyProfile             | Overrides the local ~/.conan/profiles/MyProfile if already exists    |
+--------------------------------+----------------------------------------------------------------------+
| settings.yml                   | Overrides the local ~/.conan/settings.yml                            |
+--------------------------------+----------------------------------------------------------------------+
| remotes.txt                    | Overrides remotes. Will remove remotes that are not present in file  |
+--------------------------------+----------------------------------------------------------------------+
| config/conan.conf              | Merges the variables, overriding only the declared variables         |
+--------------------------------+----------------------------------------------------------------------+

The file *remotes.txt* is the only file listed above which does not have a direct counterpart in
the ``~/.conan`` folder. Its format is a list of entries, one on each line, with the form

.. code-block:: text

    [remote name] [remote url] [bool]

where ``[bool]`` (either ``True`` or ``False``) indicates whether SSL should be used to verify that remote.

The local cache *registry.txt* file contains the remotes definitions, as well as the mapping from packages
to remotes. In general it is not a good idea to add it to the installed files. That being said, the remote
definitions part of the *registry.txt* file uses the format required for *remotes.txt*, so you may find it
provides a helpful starting point when writing a *remotes.txt* to be packaged in a Conan
client configuration.

The specified URL will be stored in the ``general.config_install`` variable of the ``conan.conf`` file,
so following calls to :command:`conan config install` command doesn't need to specify the URL.

**Examples**:

- Install the configuration from a URL:

  .. code-block:: bash

      $ conan config install http://url/to/some/config.zip

  Conan config command stores the specified URL in the conan.conf ``general.config_install`` variable.

- Install the configuration from a Git repository:

  .. code-block:: bash

      $ conan config install http://github.com/user/conan_config/.git

  You can also force the git download by using :command:`--type git` (in case it is not deduced from the URL automatically):

  .. code-block:: bash

      $ conan config install http://github.com/user/conan_config/.git --type git

- Install from a URL skipping SSL verification:

  .. code-block:: bash

      $ conan config install http://url/to/some/config.zip --verify-ssl=False

  This will disable the SSL check of the certificate. This option is defaulted to ``True``.

- Refresh the configuration again:

  .. code-block:: bash

      $ conan config install

  It's not needed to specify the url again, it is already stored.

- Install the configuration from a local path:

  .. code-block:: bash

      $ conan config install /path/to/some/config.zip
