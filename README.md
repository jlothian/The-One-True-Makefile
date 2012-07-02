The One True Makefile
=====================

The purpose of this One True Makefile project is to provide a template
for projects that use Autoconf, Libtool and Make, but not Automake.
(I prefer not to use Automake, although obviously I like some of its
brothers.)  If you're going to use this template project, you will
probably want to already have a working knowledge of how Autoconf,
Libtool and Make work, although if you're still learning, you might
like to use the One True Makefile as a working example.  (This README
won't be much of a tutorial, though.)  In order to understand the
logic of e.g. what goes in `share` directories and what goes in `etc`
directories, you should probably also have a basic familiarity with
the Linux Filesystem Hierarchy Standard.

Scheme for include directives
=============================

Please note that, if your project provedes e.g. a
`MyProject::Foo::Foodle` class, then you should specify its interface
with an include directive that mirrors the namespace hierarchy, like
so:

    #include "MyProject/Foo/Foodle.h"

Thus, if the interface file is going to end up installed in a normal
way, at `/usr/local/include/MyProject/Foo/Foodle.h`, then the people
who use your library only have to give the compiler a
`-I/usr/local/include` flag, and in fact compilers usually already
have that path specified by default, so your users actually don't have
to do anything.  The `#include` directives mirror the namespace, and
compiling Just Works.

Please note that some people use a dirty scheme whereby their include
directive for a file installed somewhere like
`/usr/local/include/MyProject/Foo/Foodle.h` looks like this:

    #include "Foo/Foodle.h" // Bad!

This means that their users have to pass the compiler a
`-I/usr/local/include/MyProject` flag, since the compiler doesn't know
about the Foo subdirectory by default.  In fact, the user has to
include another compiler flag for every dependency that does this.

But you don't do that, because you're better than that.

(Please note that my rant is somewhat spoiled by the fact that tinyxml
does not use this project's namespace (i.e. something like
`OneTrueMakefile::TinyXML::TiXmlDocument`) because someone else (Lee
Thomason) wrote the (excellent) TinyXML library, and I just dropped it
into this One True Makefile project.  I didn't change how namespaces
work in TinyXML because I didn't want to muck around in the code too
much, for fear of introducing a subtle bug somewhere.)

Features
========

The One True Makefile's build philosophy is based on the one from
Peter Miller's "Recursive Make Considered Harmful" paper, although it
also has some other nice features.  Specifically, the One True
Makefile provides:

1. Modularity.

2. Incremental builds.  (Make knows about all the dependencies, so
    everything Just Works with a minimum of compiling.  There is no
    need running make clean every single compilation cycle.)

3. Automatic `#include` directive dependency generation.  (You need to
    have a compiler that accepts the -MM flag, like gcc.)  However,
    please note that there's no way for the computer to know that
    e.g. your foo program depends on the baz module, and also on
    libz.so, so you still have to specify library dependencies and
    link targets in the module.mk files.

4. Avoidence of recursive make.  (See Miller's paper.)

5. Parallel building using Make's `-j` flag.  (This is robust and
    works well with incremental builds thanks to point 4).)

6. Unit tests, build artifacts, and final output all go in their own
    dynamically-generated subdirectories.  Because of Libtool, you can
    run executables in the final output (staging) directory in-place,
    and shared libraries will Just Work, even if you haven't run make
    install.

7. A `make uninstall` target.  (I wish more autotools-based projects
    provided that.)  (You may also be interested in the trivial "make
    help" rule.)

8. A unit testing framework that leverages Python's unittest library.
    (Yes, unittest is designed for Python projects, but it's pretty
    trivial to use the subprocess module to wrap C and/or C++
    programs, and Python is a nice "glue" language to use if your
    testing process gets sophisticated.  For example, you may want to
    start doing things like starting multiple services before running
    for a single test, and stopping them afterward, and Python's
    `unittest` framework provides hooks for that.)

9. Flexibility, in that I've tried to write everything in such a way
    that, if you don't like what the One True Makefile is doing
    (blasphemer!), it should be obvious how it works and how you can
    change it.

10. I've included a copy of Lee Thomason's excellent TinyXML library,
    since I like using xml for config files.  If you don't like and/or
    need it, removing it is as easy as removing `tinyxml` from the
    modules section of `Makefile.in`, and running `rm -rf tinyxml`.
    (This is an example of the modularity of which I spoke, up in
    point 1).)  If you like TinyXML, you actually may want to consider
    switching to TinyXML2.  Note that the actual library will be
    called `libtinyxml-otm.so`, with a `-otm` at the end, in order to
    avoid conflicts with any other `libtinyxml.so` shared library
    files that may be installed.  (Again, if you don't like the `-otm`
    suffix, it's easy to change.)

Note that I have tried to provide the above benefits using minimalist
approaches; for example, I use implicit rules whenever possible.  This
makes the build machinery more readable, easier to understand and
easier to maintain.  If you feel overwhelmed reading the Makefile.in,
I highly recommend reading the Recursive Make Considered Harmful paper
by Miller.

Usage
=====

This template setup should build a minimal example project right out
of the box, using the standard autotools commands.  For example, if
your project uses some libraries that happen to be installed in your
`$HOME/.local` directory, you would run:

    autoconf
    LDFLAGS=-L$HOME/.local/lib CPPFLAGS=-I$HOME/.local/include ./configure
    make

After the the build process completes, the targets should be sitting
in `Linux-stage`.  (I'm assuming you're running Linux, but if not, the
slightly different name of the actual staging location should be
obvious.)  You can verify everything is working by running:

    ./Linux-stage/bin/hello

Note that the hello program successfully finds the `libfoo.so` shared
library, thanks to Libtool.

You could also do a `make install` and a `make uninstall` if you're
feeling saucy.

In order to adapt this template for your own project, you will
probably want to do the following:

* Replace this `README.ms` file with one that is appropriate for your
    project.  You should probably also replace (or delete) the
    `COPYING` file, and maybe the `INSTALL` file, if you've seen fit
    to modify the build/install process.

* Add in your own modules.  (You can replace the `foo` module and/or
    the `tinyxml` module, and use them as templates.  It's unlikely
    the "Hello, world!"  functionality provided by the foo module will
    be super-useful for your project, anyway.)

* Edit Makefile.in:

    - Change `pkg_name := one_true_makefile` to refer to the name of
        your project.

    - Change the `modules :=` area to reflect the names of your
        modules.  For example, if you killed the `foo` module, kept
        the `tinyxml` module, and added two more modules called bar
        and baz, you would change this section to:

    modules := \
    	tinyxml	bar	baz

    - Change the `testmodules :=` area to reflect the names of your
        unit test modules.  For example, if you used the `testfoo`
        module as a template to make a `testbar` module, you would
        change this section to:

    modules := \
    	testbar

* Edit your `module.mk` files to list all of the files you want to be
    listed as dependencies, and all the files you want to be affected
    by the `make install` target.  Everything uses the bottom source
    directory as the base directory, and the One True Makefile is set
    up to mirror the organization of the staging directory.  Things in
    etc or shared will just get copied around, since they don't need
    to be compiled.

    For example, if you made a `baz` module with a config file at
    `baz/etc/my_project/baz/bazqux.conf`, then you want to make sure
    `baz/module.mk` has this line:

    baz_etcs := $(stage_dir)/etc/$(PKG_NAME)/baz/bazqux.conf

    On the other hand, maybe you want your `bazqux.conf` file to go in
    the bottom of `etc`, instead of in a series of subdirectories.  In
    that case, then in your `baz` module the config file would be
    located at `baz/etc/bazqux.conf`, and line in `baz/module.mk`
    would look like this:

    baz_etcs := $(stage_dir)/etc/bazqux.conf

    In the first case, running `make install` with the default
    `/usr/local` value as your install prefix would install that
    config file to `/usr/local/etc/my_project/baz/bazqux.conf`.  In
    the second case, it would get installed to
    `/usr/local/etc/bazqux.conf`.  That is, you specify build targets
    in the staging directory (even if the "building" is just trivial
    copying).  The organization of your module, the organization of
    the staging directory, and the organization of the final install
    directory (e.g. `/usr/local`) will all mirror each other.

    Note that your `module.mk` files will probably also need to
    include logic regarding linking, near the bottom.  Be sure to
    include order-only rules (with a vertical bar) to tell Make to
    create relevant directories.  The `test_foo` module has an example
    of some slightly nontrivial linking logic, near the bottom of the
    file.

    Also note that `*.h` and `*.hpp` files that aren't going to be
    installed should *not* be mentioned in the `module.mk`.  The
    dependency autogeneration feature from gcc will deal with them,
    and since they aren't going to be installed, there's no reason to
    list them manually anywhere.

* If you're keeping tinyxml, and you've renamed your package from
    something other than `one_true_makefile` (which is
    understandable), then you'll need to edit the tops of the tinyxml
    headers and source files to reflect that.  For example, if your
    new project is called MyProject, you'll have to change lines that
    say

    #include "one_true_makefile/tinyxml/tinyxml.h"

   to

    #include "MyProject/tinyxml/tinyxml.h"

* Once your modules are set up, run `autoscan`, and replace
    `configure.ac`.  It is convenient to have your own Autoconf tests
    in a separate file (or files), so you should put your own Autoconf
    tests in the `project.m4` file, and add a line like this to your
    new `configure.ac`:

    m4_include([project.m4])

    The provided `project.m4` file has a handful of tests you might
    find useful.

    Also, I've assumed that `config.h` will be in subdirectory called
    `include`, instead of with all the build stuff in the base
    directory, so you will also want to modify the `AC_CONFIG_HEADERS`
    and `AC_CONFIG_SRCDIR` lines to this:

    AC_CONFIG_SRCDIR([include/config.h.in])
    AC_CONFIG_HEADERS([include/config.h])

    If you don't like the `include/config.h` scheme, it's easy to
    change; you just have to change some lines in `Makefile.in`, near
    the configure rules.  (It should be obvious what to change.)
    Also, `foo/module.m`k has a `-Iinclude` compiler flag, so if you
    don't like the `include/config.h` scheme, your modules should
    specify `-I.` instead.  (The dot after the -I is not a typo.
    Since all paths are relative to the base directory, where
    `Makefile.in` is, the dot refers to that base directory.)

    Our default version of `configure.ac` already has these two
    changes made, but if and when you run `autoscan`, you'll have to
    re-do them.  (Minimizing the amount of stuff you have to re-do
    after running `autoscan` is the purpose of the `project.m4`
    scheme.)

* Run `autoheader`.

TROUBLESHOOTING
===============

If something is screwed up, sometimes Make will silently die, rather
than giving a helpful error message.  You can tell whether Make is
silently dying by typing `make`; if it just returns, without saying
`make: Nothing to do for 'all'.` or anything else, then it is silently
dying.  Another obvious way is to see if there is a problem is to
check the return code by running:

    make || echo err

If it says "err", something went wrong, even if you don't have an
error message.

Trying `make -d`, `make --debug=a`, `make --debug=b`, `make
--debug=v`, `make --debug=i`, `make --debug=j` and/or `make --debug=m`
may be helpful.

Sometimes renaming files can cause problems, in that this causes the
dependency files (i.e. the `*.d` files) to be inaccurate.  A way to
fix this is to delete all the dependency files so that they will be
regenerated correctly.  One way to do this is to run:

    find . -type f -name "*.d" -exec rm -vf {} \;

You may also want to start fresh by deleting all the output of the
entire build process by running:

    rm Linux-* -rf

(If you're not on Linux, it should be obvious how to change this
line.)

Often, the problem will be in a specific spot in `Makefile.in` or one
of the `module.mk` files.  A way to identify the problem child is to
edit the `modules :=` area of `Makefile.in` and remove one module,
leaving all the other modules in the list.  If you can get it to say
`make: Nothing to do for 'all'.`, then the module you've removed is
the problem child.  Once you've narrowed the problem down to one file,
it should be easier to figure out what's wrong.  (Of course, if one of
the remaining modules depends on the removed module, Make may die with
an error message without getting to the end.  In that case, you may
have to play games with removing multiple modules at once.)

Another helpful source of information is to look in the `Linux-build`
directory (or whatever you've named it); if `foo/module.mk` is the
problem area, and `build/foo/foo.lo` exists, but `Linux-stage/bin`
does not contain the `foo` executable, then it apparently died before
creating the `foo` executable but after successfully compiling
`foo.lo`, which may be suggestive.  (This isn't definitive-- maybe the
`foo` executable also has other dependencies, and one of those is
causing the problem.)

Problems with Make can be opaque sometimes.  At one point I had a
problem with the `stamp-h.in` logic in `Makefile.in`, and it seemed
like Make was dying in one of the modules, in a completely unrelated
spot, since that module happened to look at `include/config.h`.

