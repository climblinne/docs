.. _autotools_reference:

AutoToolsBuildEnvironment (configure/make)
==========================================

If you are using **configure**/**make** you can use **AutoToolsBuildEnvironment** helper.
This helper sets ``LIBS``, ``LDFLAGS``, ``CFLAGS``, ``CXXFLAGS`` and ``CPPFLAGS`` environment variables based on your requirements.

.. code-block:: python
   :emphasize-lines: 13, 14, 15
   
   from conans import ConanFile, AutoToolsBuildEnvironment

   class ExampleConan(ConanFile):
      settings = "os", "compiler", "build_type", "arch"
      requires = "Poco/1.9.0@pocoproject/stable"
      default_options = {"Poco:shared": True, "OpenSSL:shared": True}
     
      def imports(self):
         self.copy("*.dll", dst="bin", src="bin")
         self.copy("*.dylib*", dst="bin", src="lib")
   
      def build(self):
         autotools = AutoToolsBuildEnvironment(self)
         autotools.configure()
         autotools.make()

It also works using the :ref:`environment_append <environment_append_tool>` context manager applied to your **configure and make** commands,
calling `configure` and `make` manually:

.. code-block:: python

    from conans import ConanFile, AutoToolsBuildEnvironment

    class ExampleConan(ConanFile):
        ...

        def build(self):
            env_build = AutoToolsBuildEnvironment(self)
            with tools.environment_append(env_build.vars):
                self.run("./configure")
                self.run("make")

You can change some variables like ``fpic``, ``libs``, ``include_paths`` and ``defines`` before accessing the ``vars`` to override
an automatic value or add new values:

.. code-block:: python
   :emphasize-lines: 8, 9, 10

    from conans import ConanFile, AutoToolsBuildEnvironment

    class ExampleConan(ConanFile):
        ...

        def build(self):
            env_build = AutoToolsBuildEnvironment(self)
            env_build.fpic = True
            env_build.libs.append("pthread")
            env_build.defines.append("NEW_DEFINE=23")
            env_build.configure()
            env_build.make()

You can use it also with ``MSYS2``/``MinGW`` subsystems installed by setting the `win_bash` parameter
in the constructor. It will run the the ``configure`` and ``make`` commands inside a ``bash`` that
has to be in the path or declared in ``CONAN_BASH_PATH``:


.. code-block:: python
   :emphasize-lines: 12

   from conans import ConanFile, AutoToolsBuildEnvironment
   import platform

   class ExampleConan(ConanFile):
      settings = "os", "compiler", "build_type", "arch"

      def imports(self):
        self.copy("*.dll", dst="bin", src="bin")
        self.copy("*.dylib*", dst="bin", src="lib")

      def build(self):
         in_win = platform.system() == "Windows"
         env_build = AutoToolsBuildEnvironment(self, win_bash=in_win)
         env_build.configure()
         env_build.make()

Constructor
-----------

.. code-block:: python

    class AutoToolsBuildEnvironment(object):

        def __init__(self, conanfile, win_bash=False)

Parameters:
    - **conanfile** (Required): Conanfile object. Usually ``self`` in a conanfile.py
    - **win_bash**: (Optional, Defaulted to ``False``): When True, it will run the configure/make commands inside a bash.

Attributes
----------

You can adjust the automatically filled values modifying the attributes like this:

.. code-block:: python
   :emphasize-lines: 8, 9, 10

    from conans import ConanFile, AutoToolsBuildEnvironment

    class ExampleConan(ConanFile):
        ...

        def build(self):
            autotools = AutoToolsBuildEnvironment(self)
            autotools.fpic = True
            autotools.libs.append("pthread")
            autotools.defines.append("NEW_DEFINE=23")
            autotools.configure()
            autotools.make()

fpic
++++

**Defaulted to**: ``True`` if ``fPIC`` option exists and ``True`` or when ``fPIC`` exists and
                  ``False`` but option ``shared`` exists and ``True``. Otherwise ``None``.

Set it to ``True`` if you want to append the ``-fPIC`` flag.

libs
++++

List with library names of the requirements (``-l`` in ``LIBS``).

include_paths
+++++++++++++

List with the include paths of the requires (-I in CPPFLAGS).

library_paths
+++++++++++++

List with library paths of the requirements  (-L in LDFLAGS).

defines
+++++++

List with variables that will be defined with ``-D``  in ``CPPFLAGS``.

flags
+++++

List with compilation flags (``CFLAGS`` and ``CXXFLAGS``).

cxx_flags
+++++++++

List with only C++ compilation flags (``CXXFLAGS``).

link_flags
++++++++++

List with linker flags

Properties
----------

vars
++++

Environment variables ``CPPFLAGS``, ``CXXFLAGS``, ``CFLAGS``, ``LDFLAGS``, ``LIBS`` generated by the build helper to use them in the
configure, make and install steps. This variables are generated dynamically with the values of the attributes and can also be modified to be
used in the following configure, make or install steps:

.. code-block:: python

    def build():
        auotools = AutoToolsBuildEnvironment()
        autotools.fpic = True
        env_build_vars = autotools.vars
        env_build_vars['RCFLAGS'] = '-O COFF'
        autotools.configure(vars=env_build_vars)
        autotools.make(vars=env_build_vars)
        autotools.install(vars=env_build_vars)

vars_dict
+++++++++

Same behavior as ``vars`` but this property returns each variable ``CPPFLAGS``, ``CXXFLAGS``, ``CFLAGS``, ``LDFLAGS``, ``LIBS`` as
dictionaries.

Methods
-------

configure()
+++++++++++

.. code-block:: python

    def configure(self, configure_dir=None, args=None, build=None, host=None, target=None,
                  pkg_config_paths=None, vars=None)

Configures `Autotools` project with the given parameters.

.. important::

    This method sets by default the ``--prefix`` argument to ``self.package_folder`` whenever ``--prefix`` is not provided in the ``args``
    parameter during the configure step.

    There are other flags set automatically to fix the install directories by default:

    - ``--bindir``, ``--sbin`` and ``--libexec`` set to *bin* folder.
    - ``--libdir`` set to *lib* folder.
    - ``--includedir``, ``--oldincludedir`` set to *include* folder.
    - ``--datarootdir`` set to *share* folder.

    These flags will be set on demand, so only the available options in the *./configure* are actually set. They can also be totally skipped
    using ``use_default_install_dirs=False`` as described in the section below.

.. _autotools_lib64_warning:

.. warning::

    From Conan 1.8 this build helper sets the output library directory via ``--libdir`` automatically to ``${prefix}/lib``. This means that
    if you are using the ``install()`` to package with AutoTools, library artifacts will be stored in the ``lib`` directory unless indicated
    explicitly by the user.

    This change was introduced in order to fix issues detected in some Linux distributions where libraries were being installed to the
    ``lib64`` folder (instead of ``lib``) when rebuilding a package from sources. In those cases, if ``package_info()`` was declaring
    ``self.cpp_info.libdirs`` as ``lib``, the consumption of the package was broken.

    This was considered a bug in the build helper, as it should be as much deterministic as possible when building the same package for the
    same settings and generally for any other user input.

    If you were already modelling the ``lib64`` folder in your recipe, make sure you use ``lib`` for ``self.cpp_info.libdirs`` or inject
    the argument in the Autotools' ``configure()`` method:

    .. code-block:: python

        atools = AutoToolsBuildEnvironment()
        atools.configure(args=["--libdir=${prefix}/lib64"])
        atools.install()

    You can also skip its default value using the parameter ``use_default_install_dirs=False``.

Parameters:
    - **configure_dir** (Optional, Defaulted to ``None``): Directory where the ``configure`` script is. If ``None``, it will use the current
      directory.
    - **args** (Optional, Defaulted to ``None``): A list of additional arguments to be passed to the ``configure`` script. Each argument
      will be escaped according to the current shell. ``--prefix`` and ``--libdir``, will be adjusted automatically if not indicated
      specifically.
    - **build** (Optional, Defaulted to ``None``): To specify a value for the parameter ``--build``. If ``None`` it will try to detect the
      value if cross-building is detected according to the settings. If ``False``, it will not use this argument at all.
    - **host** (Optional, Defaulted to ``None``): To specify a value for the parameter ``--host``. If ``None`` it will try to detect the
      value if cross-building is detected according to the settings. If ``False``, it will not use this argument at all.
    - **target** (Optional, Defaulted to ``None``): To specify a value for the parameter ``--target``. If ``None`` it will try to detect the
      value if cross-building is detected according to the settings. If ``False``, it will not use this argument at all.
    - **pkg_config_paths** (Optional, Defaulted to ``None``): Specify folders (in a list) of relative paths to the install folder or
      absolute ones where to find ``*.pc`` files (by using the env var ``PKG_CONFIG_PATH``). If ``None`` is specified but the conanfile is
      using the ``pkg_config`` generator, the ``self.install_folder`` will be added to the ``PKG_CONFIG_PATH`` in order to locate the pc
      files of the requirements of the conanfile.
    - **vars** (Optional, Defaulted to ``None``): Overrides custom environment variables in the configure step.
    - **use_default_install_dirs** (Optional, Defaulted to ``True``): Use or not the defaulted installation dirs such as ``--libdir``,
      ``--bindir``...

make()
++++++

.. code-block:: python

    def make(self, args="", make_program=None, target=None, vars=None)

Builds `Autotools` project with the given parameters.

Parameters:
    - **args** (Optional, Defaulted to ``""``): A list of additional arguments to be passed to the ``make`` command. Each argument will be
      escaped accordingly to the current shell. No extra arguments will be added if ``args=""``.
    - **make_program** (Optional, Defaulted to ``None``): Allows to specify a different ``make`` executable, e.g., ``mingw32-make``. The
      environment variable :ref:`CONAN_MAKE_PROGRAM<conan_make_program>` can be used too.
    - **target** (Optional, Defaulted to ``None``): Choose which target to build. This allows building of e.g., docs, shared libraries or
      install for some AutoTools projects.
    - **vars** (Optional, Defaulted to ``None``): Overrides custom environment variables in the make step.

install()
+++++++++

.. code-block:: python

    def install(self, args="", make_program=None, vars=None)

Performs the install step of autotools calling ``make(target="install")``.

Parameters:
    - **args** (Optional, Defaulted to ``""``): A list of additional arguments to be passed to the ``make`` command. Each argument will be
      escaped accordingly to the current shell. No extra arguments will be added if ``args=""``.
    - **make_program** (Optional, Defaulted to ``None``): Allows to specify a different ``make`` executable, e.g., ``mingw32-make``. The
      environment variable :ref:`CONAN_MAKE_PROGRAM<conan_make_program>` can be used too.
    - **vars** (Optional, Defaulted to ``None``): Overrides custom environment variables in the install step.

Environment variables
---------------------

The following environment variables will also affect the `AutoToolsBuildEnvironment` helper class.

+--------------------+-------------------------------------------------------------------------------------+
| NAME               | DESCRIPTION                                                                         |
+====================+=====================================================================================+
| LIBS               | Library names to link                                                               |
+--------------------+-------------------------------------------------------------------------------------+
| LDFLAGS            | Link flags, (-L, -m64, -m32)                                                        |
+--------------------+-------------------------------------------------------------------------------------+
| CFLAGS             | Options for the C compiler (-g, -s, -m64, -m32, -fPIC)                              |
+--------------------+-------------------------------------------------------------------------------------+
| CXXFLAGS           | Options for the C++ compiler (-g, -s, -stdlib, -m64, -m32, -fPIC, -std)             |
+--------------------+-------------------------------------------------------------------------------------+
| CPPFLAGS           | Preprocessor definitions (-D, -I)                                                   |
+--------------------+-------------------------------------------------------------------------------------+

.. seealso::

    - :ref:`Reference/Tools/environment_append <environment_append_tool>`
