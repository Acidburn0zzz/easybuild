.. toctree::
     :maxdepth: 4

Using the EasyBuild command line
================================

Basic usage of EasyBuild is described in the following sections, covering the most important range of topics if you are new to EasyBuild.
 
Specifying builds
-----------------

By providing a single easyconfig file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
The most basic usage is to simply provide the name of an easyconfig file to ``eb``.
EasyBuild will (try and) locate the easyconfig file, and perform the installation as specified by that easyconfig file.
 
For example, to build and install GCC/4.8.3 (using the system compiler and EasyBuild/1.15.2 - this may take 30 minutes or so)::
 
  $ eb GCC-4.8.3.eb
  == temporary log file in case of crash /tmp/easybuild-0f0xKN/easybuild-oI1vAm.log
  == resolving dependencies ...
  == processing EasyBuild easyconfig /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/g/GCC/GCC-4.8.3.eb
  == building and installing GCC/4.8.3...
  == fetching files...
  == creating build dir, resetting environment...
  == unpacking...
  == patching...
  == preparing...
  == configuring...
  == building...
  == testing...
  == installing...
  == taking care of extensions...
  == packaging...
  == postprocessing...
  == sanity checking...
  == cleaning up...
  == creating module...
  == COMPLETED: Installation ended successfully
  == Results of the build can be found in the log file /home/example/.local/easybuild/software/gcc/4.8.3/easybuild/easybuild-GCC-4.8.3-20141029.013716.log
  == Build succeeded for 1 out of 1
  == temporary log file /tmp/easybuild-0f0xKN/easybuild-oI1vAm.log has been removed.
  == temporary directory /tmp/easybuild-0f0xKN has been removed.

  $ module load GCC/4.8.3
  $ which gcc
  /home/example/.local/easybuild/software/GCC/4.8.3/bin/gcc
 
All easyconfig file names' suffixes are ``.eb`` and follow format ``<name>-<version>-<toolchain>-<versionsuffix>``;
this is a crucial design aspect, since the dependency resolution mechanism (introduced below) relies upon this convention.
 
.. tip:: You may wish to modify the installation prefix (e.g., using ``--prefix`` or by defining ``$EASYBUILD_PREFIX``),
  in order to redefine the build/install/source path prefix to be used; default value is: ``$HOME/.local/easybuild``.

Via command line options
~~~~~~~~~~~~~~~~~~~~~~~~
 
An alternative approach is to only use command line options to specify which software to build.
Refer to the ``Software search and build options`` section in the ``eb --help`` output for an overview
of the available command line options for this purpose (cfr. :ref:`basic_usage_help`).
 
Here is how to build and install last version of bzip2 (that EasyBuild is aware of)
using ``dummy`` toolchain (i.e., using the system compiler)::
 
  $ eb --software-name=bzip2 --toolchain-name=dummy
  == temporary log file in case of crash /tmp/easybuild-3AzStZ/easybuild-YD_fMf.log
  == resolving dependencies ...
  == processing EasyBuild easyconfig /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/bzip2/bzip2-1.0.6.eb
  == building and installing bzip2/1.0.6...
  [...]
  == creating module...
  == COMPLETED: Installation ended successfully
  == Results of the build can be found in the log file /home/example/.local/easybuild/software/bzip2/1.0.6/easybuild/easybuild-bzip2-1.0.6-20141029.013514.log
  [...]
  
By providing a set of easyconfig files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
Multiple easyconfig files can be provided as well, either directly or by specifying a directory that contains easyconfig files.

For example, to build and install both bzip2 and GCC with a single command, simply list the easyconfigs for both on the
``eb`` command line (note that bzip2 is not being reinstalled, since a matching module is already available)::
 
  $ eb bzip2-1.0.6.eb GCC-4.8.3.eb
  == temporary log file in case of crash /tmp/easybuild-pGof8u/easybuild-GNYSey.log
  == bzip2/1.0.6 is already installed (module found), skipping
  == resolving dependencies ...
  == processing EasyBuild easyconfig /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/g/GCC/GCC-4.8.3.eb
  == building and installing GCC/4.8.3...
  == fetching files...
  == creating build dir, resetting environment...
  == unpacking...
  == patching...
  == preparing...
  == configuring...
  == building...
  == testing...
  == installing...
  == taking care of extensions...
  == packaging...
  == postprocessing...
  == sanity checking...
  == cleaning up...
  == creating module...
  == COMPLETED: Installation ended successfully
  == Results of the build can be found in the log file /home/example/.local/easybuild/software/GCC/4.8.3/easybuild/easybuild-GCC-4.8.3-20141029.024018.log
  == Build succeeded for 1 out of 1
  == temporary log file /tmp/easybuild-pGof8u/easybuild-GNYSey.log has been removed.
  == temporary directory /tmp/easybuild-pGof8u has been removed.


When one or more directories are provided, EasyBuild will (recursively) traverse them
to find easyconfig files. For example:

::

  $ find set_of_easyconfigs/ -type f             
  set_of_easyconfigs/GCC-4.8.3.eb
  set_of_easyconfigs/foo.txt
  set_of_easyconfigs/tools/bzip2-1.0.6.eb

::

  $ eb set_of_easyconfigs/
  == temporary log file in case of crash /tmp/easybuild-1yxCvv/easybuild-NeNmZr.log
  == bzip2/1.0.6 is already installed (module found), skipping
  == GCC/4.8.3 is already installed (module found), skipping
  == No easyconfigs left to be built.
  == Build succeeded for 0 out of 0
  == temporary log file /tmp/easybuild-1yxCvv/easybuild-NeNmZr.log has been removed.
  == temporary directory /tmp/easybuild-1yxCvv has been removed.
 
.. note:: EasyBuild will only pick up the files which end with ``.eb`` ; anything else will be ignored.
 
.. tip:: Calling EasyBuild is designed as an `idempotent` operation; 
  if a module is available that matches with an provided easyconfig file, the installation will simply be skipped.


Commonly used command line options
----------------------------------
 
``eb`` is EasyBuild’s main command line tool, to interact with the EasyBuild framework
and hereby the most common command line options are being documented.

Command line help, ``eb --help``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
Detailed information about the usage of the eb command is available via the ``--help``, ``-H``, ``-h`` help options.

Refer to page :ref:`basic_usage_help` for more detailed information.

.. note:: ``--help``/``-H`` spit out the long help info (i.e. including long option names), ``-h`` only includes short option names.
.. tip:: This is the best way to query for certain information, esp. recent features, since this is in sync with the actual EasyBuild version being used.

Report version
~~~~~~~~~~~~~~
 
You can query which EasyBuild version you are using with ``--version``::

  $ eb --version
  This is EasyBuild 1.15.2 (framework: 1.15.2, easyblocks: 1.15.2) on host example.local.

.. tip:: Asking EasyBuild to print its version is a quick way to ensure that ``$PYTHONPATH``
  is set up correctly, so that the entire EasyBuild installation (framework, easyblocks, easyconfigs) is available.

List of known toolchains
~~~~~~~~~~~~~~~~~~~~~~~~
 
For an overview of known toolchains, use ``eb --list-toolchains``.
 
Toolchains have brief mnemonic names, for example:

* ``goolf`` stands for ``GCC, OpenMPI, OpenBLAS/LAPACK, FFTW``
* ``iimpi`` stands for ``icc/ifort, impi``
* ``cgmvolf`` stands for ``Clang/GCC, MVAPICH2, OpenBLAS/LAPACK, FFTW``

The complete table of available toolchains is visible here: :ref:`toolchains_table`

List of available easyblocks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can obtain a list of available :ref:`easyblocks` via ``--list-easyblocks``.

The ``--list-easyblocks`` command line option prints the easyblocks in a hierarchical way,
showing the inheritance patterns, with the "base" easyblock class ``EasyBlock`` on top.

Software-specific easyblocks have a name that starts with ``EB_``; the ones that do not are generic easyblocks.
(cfr. :ref:`easyblocks` for the distinction between both types of easyblocks).

For example, a list of easyblocks can be obtained with::

  $ eb --list-easyblocks

Refer to page :ref:`basic_usage_easyblocks` for more information.

All available easyconfig parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

EasyBuild provides a significant amount of easyconfig parameters.
An overview of all available easyconfig parameters can be obtained via
``eb --avail-easyconfig-params``, or ``eb -a`` for short.

Refer to page :ref:`easyconfigs_parameters` for more information, the possible parameters are a very rich set.

Combine -a with ``--easyblock/-e`` to include parameters that are specific to a particular easyblock;
by default, the ones specific to the generic ConfigureMake easyblock are included. For example::

  $ eb -a -e EB_WRF

Enable debug logging
~~~~~~~~~~~~~~~~~~~~

Use ``eb --debug/-d`` to enable debug logging, to include all details of how EasyBuild performed a build in the log file::

  $ eb bzip2-1.0.6.eb -d

.. tip:: You may enable this by default via adding ``debug = True`` in your EasyBuild configuration file

.. note:: Debug log files are significantly larger than non-debug logs, so be aware.

Forced reinstallation
~~~~~~~~~~~~~~~~~~~~~

Use ``eb --force/-f`` to force the reinstallation of a given easyconfig/module.

.. warning:: Use with care, since the reinstallation of existing modules will be done without requesting confirmation first!

.. tip:: Combine --force with --dry-run to get a good view on which installations will be forced.
   (cfr. `Using dry-run to get an overview of planned installations`_)

Searching for easyconfigs
~~~~~~~~~~~~~~~~~~~~~~~~~

Use ``--search/-S`` (long vs short output) and an easyconfig filepath pattern, for `case-insensitive` search of easyconfigs. Example::

  $ eb --search blast
  == temporary log file in case of crash /tmp/easybuild-1qIvuB/easybuild-eYwxlR.log
  == Searching (case-insensitive) for 'blast' in /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.26-Linux_x86_64.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.27-goalf-1.1.0-no-OFED.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.27-goolf-1.4.10.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.27-ictce-4.0.6.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.27_ictce-fixes.patch
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.28-goolf-1.4.10-Python-2.7.3.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.28-goolf-1.4.10.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.28-ictce-4.1.13.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.28-ictce-5.3.0-Python-2.7.3.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.28-ictce-5.3.0.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/b/BLAST/BLAST-2.2.28_ictce-fixes.patch
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/m/mpiBLAST/mpiBLAST-1.6.0-goalf-1.1.0-no-OFED.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/m/mpiBLAST/mpiBLAST-1.6.0-goolf-1.4.10.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/m/mpiBLAST/mpiBLAST-1.6.0-ictce-4.0.6.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/m/mpiBLAST/mpiBLAST-1.6.0-ictce-5.2.0.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/m/mpiBLAST/mpiBLAST-1.6.0-ictce-5.3.0.eb
   * /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/m/mpiBLAST/mpiBLAST_disable-ncbi-X11-apps.patch
  == temporary log file /tmp/easybuild-1qIvuB/easybuild-eYwxlR.log has been removed.
  == temporary directory /tmp/easybuild-1qIvuB has been removed.

The same query with ``-S`` is far more readable, when there is a joint path that can be collapsed to a variable like ``$CFGS1``::

  eb -S blast
  == temporary log file in case of crash /tmp/easybuild-tMmLMz/easybuild-Qgfely.log
  == Searching (case-insensitive) for 'blast' in /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs
  CFGS1=/home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs
   * $CFGS1/b/BLAST/BLAST-2.2.26-Linux_x86_64.eb
   * $CFGS1/b/BLAST/BLAST-2.2.27-goalf-1.1.0-no-OFED.eb
   * $CFGS1/b/BLAST/BLAST-2.2.27-goolf-1.4.10.eb
   * $CFGS1/b/BLAST/BLAST-2.2.27-ictce-4.0.6.eb
   * $CFGS1/b/BLAST/BLAST-2.2.27_ictce-fixes.patch
   * $CFGS1/b/BLAST/BLAST-2.2.28-goolf-1.4.10-Python-2.7.3.eb
   * $CFGS1/b/BLAST/BLAST-2.2.28-goolf-1.4.10.eb
   * $CFGS1/b/BLAST/BLAST-2.2.28-ictce-4.1.13.eb
   * $CFGS1/b/BLAST/BLAST-2.2.28-ictce-5.3.0-Python-2.7.3.eb
   * $CFGS1/b/BLAST/BLAST-2.2.28-ictce-5.3.0.eb
   * $CFGS1/b/BLAST/BLAST-2.2.28_ictce-fixes.patch
   * $CFGS1/m/mpiBLAST/mpiBLAST-1.6.0-goalf-1.1.0-no-OFED.eb
   * $CFGS1/m/mpiBLAST/mpiBLAST-1.6.0-goolf-1.4.10.eb
   * $CFGS1/m/mpiBLAST/mpiBLAST-1.6.0-ictce-4.0.6.eb
   * $CFGS1/m/mpiBLAST/mpiBLAST-1.6.0-ictce-5.2.0.eb
   * $CFGS1/m/mpiBLAST/mpiBLAST-1.6.0-ictce-5.3.0.eb
   * $CFGS1/m/mpiBLAST/mpiBLAST_disable-ncbi-X11-apps.patch
  == temporary log file /tmp/easybuild-tMmLMz/easybuild-Qgfely.log has been removed.
  == temporary directory /tmp/easybuild-tMmLMz has been removed.

The supplied pattern is used to match easyconfig **filepaths**; that aspect can be exploited to trim down
the list of easyconfigs in the search result. For example, use ``/GCC`` to search for easyconfig files for GCC::

  $ eb -S /GCC-4.9
  == temporary log file in case of crash /tmp/easybuild-W40SsV/easybuild-7l96Cm.log
  == Searching (case-insensitive) for '/GCC-4.9' in /home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs
  CFGS1=/home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs/g/GCC
   * $CFGS1/GCC-4.9.0-CLooG-multilib.eb
   * $CFGS1/GCC-4.9.0-CLooG.eb
   * $CFGS1/GCC-4.9.0.eb
   * $CFGS1/GCC-4.9.1-CLooG-multilib.eb
   * $CFGS1/GCC-4.9.1-CLooG.eb
   * $CFGS1/GCC-4.9.1.eb
  == temporary log file /tmp/easybuild-W40SsV/easybuild-7l96Cm.log has been removed.
  == temporary directory /tmp/easybuild-W40SsV has been removed.

.. note:: By using a leading slash in front of a search pattern, as the last example,
  we filter out all the potential matches of easyconfigs that are built with the GCC toolchain.

.. tip:: Using ``--search`` has remarkably longer output in most cases, compared to ``-S``; the information is the same,
  however the paths towards the easyconfigs are fully expanded, taking lot of screen real estate for most people. 


Use robot for dependency resolution
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To make EasyBuild try and resolve dependencies, use the ``--robot/-r`` command line option, as follows::

  $ eb mpiBLAST-1.6.0-goolf-1.4.10.eb --robot
  [...]
  == building and installing GCC/4.7.2...
  [...]
  == building and installing hwloc/1.6.2-GCC-4.7.2...
  [...]
  == building and installing OpenMPI/1.6.4-GCC-4.7.2...
  [...]
  == building and installing gompi/1.4.10...
  [...]
  == building and installing OpenBLAS/0.2.6-gompi-1.4.10-LAPACK-3.4.2...
  [...]
  == building and installing FFTW/3.3.3-gompi-1.4.10...
  [...]
  == building and installing ScaLAPACK/2.0.2-gompi-1.4.10-OpenBLAS-0.2.6-LAPACK-3.4.2...
  [...]
  == building and installing goolf/1.4.10...
  [...]
  == building and installing mpiBLAST/1.6.0-goolf-1.4.10...
  [...]
  == Build succeeded for 10 out of 10

EasyBuild supports installing an entire software stack, including the required toolchain if needed, with a single ``eb`` invocation.

The dependency resolution mechanism will construct a full dependency graph for the software package(s)
being installed, after which a list of dependencies is composed for which no module is available yet.
Each of the retained dependencies will then be built and installed, in the required order as indicated by the dependency graph.

.. tip:: This is particularly useful for software packages that have an extensive list of dependencies,
  or when reinstalling software using a different compiler toolchain
  (you can use the ``--try-toolchain`` command line option in combination with ``--robot``).

Using dry-run to get an overview of planned installations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can do a "dry-run" overview by supplying ``-D/--dry-run`` (typically combined with ``--robot``, in the form of ``-Dr``)::

  $ eb mpiBLAST-1.6.0-goolf-1.4.10.eb -Dr
  == temporary log file in case of crash /tmp/easybuild-vyNQhw/easybuild-pO8EJv.log
  Dry run: printing build status of easyconfigs and dependencies
  CFGS=/home/example/.local/easybuild/software/EasyBuild/1.15.2/lib/python2.7/site-packages/easybuild_easyconfigs-1.15.2.0-py2.7.egg/easybuild/easyconfigs
   * [*] $CFGS/g/GCC/GCC-4.7.2.eb (module: GCC/4.7.2)
   * [*] $CFGS/h/hwloc/hwloc-1.6.2-GCC-4.7.2.eb (module: hwloc/1.6.2-GCC-4.7.2)
   * [*] $CFGS/o/OpenMPI/OpenMPI-1.6.4-GCC-4.7.2.eb (module: OpenMPI/1.6.4-GCC-4.7.2)
   * [*] $CFGS/g/gompi/gompi-1.4.10.eb (module: gompi/1.4.10)
   * [ ] $CFGS/o/OpenBLAS/OpenBLAS-0.2.6-gompi-1.4.10-LAPACK-3.4.2.eb (module: OpenBLAS/0.2.6-gompi-1.4.10-LAPACK-3.4.2)
   * [ ] $CFGS/f/FFTW/FFTW-3.3.3-gompi-1.4.10.eb (module: FFTW/3.3.3-gompi-1.4.10)
   * [ ] $CFGS/s/ScaLAPACK/ScaLAPACK-2.0.2-gompi-1.4.10-OpenBLAS-0.2.6-LAPACK-3.4.2.eb (module: ScaLAPACK/2.0.2-gompi-1.4.10-OpenBLAS-0.2.6-LAPACK-3.4.2)
   * [ ] $CFGS/g/goolf/goolf-1.4.10.eb (module: goolf/1.4.10)
   * [ ] $CFGS/m/mpiBLAST/mpiBLAST-1.6.0-goolf-1.4.10.eb (module: mpiBLAST/1.6.0-goolf-1.4.10)
  == temporary log file /tmp/easybuild-vyNQhw/easybuild-pO8EJv.log has been removed.
  == temporary directory /tmp/easybuild-vyNQhw has been removed.

Note how the different status symbols denote distinct handling states by EasyBuild:

* ``[ ]`` The build is not available, EasyBuild will deliver it
* ``[x]`` The build is available, EasyBuild will skip building this module
* ``[F]`` The build is available, however EasyBuild has been asked to force a rebuild and will do so

