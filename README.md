#obake#

Simple, developer-centric build system for OCaml.

##Motivation##

Strong static typing in a language like ocaml makes traditional build
management tools a pain to work with when actually writing code. The problem is
the compilation fails if any little thing is wrong, so if you make a small
change to a module, and want to test it, you have to fix a bunch of compiler
errors before Make will build you an executable. This can be a major annoyance
when making significant changes to a subsystem that other systems compile
against.

This problem is normally solved by an IDE, which will often let you run a unit
test as long as it's direct dependencies can be built. But in ocaml there is no
such IDE.

OBake only requires you to define a source, build, and bin directories, as well
as package dependencies via findlib. You can then compile individual .ml and
.mli files and their dependencies. ml files can also be compiled to executable
binaries simply by adding the -e switch. OBake discovers what dependencies a
source file has at run time, and compiles all dependent source files that have
changed since their last compilation.


##Quick Start (Example)##

- Generate an obake.yml configuration file for your project at the root of the
  project tree.  
      > cd example-ocaml-project
      > obake --generate-config
      generating config file: './obake.yml'
      done.

- Change the name of the sources directory if necessary in the yml file. Add
  any package dependencies (the obake.yml file tells you how).

- Enter the src directory and compile Porsche.ml. It relies on the Color module so
  it will consider Color "stale" as well b/c it won't find a prebuilt version. 
  The rules for whether just the mli or both the ml and mli files of Color are
  compiled are somewhat subtle. See the docs for the -b, -m, and -o options for
  more info.

      > cd src
      > obake Porsche.ml

      Stale Targets:
      ( 
        /vc/oss/obake/dev/example-ocaml-project/src/Color.cmi ,
        /vc/oss/obake/dev/example-ocaml-project/src/Porsche.cmo ,
        /vc/oss/obake/dev/example-ocaml-project/src/Porsche.cmi 
       )

- Can always clean up the built files and binaries

      > obake -c

- Compile the Porche module into a machine code executable

      > obake -m -e Porsche.ml
      Stale Targets:
      ( 
        /vc/oss/obake/dev/example-ocaml-project/src/Color.cmx ,
        /vc/oss/obake/dev/example-ocaml-project/src/Color.cmi ,
        /vc/oss/obake/dev/example-ocaml-project/src/Porsche.cmx ,
        /vc/oss/obake/dev/example-ocaml-project/src/Porsche.cmi 
       )
       
       > ls ../bin/
       Porsche.bin

##Pitfalls##

- Source code files may be separated into directories inside the src/ directory.
  However, obake will treat them all as if they were in a single directory. This
  means that two modules in different subdirectories may NOT have the same name,
  as obake cannot tell them apart. Source codes should therefore refer to 
  each other as if they were modules in the same directory.

- [Bug] If an executable is stale or non-existent, but the corresponding 
  .cmx or .cmo file is up-to-date, obake will not build you a new executable.

- At around 50 stale or uncompiled modules, Obake's dependency detection slows
  to the point of being useless. (There's an 2^n operation in there when it
  calls ocamldep recursively. The current implementation is very brute force.)
  To work around this, try to compile some subdirectories that have no
  dependencies on other subdirectories first, then the module that was taking
  too long to compile. Note this is only a problem after running a 'clean'
  operation. Once everything is compiling, if you edit one file and recompile,
  it will compile quickly.
  
  
##> obake --help##  
<pre>
       
