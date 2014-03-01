# Steamworks API for AIR applications #

A simple Steamworks API wrapper to be used as native extension for Adobe AIR
applications. This allows you to implement Steam achievements, leaderboards, cloud
storage, workshop support and much more in your AIR application - on Windows,
OS X and Linux.

This project initially started as a fork of [FRESteamWorks by Oldes/Amanita Design](https://github.com/Oldes/FRESteamWorks/).

# Download #

Pre-built cross-platform ANEs (for both Windows and OS X), as well as Linux
SWCs can be found on http://dump.ventero.de/FRESteamWorks/. The Linux wrapper
binary has to be built manually, see [the Linux section](#linux) for more details.

# Usage #

For Windows/OS X builds, you'll only have to include the ANE in your project like
any normal SWC, add the extension to your application descriptor and include
`extendedDesktop` in the list of supported profiles. For an example, see
[the application descriptor of the FRESteamWorks test application](https://github.com/Ventero/FRESteamWorks/blob/master/test/bin-debug/FRESteamWorksTest-app.xml#L14-17).

Testing your application is a bit more complicated and depends on how your project
is built. For detailed information, see below. For general information regarding
the testing of Steam applications, please refer to the Steamworks SDK documentation.

Since the AIR runtime on Linux doesn't support native extensions, an external
binary is used to communicate with the Steamworks API. For more information
on how to build and include this tool, see [the Linux section](#linux).

## Flash Builder ##

### Building ###

To include the ANE in your Flash Builder project, simply add it to the list of
"Native Extensions" under "ActionScript Build Path" in your project's properties.
Make sure it gets packaged when building your application by checking the "Package" box
in the "Native Extensions" tab under "ActionScript Build Packaging". The extension
will automatically be added to your application descriptor, but you still might have
to add `extendedDesktop` to the list of supported profiles.

### Running ###

Since by default, Flash Builder uses its installation path as working directory
when running applications, this however isn't enough to test your application in
Flash Builder. This can be fixed by either adding the library's directory to the
dynamic linker's search path (see [the AIR/Flex SDK section](#air-flex-sdk) on
how to do this), or by creating a new shortcut to Flash Builder where the working
directory is set to your project's directory and adding the dynamic library to
your project.

### Packaging ###

Just include the native Steamworks library at the top level of your project so
that it gets included when packaging your application.

### Known Issues ###

Flash Builder 4.6 seems to have an issue with detecting classes that implement
interfaces inside an ANE, which results in the `FRESteamWorks` class not being
visible inside the ANE within Flash Builder. To work around this, please contact
me for an ANE build that doesn't include any interfaces. This issue is fixed in
Flash Builder 4.7.

Running an application including native extensions is not possible in Flash Builder
4.6 on OS X. See [a thread on the Adobe forums](http://forums.adobe.com/thread/986589)
for possible workarounds. You can also use a Shell/Batch script to run your application
with the AIR Debug Launcher, see [the AIR/Flex SDK section](#air-flex-sdk) for more details.

## AIR/Flex SDK ##

### Building ###

For projects built with the AIR/Flex SDKs, the ANE can be included by simply
adding the ANE path to the external library path (either by adding it to your
build config file, or by adding `-external-library-path+=/path/to/FRESteamWorks.ane`
to your `mxmlc`/`compc` compiler flags). The SWF you're has to be at least version 11.

### Running ###

To run your application with the AIR Debug Launcher (`adl`), you will have to first
unpack the ANE. To do this, rename `FRESteamWorks.ane` to `FRESteamWorks.zip`,
then unzip it and rename the generated folder to `FRESteamWorks.Unpacked.ane`.
Now you can add `-extdir /path/to/folder` to the `adl` command line flags. Here,
`folder` refers the parent folder of the unzipped extension (i.e. the folder that
contains `FRESteamWorks.Unpacked.ane`).

Since the native library included in the ANE is dynamically linked against the
Steamworks library, you'll also have to make sure that the dynamic linker can find
the Steamworks library. When testing, this is easiest done by adding the folder containing
the Steamworks library to your `%PATH%` environment variable on Windows
([example](https://github.com/Ventero/FRESteamWorks/blob/master/test/bin-debug/runWin.bat#L9)),
`DYLD_FALLBACK_LIBRARY_PATH` on OS X ([example](https://github.com/Ventero/FRESteamWorks/blob/master/test/bin-debug/runMac.sh#L6))
or the library itself to `LD_PRELOAD` on Linux ([example](https://github.com/Ventero/FRESteamWorks/blob/master/test/bin-debug/runLinux.sh#L24)).

### Packaging ###

For packaged builds, you'll simply have to include the native Steamworks libraries
in the top level of your build and add the path to a directory containing the
`FRESteamWorks.ane` as `-extdir` to `adt`. For an example, see the test application's
[packaging script](https://github.com/Ventero/FRESteamWorks/blob/master/test/bin-debug/packageWin.bat#L13-18).

## Flash CS ##

TBD

# Linux #

The AIR runtime on Linux doesn't support native extensions. Instead, you'll have
to compile your AS3 application against `FRESteamWorksLibLinux.swc` and compile
a wrapper binary that handles the communication between AIR and the Steamworks API.
For details on how to do that, please contact me directly (email address see profile).

# API Documentation #

For a full list of all supported functions, see [lib/API.txt](https://github.com/Ventero/FRESteamWorks/blob/master/lib/API.txt).
In general, the FRESteamWorks API functions are as close representation of the
native Steamworks SDK functions. Thus, for detailed documentation, see the
Steamworks SDK docs.

The objects returned by certain API functions are plain data structures.
For a list of available properties, see [the corresponding source file](https://github.com/Ventero/FRESteamWorks/tree/master/lib/src/com/amanitadesign/steam).

# Building FRESteamWorks #

There shouldn't be any reason for you to manually build the FRESteamWorks.ane,
as pre-built ANEs can be downloaded from http://dump.ventero.de/FRESteamWorks/.
If you need a more recent version than the builds that are available on that site,
you can create an issue in the [bug tracker](https://github.com/Ventero/FRESteamWorks/issues).

If you still want to build the ANE yourself (or build the test application), a few
simple steps have to be followed.

1. Create a config file by copying `config_example.sh` to `config.sh` on OS X,
or `config_example.bat` to `config.bat` and correctly setting up the values within.

2. Open the Visual Studio solution or XCode project (both inside `src/`) and fix
up the include paths (for the Adobe AIR SDK as well as the Steamworks SDK).

3. Build the project in either debug or release mode. This compiles the native
library and automatically builds an ANE for the current platform containing that
library.

4. Optionally, build the test application by running `build.{bat,sh}` inside
`test/bin-debug/` and run it with `runMac.sh` or `runWin.bat`. This automatically
uses the ANE built in step 3.

To build a cross platform ANE, you'll have to compile the native library on both
Windows and OS X and then create an ANE that includes both of those. This is easiest
done by first running `mkdir.sh` in `builds/`, building FRESteamWorks.dll on Windows,
copying it over to the directory created on OS X and then running `compile.sh`
and `build.sh` in that order.
running `

---

# License #

Copyright (c) 2012-2014, Level Up Labs, LLC

Copyright (c) 2012, Oldes

All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
