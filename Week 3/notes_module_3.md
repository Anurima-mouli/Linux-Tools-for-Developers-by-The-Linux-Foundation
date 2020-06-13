Linux Tools for Developers
==========================

by The Linux Foundation

# Week 3

#### Title: Compiling, Linking and Libraries

### gcc and Other Compilers

* **gcc** is a GNU C compiler, but it really stands for **G**NU **C**ompiler **C**ollection
* We can run it as either **gcc** or **cc** and it compiles programs written in a number of languages **C**, **C++**, as well as **Objective C**
* We can directly invoke it as a **C++ compiler** by running it as either **g++** or **C++**
* **gcc** is very closely linked with **libc**, **glibc** (is the new version), and the debugger **gdb**
* gcc extends far beyond Linux
* There are working versions of gcc for almost every operating system you can imagine
* **gcc** can be used for cross-compilation on different architectures
* Without **gcc** and **glibc**, there's no way Linux could have grown to what it is today
* If you're running Ada or Fortran or Pascal and a number of other languages, the code is first transparently converted to C and then compiled with gcc.
* It is also possible to use gcc to compile Java using the GCJ compiling interpreter
	* But this is no longer used on recent Linux distributions and the package itself is considered obsolete
* There's a number of **compiling stages** when you run gcc on a C program
	|Stage|Command|Default Input|Default Output|-w Switch|
	|----|----|----|----|----|
	|Preprocessing|cpp|.c|.i|-Wp....|
	|Compilation|gcc|.i|.s|N/A|
	|Assembly|as|.s|.o|-Wa....|
	|Linking|ld|.o|a.out|-Wl....|
	* First is the **preprocessing** step
	* Second is **compiling** from the preprocessing to the actual **assembly** code, and then once you have assembly code, you have to **link** it to construct an executable
* There is actually a name for each step: **cpp**, **gcc**, **as**, and **ld**

###### LLVM

* The **LLVMLinux** project provides an alternative compiler, which can be certainly used for applications and eventually for the Linux kernel
	* It actually doesn't work very well for compiling the Linux kernel, because there are a lot of "gcc-isms" in the actual kernel source, but people are working actively hard to get it to be used
* This grows out of a more general LLVM project, which grew out of the University of Illinois Champaign-Urbana

### Major gcc Options

* The compiled code format will be **ELF** (**E**xecutable and **L**inkable **F**ormat), which makes using shared libraries easy
* A good set of options to use is
	```bash
	-O2 -Wall -pedantic
	```

> Do not use `-pedantic` when compiling code for the Linux kernel, which uses many gcc extensions

###### Compiler Path Options
#
|Option|Description|
|:---:|:---:|
|-I dir|Include dir in search for included files; cumulative|
|-L|dir	Search dir for libraries; cumulative|
|-l|Link to lib; -lfoo links to libfoo.so if it exists, or to libfoo.a as a second choice|


###### Compiler Preprocessor Options
#
|Option|Description|
|:---:|:---:|
|-M|Do not compile; give dependencies for make|
|-H|Print out names of included files|
|-E|Preprocess only|
|-D|def	Define def|
|-U|def	Undefine def|
|-d|Print #defines|

###### Compiler Warning Options
#
|Option|Description|
|:---:|:---:|
|-v|Verbose mode, gives version number|
|-pedantic|Warn very verbosely|
|-w|Suppress warnings|
|-W|More verbose warnings|
|-Wall|Enable a bunch of important warnings|

###### Compiler Debugging and Profiling Options
#
|Option|Description|
|:---:|:---:|
|-g|Include debugging information|
|-pg|Provide profile information for gprof|

###### Compiler Input and Output Options
#
|Option|Description|
|:---:|:---:|
|-c|Stop after creating object files, do not link|
|-o file|Output is file ; default is a.out|
|-x lang|Expect input to be in lang, which can be c, objective-c, c++ (and some others); otherwise, guess by input file extension|

###### Compiler Control Options
#
|Option|Description|
|:---:|:---:|
|-ansi|Enforce full ANSI compliance|
|-pipe|Use pipes between stages|
|-static|Suppress linking with shared libraries|
|-O[lev]|Optimization level; 0, 1, 2, 3; default is 0|
|-Os|Optimize for size; use all -O2 options except those that increase the size|

### Static Libraries

* **Static libraries** have the extension **.a**
	* When a program is compiled, full copies of any loaded library routines are incorporated as part of the executable.
* The following tools are used for maintaining static libraries
	* **`ar`** creates, updates, lists and extracts files from the library
		* Syntax
			```bash
			$ ar rv libsubs.a *.o
			```
		* will create **libsubs.a** if it does not exist, and insert or update any object files in the current directory
	* **ranlib** generates, and stores within an archive, an index to its contents
		* It lists each symbol defined by the relocatable object files in the archive
		* This index speeds up linking to the library
		* Syntax
			```bash
			$ ranlib libsubs.a
			```
			* is completely equivalent to running **`ar -s libsubs.a`**
		* While running **ranlib** is essential under some UNIX implementations, under Linux it is not strictly necessary, but it is a good habit to get into
	* **nm** lists symbols from object files or libraries
		* Syntax
			```bash
			$ nm -s libsubs.a
			```
* Modern applications generally prefer to use shared libraries, as it is more **efficient** and **conserves memory**
* There are at least two circumstances where static libraries are still used
	1. For programs that are used early in system startup, before the tools to work with shared libraries are fully operational
	1. For programs that want to be completely self-contained and not have to deal with potential problems from system updates of libraries that are utilized by the application
		* This is typically done by either proprietary (and even closed source) application suppliers, or by other large vendors, such as Google or Mozilla
		* This is always controversial, because it is up to the vendor to make sure that any security holes or other bugs are fixed when they are discovered upstream in the included libraries

### Shared Libraries

* A single copy of a shared library can be used by many applications at once; thus, both executable sizes and application load time are reduced
* Shared libraries have the extension **.so**
* Typically, the full name is something like **libc.so.N**
	- where **N** is a major version number
* Under Linux, shared libraries are carefully versioned. For example, a shared library might have any of the following names
	* **libmyfuncs.so.1.0** - The actual shared library
	* **libmyfuncs.so.1** - The name included in the **soname** field of the library. Used by the executable at runtime to find the latest revision of the v. 1myfuncs library
	* **libmyfuncs.so** - Used by **gcc** to resolve symbol names in the library at link time when the executable is created.
* To **create** a **shared library**, first you must compile all sources with the `-fPIC` option, which generates so-called **Position Independent Code**
	* **NOTE** - do not use `-fpic`; it produces somewhat faster code on **m68k**, **m88k**, and **Sparc chips**, but imposes arbitrary **size limits** on shared libraries
	```bash
	$ gcc -fPIC -c func1.c
	$ gcc -fPIC -c func2.c
	```
* To create a shared library, the option **-shared** must be given during compilation, giving the **soname** of the library, as well as the full library name as the output
	```bash
	$ gcc -fPIC -shared -Wl,-soname=libmyfuncs.so.1 *.o -o libmyfuncs.so.1.0 -lc
	```
	* where the **-Wl** tells gcc to pass the option to the linker
	* The **-lc** tells the linker that **libc** is also needed
* **NOTE**
	* You can usually get away ignoring the **-fPIC** and **-Wl**, **soname** options, but it is not a good idea
		* This is because **gcc** normally emits such code anyway
		* In fact, giving the option prevents the compiler from ever issuing position-dependent code
* Note that it is really the **linker** (**ld**) that is doing the work, and the above step could also have been written as
	```bash
	ld -shared -soname=libmyfuncs.so.1 *.o -o libmyfuncs.so.1.0 -lc
	```
* To get the above to link and run properly, you will also have to do
	```bash
	$ ln -s libmyfuncs.so.1.0 libmyfuncs.so
	$ ln -s libmyfuncs.so.1.0 libmyfuncs.so.1
	```
	* If you leave out the **first** symbolic link, you will not be able to compile the program
	* if you leave out the **second**, you will not be able to run it
	* It is at this point that the version check is done, and if you do not use the **-soname** option when compiling the library, no such check will be done
* In many cases, both a shared and a static version of the same library may exist on the system, and the linker will choose the shared version by default
	* This may be overridden with the **-static** option to the linker
		* However, if you do this, all libraries will be linked in statically, including **libc**, so the resulting executable will be large; you have to be more careful than that
* The GNU **libtool** script can assist in providing shared library support, helping with compiling and linking libraries and executables, and is particularly useful for distributing applications and packages
* Full documentation can be obtained by doing **info libtool**
* The linker first searches any directories specified in the environment variable LD_LIBRARY_PATH, a colon separated list of directories, as in the PATH variable
	```bash
	$ LD_LIBRARY_PATH=$HOME/foo/lib
	$ foo [args]
	# OR
	$ LD_LIBRARY_PATH=$HOME/foo/lib  foo [args]
	```

### Linking to Libraries

* Whether a library is static or fixed, executables are linked to the library with
	```bash
	$ gcc -o foo foo.c -L/mypath/lib -lfoolib
	```
	* This will link in **/mypath/lib/libfoolib.so**, if it exists, and **/mypath/lib/libfoolib.a** otherwise
* The name convention is such that **-lwhat** refers to library **libwhat.so(.a)**
* If both **libwhat.so** and **libwhat.a** exist, the shared library will be used, unless **-static** is used on the compile line
* Poorly designed projects may have circular library dependencies, and since the loader makes only one pass through the libraries requested, like
	```bash
	$ gcc .... -lA -lB -lA ..
	```
	* if libB refers to a symbol in libA which is not otherwise referred to. While there is an option to make the loader make multiple passes (see info gcc for details) it is very slow, and a proper, layered, library architecture should avoid this kind of going in circles
* The default library search path will always include **/usr/lib** and **/lib**
* To see exactly what is being searched you can do 
	```bash
	$ gcc --print-search-dirs
	```
* User-specified library paths come before the default ones, although there are extended options to **gcc** to reverse this pattern

##### Stripping executables

> **Do not use strip on either the Linux kernel or kernel modules, both of which need the symbol information!**

* Syntax
	```bash
	$ strip foobar
	```
	* where foobar is an executable program, object file, or library archive, can be used to reduce file size and save disk space


##### Getting Debug Information
* We can use the environment variable **LD_DEBUG** to obtain useful debugging information
	* Syntax
		```bash
		$ LD_DEBUG=help
		```
		* and then typing any command gives
			```bash
			$ ls
			Valid options for the LD_DEBUG environment variable are:

			  libs        display library search paths
			  reloc       display relocation processing
			  files       display progress for input file
			  symbols     display symbol table processing
			  bindings    display information about symbol binding
			  versions    display version dependencies
			  scopes      display scope information
			  all         all previous options combined
			  statistics  display relocation statistics
			  unused      determined unused DSOs
			  help        display this help message and exit

			To direct the debugging output into a file instead of standard output
			a filename can be specified using the LD_DEBUG_OUTPUT environment variable.
			$
			```
* We can also show how symbols are resolved upon program execution, and help find things like linking to the wrong version of a library
	* Syntax
		```bash
		$ LD_DEBUG=all  ls
		```

### Debugging with gdb

* The **dominant** debugger used on Linux systems is **gdb**, no matter what high level programming language was used
* **gdb** is the **GNU debugger**
	* When you launch it, it can process various arguments and configuration options from a file in your home directory, or in the current working directory
	* You can step through **C** and **C++** programs, set **breakpoints**, **display variables**, do everything imaginable you could think of to do with a debugger
	* Even work with other languages such as **Fortran** and **Ada** that use **gcc** as a backend as well, including even stepping through the source
* You need to compile with the **-g** option to get full debugging information in the executable and make it available to **gdb**
	* But even if you don't, you can still get some useful information from **gdb**
		* such as where you were in the program when it crashed, if that's the situation
* Now, there are a number of **Graphical Interfaces** to **gdb** for people who like to debug by dragging stop signs, to set a breakpoint, etc., and it's nice because you can keep the source display at the same time as assembly language as well as command input, and output, etc
* One very popular choice today are large integrated development environments, IDEs. With **Eclipse** being the most popular, from Eclipse Foundation
* **ddd** is a standard package run by **GNU** these days
	* It's usually easily installed on any distribution
	* It's not a complete integrated development environment, as Eclipse is

#
#### Title: Java Installation and Environment

### Write Once, Use Anywhere?

> The concept of **Write Once, Run Anywhere** works better on the server side than on the client side because of the difficulties of dealing with human interface standards and desktop/window managers and operating systems

* The concept of **"Write Once, Run Anywhere"** (**WORA**) or **"Write Once, Run Everywhere"** (**WORE**) was a motivating factor for the development of the **Java** language by **Sun**
* the idea is you could develop your Java code on any machine that has a **JVM**, a **J**ava **V**irtual **M**achine, then use it on any other platform that also has a **JVM**
	* And then, once you **compile** your **Java program** into **bytecode**, it should run on any hardware, from a cellphone to a mainframe, on any operating system without further rewriting
* There are **multiple JVM implementations** available, there are ones from **Oracle** and **Sun**, **openjdk**, and **IBM**
* **JVM** implementations are much more stable now and matured, and standardized compliance is much better
* If you write something for a **small embedded** type device, you may not have to run it on a **mainframe**

##### Handling Multiple JREs and JDKs: The Alternatives System

* It is possible that you will have (or want to have) multiple versions of Java components installed on your system
* While you can directly invoke a specific choice without disturbing the general system configuration, it is also possible to set the system default in an easily reversible way
	* In order to enable this, most Linux systems use the **alternatives tool** to set the system default
	* This utility is used for many alternative setting tasks on the system, not just those which are Java-related
	* For example
		* To reconfigure the choice for Java on a Red Hat-based system
			```bash
			$ sudo alternatives --config java

			There are 2 programs which provide ’java’.
			       
			  Selection    Command
			-----------------------------------------------
			   1           java-1.7.0-openjdk.x86_64 \
			                      (/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.161-2.6.12.0.el7_4.x86_64/jre/bin/java)
			*+ 2           java-1.8.0-openjdk.x86_64 \
			                      (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/bin/java)
			Enter to keep the current selection[+], or type selection number:
			```
* On **Debian-based** systems, such as Ubuntu, the command is **update-alternatives**
* The directory /etc/alternatives contains symbolic links to the proper location
	```bash
	$ ls -l /etc/alternatives/java

	lrwxrwxrwx 1 root root 73 Jan 19 07:02 /etc/alternatives/java -> \
	     /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/bin/java
	```
* Note that the generic Java binary is also just a symbolic link
	```bash
	$ which java
	/usr/bin/java
	$ ls -l /usr/bin/java
	lrwxrwxrwx 1 root root 22 Jan 19 07:02 /usr/bin/java -> /etc/alternatives/java
	```
* We can also set this for the Java compiler, javac, as in
	```bash
	$ sudo alternatives --config javac
	     
	There is 1 program that provides ’javac’
	       
	 Selection    Command
	-----------------------------------------------
	*+ 1           java-1.7.0-openjdk.x86_64 \
	                 (/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.161-2.6.12.0.el7_4.x86_64/bin/javac)
	Enter to keep the current selection[+], or type selection number:
	```
* To just change versions for a specific user, you could put something like this in the **`$HOME/.bashrc`** file
	```bash
	export JAVA_HOME=/usr/lib/jvm/java-1.6.0-sun-1.6.0.21.x86_64/jre
	export PATH=$JAVA_HOME/bin:$PATH
	```
* To check java version
	* Syntax
		```bash
		$ java -version
		```

##### Environment Variables and Class Paths

* Your basic path is set correctly as described in the **alternatives** section. You might also try
	```bash
	$ readlink -f $(which java)
	```
	* which will follow the links to print out the path to the actual installation
* The **CLASSPATH** environment variable can be used to locate user classes that are in addition to those directly part of the **jdk** or **jre** installation and extensions
* While you can always set this variable in your **`.bashrc`** initialization file, this becomes associated with the environment only of the particular user and is effective at all times
* It is thus generally better to use the **`-cp`** option when invoking Java with the application: this could be made part of a startup script, for example

### Integrated Development Environments: Eclipse and NetBeans

##### Eclipse

* Eclipse can be traced back to at least 1984. Since 2001 it has been an open source project, and it's been under the Eclipse Foundation since 2004
* It's released under its own license, the **Eclipse Public License**
* It's a little bit incompatible with the GPL, but this has no effect on any product you develop using Eclipse
* It only causes some complications if you want to extend or modify the IDE itself
* Eclipse is very modular
	* Originally, primarily for Java projects, but now it will work on almost any language

##### Netbeans

* NetBeans goes back to 1997. It has passed through Sun and Oracle, and today it's released under CDDL and the GPL version 2 licenses
* You'll find it in some distributions, in their standard packaging system
* If you have used NetBeans on another operating system, it'll look the same on Linux, and there should be no real learning curve involved

#
#### Title: Building RPM and Debian Packages

### Package Management

> Maintaining coherence and ensuring proper licensing compliance is a very difficult task

* Originally, most **Linux distributions** were simply collections of **tarballs** - archives of either binaries or sources which needed to be compiled in order to install various applications and other system services, etc.
* You can still find distributions, including **Slackware**, which is one of the oldest distributions, if not the oldest distribution, which is still operating, that still function that way
	* There are disadvantages to distributing things this way. It can be difficult to remove all the files from a software package, if you need to remove it. You might delete some package that other packages need
	* Or install a package that won't work, because it needs other packages that haven't been installed yet
* The two most common Package Management systems are **RPM** and **APT**
	* **RPM** stands for the **R**ed Hat **P**ackage **M**anager; while originally it was developed by Red Hat, it's used by other distributions as well, even ones which aren't directly related to **Red Hat**, such as **SUSE** and **OpenSUSE**
* **APT**, the **A**dvanced **P**ackaging **T**ool, produces the so-called Debian packages
	* It came, not surprisingly, from the Debian distribution, but it's the foundation of **Ubuntu**, and **Linux Mint**, and other distributions which are built on top of Debian
* The other packaging systems use, for instance
	* Gentoo uses Emerge and Portage
	* Arch Linux uses Pacman
* There's a tool called **alien** which can convert packages between RPM and Debian, but it's rather imperfect
* **BENEFITS for DEVELOPERS**
	1. Repeatable builds, so that no matter how something is installed, it comes out the same
	1. You can keep track of dependency data, you know what packages need other packages
	1. You can include the original sources and you can also show all the steps involved in, perhaps modifying them, recompiling, and doing some configuration control
	1. You can document any explanation of changes that are made to the upstream sources
	1. Simple installation and removal methods
* **BENEFITS for Admins and End-Users**
	1. You can easily verify the integrity of installation. Example: see if any files have been modified
	1. It's easy to install and remove packages
		1. And, when you do this, you can preserve any customized and modified configuration files you may have changed
	1. You can check on install and remove that you're not getting rid of any resources that you really need to keep
	1. You can easily do things like find out what files are in a package or look at a given file and ask what package it belongs to
* The maintenance of software by using package systems is a critical task on Linux distributions
* You need a method of deploying changes made by the upstream developers back to the end users and system administrators
	* You can establish clear policies for how configuration files are dealt with
	* You can make sure that you don't have bit rot, that packages are updated and upgraded in timely fashion, and bugs and security holes are not allowed to proliferate.
	* Most important thing to make sure you get all the package dependencies correctly
		* if something changes, anything that depends on is also explicitly updated, as well.

##### RPM Creation

> **You should never remove the rpm program itself!**

* The **RPM** packaging system separates its main functionality methods using two main programs
	1. **rpm**: Handles querying, installing, upgrading and erasing.
	1. **rpmbuild**: Handles creating and manipulating source and binary packages.
* Each of these programs comes packaged as part of its own rpm and comes together with other related utilities
	```bash
	$ rpm -qil rpm-build | grep bin
	/usr/bin/gendiff
	/usr/bin/rpmbuild
	/usr/bin/rpmspec
	$ rpm -qil rpm | grep bin
	/bin/rpm
	/usr/bin/rpm2cpio
	/usr/bin/rpmdb
	/usr/bin/rpmkeys
	/usr/bin/rpmquery
	/usr/bin/rpmverify
	```
* There are three basic ingredients required to assemble the source and binary packages
	1. A **tarball** file containing all source code, makefiles, default configuration files, documentation, etc.
	1. Any **patch** files needed to be applied to the upstream source encapsulated in the tarball.
	1. A **spec** file that must be written that describes the package and how to patch and build it, dependency information, etc.
* Once you have all these files together, the command
	* Syntax
		```bash
		$ rpmbuild -ba specFile
		```
		* will perform the following steps
			* Coalesce the input files into the source RPM
			* Unpack the source tar file
			* Apply all patch files
			* Build binaries for the current architecture
			* Package binaries and config files into a binary RPM file
	* RPM can be picky about where these files are placed

##### The RPM spec File

* A **spec** file is a collection of package information and shell scripts
* Many of of the shell scripts are very short (one line scripts are common). Each script performs one of the tasks necessary in building, installing, or uninstalling a package
* The main sections of the file are
	|Section Name|Required?|Section Type|Purpose|
	|:---:|:---:|:---:|:---:|
	|Header (start of file)|Y|Data fields|Provide the rpm -i data|
	|%description|Y|ASCII text|Provide a text description of the package|
	|%prep|Y|Shell script|Unpack source code and apply patches|
	|%changelog|Y|Change history|History of changes|
	|%build|Y|Shell script|Build the binary from the unpacked sources|
	|%install|Y|Shell script|Copy the binaries to their destination|
	|%files|Y|List|Every file that the package provides is listed|
	|%clean|N|Shell script|Cleans up the sources after a build is done|
	|%pre|N|Shell script|Steps to take before installation of package|
	|%post|N|Shell script|Steps to take after installation of package|
	|%preun|N|Shell script|Steps to take before removal of package|
	|%postun|N|Shell script|Steps to take after removal of package|
* **Note** that RPM makes heavy use of macros. The file **`/usr/lib/rpm/macros`** contains those already available and may be worth examining

##### Details on RPM spec Sections

###### The Header Section
* The Header section is a list assignments, of the format
	```bash
	Attribute: Value
	```
* An example showing the attributes usually used is provided below
	```bash
	Summary: A great application!
	Name: my_app
	Version: 1.0
	Release: 2
	License: GPLv2
	Group: Applications/Text
	Source: ftp://ftp.myserver.com/pub/my_app/my_app-1.0.tgz
	URL: https://www.myserver.com/my_app/index.html
	Vendor: The Best Software Company
	Packager: A genius <genius@myserver.com>
	Patch0: my_app-1.0.patch0
	Patch1: my_app-1.0.patch1
	BuildRoot: /var/tmp/%{name}-buildroot
	```
* The **Name**, **Version**, and **Release** values are used to construct the names of the source and binary RPM files. Most other values are fairly self-explanatory, and can be chosen at will by the packager
* **Note** that the **License** field was called **Copyright** in earlier versions of RPM

###### The description Section

* The **description section** is completely free form. To see the description of an existing package, try rpm -qi package. An example description
	```bash
	%description
	This program enables the user to do everything that could
	possibly be useful under Linux.
	```

###### The prep Section

* The **prep** section of the header must take the tar file of the source code, unpack it, and then apply any patch files needed
	* Often, this section is extremely short
* Example
	```bash
	%prep
	rm -rf $RPM_BUILD_DIR/my_app-1.0
	tar zxvf $RPM_SOURCE_DIR/my_app-1.0.tar.gz
	patch -p1 < my_app-1.0.patch1
	patch -p2 < my_app-1.0.patch2
	```
* Modern versions of **rpmbuild** do not use such detailed **prep** sections. They can simply do
	```bash
	%prep
	%setup -q
	%autosetup
	```

###### The changelog Section

* The **changelog** section lists a history of changes to the package
* Example
	```bash
	%changelog
	* Tue Feb 29 2001 Mr Somebody <sbody@foo.com>
	- important fix: cleaned up a bug that trashed the filesystem.
	     
	* Tue Feb 15 2001 Another Somebody <abody@foo.com>
	- made minor changes to the initialization routine.
	```


###### The build Section

* The **build** section contains the commands needed to build your program. If your prep section did its work correctly, this should be one command
	```bash
	%build
	make
	```

###### The install Section

* The **install** section copies files to their final place in the filesystem
* This is run after building the binaries, but not when the package is installed
* When the **binary** is built, **RPM** will automatically package up the files that were put in place by the install section, then during installation, it will copy them back
* Usually, developers put an install target into makefiles that do this copying, which makes the install section of the spec file simple
	```bash
	%install
	make install
	```

###### The files Section

* The **files section** of the spec file lists all of the files that the install section copied into place. Files may be prefixed by an adjective that changes how RPM treats these files
* The possible adjectives are:
	|Adjective|Description|
	|:---:|:---:|
	|%doc|Marks documentation from the source package to be included in a binary install. Multiple documents can be listed on the same line, or on separate lines. If no path for a documentation file is specified, then it will be put in a directory named /usr/share/doc/package-version-build.|
	|%config|Mark configuration files. When you uninstall a package, any configuration files which have been changed will be saved with a .rpmsave extension.|
	|%dir|By default, if you include a directory in the files section, the directory and all files it contains are put in as part of installation. Using %dir means that only the directory, and not the files that it contains, will be copied when the package is installed.|
	|%attr|Set attributes for the file. An example would be %attr(666,root,root), where the three fields are mode, owner, and group (just like the output of ls -l). Defaults are gotten by using a - in a place.|
	|%defattr|The same as %attr, but you do not add a file to it. Instead, all files listed after the %defattr will automatically get the new attributes (unless they have their own %attr directive).|
* Example
	```bash
	%files
	%doc README
	/usr/bin/my_app
	%config /usr/bin/my_app_configure
	/usr/man/man/my_app.1
	%attr(600,-,root) /usr/lib/my_app/my_data
	```

##### RPM Dependencies

> You don't have to be a superuser to build a binary RPM package on CentOS and openSUSE, but you do to install it

* In the **spec** file you may specify three types of dependency information:
	1. Capabilities that this package provides
	1. Capabilities that this package requires
	1. Packages that this package requires.
* A capability is a required function or class of functions. You can see what libraries a package requires with
	```bash
	$ rpm -qR package
	```
* The names you see as the capabilities is not the full path name of the library; instead, it is what is called the **soname** of the library.
* Any dynamic libraries in the files section of the spec are automatically added as capabilities that the package provides
* In addition, RPM will automatically run scripts (called **find-requires** and **find-provides**) that determine which dynamic libraries your binary requires, so these will automatically be added to the list of capabilities that your package requires
* If your package has other requirements besides dynamic libraries, you can specify that another package must be installed by putting code of this format in the header section of your spec file
	```bash
	requires: package
	requires: package >= version
	requires: package >= version-build
	```
* In addition to the **>= test** (which requires that a package of the version specified or later is installed), you can also use **>**, **=**, **<**, and **<=**

##### Debian Package Creation Workflow

> **Note** the use of the underscore which has replaced the hyphen in the name; this can be very confusing!

* Building a Debian package requires many more files than does the RPM system, where almost all the complications go into the spec file
* At first glance, it is almost frighteningly complicated
* However, there exist auxiliary utilities such as **debuild** and **cdbs** which can automate many tasks, provide templates for files which do not often have to be complex, and give command line interfaces for updating the various configuration files
* Basic package building workflow
	* First you will need the upstream package source, provided as a tarball in the form
		```bash
		package-version.tar.gz
		```
		* such as **someprogram-1.0.tar.gz**
			* Note the use of the hyphen (-) between the package name and the version number
		* The system is very picky about the name; it must contain only lower case letters, digits, plus and minus signs and periods
			* It cannot contain dashes or underscores
	* You will also need any patches that need to be made to the upstream source
	* The source will be expanded into **someprogram-1.0** and the original source tarball will be preserved in
		```bash
		package_version.orig.tar.gz
		```
	* A directory **someprogram-1.0/debian** is created, and will be filled with all the files needed by the package building system
	* Source and binary packages are built from these ingredients; the binary package has the **.deb** suffix
