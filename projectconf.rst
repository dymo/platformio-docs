..  Copyright 2014-present PlatformIO <contact@platformio.org>
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
       http://www.apache.org/licenses/LICENSE-2.0
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

.. _projectconf:

Project Configuration File ``platformio.ini``
=============================================

The Project configuration file is named ``platformio.ini``. This is a
`INI-style <http://en.wikipedia.org/wiki/INI_file>`_ file.

``platformio.ini`` has sections (each denoted by a ``[header]``) and
key / value pairs within the sections. Lines beginning with ``;``
are ignored and may be used to provide comments.

There are 2 system reserved sections:

* Base PlatformIO settings: :ref:`projectconf_section_platformio`
* Build Environment settings: :ref:`projectconf_section_env`

The other sections can be used by users, for example, for
:ref:`projectconf_dynamic_vars`. The sections and their allowable values are
described below.

.. contents::
  :depth: 2

.. _projectconf_dynamic_vars:

Dynamic variables
-----------------

.. versionadded:: 3.1

Dynamic variables/templates are useful when you have common configuration data
between build environments. For examples, common :ref:`projectconf_build_flags`
or project dependencies :ref:`projectconf_lib_deps`.

Each variable should have a next format: ``${<section>.<option>}``, where
``<section>`` is a value from ``[<section>]`` group, and ``<option>`` is a
first item from pair ``<option> = value``.

You can inject environment variable using ``env`` as a ``section``. For example,
``${env.HOME}``, etc.

* Variable can be applied only for the option's value
* Multiple variables are allowed
* The ``platformio`` and ``env`` sections are reserved and could not be used
  as custom section. Some good section names might be ``common`` or ``global``.

Example:

.. code-block:: ini

    [platformio]
    ; Unix
    lib_extra_dirs = ${env.HOME}/Documents/Arduino/libraries
    ; Windows
    lib_extra_dirs = ${env.HOMEDRIVE}${env.HOMEPATH}\Documents\Arduino\libraries


    [common]
    build_flags = -D VERSION=1.2.3 -D DEBUG=1
    lib_deps_builtin =
      SPI
      Wire
    lib_deps_external = ArduinoJson@>5.6.0

    [env:uno]
    platform = atmelavr
    framework = arduino
    board = uno
    build_flags = ${common.build_flags}
    lib_deps = ${common.lib_deps_builtin}, ${common.lib_deps_external}

    [env:nodemcuv2]
    platform = espressif8266
    framework = arduino
    board = nodemcuv2
    build_flags = ${common.build_flags} -DSSID_NAME=HELLO -DSSID_PASWORD=WORLD
    lib_deps =
      ${common.lib_deps_builtin}
      ${common.lib_deps_external}
      PubSubClient@2.6
      OneWire

.. _projectconf_section_platformio:

Section ``[platformio]``
------------------------

.. contents::
    :local:

A ``platformio`` section is used for overriding default configuration options

.. note::
    Relative path is allowed for directory option:

    * ``~`` will be expanded to user's home directory
    * ``../`` or ``..\`` go up to one folder

Options
~~~~~~~

.. _projectconf_pio_env_default:

``env_default``
^^^^^^^^^^^^^^^

:ref:`cmd_run` command processes all environments ``[env:***]`` by default
if :option:`platformio run --environment` option is not specified.
:ref:`projectconf_pio_env_default` allows to define environments which
should be processed by default.

Multiple environments are allowed if they *are separated with ", "
(comma+space)*. For example.

.. code-block:: ini

    [platformio]
    env_default = uno, nodemcu

    [env:uno]
    platform = atmelavr
    framework = arduino
    board = uno

    [env:nodemcu]
    platform = espressif8266
    framework = arduino
    board = nodemcu

    [env:teensy31]
    platform = teensy
    framework = arduino
    board = teensy31

    [env:lpmsp430g2553]
    platform = timsp430
    framework = energia
    board = lpmsp430g2553
    build_flags = -D LED_BUILTIN=RED_LED

.. _projectconf_pio_home_dir:

``home_dir``
^^^^^^^^^^^^

Is used to store platform toolchains, frameworks, global libraries for
:ref: `ldf`, service data and etc. The size of this folder will depend on
number of installed development platforms.

A default value is User's home directory:

* Unix ``~/.platformio``
* Windows ``%HOMEPATH%\.platformio``

This option can be overridden by global environment variable
:envvar:`PLATFORMIO_HOME_DIR`.

Example:

.. code-block:: ini

    [platformio]
    home_dir = /path/to/custom/pio/storage

.. _projectconf_pio_lib_dir:

``lib_dir``
^^^^^^^^^^^

You can put here your own/private libraries. The source code of each library
should be placed in separate directory, like
``lib/private_lib/[here are source files]``. This directory has the highest
priority for :ref:`ldf`.

A default value is ``lib`` that means that folder is located in the root of
project.

This option can be overridden by global environment variable
:envvar:`PLATFORMIO_LIB_DIR`.

For example, see how can be organized ``Foo`` and ``Bar`` libraries:

.. code::

    |--lib
    |  |--Bar
    |  |  |--docs
    |  |  |--examples
    |  |  |--src
    |  |     |- Bar.c
    |  |     |- Bar.h
    |  |--Foo
    |  |  |- Foo.c
    |  |  |- Foo.h
    |- platformio.ini
    |--src
       |- main.c


Then in ``src/main.c`` you should use:

.. code-block:: c

    #include <Foo.h>
    #include <Bar.h>

    // rest H/C/CPP code

PlatformIO will find your libraries automatically, configure preprocessor's
include paths and build them.

.. _projectconf_pio_libdeps_dir:

``libdeps_dir``
^^^^^^^^^^^^^^^

Internal storage where :ref:`librarymanager` will install project dependencies
(:ref:`projectconf_lib_deps`). A default value is ``.piolibdeps`` that means
that folder is located in the root of project.

This option can be overridden by global environment variable
:envvar:`PLATFORMIO_LIBDEPS_DIR`.

.. _projectconf_global_lib_extra_dirs:

``lib_extra_dirs``
^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.2

A list with global extra directories for a project where :ref:`ldf` will look
for libraries.

This option has the same behavior as :ref:`projectconf_lib_extra_dirs` option
for a specific build environment defined in ``[env:]`` section. The main
difference is that the option that is defined in ``[platofrmio]`` section
will be applied automatically for all ``[env:]`` sections.

For the possible values and examples please follow to :ref:`projectconf_lib_extra_dirs`.

.. _projectconf_pio_src_dir:

``src_dir``
^^^^^^^^^^^

A path to project's source directory. PlatformIO uses it for :ref:`cmd_run`
command. A default value is ``src`` that means that folder is located in the
root of project.

This option can be overridden by global environment variable
:envvar:`PLATFORMIO_SRC_DIR`.

.. note::
    This option is useful for people who migrate from Arduino/Energia IDEs where
    source directory should have the same name like the main source file.
    See `example <https://github.com/platformio/platformio-examples/tree/develop/atmelavr-and-arduino/arduino-own-src_dir>`__ project with own source directory.

.. _projectconf_pio_envs_dir:

``envs_dir``
^^^^^^^^^^^^

.. warning::
    **PLEASE DO NOT EDIT FILES IN THIS FOLDER**. PlatformIO will overwrite
    your changes on the next build. **THIS IS A CACHE DIRECTORY**.

*PlatformIO Build System* uses this folder for project
environments to store compiled object files, static libraries, firmwares and
other cached information. It allows PlatformIO to build source code extremely
fast!

*You can delete this folder without any risk!* If you modify :ref:`projectconf`,
then PlatformIO will remove this folder automatically. It will be created on the
next build operation.

A default value is ``.pioenvs`` that means that folder is located in the root of
project.

This option can be overridden by global environment variable
:envvar:`PLATFORMIO_ENVS_DIR`.

.. note::
    If you have any problems with building your Project environments which
    are defined in :ref:`projectconf`, then **TRY TO DELETE** this folder. In
    this situation you will remove all cached files without any risk.

.. _projectconf_pio_data_dir:

``data_dir``
^^^^^^^^^^^^

Data directory to store contents and :ref:`platform_espressif_uploadfs`.
A default value is ``data`` that means that folder is located in the root of
project.

This option can be overridden by global environment variable
:envvar:`PLATFORMIO_DATA_DIR`.

.. _projectconf_pio_test_dir:

``test_dir``
^^^^^^^^^^^^

Directory where :ref:`unit_testing` engine will look for the tests.
A default value is ``test`` that means that folder is located in the root of
project.

This option can be overridden by global environment variable
:envvar:`PLATFORMIO_TEST_DIR`.

.. _projectconf_pio_boards_dir:

``boards_dir``
^^^^^^^^^^^^^^

Custom board settings per project. You can change this path with your own.
A default value is ``boards`` that means that folder is located in the root of
project.

By default, PlatformIO looks for boards in this order:

1. Project :ref:`projectconf_pio_boards_dir`
2. Global :ref:`projectconf_pio_home_dir`/boards
3. Development platform :ref:`projectconf_pio_home_dir`/platforms/\*/boards.

This option can be overridden by global environment variable
:envvar:`PLATFORMIO_BOARDS_DIR`.

--------

.. _projectconf_section_env:

Section ``[env:NAME]``
----------------------

.. contents::
    :local:

A section with ``env:`` prefix is used to define virtual environment with
specific options that will be processed with :ref:`cmd_run` command. You can
define unlimited numbers of environments.

Each environment must have unique ``NAME``. The valid chars for ``NAME`` are

* letters ``a-z``
* numbers ``0-9``
* special char ``_`` (underscore)

For example, ``[env:hello_world]``.

General options
~~~~~~~~~~~~~~~

.. contents::
    :local:

.. _projectconf_env_platform:

``platform``
^^^^^^^^^^^^

:ref:`platforms` name.

PlatformIO allows to use specific version of platform using
`Semantic Versioning <http://semver.org>`_ (X.Y.Z=MAJOR.MINOR.PATCH).
Version specifications can take any of the following forms:

* ``0.1.2``: an exact version number. Use only this exact version
* ``^0.1.2``: any compatible version (exact version for ``0.x.x`` versions
* ``~0.1.2``: any version with the same major and minor versions, and an
  equal or greater patch version
* ``>0.1.2``: any version greater than ``0.1.2``. ``>=``, ``<``, and ``<=``
  are also possible
* ``>0.1.0,!=0.2.0,<0.3.0``: any version greater than ``0.1.0``, not equal to
  ``0.2.0`` and less than ``0.3.0``

Examples:

.. code-block:: ini

    [env:the_latest_version]
    platform = atmelavr

    [env:specific_major_version]
    platform = atmelavr@^0.1.2

    [env:specific_major_and_minor_version]
    platform = atmelavr@~0.1.2

.. _projectconf_env_framework:

``framework``
^^^^^^^^^^^^^

:ref:`frameworks` name.

The multiple frameworks are allowed, *split them with comma+space ", "*.

.. _projectconf_env_board:

``board``
^^^^^^^^^

*PlatformIO* has pre-configured settings for the most popular boards. You don't
need to specify ``board_mcu``, ``board_f_cpu``, ``upload_protocol`` or
``upload_speed`` options. Just define a ``board`` type and *PlatformIO* will
pre-fill options described above with appropriate values.

You can find the ``board`` type in *Boards* section of each :ref:`platforms` or
using `PlatformIO Embedded Boards Explorer <http://platformio.org/boards>`_.


Board options
~~~~~~~~~~~~~

.. contents::
    :local:

``board_mcu``
^^^^^^^^^^^^^

``board_mcu`` is a microcontroller(MCU) type that is used by compiler to
recognize MCU architecture. The correct type of ``board_mcu`` depends on
platform library. For example, the list of ``board_mcu`` for "megaAVR Devices"
is described `here <http://www.nongnu.org/avr-libc/user-manual/>`_.

The full list of ``board_mcu`` for the popular embedded platforms you can find
in *Boards* section of :ref:`platforms`. See "Microcontroller" column.

.. _projectconf_board_f_cpu:

``board_f_cpu``
^^^^^^^^^^^^^^^

An option ``board_f_cpu`` is used to define MCU frequency (Hertz, Clock). A
format of this option is ``C-like long integer`` value with ``L`` suffix. The
1 Hertz is equal to ``1L``, then 16 Mhz (Mega Hertz) is equal to ``16000000L``.

The full list of ``board_f_cpu`` for the popular embedded platforms you can
find in *Boards* section of :ref:`platforms`. See "Frequency" column. You can
overclock a board by specifying a ``board_f_cpu`` value other than the default.

.. _projectconf_board_f_flash:

``board_f_flash``
^^^^^^^^^^^^^^^^^

An option ``board_f_flash`` is used to define FLASH chip frequency (Hertz, Clock). A
format of this option is ``C-like long integer`` value with ``L`` suffix. The
1 Hertz is equal to ``1L``, then 40 Mhz (Mega Hertz) is equal to ``40000000L``.

This option isn't available for the all development platforms. The only
:ref:`platform_espressif8266` supports it.

.. _projectconf_board_flash_mode:

``board_flash_mode``
^^^^^^^^^^^^^^^^^^^^

Flash chip interface mode. This option isn't available for the all development
platforms. The only :ref:`platform_espressif8266` supports it.

Build options
~~~~~~~~~~~~~

.. contents::
    :local:

.. _projectconf_build_flags:

``build_flags``
^^^^^^^^^^^^^^^

These flags/options control preprocessing, compilation, assembly and linking
processes:

.. list-table::
    :header-rows:  1

    * - Format
      - Scope
      - Description
    * - ``-D name``
      - CPPDEFINES
      - Predefine *name* as a macro, with definition 1.
    * - ``-D name=definition``
      - CPPDEFINES
      - The contents of *definition* are tokenized and processed as if they
        appeared during translation phase three in a ``#define`` directive.
    * - ``-U name``
      - CPPDEFINES
      - Cancel any previous definition of *name*, either built in or provided
        with a ``-D`` option.
    * - ``-Wp,option``
      - CPPFLAGS
      - Bypass the compiler driver and pass *option* directly  through to the
        preprocessor
    * - ``-Wall``
      - CCFLAGS
      - Turns on all optional warnings which are desirable for normal code.
    * - ``-Werror``
      - CCFLAGS
      - Make all warnings into hard errors. Source code which triggers warnings will be rejected.
    * - ``-w``
      - CCFLAGS
      - Suppress all warnings, including those which GNU CPP issues by default.
    * - ``-include file``
      - CCFLAGS
      - Process *file* as if ``#include "file"`` appeared as the first line of
        the primary source file.
    * - ``-Idir``
      - CPPPATH
      - Add the directory *dir* to the list of directories to be searched
        for header files.
    * - ``-Wa,option``
      - ASFLAGS, CCFLAGS
      - Pass *option* as an option to the assembler. If *option* contains
        commas, it is split into multiple options at the commas.
    * - ``-Wl,option``
      - LINKFLAGS
      - Pass *option* as an option to the linker. If *option* contains
        commas, it is split into multiple options at the commas.
    * - ``-llibrary``
      - LIBS
      - Search the *library* named library when linking
    * - ``-Ldir``
      - LIBPATH
      - Add directory *dir* to the list of directories to be searched for
        ``-l``.

This option can be set by global environment variable
:envvar:`PLATFORMIO_BUILD_FLAGS`.

Example:

.. code-block:: ini

    [env:specific_defines]
    build_flags = -DFOO -DBAR=1 -DFLOAT_VALUE=1.23457e+07

    [env:string_defines]
    build_flags = '-DHELLO="World!"' '-DWIFI_PASS="My password"'

    [env:specific_inclibs]
    build_flags = -I/opt/include -L/opt/lib -lfoo

    [env:specific_ld_script]
    build_flags = -Wl,-T/path/to/ld_script.ld


.. _projectconf_dynamic_build_flags:

Dynamic build flags
'''''''''''''''''''

PlatformIO Core allows to run external command/script which outputs build flags.
PIO will automatically parse the output and append flags to a build environment.
**You can use any shell or programming language.**

This external command will be called on each :ref:`cmd_run` command before
building/uploading process.

Use Cases:

 * Macro with the latest VCS revision/tag "on-the-fly"
 * Generate dynamic headers (``*.h``)
 * Process media content before generating SPIFFS image
 * Make some changes to source code or related libraries

Example:

.. code-block:: ini

    [env:generate_flags_with_external_command]
    build_flags = !cmd_or_path_to_script


**Use Case: Get the latest GIT revision "on-the-fly"**

*Unix*

.. code-block:: ini

    [env:git_revision_macro]
    build_flags = !echo "-DPIO_SRC_REV="$(git rev-parse HEAD)

*Windows*

You need to create a separate file named ``print_git_rev.bat`` and place it
near ``platformio.ini``.

``platformio.ini``:

.. code-block:: ini

    [env:git_revision_macro]
    build_flags = !print_git_rev.bat

``print_git_rev.bat``:

.. code-block:: bat

    @echo off
    FOR /F "tokens=1 delims=" %%A in ('git rev-parse HEAD') do echo -DPIO_SRC_REV=%%A


--------------

For more detailed information about available flags/options go to:

* `Options to Request or Suppress Warnings
  <https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html>`_
* `Options for Debugging Your Program
  <https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html>`_
* `Options That Control Optimization
  <https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html>`_
* `Options Controlling the Preprocessor
  <https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html>`_
* `Passing Options to the Assembler
  <https://gcc.gnu.org/onlinedocs/gcc/Assembler-Options.html>`_
* `Options for Linking <https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html>`_
* `Options for Directory Search
  <https://gcc.gnu.org/onlinedocs/gcc/Directory-Options.html>`_

.. _projectconf_src_build_flags:

``src_build_flags``
^^^^^^^^^^^^^^^^^^^

An option ``src_build_flags`` has the same behavior like ``build_flags``
but will be applied only for the project source code from
:ref:`projectconf_pio_src_dir` directory.

This option can be set by global environment variable
:envvar:`PLATFORMIO_SRC_BUILD_FLAGS`.

.. _projectconf_build_unflags:

``build_unflags``
^^^^^^^^^^^^^^^^^

Remove base/initial flags which were set by development platform.

.. code-block:: ini

   [env:unflags]
   build_unflags = -Os -std=gnu++11
   build_flags = -O2

.. _projectconf_src_filter:

``src_filter``
^^^^^^^^^^^^^^

This option allows to specify which source files should be included/excluded
from build process. Filter supports 2 templates:

* ``+<PATH>`` include template
* ``-<PATH>`` exclude template

``PATH`` MAST BE related from :ref:`projectconf_pio_src_dir`. All patterns will
be applied in theirs order.
`GLOB Patterns <http://en.wikipedia.org/wiki/Glob_(programming)>`_ are allowed.

By default, ``src_filter`` is predefined to
``+<*> -<.git/> -<svn/> -<example/> -<examples/> -<test/> -<tests/>``,
that means "includes ALL files, then
exclude ``.git`` and ``svn`` repository folders, ``example`` ... folder.

This option can be set by global environment variable
:envvar:`PLATFORMIO_SRC_FILTER`.

.. _projectconf_targets:

``targets``
^^^^^^^^^^^

A list with targets which will be processed by :ref:`cmd_run` command by
default. You can enter more than one target separated with "space".

The list with available targets is located in :option:`platformio run --target`.

**Tip!** You can use these targets like an option to
:option:`platformio run --target` command. For example:

.. code-block:: bash

    # clean project
    platformio run -t clean

    # dump current build environment
    platformio run --target envdump

When no targets are defined, *PlatformIO* will build only sources by default.


Upload options
~~~~~~~~~~~~~~

.. contents::
    :local:

.. _projectconf_upload_port:

``upload_port``
^^^^^^^^^^^^^^^

This option is used by "uploader" tool when sending firmware to board via
``upload_port``. For example,

* ``/dev/ttyUSB0`` - Serial port (Unix-based OS)
* ``COM3`` - Serial port (Windows OS)
* ``192.168.0.13`` - IP address when using OTA
* ``/media/disk`` - physical path to media disk/flash drive
  (:ref:`framework_mbed` enabled boards)
* ``D:`` - physical path to media disk/flash drive (Windows OS).

If ``upload_port`` isn't specified, then *PlatformIO* will try to detect it
automatically.

To print all available serial ports use :ref:`cmd_device_list` command.

This option can be set by global environment variable
:envvar:`PLATFORMIO_UPLOAD_PORT`.

Please note that you can use Unix shell-style wildcards:

.. list-table::
    :header-rows:  1

    * - Pattern
      - Meaning

    * - ``*``
      - matches everything

    * - ``?``
      - matches any single character

    * - ``[seq]``
      - matches any character in seq

    * - ``[!seq]``
      - matches any character not in seq

Example:

.. code-block:: ini

    [env:uno]
    platform = atmelavr
    framework = arduino
    ; any port that stats with /dev/ttyUSB
    upload_port = /dev/ttyUSB*

    ; COM1 or COM3
    upload_port = COM[13]

``upload_protocol``
^^^^^^^^^^^^^^^^^^^

A protocol that "uploader" tool uses to talk to the board.

.. _projectconf_upload_speed:

``upload_speed``
^^^^^^^^^^^^^^^^

A connection speed (`baud rate <http://en.wikipedia.org/wiki/Baud>`_)
which "uploader" tool uses when sending firmware to board.

.. _projectconf_upload_flags:

``upload_flags``
^^^^^^^^^^^^^^^^

Extra flags for uploader. Will be added to the end of uploader command. If you
need to override uploader command or base flags please use :ref:`projectconf_extra_script`.

This option can be set by global environment variable
:envvar:`PLATFORMIO_UPLOAD_FLAGS`.

.. _projectconf_upload_resetmethod:

``upload_resetmethod``
^^^^^^^^^^^^^^^^^^^^^^

Specify reset method for "uploader" tool. This option isn't available for all
development platforms. The only :ref:`platform_espressif8266` supports it.

Library options
~~~~~~~~~~~~~~~

.. contents::
    :local:

.. _projectconf_lib_deps:

``lib_deps``
^^^^^^^^^^^^

.. versionadded:: 3.0
.. seealso::
    Please make sure to read :ref:`ldf` guide first.

Specify project dependencies that should be installed automatically to
:ref:`projectconf_pio_libdeps_dir` before environment processing.
Multiple dependencies are allowed (*multi-lines or separated with comma+space ", "*).

If you have multiple build environments that depend on the same libraries,
you can use :ref:`projectconf_dynamic_vars` to use common configuration.

**Valid forms**

.. code-block:: ini

  ; one line definition (comma + space)
  [env:myenv]
  lib_deps = LIBRARY_1, LIBRARY_2, LIBRARY_N

  ; multi-line definition
  [env:myenv2]
  lib_deps =
    LIBRARY_1
    LIBRARY_2
    LIBRARY_N

The each line with ``LIBRARY_1... LIBRARY_N`` will be passed automatically to
:ref:`cmd_lib_install` command. Please follow to :ref:`cmd_lib_install` for
detailed documentation about possible values.

Example:

.. code-block:: ini

  [env:depends_on_some_libs]
  lib_deps =
    13
    PubSubClient
    Json@~5.6,!=5.4
    https://github.com/gioblu/PJON.git#v2.0
    https://github.com/me-no-dev/ESPAsyncTCP.git

.. _projectconf_lib_ignore:

``lib_ignore``
^^^^^^^^^^^^^^

.. seealso::
    Please make sure to read :ref:`ldf` guide first.

Specify libraries which should be ignored by Library Dependency Finder.

The correct value for this option is library name (not
folder name). In the most cases, library name is pre-defined in manifest file
(:ref:`library_config`, ``library.properties``, ``module.json``). The multiple
library names are allowed, *split them with comma+space ", "*.

There is ability to ignore built-in :ref:`framework_mbed` libraries: mbed-rtos,
mbed-events, mbed-fs, mbed-net, mbed-rpc, mbed-dsp, mbed-USBHost, mbed-USBDevice.
See full list `here <https://github.com/platformio/builder-framework-mbed/blob/develop/mbed.py#L323>`__.

Example:

.. code-block:: ini

    [env:ignore_some_libs]
    lib_ignore = SPI, Ethernet, mbed-fs

.. _projectconf_lib_extra_dirs:

``lib_extra_dirs``
^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.0
.. seealso::
    Please make sure to read :ref:`ldf` guide first.

A list with extra directories/storages where :ref:`ldf` will
look for dependencies. Multiple paths are allowed. *Please separate them
using comma+space ", "*.

This option can be set by global environment variable
:envvar:`PLATFORMIO_LIB_EXTRA_DIRS` or using global ``[platformio]`` section
and :ref:`projectconf_global_lib_extra_dirs` option.

.. warning::
  This is a not direct path to library with source code. It should be the path
  to storage that contains libraries grouped by folders. For example,
  ``/extra/lib/storage/`` but not ``/extra/lib/storage/MyLibrary``.

Example:

.. code-block:: ini

    [env:custom_lib_dirs]
    lib_extra_dirs = /path/to/private/dir1,/path/to/private/dir2

.. _projectconf_lib_ldf_mode:

``lib_ldf_mode``
^^^^^^^^^^^^^^^^

.. versionadded:: 3.0
.. seealso::
    Please make sure to read :ref:`ldf` guide first.

This option specifies how does Library Dependency Finder should analyze
dependencies (``#include`` directives). See :ref:`ldf_mode` for details.

.. _projectconf_lib_compat_mode:

``lib_compat_mode``
^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.0
.. seealso::
    Please make sure to read :ref:`ldf` guide first.

Library compatibility mode allows to control strictness of Library Dependency
Finder. More details :ref:`ldf_compat_mode`.

By default, this value is set to ``lib_compat_mode = 1`` and means that LDF
will check only for framework compatibility.


Test options
~~~~~~~~~~~~

.. contents::
    :local:

.. _projectconf_test_ignore:

``test_ignore``
^^^^^^^^^^^^^^^

.. versionadded:: 3.0
.. seealso::
    Please make sure to read :ref:`unit_testing` guide first.

Ignore :ref:`unit_testing` tests where the name matches specified patterns.
Multiple names are allowed. *Please separate them using comma+space ", "*.
Also, you can ignore some tests using :option:`platformio test --ignore` command.

.. list-table::
    :header-rows:  1

    * - Pattern
      - Meaning

    * - ``*``
      - matches everything

    * - ``?``
      - matches any single character

    * - ``[seq]``
      - matches any character in seq

    * - ``[!seq]``
      - matches any character not in seq

**Example**

.. code-block:: ini

  [env:myenv]
  test_ignore = footest, bartest_*, test[13]

.. _projectconf_test_port:

``test_port``
^^^^^^^^^^^^^

This option is used as communication interface (Serial/UART) between PlatformIO
:ref:`unit_testing` Engine and target device. For example,

* ``/dev/ttyUSB0`` - Unix-based OS
* ``COM3`` - Windows OS

If ``test_port`` isn't specified, then *PlatformIO* will try to detect it
automatically.

To print all available serial ports use :ref:`cmd_device_list` command.


Debugging options
~~~~~~~~~~~~~~~~~

.. versionadded:: 3.4
.. seealso::
    Please make sure to read :ref:`debugging` guide first.

.. _projectconf_debug_tool:

``debug_tool``
^^^^^^^^^^^^^^

A name of debugging tool. This option is useful when board support more than
one debugger tool/adapter.

See available tools per board in :ref:`debugging`.

Example

.. code-block:: ini

    [env:debug]
    platform = ...
    board = ...
    debug_tool = jlink

.. _projectconf_debug_port:

``debug_port``
^^^^^^^^^^^^^^

A debugging port of a remote target. Could be a serial device or network address.
PlatformIO detects it automatically if is not specified.

For example:

* ``/dev/ttyUSB0`` - Unix-based OS
* ``COM3`` - Windows OS
* ``localhost:3333``


.. _projectconf_debug_init_cmds:

``debug_init_cmds``
^^^^^^^^^^^^^^^^^^^

Initial commands that will be passed to back-end debugger.

PlatformIO dynamically configures back-end debugger depending on a debug environment.
**Highly recommended to DO NOT override this option.**

For example, the custom initial commands for GDB:

.. code-block:: ini

    [env:debug]
    platform = ...
    board = ...
    debug_init_cmds =
      target remote $DEBUG_PORT
      file "$PROG_PATH"
      load "$PROG_PATH"
      monitor init
      monitor reset halt
      tb main

.. _projectconf_debug_extra_cmds:

``debug_extra_cmds``
^^^^^^^^^^^^^^^^^^^^

Extra commands that will be passed to back-end debugger after initialization.
For example, add custom breakpoint and load ``.gdbinit`` from a project directory
for GDB:

.. code-block:: ini

    [env:debug]
    platform = ...
    board = ...
    debug_extra_cmds =
      break main.cpp:13
      break foo.cpp:100
      source .gdbinit

.. note::

  **Initial Project Breakpoints**: Use ``break path/to/file:LINE_NUMBER`` to
  define initial breakpoints for debug environment. Multiple breakpoints are
  allowed.

  To save session breakpoints, please use ``save breakpoints [filename]``
  command in Debug Console. For example, ``save breakpoints .gdbinit``. Later,
  this file could be loaded via ``source [filename]`` command. See above.


Advanced options
~~~~~~~~~~~~~~~~

.. _projectconf_extra_script:

``extra_script``
^^^^^^^^^^^^^^^^

.. warning::

  This option is recommended for Advanced Users and requires Python language knowledges.

  We highly recommended to take a look at :ref:`projectconf_dynamic_build_flags`
  option where you can use any programming language. Also, this option is very
  good if you need to apply changes to the project before building/uploading
  process:

  * Macro with the latest VCS revision/tag "on-the-fly"
  * Generate dynamic headers (``*.h``)
  * Process media content before generating SPIFFS image
  * Make some changes to source code or related libraries

  More details :ref:`projectconf_dynamic_build_flags`.

.. contents::
    :local:

Allows to launch extra script using `SCons <http://www.scons.org>`_ software
construction tool. For more details please follow to "Construction Environments"
section of
`SCons documentation <http://www.scons.org/doc/production/HTML/scons-user.html#chap-environments>`_.

This option can be set by global environment variable
:envvar:`PLATFORMIO_EXTRA_SCRIPT`.

Take a look at the multiple snippets/answers for the user questions:

  - `#462 Split C/C++ build flags <https://github.com/platformio/platformio-core/issues/462#issuecomment-172667342>`_
  - `#365 Extra configuration for ESP8266 uploader <https://github.com/platformio/platformio-core/issues/365#issuecomment-163695011>`_
  - `#351 Specific reset method for ESP8266 <https://github.com/platformio/platformio-core/issues/351#issuecomment-161789165>`_
  - `#247 Specific options for avrdude <https://github.com/platformio/platformio-core/issues/247#issuecomment-118169728>`_.

Extra Linker Flags without ``-Wl,`` prefix
''''''''''''''''''''''''''''''''''''''''''

Sometimes you need to pass extra flags to GCC linker without ``Wl,``. You could
use :ref:`projectconf_build_flags` option but it will not work. PlatformIO
will not parse these flags to ``LINKFLAGS`` scope. In this case, simple
extra script will help:

``platformio.ini``:

.. code-block:: ini

    [env:env_extra_link_flags]
    platform = windows_x86
    extra_script = extra_script.py

``extra_script.py`` (place it near ``platformio.ini``):

.. code-block:: python

    Import('env')

    env.Append(
      LINKFLAGS=[
          "-static",
          "-static-libgcc",
          "-static-libstdc++"
      ]
    )

Custom Uploader
'''''''''''''''

Example, specify own upload command for :ref:`platform_atmelavr`:

``platformio.ini``:

.. code-block:: ini

    [env:env_custom_uploader]
    platform = atmelavr
    extra_script = /path/to/extra_script.py
    custom_option = hello

``extra_script.py``:

.. code-block:: python

    Import('env')
    from base64 import b64decode

    env.Replace(UPLOADHEXCMD='"$UPLOADER" ' + b64decode(ARGUMENTS.get("CUSTOM_OPTION")) + ' --uploader --flags')

    # uncomment line below to see environment variables
    # print env.Dump()
    # print ARGUMENTS

Before/Pre and After/Post actions
'''''''''''''''''''''''''''''''''

PlatformIO Build System has rich API that allows to attach different pre-/post
actions (hooks) using ``env.AddPreAction(target, callback)`` or
``env.AddPreAction(target, [callback1, callback2, ...])`` function. A first
argument ``target`` can be a name of target that is passed using
:option:`platformio run --target` command, a name of built-in targets
(buildprog, size, upload, program, buildfs, uploadfs, uploadfsota) or path
to file which PlatformIO processes (ELF, HEX, BIN, OBJ, etc.).

The example below demonstrates how to call different functions
when :option:`platformio run --target` is called with ``upload`` value.
`extra_script.py` file is located on the same level as ``platformio.ini``.

``platformio.ini``:

.. code-block:: ini

    [env:pre_and_post_hooks]
    extra_script = extra_script.py

``extra_script.py``:

.. code-block:: python

    Import("env")

    #
    # Upload actions
    #

    def before_upload(source, target, env):
        print "before_upload"
        # do some actions


    def after_upload(source, target, env):
        print "after_upload"
        # do some actions

    print "Current build targets", map(str, BUILD_TARGETS)

    env.AddPreAction("upload", before_upload)
    env.AddPostAction("upload", after_upload)

    #
    # Custom actions when building program/firmware
    #

    env.AddPreAction("buildprog", callback...)
    env.AddPostAction("buildprog", callback...)

    #
    # Custom actions for specific files/objects
    #

    env.AddPreAction("$BUILD_DIR/firmware.elf", [callback1, callback2,...])
    env.AddPostAction("$BUILD_DIR/firmware.hex", callback...)

    # custom action before building SPIFFS image. For example, compress HTML, etc.
    env.AddPreAction("$BUILD_DIR/spiffs.bin", callback...)

    # custom action for project's main.cpp
    env.AddPostAction("$BUILD_DIR/src/main.cpp.o", callback...)


-----------

.. _projectconf_examples:

Examples
--------

.. note::
    A full list with project examples can be found in
    `PlatformIO Repository <https://github.com/platformio/platformio-examples/tree/develop>`_.

1. :ref:`platform_atmelavr`: Arduino UNO board with auto pre-configured
   ``board_*`` and ``upload_*`` options (use only ``board`` option) and Arduino
   Wiring-based Framework

.. code-block:: ini

    [env:atmelavr_arduino_uno_board]
    platform = atmelavr
    framework = arduino
    board = uno

    ; enable auto-uploading
    targets = upload


2. :ref:`platform_atmelavr`: Embedded board that is based on ATmega168 MCU with
   "arduino" bootloader

.. code-block:: ini

    [env:atmelavr_atmega168_board]
    platform = atmelavr
    board_mcu = atmega168
    board_f_cpu = 16000000L

    upload_port = /dev/ttyUSB0
    ; for Windows OS
    ; upload_port = COM3
    upload_protocol = arduino
    upload_speed = 19200

    ; enable auto-uploading
    targets = upload


3. Upload firmware via USB programmer (USBasp) to :ref:`platform_atmelavr`
   microcontrollers

.. code-block:: ini

    [env:atmelavr_usbasp]
    platform = atmelavr
    framework = arduino
    board = pro8MHzatmega328
    upload_protocol = usbasp
    upload_flags = -Pusb -B5

Then upload firmware using target ``program`` for :option:`platformio run --target`.
command. To use other programmers see :ref:`atmelavr_upload_via_programmer`.


4. :ref:`platform_ststm32`: Upload firmware using GDB script ``upload.gdb``,
   `issue #175 <https://github.com/platformio/platformio-core/issues/175>`_

.. code-block:: ini

    [env:st_via_gdb]
    platform = ststm32
    board = armstrap_eagle512
    upload_protocol = gdb

Also, take a look at this article `Armstrap Eagle and PlatformIO <http://isobit.io/2015/08/08/armstrap.html>`_.

5. :ref:`platform_ststm32`: Upload firmware using ST-Link instead mbed's media
   disk

.. code-block:: ini

    [env:stlink_for_mbed]
    platform = ststm32
    board = disco_f100rb
    upload_protocol = stlink
