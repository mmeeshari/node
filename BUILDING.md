# Building Node.js

Depending on what platform or features you need, the build process may
differ. After you've built a binary, running the
test suite to confirm that the binary works as intended is a good next step.

If you can reproduce a test failure, search for it in the
[Node.js issue tracker](https://github.com/nodejs/node/issues) or
file a new issue.

## Table of Contents

* [Supported platforms](#supported-platforms)
  * [Input](#input)
  * [Strategy](#strategy)
  * [Platform list](#platform-list)
  * [Supported toolchains](#supported-toolchains)
  * [Official binary platforms and toolchains](#official-binary-platforms-and-toolchains)
    * [OpenSSL asm support](#openssl-asm-support)
  * [Previous versions of this document](#previous-versions-of-this-document)
* [Building Node.js on supported platforms](#building-nodejs-on-supported-platforms)
  * [Note about Python 2 and Python 3](#note-about-python-2-and-python-3)
  * [Unix and macOS](#unix-and-macos)
    * [Unix prerequisites](#unix-prerequisites)
    * [macOS prerequisites](#macos-prerequisites)
    * [Building Node.js](#building-nodejs)
    * [Running Tests](#running-tests)
    * [Running Coverage](#running-coverage)
    * [Building the documentation](#building-the-documentation)
    * [Building a debug build](#building-a-debug-build)
  * [Windows](#windows)
    * [Prerequisites](#prerequisites)
      * [Option 1: Manual install](#option-1-manual-install)
      * [Option 1: Automated install with Boxstarter](#option-1-automated-install-with-boxstarter)
    * [Building Node.js](#building-nodejs-1)
  * [Android/Android-based devices (e.g. Firefox OS)](#androidandroid-based-devices-eg-firefox-os)
* [`Intl` (ECMA-402) support](#intl-ecma-402-support)
  * [Default: `small-icu` (English only) support](#default-small-icu-english-only-support)
  * [Build with full ICU support (all locales supported by ICU)](#build-with-full-icu-support-all-locales-supported-by-icu)
    * [Unix/macOS](#unixmacos)
    * [Windows](#windows-1)
  * [Building without Intl support](#building-without-intl-support)
    * [Unix/macOS](#unixmacos-1)
    * [Windows](#windows-2)
  * [Use existing installed ICU (Unix/macOS only)](#use-existing-installed-icu-unixmacOS-only)
  * [Build with a specific ICU](#build-with-a-specific-icu)
    * [Unix/macOS](#unixmacos-2)
    * [Windows](#windows-3)
* [Building Node.js with FIPS-compliant OpenSSL](#building-nodejs-with-fips-compliant-openssl)
* [Building Node.js with external core modules](#building-nodejs-with-external-core-modules)
  * [Unix/macOS](#unixmacos-3)
  * [Windows](#windows-4)
* [Note for downstream distributors of Node.js](#note-for-downstream-distributors-of-nodejs)

## Supported platforms

This list of supported platforms is current as of the branch/release to
which it belongs.

### Input

Node.js relies on V8 and libuv. We adopt a subset of their supported platforms.

### Strategy

There are three support tiers:

* **Tier 1**: These platforms represent the majority of Node.js users. The
  Node.js Build Working Group maintains infrastructure for full test coverage.
  Maintenance is supported by the Node.js core team. All commits to the
  Node.js repository are tested on multiple variants of these platforms. Test
  failures on tier 1 platforms will block releases.
* **Tier 2**: These platforms represent smaller segments of the Node.js user
  base. The Node.js Build Working Group maintains infrastructure for full test
  coverage. Maintenance is supported by smaller groups or individuals within
  the Node.js core team, or the vendor of the platform itself. All commits to
  the Node.js repository are tested on multiple variants of these platforms
  where practical. Test failures on tier 2 platforms will block releases.
  Delays in release of binaries for these platforms are acceptable
  where necessary due to infrastructure concerns.
* **Experimental**: May not compile or test suite may not pass. The core team
  does not create releases for these platforms. Test failures on experimental
  platforms do not block releases. Contributions to improve support for these
  platforms are welcome.

Platforms may move between tiers between major release lines. The table below
will be updated to reflect those changes.

### Platform list

Compiling and running Node.js is supported for a limited set of operating
systems, architectures and libc versions. The table below lists the
combinations that the core team has committed to supporting and the nature of
that support as per the support tiers above. A list of
[supported compile toolchains](#supported-toolchains) is also supplied for
tier 1 platforms.

**For production applications, run Node.js on supported platforms only.**

Node.js does not support a platform version if a vendor has expired support
for it. In other words, Node.js does not support running on End-of-Life (EoL)
platforms. This is true regardless of entries in the table below.

| Operating System | Architectures    | Versions                        | Support Type | Notes                             |
| ---------------- | ---------------- | ------------------------------- | ------------ | --------------------------------- |
| GNU/Linux        | x64              | kernel >= 3.10, glibc >= 2.17   | Tier 1       | e.g. Ubuntu 16.04 <sup>[1](#fn1)</sup>, Debian 9, EL 7 <sup>[2](#fn2)</sup> |
| GNU/Linux        | x64              | kernel >= 3.10, musl >= 1.1.19  | Experimental | e.g. Alpine 3.8                   |
| GNU/Linux        | x86              | kernel >= 3.10, glibc >= 2.17   | Experimental | Downgraded as of Node.js 10       |
| GNU/Linux        | arm64            | kernel >= 4.5, glibc >= 2.17    | Tier 1       | e.g. Ubuntu 16.04, Debian 9, EL 7 <sup>[3](#fn3)</sup> |
| GNU/Linux        | armv7            | kernel >= 4.14, glibc >= 2.24   | Tier 1       | e.g. Ubuntu 18.04, Debian 9       |
| GNU/Linux        | armv6            | kernel >= 4.14, glibc >= 2.24   | Experimental | Downgraded as of Node.js 12       |
| GNU/Linux        | ppc64le >=power8 | kernel >= 3.10.0, glibc >= 2.17 | Tier 2       | e.g. Ubuntu 16.04 <sup>[1](#fn1)</sup>, EL 7  <sup>[2](#fn2)</sup> |
| GNU/Linux        | s390x            | kernel >= 3.10.0, glibc >= 2.17 | Tier 2       | e.g. EL 7 <sup>[2](#fn2)</sup>    |
| Windows          | x64, x86 (WoW64) | >= Windows 7/2008 R2/2012 R2    | Tier 1       | <sup>[4](#fn4),[5](#fn5)</sup>    |
| Windows          | x86 (native)     | >= Windows 7/2008 R2/2012 R2    | Tier 1 (running) / Experimental (compiling) <sup>[6](#fn6)</sup> | |
| Windows          | arm64            | >= Windows 10                   | Experimental |                                   |
| macOS            | x64              | >= 10.11                        | Tier 1       |                                   |
| SmartOS          | x64              | >= 18                           | Tier 2       |                                   |
| AIX              | ppc64be >=power7 | >= 7.1 TL05                     | Tier 2       |                                   |
| FreeBSD          | x64              | >= 11                           | Experimental | Downgraded as of Node.js 12       |

<em id="fn1">1</em>: GCC 6 is not provided on the base platform, users will
  need the
  [Toolchain test builds PPA](https://launchpad.net/~ubuntu-toolchain-r/+archive/ubuntu/test?field.series_filter=xenial)
  or similar to source a newer compiler.

<em id="fn2">2</em>: GCC 6 is not provided on the base platform, users will
  need the
  [devtoolset-6](https://www.softwarecollections.org/en/scls/rhscl/devtoolset-6/)
  or later to source a newer compiler.

<em id="fn3">3</em>: Older kernel versions may work for ARM64, however the
  Node.js test infrastructure only tests >= 4.5.

<em id="fn4">4</em>: On Windows, running Node.js in Windows terminal emulators
  like `mintty` requires the usage of [winpty](https://github.com/rprichard/winpty)
  for the tty channels to work correctly (e.g. `winpty node.exe script.js`).
  In "Git bash" if you call the node shell alias (`node` without the `.exe`
  extension), `winpty` is used automatically.

<em id="fn5">5</em>: The Windows Subsystem for Linux (WSL) is not directly
  supported, but the GNU/Linux build process and binaries should work. The
  community will only address issues that reproduce on native GNU/Linux
  systems. Issues that only reproduce on WSL should be reported in the
  [WSL issue tracker](https://github.com/Microsoft/WSL/issues). Running the
  Windows binary (`node.exe`) in WSL is not recommended. It will not work
  without workarounds such as stdio redirection.

<em id="fn6">6</em>: Running Node.js on x86 Windows should work and binaries
are provided. However, tests in our infrastructure only run on WoW64.
Furthermore, compiling on x86 Windows is currently considered Experimental and
may not be possible.

### Supported toolchains

Depending on the host platform, the selection of toolchains may vary.

| Operating System | Compiler Versions                                              |
| ---------------- | -------------------------------------------------------------- |
| Linux            | GCC >= 6.3                                                     |
| Windows          | Visual Studio >= 2017 with the Windows 10 SDK on a 64-bit host |
| macOS            | Xcode >= 8 (Apple LLVM >= 8)                                   |

### Official binary platforms and toolchains

Binaries at <https://nodejs.org/download/release/> are produced on:

| Binary package        | Platform and Toolchain                                                   |
| --------------------- | ------------------------------------------------------------------------ |
| aix-ppc64             | AIX 7.1 TL05 on PPC64BE with GCC 6                                       |
| darwin-x64 (and .pkg) | macOS 10.11, Xcode Command Line Tools 8 with -mmacosx-version-min=10.10  |
| linux-arm64           | CentOS 7 with devtoolset-6 / GCC 6                                       |
| linux-armv7l          | Cross-compiled on Ubuntu 16.04 x64 with [custom GCC toolchain](https://github.com/rvagg/rpi-newer-crosstools)   |
| linux-ppc64le         | CentOS 7 with devtoolset-6 / GCC 6 <sup>[7](#fn7)</sup>                  |
| linux-s390x           | RHEL 7 with devtoolset-6 / GCC 6 <sup>[7](#fn7)</sup>                    |
| linux-x64             | CentOS 7 with devtoolset-6 / GCC 6 <sup>[7](#fn7)</sup>                  |
| sunos-x64             | SmartOS 18 with GCC 7                                                    |
| win-x64 and win-x86   | Windows 2012 R2 (x64) with Visual Studio 2017                            |

<em id="fn7">7</em>: The Enterprise Linux devtoolset-6 allows us to compile
binaries with GCC 6 but linked to the glibc and libstdc++ versions of the host
platforms (CentOS 7 / RHEL 7). Therefore, binaries produced on these systems
are compatible with glibc >= 2.17 and libstdc++ >= 6.0.20 (`GLIBCXX_3.4.20`).
These are available on distributions natively supporting GCC 4.9, such as
Ubuntu 14.04 and Debian 8.

#### OpenSSL asm support

OpenSSL-1.1.1 requires the following assembler version for use of asm
support on x86_64 and ia32.

For use of AVX-512,

* gas (GNU assembler) version 2.26 or higher
* nasm version 2.11.8 or higher in Windows

AVX-512 is disabled for Skylake-X by OpenSSL-1.1.1.

For use of AVX2,

* gas (GNU assembler) version 2.23 or higher
* Xcode version 5.0 or higher
* llvm version 3.3 or higher
* nasm version 2.10 or higher in Windows

Please refer to
 https://www.openssl.org/docs/man1.1.1/man3/OPENSSL_ia32cap.html for details.

 If compiling without one of the above, use `configure` with the
`--openssl-no-asm` flag. Otherwise, `configure` will fail.

### Previous versions of this document

Supported platforms and toolchains change with each major version of Node.js.
This document is only valid for the current major version of Node.js.
Consult previous versions of this document for older versions of Node.js:

* [Node.js 10](https://github.com/nodejs/node/blob/v10.x/BUILDING.md)
* [Node.js 8](https://github.com/nodejs/node/blob/v8.x/BUILDING.md)
* [Node.js 6](https://github.com/nodejs/node/blob/v6.x/BUILDING.md)

## Building Node.js on supported platforms

### Note about Python 2 and Python 3

The Node.js project uses Python as part of its build process and has
historically only been Python 2 compatible.

Python 2 will reach its _end-of-life_ at the end of 2019 at which point the
interpreter will cease receiving updates. See https://python3statement.org/
for more information.

The Node.js project is in the process of transitioning its Python code to
Python 3 compatibility. Installing both versions of Python while building
and testing Node.js allows developers and end users to test, benchmark,
and debug Node.js running on both versions to ensure a smooth and complete
transition before the year-end deadline.

### Unix and macOS

#### Unix prerequisites

* `gcc` and `g++` >= 6.3 or newer, or
* GNU Make 3.81 or newer
* Python (see note above)
  * Python 2.7
  * Python 3.5, 3.6, and 3.7 are experimental.

Installation via Linux package manager can be achieved with:

* Ubuntu, Debian: `sudo apt-get install python g++ make`
* Fedora: `sudo dnf install python gcc-c++ make`
* CentOS and RHEL: `sudo yum install python gcc-c++ make`
* OpenSUSE: `sudo zypper install python gcc-c++ make`

FreeBSD and OpenBSD users may also need to install `libexecinfo`.

#### macOS prerequisites

* Xcode Command Line Tools >= 8 for macOS
* Python (see note above)
  * Python 2.7
  * Python 3.5, 3.6, and 3.7 are experimental.

macOS users can install the `Xcode Command Line Tools` by running
`xcode-select --install`. Alternatively, if you already have the full Xcode
installed, you can find them under the menu `Xcode -> Open Developer Tool ->
More Developer Tools...`. This step will install `clang`, `clang++`, and
`make`.

#### Building Node.js

If the path to your build directory contains a space, the build will likely
fail.

To build Node.js:

```console
$ ./configure
$ make -j4
```

The `-j4` option will cause `make` to run 4 simultaneous compilation jobs which
may reduce build time. For more information, see the
[GNU Make Documentation](https://www.gnu.org/software/make/manual/html_node/Parallel.html).

The above requires that `python` resolves to a supported version of
Python. See [Prerequisites](#prerequisites).

After building, setting up [firewall rules](tools/macos-firewall.sh) can avoid
popups asking to accept incoming network connections when running tests.

Running the following script on macOS will add the firewall rules for the
executable `node` in the `out` directory and the symbolic `node` link in the
project's root directory.

```console
$ sudo ./tools/macos-firewall.sh
```

#### Running Tests

To verify the build:

```console
$ make test-only
```

At this point, you are ready to make code changes and re-run the tests.

If you are running tests before submitting a Pull Request, the recommended
command is:

```console
$ make -j4 test
```

`make -j4 test` does a full check on the codebase, including running linters and
documentation tests.

Make sure the linter does not report any issues and that all tests pass. Please
do not submit patches that fail either check.

If you want to run the linter without running tests, use
`make lint`/`vcbuild lint`. It will lint JavaScript, C++, and Markdown files.

If you are updating tests and want to run tests in a single test file
(e.g. `test/parallel/test-stream2-transform.js`):

```text
$ python tools/test.py test/parallel/test-stream2-transform.js
```

You can execute the entire suite of tests for a given subsystem
by providing the name of a subsystem:

```text
$ python tools/test.py -J --mode=release child-process
```

If you want to check the other options, please refer to the help by using
the `--help` option:

```text
$ python tools/test.py --help
```

You can usually run tests directly with node:

```text
$ ./node ./test/parallel/test-stream2-transform.js
```

Remember to recompile with `make -j4` in between test runs if you change code in
the `lib` or `src` directories.

The tests attempt to detect support for IPv6 and exclude IPv6 tests if
appropriate. If your main interface has IPv6 addresses, then your
loopback interface must also have '::1' enabled. For some default installations
on Ubuntu that does not seem to be the case. To enable '::1' on the
loopback interface on Ubuntu:

```bash
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=0
```

#### Running Coverage

It's good practice to ensure any code you add or change is covered by tests.
You can do so by running the test suite with coverage enabled:

```console
$ ./configure --coverage
$ make coverage
```

A detailed coverage report will be written to `coverage/index.html` for
JavaScript coverage and to `coverage/cxxcoverage.html` for C++ coverage
(if you only want to run the JavaScript tests then you do not need to run
the first command `./configure --coverage`).

_Generating a test coverage report can take several minutes._

To collect coverage for a subset of tests you can set the `CI_JS_SUITES` and
`CI_NATIVE_SUITES` variables (to run specific suites, e.g., `child-process`, in
isolation, unset the opposing `_SUITES` variable):

```text
$ CI_JS_SUITES=child-process CI_NATIVE_SUITES= make coverage
```

The above command executes tests for the `child-process` subsystem and
outputs the resulting coverage report.

Alternatively, you can run `make coverage-run-js`, to execute JavaScript tests
independently of the C++ test suite:

```text
$ CI_JS_SUITES=fs CI_NATIVE_SUITES= make coverage-run-js
```

The `make coverage` command downloads some tools to the project root directory.
To clean up after generating the coverage reports:

```console
$ make coverage-clean
```

#### Building the documentation

To build the documentation:

This will build Node.js first (if necessary) and then use it to build the docs:

```console
$ make doc
```

If you have an existing Node.js build, you can build just the docs with:

```console
$ NODE=/path/to/node make doc-only
```

To read the documentation:

```console
$ man doc/node.1
```

If you prefer to read the documentation in a browser,
run the following after `make doc` is finished:

```console
$ make docopen
```

This will open a browser with the documentation.

To test if Node.js was built correctly:

```console
$ ./node -e "console.log('Hello from Node.js ' + process.version)"
```

To install this version of Node.js into a system directory:

```console
$ [sudo] make install
```

#### Building a debug build

If you run into an issue where the information provided by the JS stack trace
is not enough, or if you suspect the error happens outside of the JS VM, you
can try to build a debug enabled binary:

```console
$ ./configure --debug
$ make -j4
```

`make` with `./configure --debug` generates two binaries, the regular release
one in `out/Release/node` and a debug binary in `out/Debug/node`, only the
release version is actually installed when you run `make install`.

To use the debug build with all the normal dependencies overwrite the release
version in the install directory:

``` console
$ make install --prefix=/opt/node-debug/
$ cp -a -f out/Debug/node /opt/node-debug/node
```

When using the debug binary, core dumps will be generated in case of crashes.
These core dumps are useful for debugging when provided with the
corresponding original debug binary and system information.

Reading the core dump requires `gdb` built on the same platform the core dump
was captured on (i.e. 64-bit `gdb` for `node` built on a 64-bit system, Linux
`gdb` for `node` built on Linux) otherwise you will get errors like
`not in executable format: File format not recognized`.

Example of generating a backtrace from the core dump:

``` console
$ gdb /opt/node-debug/node core.node.8.1535359906
$ backtrace
```

### Windows

#### Prerequisites

##### Option 1: Manual install

* [Python 2.7](https://www.python.org/downloads/)
* The "Desktop development with C++" workload from
  [Visual Studio 2017](https://www.visualstudio.com/downloads/) or the
  "Visual C++ build tools" workload from the
  [Build Tools](https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2017),
  with the default optional components.
* Basic Unix tools required for some tests,
  [Git for Windows](https://git-scm.com/download/win) includes Git Bash
  and tools which can be included in the global `PATH`.
* The [NetWide Assembler](https://www.nasm.us/), for OpenSSL assembler modules.
  If not installed in the default location, it needs to be manually added
  to `PATH`. A build with the `openssl-no-asm` option does not need this, nor
  does a build targeting ARM64 Windows.

Optional requirements to build the MSI installer package:

* The [WiX Toolset v3.11](https://wixtoolset.org/releases/) and the
  [Wix Toolset Visual Studio 2017 Extension](https://marketplace.visualstudio.com/items?itemName=RobMensching.WixToolsetVisualStudio2017Extension).

Optional requirements for compiling for Windows 10 on ARM (ARM64):

* ARM64 Windows build machine
  * Due to a GYP limitation, this is required to run compiled code
    generation tools (like V8's builtins and mksnapshot tools)
* Visual Studio 15.9.0 or newer
* Visual Studio optional components
  * Visual C++ compilers and libraries for ARM64
  * Visual C++ ATL for ARM64
* Windows 10 SDK 10.0.17763.0 or newer

##### Option 2: Automated install with Boxstarter
<a name="boxstarter"></a>

A [Boxstarter](https://boxstarter.org/) script can be used for easy setup of
Windows systems with all the required prerequisites for Node.js development.
This script will install the following [Chocolatey](https://chocolatey.org/)
packages:

* [Git for Windows](https://chocolatey.org/packages/git) with the `git` and
  Unix tools added to the `PATH`.
* [Python 3.x](https://chocolatey.org/packages/python) and
  [legacy Python](https://chocolatey.org/packages/python2)
* [Visual Studio 2017 Build Tools](https://chocolatey.org/packages/visualstudio2017buildtools)
  with [Visual C++ workload](https://chocolatey.org/packages/visualstudio2017-workload-vctools)
* [NetWide Assembler](https://chocolatey.org/packages/nasm)

To install Node.js prerequisites using
[Boxstarter WebLauncher](https://boxstarter.org/WebLauncher), open
<https://boxstarter.org/package/nr/url?https://raw.githubusercontent.com/nodejs/node/master/tools/bootstrap/windows_boxstarter>
with Internet Explorer or Edge browser on the target machine.

Alternatively, you can use PowerShell. Run those commands from an elevated
PowerShell terminal:

```powershell
Set-ExecutionPolicy Unrestricted -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://boxstarter.org/bootstrapper.ps1'))
get-boxstarter -Force
Install-BoxstarterPackage https://raw.githubusercontent.com/nodejs/node/master/tools/bootstrap/windows_boxstarter -DisableReboots
```

The entire installation using Boxstarter will take up approximately 10 GB of
disk space.

#### Building Node.js

If the path to your build directory contains a space or a non-ASCII character,
the build will likely fail.

```console
> .\vcbuild
```

To run the tests:

```console
> .\vcbuild test
```

To test if Node.js was built correctly:

```console
> Release\node -e "console.log('Hello from Node.js', process.version)"
```

### Android/Android-based devices (e.g. Firefox OS)

Android is not a supported platform. Patches to improve the Android build are
welcome. There is no testing on Android in the current continuous integration
environment. The participation of people dedicated and determined to improve
Android building, testing, and support is encouraged.

Be sure you have downloaded and extracted
[Android NDK](https://developer.android.com/tools/sdk/ndk/index.html) before in
a folder. Then run:

```console
$ ./android-configure /path/to/your/android-ndk
$ make
```

## `Intl` (ECMA-402) support

[Intl](https://github.com/nodejs/node/blob/master/doc/api/intl.md) support is
enabled by default, with English data only.

### Default: `small-icu` (English only) support

By default, only English data is included, but
the full `Intl` (ECMA-402) APIs.  It does not need to download
any dependencies to function. You can add full
data at runtime.

### Build with full ICU support (all locales supported by ICU)

With the `--download=all`, this may download ICU if you don't have an
ICU in `deps/icu`. (The embedded `small-icu` included in the default
Node.js source does not include all locales.)

#### Unix/macOS

```console
$ ./configure --with-intl=full-icu --download=all
```

#### Windows

```console
> .\vcbuild full-icu download-all
```

### Building without Intl support

The `Intl` object will not be available, nor some other APIs such as
`String.normalize`.

#### Unix/macOS

```console
$ ./configure --without-intl
```

#### Windows

```console
> .\vcbuild without-intl
```

### Use existing installed ICU (Unix/macOS only)

```console
$ pkg-config --modversion icu-i18n && ./configure --with-intl=system-icu
```

If you are cross-compiling, your `pkg-config` must be able to supply a path
that works for both your host and target environments.

### Build with a specific ICU

You can find other ICU releases at
[the ICU homepage](http://icu-project.org/download).
Download the file named something like `icu4c-**##.#**-src.tgz` (or
`.zip`).

To check the minimum recommended ICU, run `./configure --help` and see
the help for the `--with-icu-source` option. A warning will be printed
during configuration if the ICU version is too old.

#### Unix/macOS

From an already-unpacked ICU:

```console
$ ./configure --with-intl=[small-icu,full-icu] --with-icu-source=/path/to/icu
```

From a local ICU tarball:

```console
$ ./configure --with-intl=[small-icu,full-icu] --with-icu-source=/path/to/icu.tgz
```

From a tarball URL:

```console
$ ./configure --with-intl=full-icu --with-icu-source=http://url/to/icu.tgz
```

#### Windows

First unpack latest ICU to `deps/icu`
[icu4c-**##.#**-src.tgz](http://icu-project.org/download) (or `.zip`)
as `deps/icu` (You'll have: `deps/icu/source/...`)

```console
> .\vcbuild full-icu
```

## Building Node.js with FIPS-compliant OpenSSL

The current version of Node.js does not support FIPS.

## Building Node.js with external core modules

It is possible to specify one or more JavaScript text files to be bundled in
the binary as built-in modules when building Node.js.

### Unix/macOS

This command will make `/root/myModule.js` available via
`require('/root/myModule')` and `./myModule2.js` available via
`require('myModule2')`.

```console
$ ./configure --link-module '/root/myModule.js' --link-module './myModule2.js'
```

### Windows

To make `./myModule.js` available via `require('myModule')` and
`./myModule2.js` available via `require('myModule2')`:

```console
> .\vcbuild link-module './myModule.js' link-module './myModule2.js'
```

## Note for downstream distributors of Node.js

The Node.js ecosystem is reliant on ABI compatibility within a major release.
To maintain ABI compatibility it is required that distributed builds of Node.js
be built against the same version of dependencies, or similar versions that do
not break their ABI compatibility, as those released by Node.js for any given
`NODE_MODULE_VERSION` (located in `src/node_version.h`).

When Node.js is built (with an intention to distribute) with an ABI
incompatible with the official Node.js builds (e.g. using a ABI incompatible
version of a dependency), please reserve and use a custom `NODE_MODULE_VERSION`
by opening a pull request against the registry available at
<https://github.com/nodejs/node/blob/master/doc/abi_version_registry.json>.
