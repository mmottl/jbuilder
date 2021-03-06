*****************************************
Project Layout and Metadata Specification
*****************************************

A typical jbuilder project will have one or more ``<package>.opam`` file
at toplevel as well as ``jbuild`` files wherever interesting things are:
libraries, executables, tests, documents to install, etc...

It is recommended to organize your project so that you have exactly one
library per directory. You can have several executables in the same
directory, as long as they share the same build configuration. If you'd
like to have multiple executables with different configurations in the
same directory, you will have to make an explicit module list for every
executable using ``modules``.

The next sections describe the format of Jbuilder metadata files.

Note that the Jbuilder metadata format is versioned in order to ensure
forward compatibility. Jane Street packages use a special
``jane_street`` version which correspond to a rolling and unstable
version that follows the internal Jane Street development. You shouldn't
use this in your project, it is only intended to make the publication of
Jane Street packages easier.

Except for the special ``jane_street`` version, there is currently only
one version available, but to be future proof, you should still specify
it in your ``jbuild`` files. If no version is specified, the latest one
will be used.

Metadata format
===============

Most configuration files read by Jbuilder are using the S-expression
syntax, which is very simple. Everything is either an atom or a list.
The exact specification of S-expressions is described in the
documentation of the `parsexp <https://github.com/janestreet/parsexp>`__
library.

Note that the format is completely static. However you can do
meta-programming on jbuilds files by writing them in :ref:`ocaml-syntax`.

.. _opam-files:

<package>.opam files
====================

When a ``<package>.opam`` file is present, Jbuilder will know that the
package named ``<package>`` exists. It will know how to construct a
``<package>.install`` file in the same directory to handle installation
via `opam <https://opam.ocaml.org/>`__. Jbuilder also defines the
recursive ``install`` alias, which depends on all the buildable
``<package>.install`` files in the workspace. So for instance to build
everything that is installable in a workspace, run at the root:

::

    $ jbuilder build @install

Declaring a package this way will allow you to add elements such as
libraries, executables, documentations, ... to your package by declaring
them in ``jbuild`` files.

Jbuilder will only register the existence of ``<package>`` in the
subtree starting where the ``<package>.opam`` file lives, so you can
only declare parts of the packages in this subtree. Typically your
``<package>.opam`` files should be at the root of your project, since
this is where ``opam pin ...`` will look for them.

Note that ``<package>`` must be non empty, so in particular ``.opam``
files are ignored.

Package version
---------------

Note that Jbuilder will try to determine the version number of packages
defined in the workspace. While Jbuilder itself makes no use of version
numbers, it can be use by external tools such as
`ocamlfind <http://projects.camlcity.org/projects/findlib.html>`__.

Jbuilder determines the version of a package by first looking in the
``<package>.opam`` for a ``version`` variable. If not found, it will try
to read the first line of a version file in the same directory as the
``<package>.opam`` file. The version file is any file whose name is, in
order in which they are looked for:

-  ``<package>.version``
-  ``version``
-  ``VERSION``

The version file can be generated by a user rule.

If the version can't be determined, Jbuilder just won't assign one.

Note that if you are using `Topkg <https://github.com/dbuenzli/topkg>`__
as well in your project, you shouldn't manually set a version in your
``<package>.opam`` file or write/generate on of the file listed above.
See the section about :ref:`using-topkg` for more details.

Odig conventions
----------------

Jbuilder follows the `odig <http://erratique.ch/software/odig>`__
conventions and automatically installs any README\*, CHANGE\*, HISTORY\*
and LICENSE\* files in the same directory as the ``<package>.opam`` file
to a location where odig will find them.

Note that this include files present in the source tree as well as
generated files. So for instance a changelog generated by a user rule
will be automatically installed as well.

jbuild-ignore
=============

By default Jbuilder traverses the whole source tree, ignoring the
following files and directories:

- any file that start with ``.#``
- any directory that start with either ``.`` or ``_``

To ignore a subtree, simply write a ``jbuild-ignore`` file in the
parent directory containing the name of the sub-directories to ignore.

So for instance, if you write ``foo`` in ``src/jbuild-ignore``, then
``src/foo`` won't be traversed and any ``jbuild`` file it contains will
be ignored.

``jbuild-ignore`` files contain a list of directory names, one per line.
