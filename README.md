What OCaml Compiles when it Compiles
====================================

This document is written from a state of ignorance. Please forward all
corrections to john@coherentgraphics.co.uk or make an issue or pull request
here:

http://www.github.com/johnwhitington/what-ocaml-compiles

This document describes what happens when one runs "./configure; make world;
make opt; make opt.opt; make install" on a MAC OS X system. It does not attempt
to explain what happens on every platform with every option.

1. What's in the checkout
-------------------------

2. Running configure
--------------------

```
Configuring OCaml version 4.03.0+dev10-2015-07-29
Configuring for host x86_64-apple-darwin15.0.0 ...
Configuring for target x86_64-apple-darwin15.0.0 ...
Using compiler gcc.
Compiler family and version: clang-__clang_major-1.
The C compiler is ISO C99 compliant.
Checking the sizes of integers and pointers...
Wow! A 64 bit architecture!
This is a little-endian architecture.
Doubles can be word-aligned.
64-bit integers can be word-aligned.
ranlib found
#! appears to work in shell scripts.
POSIX signal handling found.
expm1(), log1p(), hypot(), copysign() found.
getrusage() found.
times() found.
termcap functions found (with libraries '-lcurses')
You have BSD sockets.
socklen_t is defined in <sys/socket.h>
inet_aton() found.
IPv6 is supported.
stdint.h found.
unistd.h found.
off_t is defined in <sys/types.h>
dirent.h found.
rewinddir() found.
lockf() found.
mkfifo() found.
getcwd() found.
getwd() found.
getpriority() found.
utime() found.
utimes() found.
dup2() found.
fchmod() found.
truncate() found.
sys/select.h found.
select() found.
symlink() found.
waitpid() found.
wait4() found.
getgroups() found.
setgroups() found.
initgroups() found.
POSIX termios found.
Asynchronous I/O are supported.
setitimer() found.
gethostname() found.
uname() found.
gettimeofday() found.
mktime() found.
setsid() found.
putenv() found.
setlocale() and <locale.h> found.
dlopen() found.
Dynamic loading of shared libraries is supported.
mmap() found.
pwrite() found
stat() supports nanosecond precision.
mkstemp() found
nice() found
Replay debugger supported.
System stack overflow can be detected.
POSIX threads library supported.
Options for linking with POSIX threads: -lpthread
sigwait() found
Bytecode threads library supported.
X11 works
Options for compiling for X11: -I/opt/X11/include
Options for linking with X11: -L/opt/X11/lib -lX11
[WARNING] BFD library not found, 'objinfo' will be unable to display info  on .cmxs files.
Assembler supports CFI

** Configuration summary **

Directories where OCaml will be installed:
        binaries.................. /usr/local/bin
        standard library.......... /usr/local/lib/ocaml
        manual pages.............. /usr/local/man (with extension .1)
Configuration for the bytecode compiler:
        C compiler used........... gcc
        options for compiling.....  -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT
        options for linking.......     -lcurses -lpthread
        shared libraries are supported
        options for compiling.....   -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT
        command for building...... gcc -bundle -flat_namespace -undefined suppress                    -Wl,-no_compact_unwind -o lib.so /a/path objs
Configuration for the native-code compiler:
        hardware architecture..... amd64
        OS variant................ macosx
        C compiler used........... gcc
        options for compiling.....  -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT
        options for linking.......   
        assembler ................ clang -arch x86_64 -c
        preprocessed assembler ... clang -arch x86_64 -c
        assembler supports CFI ... yes
        with frame pointers....... no
        naked pointers forbidden.. no
        native dynlink ........... true
        profiling with gprof ..... supported
Source-level replay debugger: supported
Additional libraries supported:
        unix str num dynlink bigarray systhreads threads graph
Configuration for the "num" library:
        target architecture ...... amd64 (asm level 1)
Configuration for the "graph" library:
        options for compiling .... -I/opt/X11/include
        options for linking ...... -L/opt/X11/lib -lX11

** OCaml configuration completed successfully **
```

(state of directory now)

3. Building the bytecode compiler
---------------------------------

We type "make world", which is defined in the top-level makefile as:

```
# Compile everything the first time
world:
	$(MAKE) coldstart
	$(MAKE) all

```

So "make coldstart" is called:

```
/Applications/Xcode.app/Contents/Developer/usr/bin/make coldstart
```

This is defined in the top-level Makefile as

```
# Start up the system from the distribution compiler
coldstart:
	cd byterun; $(MAKE) all
	cp byterun/ocamlrun$(EXE) boot/ocamlrun$(EXE)
	cd yacc; $(MAKE) all
	cp yacc/ocamlyacc$(EXE) boot/ocamlyacc$(EXE)
	cd stdlib; $(MAKE) COMPILER=../boot/ocamlc all
	cd stdlib; cp $(LIBFILES) ../boot
	if test -f boot/libcamlrun.a; then :; else \
	  ln -s ../byterun/libcamlrun.a boot/libcamlrun.a; fi
	if test -d stdlib/caml; then :; else \
	  ln -s ../byterun/caml stdlib/caml; fi

```

So first, coldstart runs "make all" in the byterun/ subdirectory:

```
cd byterun; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
```

The "all" target in byterun/Makefile is different depending upon whether
configure enabled shared libraries or not. In our case it did.


From the caml/jumptbl.h target in byterun/Makefile.common, the following
invocation of `sed` is run:

```
sed -n -e '/^  /s/ \([A-Z]\)/ \&\&lbl_\1/gp' \
	       -e '/^}/q' caml/instruct.h > caml/jumptbl.h
```

It processes the following enum from byterun/caml/instruct.h:

```
enum instructions {
  ACC0, ACC1, ACC2, ACC3, ACC4, ACC5, ACC6, ACC7,
  ACC, PUSH,
  PUSHACC0, PUSHACC1, PUSHACC2, PUSHACC3,
  PUSHACC4, PUSHACC5, PUSHACC6, PUSHACC7,
  PUSHACC, POP, ASSIGN,
  ENVACC1, ENVACC2, ENVACC3, ENVACC4, ENVACC,
  PUSHENVACC1, PUSHENVACC2, PUSHENVACC3, PUSHENVACC4, PUSHENVACC,
  PUSH_RETADDR, APPLY, APPLY1, APPLY2, APPLY3,
  APPTERM, APPTERM1, APPTERM2, APPTERM3,
  RETURN, RESTART, GRAB,
  CLOSURE, CLOSUREREC,
  OFFSETCLOSUREM2, OFFSETCLOSURE0, OFFSETCLOSURE2, OFFSETCLOSURE,
  PUSHOFFSETCLOSUREM2, PUSHOFFSETCLOSURE0,
  PUSHOFFSETCLOSURE2, PUSHOFFSETCLOSURE,
  GETGLOBAL, PUSHGETGLOBAL, GETGLOBALFIELD, PUSHGETGLOBALFIELD, SETGLOBAL,
  ATOM0, ATOM, PUSHATOM0, PUSHATOM,
  MAKEBLOCK, MAKEBLOCK1, MAKEBLOCK2, MAKEBLOCK3, MAKEFLOATBLOCK,
  GETFIELD0, GETFIELD1, GETFIELD2, GETFIELD3, GETFIELD, GETFLOATFIELD,
  SETFIELD0, SETFIELD1, SETFIELD2, SETFIELD3, SETFIELD, SETFLOATFIELD,
  VECTLENGTH, GETVECTITEM, SETVECTITEM,
  GETSTRINGCHAR, SETSTRINGCHAR,
  BRANCH, BRANCHIF, BRANCHIFNOT, SWITCH, BOOLNOT,
  PUSHTRAP, POPTRAP, RAISE,
  CHECK_SIGNALS,
  C_CALL1, C_CALL2, C_CALL3, C_CALL4, C_CALL5, C_CALLN,
  CONST0, CONST1, CONST2, CONST3, CONSTINT,
  PUSHCONST0, PUSHCONST1, PUSHCONST2, PUSHCONST3, PUSHCONSTINT,
  NEGINT, ADDINT, SUBINT, MULINT, DIVINT, MODINT,
  ANDINT, ORINT, XORINT, LSLINT, LSRINT, ASRINT,
  EQ, NEQ, LTINT, LEINT, GTINT, GEINT,
  OFFSETINT, OFFSETREF, ISINT,
  GETMETHOD,
  BEQ, BNEQ,  BLTINT, BLEINT, BGTINT, BGEINT,
  ULTINT, UGEINT,
  BULTINT, BUGEINT,
  GETPUBMET, GETDYNMET,
  STOP,
  EVENT, BREAK,
  RERAISE, RAISE_NOTRACE,
FIRST_UNIMPLEMENTED_OP};
```

And from it builds the following in byterun/caml/jumptbl.h:

```
  &&lbl_ACC0, &&lbl_ACC1, &&lbl_ACC2, &&lbl_ACC3, &&lbl_ACC4, &&lbl_ACC5, &&lbl_ACC6, &&lbl_ACC7,
  &&lbl_ACC, &&lbl_PUSH,
  &&lbl_PUSHACC0, &&lbl_PUSHACC1, &&lbl_PUSHACC2, &&lbl_PUSHACC3,
  &&lbl_PUSHACC4, &&lbl_PUSHACC5, &&lbl_PUSHACC6, &&lbl_PUSHACC7,
  &&lbl_PUSHACC, &&lbl_POP, &&lbl_ASSIGN,
  &&lbl_ENVACC1, &&lbl_ENVACC2, &&lbl_ENVACC3, &&lbl_ENVACC4, &&lbl_ENVACC,
  &&lbl_PUSHENVACC1, &&lbl_PUSHENVACC2, &&lbl_PUSHENVACC3, &&lbl_PUSHENVACC4, &&lbl_PUSHENVACC,
  &&lbl_PUSH_RETADDR, &&lbl_APPLY, &&lbl_APPLY1, &&lbl_APPLY2, &&lbl_APPLY3,
  &&lbl_APPTERM, &&lbl_APPTERM1, &&lbl_APPTERM2, &&lbl_APPTERM3,
  &&lbl_RETURN, &&lbl_RESTART, &&lbl_GRAB,
  &&lbl_CLOSURE, &&lbl_CLOSUREREC,
  &&lbl_OFFSETCLOSUREM2, &&lbl_OFFSETCLOSURE0, &&lbl_OFFSETCLOSURE2, &&lbl_OFFSETCLOSURE,
  &&lbl_PUSHOFFSETCLOSUREM2, &&lbl_PUSHOFFSETCLOSURE0,
  &&lbl_PUSHOFFSETCLOSURE2, &&lbl_PUSHOFFSETCLOSURE,
  &&lbl_GETGLOBAL, &&lbl_PUSHGETGLOBAL, &&lbl_GETGLOBALFIELD, &&lbl_PUSHGETGLOBALFIELD, &&lbl_SETGLOBAL,
  &&lbl_ATOM0, &&lbl_ATOM, &&lbl_PUSHATOM0, &&lbl_PUSHATOM,
  &&lbl_MAKEBLOCK, &&lbl_MAKEBLOCK1, &&lbl_MAKEBLOCK2, &&lbl_MAKEBLOCK3, &&lbl_MAKEFLOATBLOCK,
  &&lbl_GETFIELD0, &&lbl_GETFIELD1, &&lbl_GETFIELD2, &&lbl_GETFIELD3, &&lbl_GETFIELD, &&lbl_GETFLOATFIELD,
  &&lbl_SETFIELD0, &&lbl_SETFIELD1, &&lbl_SETFIELD2, &&lbl_SETFIELD3, &&lbl_SETFIELD, &&lbl_SETFLOATFIELD,
  &&lbl_VECTLENGTH, &&lbl_GETVECTITEM, &&lbl_SETVECTITEM,
  &&lbl_GETSTRINGCHAR, &&lbl_SETSTRINGCHAR,
  &&lbl_BRANCH, &&lbl_BRANCHIF, &&lbl_BRANCHIFNOT, &&lbl_SWITCH, &&lbl_BOOLNOT,
  &&lbl_PUSHTRAP, &&lbl_POPTRAP, &&lbl_RAISE,
  &&lbl_CHECK_SIGNALS,
  &&lbl_C_CALL1, &&lbl_C_CALL2, &&lbl_C_CALL3, &&lbl_C_CALL4, &&lbl_C_CALL5, &&lbl_C_CALLN,
  &&lbl_CONST0, &&lbl_CONST1, &&lbl_CONST2, &&lbl_CONST3, &&lbl_CONSTINT,
  &&lbl_PUSHCONST0, &&lbl_PUSHCONST1, &&lbl_PUSHCONST2, &&lbl_PUSHCONST3, &&lbl_PUSHCONSTINT,
  &&lbl_NEGINT, &&lbl_ADDINT, &&lbl_SUBINT, &&lbl_MULINT, &&lbl_DIVINT, &&lbl_MODINT,
  &&lbl_ANDINT, &&lbl_ORINT, &&lbl_XORINT, &&lbl_LSLINT, &&lbl_LSRINT, &&lbl_ASRINT,
  &&lbl_EQ, &&lbl_NEQ, &&lbl_LTINT, &&lbl_LEINT, &&lbl_GTINT, &&lbl_GEINT,
  &&lbl_OFFSETINT, &&lbl_OFFSETREF, &&lbl_ISINT,
  &&lbl_GETMETHOD,
  &&lbl_BEQ, &&lbl_BNEQ,  &&lbl_BLTINT, &&lbl_BLEINT, &&lbl_BGTINT, &&lbl_BGEINT,
  &&lbl_ULTINT, &&lbl_UGEINT,
  &&lbl_BULTINT, &&lbl_BUGEINT,
  &&lbl_GETPUBMET, &&lbl_GETDYNMET,
  &&lbl_STOP,
  &&lbl_EVENT, &&lbl_BREAK,
  &&lbl_RERAISE, &&lbl_RAISE_NOTRACE,
```

Now, to compiling. The first five files from the bytecode runtime system are compiled:

```
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o interp.o interp.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o misc.o misc.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o stacks.o stacks.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o fix_code.o fix_code.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o startup_aux.o startup_aux.c
```

The next one, `startup.c` requires the file version.h, which is autogenerated
from the `VERSION` file. So we must do that now:

```
../tools/make-version-header.sh ../VERSION > caml/version.h
```

The file VERSION is as follows:

```
4.03.0+dev10-2015-07-29

# The version string is the first line of this file.
# It must be in the format described in stdlib/sys.mli
```

The resultant `caml/version.h` file is:

```
#define OCAML_VERSION_MAJOR 4
#define OCAML_VERSION_MINOR 3
#define OCAML_VERSION_PATCHLEVEL 0
#define OCAML_VERSION_ADDITIONAL "dev10-2015-07-29"
#define OCAML_VERSION 40300
#define OCAML_VERSION_STRING "4.03.0+dev10-2015-07-29"
```

We compile `startup.c` and the rest of the bytecode runtime system:

```
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o startup.o startup.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o freelist.o freelist.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o major_gc.o major_gc.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o minor_gc.o minor_gc.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o memory.o memory.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o alloc.o alloc.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o roots.o roots.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o globroots.o globroots.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o fail.o fail.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o signals.o signals.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o signals_byt.o signals_byt.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o printexc.o printexc.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o backtrace_prim.o backtrace_prim.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o backtrace.o backtrace.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o compare.o compare.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o ints.o ints.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o floats.o floats.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o str.o str.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o array.o array.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o io.o io.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o extern.o extern.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o intern.o intern.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o hash.o hash.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o sys.o sys.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o meta.o meta.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o parsing.o parsing.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o gc_ctrl.o gc_ctrl.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o terminfo.o terminfo.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o md5.o md5.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o obj.o obj.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o lexing.o lexing.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o callback.o callback.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o debugger.o debugger.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o weak.o weak.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o compact.o compact.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o finalise.o finalise.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o custom.o custom.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o dynlink.o dynlink.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o unix.o unix.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o main.o main.c
```

The unix archiver is then used to combine these object files into the OCaml
bytecode runtime library `libcamlrun.a`:

```
ar rc libcamlrun.a interp.o misc.o stacks.o fix_code.o startup_aux.o startup.o freelist.o major_gc.o minor_gc.o memory.o alloc.o roots.o globroots.o fail.o signals.o signals_byt.o printexc.o backtrace_prim.o backtrace.o compare.o ints.o floats.o str.o array.o io.o extern.o intern.o hash.o sys.o meta.o parsing.o gc_ctrl.o terminfo.o md5.o obj.o lexing.o callback.o debugger.o weak.o compact.o finalise.o custom.o dynlink.o unix.o main.o

ranlib libcamlrun.a
```

The `ranlib` command builds an internal index for the archive.

```
sed -n -e "s/CAMLprim value \([a-z0-9_][a-z0-9_]*\).*/\1/p" alloc.c array.c compare.c extern.c floats.c gc_ctrl.c hash.c intern.c interp.c ints.c io.c lexing.c md5.c meta.c obj.c parsing.c signals.c str.c sys.c terminfo.c callback.c weak.c finalise.c stacks.c dynlink.c backtrace_prim.c backtrace.c \
	  | sort | uniq > primitives
(echo '#include "caml/mlvalues.h"'; \
	 echo '#include "caml/prims.h"'; \
	 sed -e 's/.*/extern value &();/' primitives; \
	 echo 'c_primitive caml_builtin_cprim[] = {'; \
	 sed -e 's/.*/	&,/' primitives; \
	 echo '	 0 };'; \
	 echo 'char * caml_names_of_builtin_cprim[] = {'; \
	 sed -e 's/.*/	"&",/' primitives; \
	 echo '	 0 };') > prims.c
```

```
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o prims.o prims.c
```

```
gcc -Wl,-no_compact_unwind  -o ocamlrun \
		  prims.o libcamlrun.a -lcurses -lpthread
```

```
echo "/usr/local/lib/ocaml/stublibs" > ld.conf
echo "/usr/local/lib/ocaml" >> ld.conf
```


4. Building the native code compiler
------------------------------------

5. More native code
-------------------

6. Installing
---------------


