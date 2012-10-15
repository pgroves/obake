#obake#

Simple, developer-centric build system for OCaml.

##Motivation##

Strong static typing in a language like ocaml makes traditional build
management tools a pain to work with when actually writing code. The problem is
the compilation fails if any little thing is wrong, so if you make a small
change to a module, and want to test it, you have to fix a bunch of compiler
errors before Make will build you an executable. This can be a major annoyance
when making significant changes to a subsystem that other systems compile
against. This is a mismatch between build systems designed to build and deploy
the contents of a code repository and the workflow of software developers.

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

- Can always clean up the built files and binaries.

  ```
  > obake -c
  ```

- Compile the Porche module into a machine code executable.
  
  ```
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
  ```

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
  
##Prerequisites##

- Ruby 1.8.7 and text/highlight gem

```
> [sudo] apt-get install ruby
> [sudo] gem install text-highlight
```

- OCaml (3.09+) and ocamlfind

```
> [sudo] apt-get install ocaml
> [sudo] apt-get install ocaml-findlib
```

  
## \> obake \-\-help ##


<pre>
       

                                         OBake 


Synopsis

   obake {-q|-v} {-f  <filename>} {-h} {-gt} {-m|-b} {-e} {-c} {-p} {-o}

Description

   OBake is a Bare-minimum build tool for Ocaml that is not make. 


Options
   '+' indicates option can be set in config file.

   _________
 + Verbosity
   How much info to print during execution 

      -q, --quiet
         Print errors only 

      -v, --verbose
         Print full config report on init, plus other info 

   ____________________
   Config File Location
   The name of a config file to use other than the default 

      -f <filename>,
      --config-file <filename>
   __________
   Print Help

      -h, --help
         Prints this help message 

   ____________________
   Generate Config File

      -gt, --generate-config
         Generates an example config file and places it at ./obake.yml 

   _____________
 + Compiler Mode
   Ocaml can compile to machine code or byte code. The two are functionally 
   equivalent at both compile time and runtime, but byte code is faster to compile 
   and slower at runtime. Has no real effect when compiling .mli (header) files. 

      -m, --machine-code
         Compile all files to machine code using ocamlopt 

      -b, --byte-code
         Compile all files to byte code using ocamlc. This is the default. 

   _________________
   Build Executables
   OBake can build executables by deducing exactly which modules are needed to link 
   the target source files' compiled version and then perform the linking step. 
   Executables are given extension .bin and placed in the binDir defined in the 
   config file. 

      -e, --executable
         Build executables for the input filenames 

   _____
   Clean
   Delete contents of buildDir and binDir before doing any compilation. Can be used 
   independently of compiling operations, that is ">obake -c myFile.ml" will clear 
   the output directories and then compile myFile.ml, while ">obake -c" will simply 
   clear the output directories and quit. 

      -c, --clean
         Clean before proceeding to compilation. 

   ________________
   Enable Profiling
   Enable profiling of executables. All files will need to be recompiled, as well as 
   the linking step. Running the exec will then generate a dump file, which can be 
   viewed with gprof or XXX. Only affects machine-code compiled files. 

      -p, --profiling
         Enable executables to be analyzed by a profiler. 

   _________________________
   Force ByteCode Impl Files
   When compiling byte code files, the default is to only requirecmi (compiled 
   headers) of dependencies to be present. This will force the cmo (compiled impls) 
   of dependency modules to be built. This is turned on automatically when building 
   byte code executables. 

      -o, --force-cmos
         Force implementation (cmo) dependencies for byte code comp. 


Config File
   The config file used will be the first instance of obake.yml found when ascending 
   the directories of the local machine's filesytem, starting at the directory this 
   command is run from. Use --generate-config to create a template config file at 
   './obake.yml'. 

Config File Only Options
   These options can only be set through a config file. 

   _________________
   Sources Directory
      The root directory of where the project sources are. While the specific files 
      to compile will be given as arguments on the commandline, the location of all 
      source files to consider is necessary to auto-detect dependencies. 

   _______________
   Build Directory
      The directory where files generated by obake will be output to. This includes 
      .ml and .mli files genrated by the lexer and parser but not executable .bin 
      files. 

   _____________________
   Executables Directory
      The directory where files linked into executables by obake will beoutput to. 

   _________
   Libraries
      A list of ocamlfind package names to include in the compilation. Valid values 
      can be found by the command'>ocamlfind list' 

-/end help
</pre>
