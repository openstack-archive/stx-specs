..  This work is licensed under a Creative Commons Attribution 3.0 Unported
    License.
    http://creativecommons.org/licenses/by/3.0/legalcode

======================
Create stx-mirror tool
======================

Storyboard: https://storyboard.openstack.org/#!/story/2004413

Create a new tool called *stx-mirror* in favor of the existing
*download_mirror.sh* and helpers to improve download time, integrity checking,
logging and error handling.


Problem description
===================

The *download_mirror.sh* script was a short-term solution in the context of
the StarlingX opensource effort that helped to get all needed packages to build
the ISO. This script downloads RPM packages from CentOS and third party
repositories, tarballs from specific software releases and binary files from
CentOS releases.

There are several problems with the structure of this script that are
described below.

Serial downloading
------------------

For every item in the .lst files the download process is performed. There are
around 2180 items to be downloaded. Depending on the network speed, the entire
process takes from 1 hour to 3 hours to complete.

lst file formats
----------------

A set of .lst files were created to treat every different item. There is
a list for packages that shall be downloaded with `yum`, other for direct
download and another for downloading with required post-processing.

For example, the ``tarball-dl.lst`` uses a ``#`` separated field line to
specify:

- The name of the downloaded file.
- The folder name included in the resulting tarball: In some cases the tarball
  from upstream does not contain the folder name required by the StarlingX
  build system.
- The URL to download the file.
- The protocol to use to download the file.
- The post processing script to be used in the downloaded file.


These files require that the script should be too specific in the handling of
each case and is not readable for the users. As well it is not an extensible
file format, if new features arise more fields needs to be appended. For
example, the last two fields were recently added for a new feature support.


CentOS specific implementation
------------------------------

As CentOS was considered the only supported base OS for StarlingX, the code in
the download scripts assumed that, therefore the extension to support more
base OSs will require a refactor to decouple the generic and specific parts of
the downloading.


Integrity checking
------------------

The only verification that occurs now is done with the RPM files. The
``rpm -K`` command is run at the end of the entire process to check the
signature of every file with an implicit checksum verification.

However the download scripts doesn't check for integrity in other files,
tarball or binary. This lets to a common issue where a tarball with zero sized
length is downloaded but the user is not notified of this problem.


Spread logging
--------------

During download a ``logs`` folder is created with the details of the files that
weren't downloaded, or are missing from the repositories. This forces the user
to look into different files for each failure case.

Also, the logging is made without a defined structure and anyone can create new
entries in the log files.


Lack of tests
-------------

The download scripts does not have unit tests. A small set of tests was
included in the ``utils_tests.sh`` as an attempt to solve this problem.
however this is not sufficient to cover all the cases.

It is common to find bugs after new features or changes as there are no tests
to verify.


Use cases
=========

The following use cases were identified:

- In a clean environment, a user runs the *stx-mirror* tool to download the
  mirror: The tool shall read and parse the package list from a yaml file, the
  mirror shall be downloaded from the official repository and stored in a
  default folder name in the current directory. The mirror shall be verified
  with GPG keys and MD5 checksums.
- In a clean environment, a user runs the *stx-mirror* tool to download the
  mirror: The tool shall generate a log file with the information of download
  commands executed, their return status and post processing results. At the
  end of this file, the tool will store a summary of the execution with the
  number of the failing commands and missing packages.
- In a clean environment, a user runs the *stx-mirror* tool but some files
  failed to download: The tools shall report the user of the failed items using
  the stderr output.
- A user runs the *stx-mirror* tool with a previous downloaded mirror: The
  tool shall verify the existence of each file before downloading, avoiding
  this action if the downloaded file exists.
- In a clean environment, a user runs the *stx-mirror* tool to download the
  mirror: when a download command for a package failed, the tool shall retry to
  download from the upstream repository.
- In a clean environment, a user runs the *stx-mirror* tool to download the
  mirror and specify the destination directory with the ``--output`` option:
  The mirror shall be downloaded and verified in the specified destination
  directory.
- In a clean environment, a user runs the *stx-mirror* tool to download the
  mirror and the ``--upstream`` option is selected. The tool download the
  mirror from upstream repositories, if an item fails the tool will retry using
  the official repository.
- A user runs the *stx-mirror* tool with a previous downloaded mirror using
  the ``--prune`` option: The tool will download only the new packages. If any
  package has been deleted from the package list, the package will be deleted.


Proposed changed
================

Implement a new python tool called *stx-mirror* with the following parameters:
::

 stx-mirror --output <directory>
 stx-mirror --upstream
 stx-mirror --prune

This tool shall solve all the problems described above.


Alternatives
============

The implementation of this tool can be done in Go and make use of the benefits
of goroutines, however using python will help in community adoption.

There are other alternatives to solve the problems in the current
implementation, however python can be more helpful on the handling of yaml
files for list of items and concurrent execution of the downloading.


Data model impact
=================

This is an example of the yaml package list:

::

 - information: micromanifest
   starlingx-version: stx-r1
   distribution: centos
 - name: centos
   packages:
     - package: some-package-1.2.3.el7.noarch.rpm
     - package: another-package-0.2.3.el7.x86_64.rpm
 - name: 3rdParty
   packages:
     - package: http://someurl.org/go-srpm-macros-2-3.el7.noarch.rpm
     - package: http://someurl.org/golang-1.10.2-1.el7.x86_64.rpm
 - name: Customized
   packages:
   - package: http://http.debian.net/debian/pool/main/d/dpkg/dpkg_1.18.24.tar.xz
     md5: 155fe5c91728bdf82756674d5aa85e4ff2e3eac6
   - custom: https://github.com/pypa/setuptools/archive/v38.5.1.tar.gz
     script: |
           #!/bin/bash
           var=$(ls | wc -l)
           var=$((var+10))
           echo $var


REST API impact
===============

None


Security impact
===============

None


Other end user impact
=====================

None


Performance impact
==================

The download speed will increase using concurrent downloading.


Other deployer impact
=====================

None


Developer impact
================

None


Upgrade impact
==============

None


Implementation
==============


General overview
----------------

The following components were identified:

- CLI parser: Responsible to parse command line arguments.
- YAML parser: Responsible to find and parse yaml files to get the object of
  every download item.
- Item downloader: Responsible to identify, download and process every item.
- Logger: Responsible to log information into the standard output and log file.


YAML parser
-----------

This module is responsible of:

- Check for existence of yaml files in the default folder: A ``centos`` folder
  shall contain all the yaml files related with the packages for CentOS based
  building.
- Parse yaml content: This will read the yaml files and create objects to
  represent the items to be downloaded.


Item downloader
---------------

With the objects generated in the YAML parser, the item downloader does the
following:

- Create a thread pool to handle each object
- Send each item to the thread pool.
- Interactively select the download process based on the object type (use yum,
  direct download or other mechanism)



Assignee(s)
===========

Marcela Rosales
Erich Cordoba


Repos impacted
==============

  - stx-tools


Work items
==========

- Implement the stx-mirror tool: Marcela Rosales/Erich Cordoba
  - Implement YAML parser
  - Implement CentOS downloader
  - Implement Item downloader
  - Implement logger
- Implement tool to migrate .lst to .yaml files.: Erich Cordoba
- Update documentation with instructions to use the tool: Erich Cordoba


Dependencies
============

The only dependency identified so far is the python yaml module. This
dependency will be solved by a ``requirements.txt`` file and included in the
``Dockerfile`` for the build image.

Also, the ``generate-cgcs-centos-repo.sh`` script needs the .lst files to
create the symlinks. This script needs to be changed to meet this new changes
once are implemented.


Testing
=======

The target of this implementation is to have proper unit testing with a
coverage around 90%, both branching and functional.


Documentation impact
====================

The ``Readme.rst`` file needs to be updated accordingly to use this tool.


References
==========

None
