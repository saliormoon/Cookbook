# Compiling ArangoDB under Windows

## Problem

I want to compile ArangoDB under Windows.

**Note:** For this recipe you need at least ArangoDB 2.4; it also works for 2.5 and 2.6. For ArangoDB version before 2.4 look at the [old Compiling ArangoDB under Windows](CompilingUnderWindowsLegacy.md).

## Solution

While compiling ArangoDB under Linux is straight forward - simply execute `configure` and `make` - compiling under Windows 
is a bit more demanding.

### Ingredients

For this recipe you need to install the following programs under Windows:

* [cygwin](https://www.cygwin.com/)

You need at least `make` from cygwin. Cygwin also offers a `cmake`. Do **not** install this version. The unit tests require the bash.

* [cmake](http://www.cmake.org/)

Either version 2.8.12, 3.0.2 or 3.1.2 should work. Attention - pitfall: the cygwin version doesn't work.

* [python](http://www.python.org)

Either version 2.x or 3.x (excluding 3.4) should work - it's used to run V8s GYP. Make sure you add python.exe to your path environment variable; Restarting your running shell may be necessary.

* Visual Studio Express 2013 for Windows Desktop

Please note that there are different versions of Visual Studio. The `Visual Studio for Windows` will **not work**. You need to
install `Visual Studio (Express) for Windows Desktop`. You must configure your path in such a way that the compiler can
be found. One way is to execute the `vcvarsall.bat` script from the `VC` folder.

<!--
* BOOST (if you intend to run the tests for development)

[installing boost under windows](https://niuquant.wordpress.com/2013/11/16/setup-boost-1-55-0-unit-test-framework-with-visual-stuido-2010-on-windows-8-desktop/) has more details on howto.
-->

* [Nullsoft Scriptable Install System](http://nsis.sourceforge.net/Download)

* [Ruby](https://www.ruby-lang.org/en/downloads/) (for the unittests, if you intend to run them)

* [procdump](https://technet.microsoft.com/en-us/sysinternals/dd996900.aspx) (for the unittests; run once to accept the eula)

* [GitLink](https://github.com/GitTools/GitLink) to adjust the pdb files to github

### Building the required libraries

First of all, start a `bash` from cygwin and clone the repository using the corresponding branch, e.g. for ArangoDB 2.6:

    git clone -b 2.6 https://github.com/arangodb/arangodb-windows-libraries.git

and switch into the directory `arangodb-windows-libraries`. This repository contains the open-source libraries which
are needed by ArangoDB:

* etcd from CoreOS
* getopt
* libev
* linenoise
* openssl
* regex
* zlib

In order to build the corresponding 32bit and 64bit versions of these libraries, execute

    make
    make install
    make 3rdParty

This will create a folder `WindowsLibraries` containing the headers and libraries.

### Building ArangoDB itself

Clone the repository

    https://github.com/arangodb/arangodb.git

and copy the previously created `WindowsLibraries` folder into this directory `ArangoDB`.

Next you need to build Google's V8 version, that is part of ArangoDB. This version contains V8 and ICU. Switch into the directory

    ArangoDB/3rdParty

and execute

    make -f Makefile.v8-windows
    make -f Makefile.v8-windows install


Now switch back into the `ArangoDB` root folder and execute

    make pack-win32

or

    make pack-win64

in order to build the installer file for either 32 bit or 64 bit.

## Development builds
For development builds which are able to run the unit tests run
<!--
(adjust the boost directory to your situation):
-->
<!--
    export BOOST_ROOT='c:\Program Files\boost_1_56_0\'
    export BOOST_LIBRARYDIR='c:\Program Files\boost_1_56_0\lib64-msvc-12.0'
    BOOSTROOT='c:\Program Files\boost_1_56_0'
    BOOST_INCLUDE='c:\Program Files\boost_1_56_0\'
    BOOST_LIBRARYDIR='c:\Program Files\boost_1_56_0\lib64-msvc-12.0'
    BOOST_ROOT='c:\Program Files\boost_1_56_0\'
    CMAKE_INCLUDE_PATHE='c:\Program Files\boost_1_56_0\'
-->

    make pack-win64-relative

Since most of the scripts assume they're running on a unix system, some directories are treated as mandatory and thus have to be created on the drive where you checked out your source:

  mkdir -p /cygdrive/c/var/tmp/

You probably already downloaded and installed [Ruby](https://www.ruby-lang.org/en/downloads/). Now we need to install the httparty and rspec gems. In order to use gem, you need to [Install certificates](https://gist.github.com/fnichol/867550). Once you did that you can run:

    gem install httparty rspec

The *bin/rspec.bat* created by gem is not sufficient for being run by the unittesting facility; replace it with something like this:

    @ECHO OFF
    IF NOT "%~f0" == "~f0" GOTO :WinNT
    @"C:\Program Files (x86)\Rub21-x64\bin\ruby.exe" "C:/Program Files (x86)/Ruby21-x64/bin/rspec" %1 %2 %3 %4 %5 %6 %7 %8 %9
    GOTO :EOF
    :WinNT
    @"C:\Program Files (x86)\Ruby21-x64\bin\ruby.exe" "C:/Program Files (x86)/Ruby21-x64/bin/rspec" %*


  
You can then run the unittests like that:

  bash ./scripts/run_cygwin.sh scripts/unittest.js all '{"skipBoost":true,"skipGeo":true}'

### Executables only

If you do not need the installer file, you can use the `cmake` to build the executables. Instead of `make pack-win32`
use

    mkdir Build32
    cd Build32
    cmake -G "Visual Studio 12" ..

This will create a solution file in the `Build32` folder. You can now start Visual Studio and open this
solution file.

To build the 64 bit version, use

    mkdir Build64
    cd Build64
    cmake -G "Visual Studio 12 Win64" ..

Alternatively use `cmake` to build the executables.

    cmake --build . --config Release

In order to execute the binaries you need to copy the ICU datafile into the directory containing the
executable

    cp WindowsLibraries/64/icudtl.dat Build64/bin/Debug/icudt52l.dat

**Author**: [Frank Celler](https://github.com/fceller)

**Tags**: #windows
