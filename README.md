What OCaml Compiles when it Compiles
====================================

This document is written from a state of ignorance. Please forward all
corrections to john / coherentgraphics co uk, or make an issue or pull request
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

The `ranlib` command builds an internal index for the archive. More `sed` now,
to collect together a list of OCaml primitives defined in the runtime system
source files using the CAMLprim macro. They are sorted and made unique.


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

The result is the file `prims.c`, which looks like this:

```
#include "caml/mlvalues.h"
#include "caml/prims.h"
extern value caml_abs_float();
extern value caml_acos_float();
extern value caml_add_debug_info();
extern value caml_add_float();
extern value caml_alloc_dummy();
extern value caml_alloc_dummy_float();
extern value caml_alloc_dummy_function();
extern value caml_array_append();
extern value caml_array_blit();
extern value caml_array_concat();
extern value caml_array_get();
extern value caml_array_get_addr();
extern value caml_array_get_float();
extern value caml_array_set();
extern value caml_array_set_addr();
extern value caml_array_set_float();
extern value caml_array_sub();
extern value caml_array_unsafe_get();
extern value caml_array_unsafe_get_float();
extern value caml_array_unsafe_set();
extern value caml_array_unsafe_set_addr();
extern value caml_array_unsafe_set_float();
extern value caml_asin_float();
extern value caml_atan2_float();
extern value caml_atan_float();
extern value caml_backtrace_status();
extern value caml_bitvect_test();
extern value caml_blit_string();
extern value caml_bswap16();
extern value caml_ceil_float();
extern value caml_channel_descriptor();
extern value caml_classify_float();
extern value caml_compare();
extern value caml_convert_raw_backtrace_slot();
extern value caml_copysign_float();
extern value caml_cos_float();
extern value caml_cosh_float();
extern value caml_create_string();
extern value caml_div_float();
extern value caml_dynlink_add_primitive();
extern value caml_dynlink_close_lib();
extern value caml_dynlink_get_current_libs();
extern value caml_dynlink_lookup_symbol();
extern value caml_dynlink_open_lib();
extern value caml_ensure_stack_capacity();
extern value caml_eq_float();
extern value caml_equal();
extern value caml_exp_float();
extern value caml_expm1_float();
extern value caml_fill_string();
extern value caml_final_register();
extern value caml_final_release();
extern value caml_float_compare();
extern value caml_float_of_int();
extern value caml_float_of_string();
extern value caml_floor_float();
extern value caml_fmod_float();
extern value caml_format_float();
extern value caml_format_int();
extern value caml_frexp_float();
extern value caml_gc_compaction();
extern value caml_gc_counters();
extern value caml_gc_full_major();
extern value caml_gc_get();
extern value caml_gc_major();
extern value caml_gc_major_slice();
extern value caml_gc_minor();
extern value caml_gc_quick_stat();
extern value caml_gc_set();
extern value caml_gc_stat();
extern value caml_ge_float();
extern value caml_get_current_callstack();
extern value caml_get_current_environment();
extern value caml_get_exception_backtrace();
extern value caml_get_exception_raw_backtrace();
extern value caml_get_global_data();
extern value caml_get_public_method();
extern value caml_get_section_table();
extern value caml_greaterequal();
extern value caml_greaterthan();
extern value caml_gt_float();
extern value caml_hash();
extern value caml_hash_univ_param();
extern value caml_hypot_float();
extern value caml_input_value();
extern value caml_input_value_from_string();
extern value caml_install_signal_handler();
extern value caml_int32_add();
extern value caml_int32_and();
extern value caml_int32_bits_of_float();
extern value caml_int32_bswap();
extern value caml_int32_compare();
extern value caml_int32_div();
extern value caml_int32_float_of_bits();
extern value caml_int32_format();
extern value caml_int32_mod();
extern value caml_int32_mul();
extern value caml_int32_neg();
extern value caml_int32_of_float();
extern value caml_int32_of_int();
extern value caml_int32_of_string();
extern value caml_int32_or();
extern value caml_int32_shift_left();
extern value caml_int32_shift_right();
extern value caml_int32_shift_right_unsigned();
extern value caml_int32_sub();
extern value caml_int32_to_float();
extern value caml_int32_to_int();
extern value caml_int32_xor();
extern value caml_int64_add();
extern value caml_int64_and();
extern value caml_int64_bits_of_float();
extern value caml_int64_bswap();
extern value caml_int64_compare();
extern value caml_int64_div();
extern value caml_int64_float_of_bits();
extern value caml_int64_format();
extern value caml_int64_mod();
extern value caml_int64_mul();
extern value caml_int64_neg();
extern value caml_int64_of_float();
extern value caml_int64_of_int();
extern value caml_int64_of_int32();
extern value caml_int64_of_nativeint();
extern value caml_int64_of_string();
extern value caml_int64_or();
extern value caml_int64_shift_left();
extern value caml_int64_shift_right();
extern value caml_int64_shift_right_unsigned();
extern value caml_int64_sub();
extern value caml_int64_to_float();
extern value caml_int64_to_int();
extern value caml_int64_to_int32();
extern value caml_int64_to_nativeint();
extern value caml_int64_xor();
extern value caml_int_as_pointer();
extern value caml_int_compare();
extern value caml_int_of_float();
extern value caml_int_of_string();
extern value caml_invoke_traced_function();
extern value caml_lazy_follow_forward();
extern value caml_lazy_make_forward();
extern value caml_ldexp_float();
extern value caml_le_float();
extern value caml_lessequal();
extern value caml_lessthan();
extern value caml_lex_engine();
extern value caml_log10_float();
extern value caml_log1p_float();
extern value caml_log_float();
extern value caml_lt_float();
extern value caml_make_array();
extern value caml_make_float_vect();
extern value caml_make_vect();
extern value caml_marshal_data_size();
extern value caml_md5_chan();
extern value caml_md5_string();
extern value caml_ml_channel_size();
extern value caml_ml_channel_size_64();
extern value caml_ml_close_channel();
extern value caml_ml_enable_runtime_warnings();
extern value caml_ml_flush();
extern value caml_ml_flush_partial();
extern value caml_ml_input();
extern value caml_ml_input_char();
extern value caml_ml_input_int();
extern value caml_ml_input_scan_line();
extern value caml_ml_open_descriptor_in();
extern value caml_ml_open_descriptor_out();
extern value caml_ml_out_channels_list();
extern value caml_ml_output();
extern value caml_ml_output_char();
extern value caml_ml_output_int();
extern value caml_ml_output_partial();
extern value caml_ml_pos_in();
extern value caml_ml_pos_in_64();
extern value caml_ml_pos_out();
extern value caml_ml_pos_out_64();
extern value caml_ml_runtime_warnings_enabled();
extern value caml_ml_seek_in();
extern value caml_ml_seek_in_64();
extern value caml_ml_seek_out();
extern value caml_ml_seek_out_64();
extern value caml_ml_set_binary_mode();
extern value caml_ml_set_channel_name();
extern value caml_ml_string_length();
extern value caml_modf_float();
extern value caml_mul_float();
extern value caml_nativeint_add();
extern value caml_nativeint_and();
extern value caml_nativeint_bswap();
extern value caml_nativeint_compare();
extern value caml_nativeint_div();
extern value caml_nativeint_format();
extern value caml_nativeint_mod();
extern value caml_nativeint_mul();
extern value caml_nativeint_neg();
extern value caml_nativeint_of_float();
extern value caml_nativeint_of_int();
extern value caml_nativeint_of_int32();
extern value caml_nativeint_of_string();
extern value caml_nativeint_or();
extern value caml_nativeint_shift_left();
extern value caml_nativeint_shift_right();
extern value caml_nativeint_shift_right_unsigned();
extern value caml_nativeint_sub();
extern value caml_nativeint_to_float();
extern value caml_nativeint_to_int();
extern value caml_nativeint_to_int32();
extern value caml_nativeint_xor();
extern value caml_neg_float();
extern value caml_neq_float();
extern value caml_new_lex_engine();
extern value caml_notequal();
extern value caml_obj_add_offset();
extern value caml_obj_block();
extern value caml_obj_dup();
extern value caml_obj_is_block();
extern value caml_obj_set_tag();
extern value caml_obj_tag();
extern value caml_obj_truncate();
extern value caml_output_value();
extern value caml_output_value_to_buffer();
extern value caml_output_value_to_string();
extern value caml_parse_engine();
extern value caml_power_float();
extern value caml_realloc_global();
extern value caml_record_backtrace();
extern value caml_register_code_fragment();
extern value caml_register_named_value();
extern value caml_reify_bytecode();
extern value caml_remove_debug_info();
extern value caml_set_oo_id();
extern value caml_set_parser_trace();
extern value caml_sin_float();
extern value caml_sinh_float();
extern value caml_sqrt_float();
extern value caml_static_alloc();
extern value caml_static_free();
extern value caml_static_release_bytecode();
extern value caml_static_resize();
extern value caml_string_compare();
extern value caml_string_equal();
extern value caml_string_get();
extern value caml_string_get16();
extern value caml_string_get32();
extern value caml_string_get64();
extern value caml_string_greaterequal();
extern value caml_string_greaterthan();
extern value caml_string_lessequal();
extern value caml_string_lessthan();
extern value caml_string_notequal();
extern value caml_string_set();
extern value caml_string_set16();
extern value caml_string_set32();
extern value caml_string_set64();
extern value caml_sub_float();
extern value caml_sys_chdir();
extern value caml_sys_close();
extern value caml_sys_const_big_endian();
extern value caml_sys_const_int_size();
extern value caml_sys_const_max_wosize();
extern value caml_sys_const_ostype_cygwin();
extern value caml_sys_const_ostype_unix();
extern value caml_sys_const_ostype_win32();
extern value caml_sys_const_word_size();
extern value caml_sys_exit();
extern value caml_sys_file_exists();
extern value caml_sys_get_argv();
extern value caml_sys_get_config();
extern value caml_sys_getcwd();
extern value caml_sys_getenv();
extern value caml_sys_is_directory();
extern value caml_sys_isatty();
extern value caml_sys_open();
extern value caml_sys_random_seed();
extern value caml_sys_read_directory();
extern value caml_sys_remove();
extern value caml_sys_rename();
extern value caml_sys_system_command();
extern value caml_sys_time();
extern value caml_tan_float();
extern value caml_tanh_float();
extern value caml_terminfo_backup();
extern value caml_terminfo_resume();
extern value caml_terminfo_setup();
extern value caml_terminfo_standout();
extern value caml_update_dummy();
extern value caml_weak_blit();
extern value caml_weak_check();
extern value caml_weak_create();
extern value caml_weak_get();
extern value caml_weak_get_copy();
extern value caml_weak_set();
c_primitive caml_builtin_cprim[] = {
	caml_abs_float,
	caml_acos_float,
	caml_add_debug_info,
	caml_add_float,
	caml_alloc_dummy,
	caml_alloc_dummy_float,
	caml_alloc_dummy_function,
	caml_array_append,
	caml_array_blit,
	caml_array_concat,
	caml_array_get,
	caml_array_get_addr,
	caml_array_get_float,
	caml_array_set,
	caml_array_set_addr,
	caml_array_set_float,
	caml_array_sub,
	caml_array_unsafe_get,
	caml_array_unsafe_get_float,
	caml_array_unsafe_set,
	caml_array_unsafe_set_addr,
	caml_array_unsafe_set_float,
	caml_asin_float,
	caml_atan2_float,
	caml_atan_float,
	caml_backtrace_status,
	caml_bitvect_test,
	caml_blit_string,
	caml_bswap16,
	caml_ceil_float,
	caml_channel_descriptor,
	caml_classify_float,
	caml_compare,
	caml_convert_raw_backtrace_slot,
	caml_copysign_float,
	caml_cos_float,
	caml_cosh_float,
	caml_create_string,
	caml_div_float,
	caml_dynlink_add_primitive,
	caml_dynlink_close_lib,
	caml_dynlink_get_current_libs,
	caml_dynlink_lookup_symbol,
	caml_dynlink_open_lib,
	caml_ensure_stack_capacity,
	caml_eq_float,
	caml_equal,
	caml_exp_float,
	caml_expm1_float,
	caml_fill_string,
	caml_final_register,
	caml_final_release,
	caml_float_compare,
	caml_float_of_int,
	caml_float_of_string,
	caml_floor_float,
	caml_fmod_float,
	caml_format_float,
	caml_format_int,
	caml_frexp_float,
	caml_gc_compaction,
	caml_gc_counters,
	caml_gc_full_major,
	caml_gc_get,
	caml_gc_major,
	caml_gc_major_slice,
	caml_gc_minor,
	caml_gc_quick_stat,
	caml_gc_set,
	caml_gc_stat,
	caml_ge_float,
	caml_get_current_callstack,
	caml_get_current_environment,
	caml_get_exception_backtrace,
	caml_get_exception_raw_backtrace,
	caml_get_global_data,
	caml_get_public_method,
	caml_get_section_table,
	caml_greaterequal,
	caml_greaterthan,
	caml_gt_float,
	caml_hash,
	caml_hash_univ_param,
	caml_hypot_float,
	caml_input_value,
	caml_input_value_from_string,
	caml_install_signal_handler,
	caml_int32_add,
	caml_int32_and,
	caml_int32_bits_of_float,
	caml_int32_bswap,
	caml_int32_compare,
	caml_int32_div,
	caml_int32_float_of_bits,
	caml_int32_format,
	caml_int32_mod,
	caml_int32_mul,
	caml_int32_neg,
	caml_int32_of_float,
	caml_int32_of_int,
	caml_int32_of_string,
	caml_int32_or,
	caml_int32_shift_left,
	caml_int32_shift_right,
	caml_int32_shift_right_unsigned,
	caml_int32_sub,
	caml_int32_to_float,
	caml_int32_to_int,
	caml_int32_xor,
	caml_int64_add,
	caml_int64_and,
	caml_int64_bits_of_float,
	caml_int64_bswap,
	caml_int64_compare,
	caml_int64_div,
	caml_int64_float_of_bits,
	caml_int64_format,
	caml_int64_mod,
	caml_int64_mul,
	caml_int64_neg,
	caml_int64_of_float,
	caml_int64_of_int,
	caml_int64_of_int32,
	caml_int64_of_nativeint,
	caml_int64_of_string,
	caml_int64_or,
	caml_int64_shift_left,
	caml_int64_shift_right,
	caml_int64_shift_right_unsigned,
	caml_int64_sub,
	caml_int64_to_float,
	caml_int64_to_int,
	caml_int64_to_int32,
	caml_int64_to_nativeint,
	caml_int64_xor,
	caml_int_as_pointer,
	caml_int_compare,
	caml_int_of_float,
	caml_int_of_string,
	caml_invoke_traced_function,
	caml_lazy_follow_forward,
	caml_lazy_make_forward,
	caml_ldexp_float,
	caml_le_float,
	caml_lessequal,
	caml_lessthan,
	caml_lex_engine,
	caml_log10_float,
	caml_log1p_float,
	caml_log_float,
	caml_lt_float,
	caml_make_array,
	caml_make_float_vect,
	caml_make_vect,
	caml_marshal_data_size,
	caml_md5_chan,
	caml_md5_string,
	caml_ml_channel_size,
	caml_ml_channel_size_64,
	caml_ml_close_channel,
	caml_ml_enable_runtime_warnings,
	caml_ml_flush,
	caml_ml_flush_partial,
	caml_ml_input,
	caml_ml_input_char,
	caml_ml_input_int,
	caml_ml_input_scan_line,
	caml_ml_open_descriptor_in,
	caml_ml_open_descriptor_out,
	caml_ml_out_channels_list,
	caml_ml_output,
	caml_ml_output_char,
	caml_ml_output_int,
	caml_ml_output_partial,
	caml_ml_pos_in,
	caml_ml_pos_in_64,
	caml_ml_pos_out,
	caml_ml_pos_out_64,
	caml_ml_runtime_warnings_enabled,
	caml_ml_seek_in,
	caml_ml_seek_in_64,
	caml_ml_seek_out,
	caml_ml_seek_out_64,
	caml_ml_set_binary_mode,
	caml_ml_set_channel_name,
	caml_ml_string_length,
	caml_modf_float,
	caml_mul_float,
	caml_nativeint_add,
	caml_nativeint_and,
	caml_nativeint_bswap,
	caml_nativeint_compare,
	caml_nativeint_div,
	caml_nativeint_format,
	caml_nativeint_mod,
	caml_nativeint_mul,
	caml_nativeint_neg,
	caml_nativeint_of_float,
	caml_nativeint_of_int,
	caml_nativeint_of_int32,
	caml_nativeint_of_string,
	caml_nativeint_or,
	caml_nativeint_shift_left,
	caml_nativeint_shift_right,
	caml_nativeint_shift_right_unsigned,
	caml_nativeint_sub,
	caml_nativeint_to_float,
	caml_nativeint_to_int,
	caml_nativeint_to_int32,
	caml_nativeint_xor,
	caml_neg_float,
	caml_neq_float,
	caml_new_lex_engine,
	caml_notequal,
	caml_obj_add_offset,
	caml_obj_block,
	caml_obj_dup,
	caml_obj_is_block,
	caml_obj_set_tag,
	caml_obj_tag,
	caml_obj_truncate,
	caml_output_value,
	caml_output_value_to_buffer,
	caml_output_value_to_string,
	caml_parse_engine,
	caml_power_float,
	caml_realloc_global,
	caml_record_backtrace,
	caml_register_code_fragment,
	caml_register_named_value,
	caml_reify_bytecode,
	caml_remove_debug_info,
	caml_set_oo_id,
	caml_set_parser_trace,
	caml_sin_float,
	caml_sinh_float,
	caml_sqrt_float,
	caml_static_alloc,
	caml_static_free,
	caml_static_release_bytecode,
	caml_static_resize,
	caml_string_compare,
	caml_string_equal,
	caml_string_get,
	caml_string_get16,
	caml_string_get32,
	caml_string_get64,
	caml_string_greaterequal,
	caml_string_greaterthan,
	caml_string_lessequal,
	caml_string_lessthan,
	caml_string_notequal,
	caml_string_set,
	caml_string_set16,
	caml_string_set32,
	caml_string_set64,
	caml_sub_float,
	caml_sys_chdir,
	caml_sys_close,
	caml_sys_const_big_endian,
	caml_sys_const_int_size,
	caml_sys_const_max_wosize,
	caml_sys_const_ostype_cygwin,
	caml_sys_const_ostype_unix,
	caml_sys_const_ostype_win32,
	caml_sys_const_word_size,
	caml_sys_exit,
	caml_sys_file_exists,
	caml_sys_get_argv,
	caml_sys_get_config,
	caml_sys_getcwd,
	caml_sys_getenv,
	caml_sys_is_directory,
	caml_sys_isatty,
	caml_sys_open,
	caml_sys_random_seed,
	caml_sys_read_directory,
	caml_sys_remove,
	caml_sys_rename,
	caml_sys_system_command,
	caml_sys_time,
	caml_tan_float,
	caml_tanh_float,
	caml_terminfo_backup,
	caml_terminfo_resume,
	caml_terminfo_setup,
	caml_terminfo_standout,
	caml_update_dummy,
	caml_weak_blit,
	caml_weak_check,
	caml_weak_create,
	caml_weak_get,
	caml_weak_get_copy,
	caml_weak_set,
	 0 };
char * caml_names_of_builtin_cprim[] = {
	"caml_abs_float",
	"caml_acos_float",
	"caml_add_debug_info",
	"caml_add_float",
	"caml_alloc_dummy",
	"caml_alloc_dummy_float",
	"caml_alloc_dummy_function",
	"caml_array_append",
	"caml_array_blit",
	"caml_array_concat",
	"caml_array_get",
	"caml_array_get_addr",
	"caml_array_get_float",
	"caml_array_set",
	"caml_array_set_addr",
	"caml_array_set_float",
	"caml_array_sub",
	"caml_array_unsafe_get",
	"caml_array_unsafe_get_float",
	"caml_array_unsafe_set",
	"caml_array_unsafe_set_addr",
	"caml_array_unsafe_set_float",
	"caml_asin_float",
	"caml_atan2_float",
	"caml_atan_float",
	"caml_backtrace_status",
	"caml_bitvect_test",
	"caml_blit_string",
	"caml_bswap16",
	"caml_ceil_float",
	"caml_channel_descriptor",
	"caml_classify_float",
	"caml_compare",
	"caml_convert_raw_backtrace_slot",
	"caml_copysign_float",
	"caml_cos_float",
	"caml_cosh_float",
	"caml_create_string",
	"caml_div_float",
	"caml_dynlink_add_primitive",
	"caml_dynlink_close_lib",
	"caml_dynlink_get_current_libs",
	"caml_dynlink_lookup_symbol",
	"caml_dynlink_open_lib",
	"caml_ensure_stack_capacity",
	"caml_eq_float",
	"caml_equal",
	"caml_exp_float",
	"caml_expm1_float",
	"caml_fill_string",
	"caml_final_register",
	"caml_final_release",
	"caml_float_compare",
	"caml_float_of_int",
	"caml_float_of_string",
	"caml_floor_float",
	"caml_fmod_float",
	"caml_format_float",
	"caml_format_int",
	"caml_frexp_float",
	"caml_gc_compaction",
	"caml_gc_counters",
	"caml_gc_full_major",
	"caml_gc_get",
	"caml_gc_major",
	"caml_gc_major_slice",
	"caml_gc_minor",
	"caml_gc_quick_stat",
	"caml_gc_set",
	"caml_gc_stat",
	"caml_ge_float",
	"caml_get_current_callstack",
	"caml_get_current_environment",
	"caml_get_exception_backtrace",
	"caml_get_exception_raw_backtrace",
	"caml_get_global_data",
	"caml_get_public_method",
	"caml_get_section_table",
	"caml_greaterequal",
	"caml_greaterthan",
	"caml_gt_float",
	"caml_hash",
	"caml_hash_univ_param",
	"caml_hypot_float",
	"caml_input_value",
	"caml_input_value_from_string",
	"caml_install_signal_handler",
	"caml_int32_add",
	"caml_int32_and",
	"caml_int32_bits_of_float",
	"caml_int32_bswap",
	"caml_int32_compare",
	"caml_int32_div",
	"caml_int32_float_of_bits",
	"caml_int32_format",
	"caml_int32_mod",
	"caml_int32_mul",
	"caml_int32_neg",
	"caml_int32_of_float",
	"caml_int32_of_int",
	"caml_int32_of_string",
	"caml_int32_or",
	"caml_int32_shift_left",
	"caml_int32_shift_right",
	"caml_int32_shift_right_unsigned",
	"caml_int32_sub",
	"caml_int32_to_float",
	"caml_int32_to_int",
	"caml_int32_xor",
	"caml_int64_add",
	"caml_int64_and",
	"caml_int64_bits_of_float",
	"caml_int64_bswap",
	"caml_int64_compare",
	"caml_int64_div",
	"caml_int64_float_of_bits",
	"caml_int64_format",
	"caml_int64_mod",
	"caml_int64_mul",
	"caml_int64_neg",
	"caml_int64_of_float",
	"caml_int64_of_int",
	"caml_int64_of_int32",
	"caml_int64_of_nativeint",
	"caml_int64_of_string",
	"caml_int64_or",
	"caml_int64_shift_left",
	"caml_int64_shift_right",
	"caml_int64_shift_right_unsigned",
	"caml_int64_sub",
	"caml_int64_to_float",
	"caml_int64_to_int",
	"caml_int64_to_int32",
	"caml_int64_to_nativeint",
	"caml_int64_xor",
	"caml_int_as_pointer",
	"caml_int_compare",
	"caml_int_of_float",
	"caml_int_of_string",
	"caml_invoke_traced_function",
	"caml_lazy_follow_forward",
	"caml_lazy_make_forward",
	"caml_ldexp_float",
	"caml_le_float",
	"caml_lessequal",
	"caml_lessthan",
	"caml_lex_engine",
	"caml_log10_float",
	"caml_log1p_float",
	"caml_log_float",
	"caml_lt_float",
	"caml_make_array",
	"caml_make_float_vect",
	"caml_make_vect",
	"caml_marshal_data_size",
	"caml_md5_chan",
	"caml_md5_string",
	"caml_ml_channel_size",
	"caml_ml_channel_size_64",
	"caml_ml_close_channel",
	"caml_ml_enable_runtime_warnings",
	"caml_ml_flush",
	"caml_ml_flush_partial",
	"caml_ml_input",
	"caml_ml_input_char",
	"caml_ml_input_int",
	"caml_ml_input_scan_line",
	"caml_ml_open_descriptor_in",
	"caml_ml_open_descriptor_out",
	"caml_ml_out_channels_list",
	"caml_ml_output",
	"caml_ml_output_char",
	"caml_ml_output_int",
	"caml_ml_output_partial",
	"caml_ml_pos_in",
	"caml_ml_pos_in_64",
	"caml_ml_pos_out",
	"caml_ml_pos_out_64",
	"caml_ml_runtime_warnings_enabled",
	"caml_ml_seek_in",
	"caml_ml_seek_in_64",
	"caml_ml_seek_out",
	"caml_ml_seek_out_64",
	"caml_ml_set_binary_mode",
	"caml_ml_set_channel_name",
	"caml_ml_string_length",
	"caml_modf_float",
	"caml_mul_float",
	"caml_nativeint_add",
	"caml_nativeint_and",
	"caml_nativeint_bswap",
	"caml_nativeint_compare",
	"caml_nativeint_div",
	"caml_nativeint_format",
	"caml_nativeint_mod",
	"caml_nativeint_mul",
	"caml_nativeint_neg",
	"caml_nativeint_of_float",
	"caml_nativeint_of_int",
	"caml_nativeint_of_int32",
	"caml_nativeint_of_string",
	"caml_nativeint_or",
	"caml_nativeint_shift_left",
	"caml_nativeint_shift_right",
	"caml_nativeint_shift_right_unsigned",
	"caml_nativeint_sub",
	"caml_nativeint_to_float",
	"caml_nativeint_to_int",
	"caml_nativeint_to_int32",
	"caml_nativeint_xor",
	"caml_neg_float",
	"caml_neq_float",
	"caml_new_lex_engine",
	"caml_notequal",
	"caml_obj_add_offset",
	"caml_obj_block",
	"caml_obj_dup",
	"caml_obj_is_block",
	"caml_obj_set_tag",
	"caml_obj_tag",
	"caml_obj_truncate",
	"caml_output_value",
	"caml_output_value_to_buffer",
	"caml_output_value_to_string",
	"caml_parse_engine",
	"caml_power_float",
	"caml_realloc_global",
	"caml_record_backtrace",
	"caml_register_code_fragment",
	"caml_register_named_value",
	"caml_reify_bytecode",
	"caml_remove_debug_info",
	"caml_set_oo_id",
	"caml_set_parser_trace",
	"caml_sin_float",
	"caml_sinh_float",
	"caml_sqrt_float",
	"caml_static_alloc",
	"caml_static_free",
	"caml_static_release_bytecode",
	"caml_static_resize",
	"caml_string_compare",
	"caml_string_equal",
	"caml_string_get",
	"caml_string_get16",
	"caml_string_get32",
	"caml_string_get64",
	"caml_string_greaterequal",
	"caml_string_greaterthan",
	"caml_string_lessequal",
	"caml_string_lessthan",
	"caml_string_notequal",
	"caml_string_set",
	"caml_string_set16",
	"caml_string_set32",
	"caml_string_set64",
	"caml_sub_float",
	"caml_sys_chdir",
	"caml_sys_close",
	"caml_sys_const_big_endian",
	"caml_sys_const_int_size",
	"caml_sys_const_max_wosize",
	"caml_sys_const_ostype_cygwin",
	"caml_sys_const_ostype_unix",
	"caml_sys_const_ostype_win32",
	"caml_sys_const_word_size",
	"caml_sys_exit",
	"caml_sys_file_exists",
	"caml_sys_get_argv",
	"caml_sys_get_config",
	"caml_sys_getcwd",
	"caml_sys_getenv",
	"caml_sys_is_directory",
	"caml_sys_isatty",
	"caml_sys_open",
	"caml_sys_random_seed",
	"caml_sys_read_directory",
	"caml_sys_remove",
	"caml_sys_rename",
	"caml_sys_system_command",
	"caml_sys_time",
	"caml_tan_float",
	"caml_tanh_float",
	"caml_terminfo_backup",
	"caml_terminfo_resume",
	"caml_terminfo_setup",
	"caml_terminfo_standout",
	"caml_update_dummy",
	"caml_weak_blit",
	"caml_weak_check",
	"caml_weak_create",
	"caml_weak_get",
	"caml_weak_get_copy",
	"caml_weak_set",
	 0 };
```

And which we shall now compile:

```
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o prims.o prims.c
```

This is linked with `libcamlrun.a`:

```
gcc -Wl,-no_compact_unwind  -o ocamlrun \
		  prims.o libcamlrun.a -lcurses -lpthread
```

The bytecode interpreter `ocamlrun` is thus produced in the current directory:

```
feast:byterun john$ ./ocamlrun -version
The OCaml runtime, version 4.03.0+dev10-2015-07-29
```

The file ld.conf is created:

```
echo "/usr/local/lib/ocaml/stublibs" > ld.conf
echo "/usr/local/lib/ocaml" >> ld.conf
```

So it reads, referring to the eventual install location:

```
/usr/local/lib/ocaml/stublibs
/usr/local/lib/ocaml
```

The compilation process is repeated (minus the sed and tools/ steps, since they
are done already) for position-independent code:


```
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   interp.c -o interp.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   misc.c -o misc.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   stacks.c -o stacks.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   fix_code.c -o fix_code.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   startup_aux.c -o startup_aux.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   startup.c -o startup.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   freelist.c -o freelist.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   major_gc.c -o major_gc.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   minor_gc.c -o minor_gc.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   memory.c -o memory.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   alloc.c -o alloc.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   roots.c -o roots.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   globroots.c -o globroots.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   fail.c -o fail.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   signals.c -o signals.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   signals_byt.c -o signals_byt.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   printexc.c -o printexc.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   backtrace_prim.c -o backtrace_prim.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   backtrace.c -o backtrace.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   compare.c -o compare.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   ints.c -o ints.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   floats.c -o floats.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   str.c -o str.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   array.c -o array.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   io.c -o io.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   extern.c -o extern.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   intern.c -o intern.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   hash.c -o hash.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   sys.c -o sys.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   meta.c -o meta.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   parsing.c -o parsing.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   gc_ctrl.c -o gc_ctrl.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   terminfo.c -o terminfo.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   md5.c -o md5.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   obj.c -o obj.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   lexing.c -o lexing.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   callback.c -o callback.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   debugger.c -o debugger.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   weak.c -o weak.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   compact.c -o compact.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   finalise.c -o finalise.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   custom.c -o custom.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   dynlink.c -o dynlink.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   unix.c -o unix.pic.o
gcc -c -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   main.c -o main.pic.o
ar rc libcamlrun_pic.a interp.pic.o misc.pic.o stacks.pic.o fix_code.pic.o startup_aux.pic.o startup.pic.o freelist.pic.o major_gc.pic.o minor_gc.pic.o memory.pic.o alloc.pic.o roots.pic.o globroots.pic.o fail.pic.o signals.pic.o signals_byt.pic.o printexc.pic.o backtrace_prim.pic.o backtrace.pic.o compare.pic.o ints.pic.o floats.pic.o str.pic.o array.pic.o io.pic.o extern.pic.o intern.pic.o hash.pic.o sys.pic.o meta.pic.o parsing.pic.o gc_ctrl.pic.o terminfo.pic.o md5.pic.o obj.pic.o lexing.pic.o callback.pic.o debugger.pic.o weak.pic.o compact.pic.o finalise.pic.o custom.pic.o dynlink.pic.o unix.pic.o main.pic.o
ranlib libcamlrun_pic.a
gcc -bundle -flat_namespace -undefined suppress                    -Wl,-no_compact_unwind -o libcamlrun_shared.so interp.pic.o misc.pic.o stacks.pic.o fix_code.pic.o startup_aux.pic.o startup.pic.o freelist.pic.o major_gc.pic.o minor_gc.pic.o memory.pic.o alloc.pic.o roots.pic.o globroots.pic.o fail.pic.o signals.pic.o signals_byt.pic.o printexc.pic.o backtrace_prim.pic.o backtrace.pic.o compare.pic.o ints.pic.o floats.pic.o str.pic.o array.pic.o io.pic.o extern.pic.o intern.pic.o hash.pic.o sys.pic.o meta.pic.o parsing.pic.o gc_ctrl.pic.o terminfo.pic.o md5.pic.o obj.pic.o lexing.pic.o callback.pic.o debugger.pic.o weak.pic.o compact.pic.o finalise.pic.o custom.pic.o dynlink.pic.o unix.pic.o main.pic.o -lcurses -lpthread
```

We copy `ocamlrun` into the boot directory, overwriting the one already there:

```
cp byterun/ocamlrun boot/ocamlrun
```


Now, we build `ocamlyacc`, the parser generator, which we shall need since
OCaml's parser is defined using it:

```
cd yacc; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o closure.o closure.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o error.o error.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o lalr.o lalr.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o lr0.o lr0.c
echo "#define OCAML_VERSION \"`sed -e 1q ../VERSION`\"" >version.h
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o main.o main.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o mkpar.o mkpar.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o output.o output.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o reader.o reader.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o skeleton.o skeleton.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o symtab.o symtab.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o verbose.o verbose.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT   -c -o warshall.o warshall.c
gcc -DNDEBUG -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT  -o ocamlyacc closure.o error.o lalr.o lr0.o main.o mkpar.o output.o reader.o skeleton.o symtab.o verbose.o warshall.o
```

This, too, is copied into `boot/`, overwriting the one there:

```
cp yacc/ocamlyacc boot/ocamlyacc
```

Time to build the standard library in the directory `stdlib`. Since this is mostly written in OCaml, we
shall need an OCaml compiler. But we haven't built one yet, so we use the copy
of ocamlc found in the `boot/` directory.

```
cd stdlib; /Applications/Xcode.app/Contents/Developer/usr/bin/make COMPILER=../boot/ocamlc all
```

This copy of `ocamlc` is a bytecode-compiled version of the bytecode compiler.
It can be invoked using the `ocamlrun` which we copied into the `boot` directory
earlier:

FIXME: What does ./Compflags do?
FIXME: What is -nostdlib?

```
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalFormatBasics.cmi` -c camlinternalFormatBasics.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalFormatBasics.cmo` -c camlinternalFormatBasics.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags pervasives.cmi` -c pervasives.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags pervasives.cmo` -c pervasives.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags array.cmi` -c array.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags array.cmo` -c array.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags list.cmi` -c list.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags list.cmo` -c list.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags char.cmi` -c char.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags char.cmo` -c char.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags bytes.cmi` -c bytes.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags bytes.cmo` -c bytes.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags string.cmi` -c string.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags string.cmo` -c string.ml
sed -e "s|%%VERSION%%|`sed -e 1q ../VERSION`|" sys.mlp >sys.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags sys.cmi` -c sys.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags sys.cmo` -c sys.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags sort.cmi` -c sort.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags sort.cmo` -c sort.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags marshal.cmi` -c marshal.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags marshal.cmo` -c marshal.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags int32.cmi` -c int32.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags obj.cmi` -c obj.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags obj.cmo` -c obj.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags int32.cmo` -c int32.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags int64.cmi` -c int64.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags int64.cmo` -c int64.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags nativeint.cmi` -c nativeint.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags nativeint.cmo` -c nativeint.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags lexing.cmi` -c lexing.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags lexing.cmo` -c lexing.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags parsing.cmi` -c parsing.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags parsing.cmo` -c parsing.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags set.cmi` -c set.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags set.cmo` -c set.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags map.cmi` -c map.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags map.cmo` -c map.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stack.cmi` -c stack.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stack.cmo` -c stack.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags queue.cmi` -c queue.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags queue.cmo` -c queue.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalLazy.cmi` -c camlinternalLazy.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalLazy.cmo` -c camlinternalLazy.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags lazy.cmi` -c lazy.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags lazy.cmo` -c lazy.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stream.cmi` -c stream.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stream.cmo` -c stream.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags buffer.cmi` -c buffer.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags buffer.cmo` -c buffer.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalFormat.cmi` -c camlinternalFormat.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalFormat.cmo` -c camlinternalFormat.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags printf.cmi` -c printf.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags printf.cmo` -c printf.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags arg.cmi` -c arg.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags arg.cmo` -c arg.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags printexc.cmi` -c printexc.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags printexc.cmo` -c printexc.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags gc.cmi` -c gc.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags gc.cmo` -c gc.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags digest.cmi` -c digest.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags digest.cmo` -c digest.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags random.cmi` -c random.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags random.cmo` -c random.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags hashtbl.cmi` -c hashtbl.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags hashtbl.cmo` -c hashtbl.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags weak.cmi` -c weak.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags weak.cmo` -c weak.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags format.cmi` -c format.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags format.cmo` -c format.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags scanf.cmi` -c scanf.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags scanf.cmo` -c scanf.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags callback.cmi` -c callback.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags callback.cmo` -c callback.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalOO.cmi` -c camlinternalOO.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalOO.cmo` -c camlinternalOO.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags oo.cmi` -c oo.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags oo.cmo` -c oo.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalMod.cmi` -c camlinternalMod.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalMod.cmo` -c camlinternalMod.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags genlex.cmi` -c genlex.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags genlex.cmo` -c genlex.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags filename.cmi` -c filename.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags filename.cmo` -c filename.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags complex.cmi` -c complex.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags complex.cmo` -c complex.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags arrayLabels.cmi` -c arrayLabels.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags arrayLabels.cmo` -c arrayLabels.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags listLabels.cmi` -c listLabels.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags listLabels.cmo` -c listLabels.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags bytesLabels.cmi` -c bytesLabels.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags bytesLabels.cmo` -c bytesLabels.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stringLabels.cmi` -c stringLabels.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stringLabels.cmo` -c stringLabels.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags moreLabels.cmi` -c moreLabels.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags moreLabels.cmo` -c moreLabels.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stdLabels.cmi` -c stdLabels.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stdLabels.cmo` -c stdLabels.ml
```

```
../boot/ocamlrun ../boot/ocamlc -a -o stdlib.cma camlinternalFormatBasics.cmo pervasives.cmo array.cmo list.cmo char.cmo bytes.cmo string.cmo sys.cmo sort.cmo marshal.cmo obj.cmo int32.cmo int64.cmo nativeint.cmo lexing.cmo parsing.cmo set.cmo map.cmo stack.cmo queue.cmo camlinternalLazy.cmo lazy.cmo stream.cmo buffer.cmo camlinternalFormat.cmo printf.cmo arg.cmo printexc.cmo gc.cmo digest.cmo random.cmo hashtbl.cmo weak.cmo format.cmo scanf.cmo callback.cmo camlinternalOO.cmo oo.cmo camlinternalMod.cmo genlex.cmo filename.cmo complex.cmo arrayLabels.cmo listLabels.cmo bytesLabels.cmo stringLabels.cmo moreLabels.cmo stdLabels.cmo
```

The file std_exit.ml is now compiled, which is special because it is ensures
that at_exit functions are called at the end of every program. It is kept
separate from stdlib.cma.

```
../boot/ocamlrun ../boot/ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags std_exit.cmo` -c std_exit.ml
```

The following code now constructs the file `camlheader` which contains code such
as:

```
#!/usr/local/bin/ocamlrun
```

This allows bytecode executables to find `ocamlrun' themselves. If this isn't
working on the current platform, the code in header.c is used instead.

```
if true; then \
	  echo '#!/usr/local/bin/ocamlrun' > camlheader && \
	  echo '#!/usr/local/bin/ocamlrun' > target_camlheader && \
	  echo '#!/usr/local/bin/ocamlrund' > camlheaderd && \
	  echo '#!/usr/local/bin/ocamlrund' > target_camlheaderd && \
	  echo '#!' | tr -d '\012' > camlheader_ur; \
	else \
	  for suff in '' d; do \
	    gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT  \
	              -DRUNTIME_NAME='"/usr/local/bin/ocamlrun'$suff'"' \
	              header.c -o tmpheader && \
	    strip tmpheader && \
	    mv tmpheader camlheader$suff && \
	    gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT  \
	              -DRUNTIME_NAME='"/usr/local/bin/ocamlrun'$suff'"' \
	              header.c -o tmpheader && \
	    strip tmpheader && \
	    mv tmpheader target_camlheader$suff; \
	  done && \
	  cp camlheader camlheader_ur; \
	fi
```

The standard library compiled interface .cmi, libraries of compiled bytecode .cma,
and compiled bytecode std_exit.cmo are copied into the `boot` directory,
together with `camlheader'.

```
cd stdlib; cp stdlib.cma std_exit.cmo *.cmi camlheader ../boot
```

```
if test -f boot/libcamlrun.a; then :; else \
	  ln -s ../byterun/libcamlrun.a boot/libcamlrun.a; fi
if test -d stdlib/caml; then :; else \
	  ln -s ../byterun/caml stdlib/caml; fi
```

You can see that we are finished with `make coldstart` now. We have done
`byterun`, `yacc`, `stdlib` and made the symbolic links.

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

Now on to `make all`.

```
/Applications/Xcode.app/Contents/Developer/usr/bin/make all
/Applications/Xcode.app/Contents/Developer/usr/bin/make runtime
cd byterun; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
make[3]: Nothing to be done for `all'.
```

```
if test -f stdlib/libcamlrun.a; then :; else \
	  ln -s ../byterun/libcamlrun.a stdlib/libcamlrun.a; fi
```

```
/Applications/Xcode.app/Contents/Developer/usr/bin/make coreall
```

```
/Applications/Xcode.app/Contents/Developer/usr/bin/make ocamlc
```

```
sed -e 's|%%LIBDIR%%|/usr/local/lib/ocaml|' \
	    -e 's|%%BYTERUN%%|/usr/local/bin/ocamlrun|' \
	    -e 's|%%CCOMPTYPE%%|cc|' \
	    -e 's|%%BYTECC%%|gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT |' \
	    -e 's|%%NATIVECC%%|gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT|' \
	    -e '/c_compiler =/s| -Werror||' \
	    -e 's|%%PACKLD%%|ld -r -arch x86_64  -o\ |' \
	    -e 's|%%BYTECCLIBS%%|-lcurses -lpthread|' \
	    -e 's|%%NATIVECCLIBS%%||' \
	    -e 's|%%RANLIBCMD%%|ranlib|' \
	    -e 's|%%ARCMD%%|ar|' \
	    -e 's|%%CC_PROFILE%%|-pg|' \
	    -e 's|%%ARCH%%|amd64|' \
	    -e 's|%%MODEL%%|default|' \
	    -e 's|%%SYSTEM%%|macosx|' \
	    -e 's|%%EXT_OBJ%%|.o|' \
	    -e 's|%%EXT_ASM%%|.s|' \
	    -e 's|%%EXT_LIB%%|.a|' \
	    -e 's|%%EXT_DLL%%|.so|' \
	    -e 's|%%SYSTHREAD_SUPPORT%%|true|' \
	    -e 's|%%ASM%%|clang -arch x86_64 -c|' \
	    -e 's|%%ASM_CFI_SUPPORTED%%|true|' \
	    -e 's|%%WITH_FRAME_POINTERS%%|false|' \
	    -e 's|%%MKDLL%%|gcc -bundle -flat_namespace -undefined suppress                    -Wl,-no_compact_unwind|' \
	    -e 's|%%MKEXE%%|gcc -Wl,-no_compact_unwind|' \
	    -e 's|%%MKMAINDLL%%|gcc -bundle -flat_namespace -undefined suppress                    -Wl,-no_compact_unwind|' \
	    -e 's|%%HOST%%|x86_64-apple-darwin15.0.0|' \
	    -e 's|%%TARGET%%|x86_64-apple-darwin15.0.0|' \
	    utils/config.mlp > utils/config.ml
```

```
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/config.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/config.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/clflags.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/clflags.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/misc.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/misc.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/tbl.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/tbl.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/terminfo.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/terminfo.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/ccomp.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/ccomp.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/warnings.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/warnings.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/consistbl.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c utils/consistbl.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/location.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/location.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/longident.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/longident.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/asttypes.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/parsetree.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/docstrings.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/docstrings.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/ast_helper.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/ast_helper.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/syntaxerr.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/syntaxerr.ml
```

boot/ocamlyacc -v parsing/parser.mly

boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/parser.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/parser.ml

202 states, 5088 transitions, table size 21564 bytes
2426 additional bytes used for bindings

boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/lexer.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/lexer.ml

boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/parse.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/parse.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/printast.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/printast.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/pprintast.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/pprintast.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/ast_mapper.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/ast_mapper.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/attr_helper.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c parsing/attr_helper.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/ident.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/ident.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/path.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/path.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/outcometree.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/primitive.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/primitive.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/types.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/types.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/btype.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/btype.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/oprint.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/oprint.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/subst.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/subst.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/predef.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/predef.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/datarepr.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/datarepr.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/cmi_format.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/cmi_format.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/env.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/env.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typedtree.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typedtree.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/printtyped.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/printtyped.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/ctype.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/ctype.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/printtyp.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/printtyp.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/includeclass.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/includeclass.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/mtype.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/mtype.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/envaux.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/envaux.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/includecore.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/includecore.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typedtreeIter.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typedtreeIter.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typedtreeMap.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typedtreeMap.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/tast_mapper.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/tast_mapper.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/cmt_format.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/cmt_format.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/includemod.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/includemod.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typetexp.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typetexp.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/parmatch.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/parmatch.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/annot.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/stypes.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/stypes.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typecore.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typecore.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typedecl.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typedecl.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typeclass.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typeclass.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typemod.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/typemod.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/untypeast.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c typing/untypeast.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/lambda.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/lambda.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/printlambda.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/printlambda.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/typeopt.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/typeopt.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/switch.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/switch.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/matching.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/matching.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/translobj.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/translobj.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/translcore.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/translcore.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/translclass.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/translclass.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/translmod.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/translmod.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/simplif.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/simplif.ml

(echo 'let builtin_exceptions = [|'; \
	 sed -n -e 's|.*/\* \("[A-Za-z_]*"\) \*/$|  \1;|p' byterun/caml/fail.h | \
	 sed -e '$s/;$//'; \
	 echo '|]'; \
	 echo 'let builtin_primitives = [|'; \
	 sed -e 's/.*/  "&";/' -e '$s/;$//' byterun/primitives; \
	 echo '|]') > bytecomp/runtimedef.ml

boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/runtimedef.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/runtimedef.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/pparse.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/pparse.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/main_args.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/main_args.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/compenv.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/compenv.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/compmisc.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/compmisc.ml

boot/ocamlrun boot/ocamlc -nostdlib -I boot -a -linkall -o compilerlibs/ocamlcommon.cma utils/config.cmo utils/clflags.cmo utils/misc.cmo utils/tbl.cmo utils/terminfo.cmo utils/ccomp.cmo utils/warnings.cmo utils/consistbl.cmo parsing/location.cmo parsing/longident.cmo parsing/docstrings.cmo parsing/ast_helper.cmo parsing/syntaxerr.cmo parsing/parser.cmo parsing/lexer.cmo parsing/parse.cmo parsing/printast.cmo parsing/pprintast.cmo parsing/ast_mapper.cmo parsing/attr_helper.cmo typing/ident.cmo typing/path.cmo typing/primitive.cmo typing/types.cmo typing/btype.cmo typing/oprint.cmo typing/subst.cmo typing/predef.cmo typing/datarepr.cmo typing/cmi_format.cmo typing/env.cmo typing/typedtree.cmo typing/printtyped.cmo typing/ctype.cmo typing/printtyp.cmo typing/includeclass.cmo typing/mtype.cmo typing/envaux.cmo typing/includecore.cmo typing/typedtreeIter.cmo typing/typedtreeMap.cmo typing/tast_mapper.cmo typing/cmt_format.cmo typing/includemod.cmo typing/typetexp.cmo typing/parmatch.cmo typing/stypes.cmo typing/typecore.cmo typing/typedecl.cmo typing/typeclass.cmo typing/typemod.cmo typing/untypeast.cmo bytecomp/lambda.cmo bytecomp/printlambda.cmo bytecomp/typeopt.cmo bytecomp/switch.cmo bytecomp/matching.cmo bytecomp/translobj.cmo bytecomp/translcore.cmo bytecomp/translclass.cmo bytecomp/translmod.cmo bytecomp/simplif.cmo bytecomp/runtimedef.cmo driver/pparse.cmo driver/main_args.cmo driver/compenv.cmo driver/compmisc.cmo

boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/instruct.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/meta.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/meta.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/instruct.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytegen.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytegen.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/printinstr.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/printinstr.ml

sed -n -e '/^enum/p' -e 's/,//g' -e '/^  /p' byterun/caml/instruct.h | \
	awk -f tools/make-opcodes > bytecomp/opcodes.ml

boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/opcodes.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/cmo_format.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/emitcode.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/emitcode.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytesections.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytesections.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/dll.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/dll.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/symtable.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/symtable.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytelink.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytelink.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytelibrarian.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytelibrarian.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytepackager.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c bytecomp/bytepackager.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/errors.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/errors.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/compile.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/compile.ml

boot/ocamlrun boot/ocamlc -nostdlib -I boot -a -o compilerlibs/ocamlbytecomp.cma bytecomp/meta.cmo bytecomp/instruct.cmo bytecomp/bytegen.cmo bytecomp/printinstr.cmo bytecomp/opcodes.cmo bytecomp/emitcode.cmo bytecomp/bytesections.cmo bytecomp/dll.cmo bytecomp/symtable.cmo bytecomp/bytelink.cmo bytecomp/bytelibrarian.cmo bytecomp/bytepackager.cmo driver/errors.cmo driver/compile.cmo

boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/main.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c driver/main.ml

boot/ocamlrun boot/ocamlc -nostdlib -I boot  -compat-32 -o ocamlc \
	   compilerlibs/ocamlcommon.cma compilerlibs/ocamlbytecomp.cma driver/main.cmo

/Applications/Xcode.app/Contents/Developer/usr/bin/make ocamllex ocamlyacc ocamltools library
cd yacc; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
make[4]: Nothing to be done for `all'.

cd lex; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string cset.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string cset.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string syntax.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string syntax.ml
../boot/ocamlyacc -v parser.mly
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string parser.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string parser.ml
../boot/ocamlrun ../boot/ocamllex lexer.mll
98 states, 1199 transitions, table size 5384 bytes
1802 additional bytes used for bindings
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string lexer.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string lexer.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string table.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string table.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string lexgen.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string lexgen.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string compact.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string compact.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string common.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string common.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string output.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string output.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string outputbis.mli
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string outputbis.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot -c -w +33..39 -warn-error A -bin-annot -safe-string main.ml
../boot/ocamlrun ../boot/ocamlc -strict-sequence -nostdlib -I ../boot  -compat-32 -o ocamllex cset.cmo syntax.cmo parser.cmo lexer.cmo table.cmo lexgen.cmo compact.cmo common.cmo output.cmo outputbis.cmo main.cmo

make[3]: Nothing to be done for `ocamlyacc'.

boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c asmcomp/debuginfo.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c asmcomp/clambda.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c asmcomp/cmx_format.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c asmcomp/printclambda.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c asmcomp/printclambda.ml

cd tools; /Applications/Xcode.app/Contents/Developer/usr/bin/make all

../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel depend.mli
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel depend.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel ocamldep.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel -compat-32 -o ocamldep misc.cmo config.cmo clflags.cmo terminfo.cmo warnings.cmo location.cmo longident.cmo docstrings.cmo syntaxerr.cmo ast_helper.cmo parser.cmo lexer.cmo parse.cmo ccomp.cmo ast_mapper.cmo pparse.cmo compenv.cmo depend.cmo ocamldep.cmo
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel ocamlprof.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel profiling.mli
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel profiling.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel -o ocamlprof misc.cmo config.cmo clflags.cmo terminfo.cmo warnings.cmo location.cmo longident.cmo docstrings.cmo syntaxerr.cmo ast_helper.cmo parser.cmo lexer.cmo parse.cmo ocamlprof.cmo
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel ocamlcp.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel -o ocamlcp warnings.cmo main_args.cmo ocamlcp.cmo
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel ocamloptp.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel -o ocamloptp warnings.cmo main_args.cmo \
	         ocamloptp.cmo
sed -e 's|%%BINDIR%%|/usr/local/bin|' ocamlmktop.tpl > ocamlmktop
chmod +x ocamlmktop
(echo 'let bindir = "/usr/local/bin"'; \
         echo 'let ext_lib = ".a"'; \
         echo 'let ext_dll = ".so"'; \
         echo 'let supports_shared_libraries = true';\
         echo 'let mkdll = "gcc -bundle -flat_namespace -undefined suppress                    -Wl,-no_compact_unwind"'; \
         echo 'let byteccrpath = ""'; \
         echo 'let nativeccrpath = ""'; \
         echo 'let mksharedlibrpath = ""'; \
         echo 'let toolpref = ""'; \
         sed -n -e 's/^#ml //p' ../config/Makefile) \
        > ocamlmklibconfig.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel ocamlmklibconfig.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel ocamlmklib.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel -o ocamlmklib ocamlmklibconfig.cmo ocamlmklib.cmo
unset LC_ALL || : ; \
	unset LC_CTYPE || : ; \
	unset LC_COLLATE LANG || : ; \
	sed -e '/\/\*/d' \
	    -e '/^#/d' \
	    -e 's/enum \(.*\) {/let names_of_\1 = [|/' \
	    -e 's/.*};$/ |]/' \
	    -e 's/\([A-Z][A-Z_0-9a-z]*\)/"\1"/g' \
	    -e 's/,/;/g' \
	../byterun/caml/instruct.h > opnames.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel opnames.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel dumpobj.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel -o dumpobj \
	         misc.cmo tbl.cmo config.cmo ident.cmo \
	         opcodes.cmo bytesections.cmo opnames.cmo dumpobj.cmo
gcc -o objinfo_helper -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT \
          '-Dsymbol_prefix="_"'  objinfo_helper.c 
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel objinfo.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -o objinfo ../compilerlibs/ocamlcommon.cma ../compilerlibs/ocamlbytecomp.cma ../asmcomp/printclambda.cmo objinfo.cmo
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel cmt2annot.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -c -strict-sequence -w +27+32..39 -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel read_cmt.ml
../boot/ocamlrun ../boot/ocamlc -nostdlib -I ../boot -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../asmcomp -I ../driver -I ../toplevel -o read_cmt ../compilerlibs/ocamlcommon.cma ../compilerlibs/ocamlbytecomp.cma cmt2annot.cmo read_cmt.cmo
cd stdlib; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalFormatBasics.cmi` -c camlinternalFormatBasics.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalFormatBasics.cmo` -c camlinternalFormatBasics.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags pervasives.cmi` -c pervasives.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags pervasives.cmo` -c pervasives.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags array.cmi` -c array.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags array.cmo` -c array.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags list.cmi` -c list.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags list.cmo` -c list.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags char.cmi` -c char.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags char.cmo` -c char.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags bytes.cmi` -c bytes.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags bytes.cmo` -c bytes.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags string.cmi` -c string.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags string.cmo` -c string.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags sys.cmi` -c sys.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags sys.cmo` -c sys.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags sort.cmi` -c sort.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags sort.cmo` -c sort.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags marshal.cmi` -c marshal.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags marshal.cmo` -c marshal.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags int32.cmi` -c int32.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags obj.cmi` -c obj.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags obj.cmo` -c obj.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags int32.cmo` -c int32.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags int64.cmi` -c int64.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags int64.cmo` -c int64.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags nativeint.cmi` -c nativeint.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags nativeint.cmo` -c nativeint.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags lexing.cmi` -c lexing.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags lexing.cmo` -c lexing.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags parsing.cmi` -c parsing.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags parsing.cmo` -c parsing.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags set.cmi` -c set.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags set.cmo` -c set.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags map.cmi` -c map.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags map.cmo` -c map.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stack.cmi` -c stack.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stack.cmo` -c stack.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags queue.cmi` -c queue.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags queue.cmo` -c queue.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalLazy.cmi` -c camlinternalLazy.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalLazy.cmo` -c camlinternalLazy.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags lazy.cmi` -c lazy.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags lazy.cmo` -c lazy.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stream.cmi` -c stream.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stream.cmo` -c stream.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags buffer.cmi` -c buffer.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags buffer.cmo` -c buffer.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalFormat.cmi` -c camlinternalFormat.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalFormat.cmo` -c camlinternalFormat.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags printf.cmi` -c printf.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags printf.cmo` -c printf.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags arg.cmi` -c arg.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags arg.cmo` -c arg.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags printexc.cmi` -c printexc.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags printexc.cmo` -c printexc.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags gc.cmi` -c gc.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags gc.cmo` -c gc.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags digest.cmi` -c digest.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags digest.cmo` -c digest.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags random.cmi` -c random.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags random.cmo` -c random.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags hashtbl.cmi` -c hashtbl.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags hashtbl.cmo` -c hashtbl.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags weak.cmi` -c weak.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags weak.cmo` -c weak.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags format.cmi` -c format.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags format.cmo` -c format.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags scanf.cmi` -c scanf.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags scanf.cmo` -c scanf.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags callback.cmi` -c callback.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags callback.cmo` -c callback.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalOO.cmi` -c camlinternalOO.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalOO.cmo` -c camlinternalOO.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags oo.cmi` -c oo.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags oo.cmo` -c oo.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalMod.cmi` -c camlinternalMod.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags camlinternalMod.cmo` -c camlinternalMod.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags genlex.cmi` -c genlex.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags genlex.cmo` -c genlex.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags filename.cmi` -c filename.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags filename.cmo` -c filename.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags complex.cmi` -c complex.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags complex.cmo` -c complex.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags arrayLabels.cmi` -c arrayLabels.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags arrayLabels.cmo` -c arrayLabels.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags listLabels.cmi` -c listLabels.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags listLabels.cmo` -c listLabels.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags bytesLabels.cmi` -c bytesLabels.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags bytesLabels.cmo` -c bytesLabels.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stringLabels.cmi` -c stringLabels.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stringLabels.cmo` -c stringLabels.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags moreLabels.cmi` -c moreLabels.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags moreLabels.cmo` -c moreLabels.ml
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stdLabels.cmi` -c stdLabels.mli
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags stdLabels.cmo` -c stdLabels.ml
../boot/ocamlrun ../ocamlc -a -o stdlib.cma camlinternalFormatBasics.cmo pervasives.cmo array.cmo list.cmo char.cmo bytes.cmo string.cmo sys.cmo sort.cmo marshal.cmo obj.cmo int32.cmo int64.cmo nativeint.cmo lexing.cmo parsing.cmo set.cmo map.cmo stack.cmo queue.cmo camlinternalLazy.cmo lazy.cmo stream.cmo buffer.cmo camlinternalFormat.cmo printf.cmo arg.cmo printexc.cmo gc.cmo digest.cmo random.cmo hashtbl.cmo weak.cmo format.cmo scanf.cmo callback.cmo camlinternalOO.cmo oo.cmo camlinternalMod.cmo genlex.cmo filename.cmo complex.cmo arrayLabels.cmo listLabels.cmo bytesLabels.cmo stringLabels.cmo moreLabels.cmo stdLabels.cmo
../boot/ocamlrun ../ocamlc -strict-sequence -w +33..39 -g -warn-error A -bin-annot -nostdlib -safe-string `./Compflags std_exit.cmo` -c std_exit.ml
/Applications/Xcode.app/Contents/Developer/usr/bin/make ocaml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/genprintval.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/genprintval.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/toploop.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/toploop.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/trace.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/trace.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/topdirs.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/topdirs.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/topmain.mli
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/topmain.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -a -o compilerlibs/ocamltoplevel.cma toplevel/genprintval.cmo toplevel/toploop.cmo toplevel/trace.cmo toplevel/topdirs.cmo toplevel/topmain.cmo
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/topstart.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot -strict-sequence -w +33..39+48+50 -warn-error A -bin-annot -safe-string -I utils -I parsing -I typing -I bytecomp -I asmcomp -I driver -I toplevel -c toplevel/expunge.ml
boot/ocamlrun boot/ocamlc -nostdlib -I boot  -o expunge compilerlibs/ocamlcommon.cma \
	         compilerlibs/ocamlbytecomp.cma toplevel/expunge.cmo
boot/ocamlrun boot/ocamlc -nostdlib -I boot  -linkall -o ocaml.tmp \
	  compilerlibs/ocamlcommon.cma compilerlibs/ocamlbytecomp.cma \
	  compilerlibs/ocamltoplevel.cma toplevel/topstart.cmo
boot/ocamlrun ./expunge ocaml.tmp ocaml arg array arrayLabels buffer bytes bytesLabels callback camlinternalFormat camlinternalFormatBasics camlinternalLazy camlinternalMod camlinternalOO char complex digest filename format gc genlex hashtbl int32 int64 lazy lexing list listLabels map marshal moreLabels nativeint obj oo parsing pervasives printexc printf queue random scanf set sort stack stdLabels stream string stringLabels sys weak outcometree topdirs toploop
rm -f ocaml.tmp
/Applications/Xcode.app/Contents/Developer/usr/bin/make otherlibraries ocamlbuild.byte ocamldebugger \
	  ocamldoc
cd yacc; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
make[3]: Nothing to be done for `all'.
cd lex; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
make[3]: Nothing to be done for `all'.
cd tools; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
make[3]: Nothing to be done for `all'.
for i in unix str num dynlink bigarray systhreads threads graph; do \
	  (cd otherlibs/$i; /Applications/Xcode.app/Contents/Developer/usr/bin/make all) || exit $?; \
	done
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c accept.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c access.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c addrofstr.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c alarm.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c bind.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c chdir.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c chmod.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c chown.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c chroot.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c close.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c closedir.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c connect.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c cst2constr.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c cstringv.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c dup.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c dup2.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c envir.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c errmsg.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c execv.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c execve.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c execvp.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c exit.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c fchmod.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c fchown.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c fcntl.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c fork.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c ftruncate.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getaddrinfo.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getcwd.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getegid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c geteuid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getgid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getgr.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getgroups.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c gethost.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c gethostname.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getlogin.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getnameinfo.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getpeername.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getpid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getppid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getproto.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getpw.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c gettimeofday.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getserv.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getsockname.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c getuid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c gmtime.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c initgroups.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c isatty.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c itimer.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c kill.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c link.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c listen.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c lockf.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c lseek.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c mkdir.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c mkfifo.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c nice.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c open.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c opendir.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c pipe.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c putenv.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c read.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c readdir.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c readlink.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c rename.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c rewinddir.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c rmdir.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c select.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c sendrecv.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c setgid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c setgroups.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c setsid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c setuid.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c shutdown.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c signals.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c sleep.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c socket.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c socketaddr.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c socketpair.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c sockopt.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c stat.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c strofaddr.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c symlink.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c termios.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c time.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c times.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c truncate.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c umask.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c unixsupport.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c unlink.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c utimes.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c wait.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c write.c
../../boot/ocamlrun ../../tools/ocamlmklib -oc unix accept.o access.o addrofstr.o alarm.o bind.o chdir.o chmod.o chown.o chroot.o close.o closedir.o connect.o cst2constr.o cstringv.o dup.o dup2.o envir.o errmsg.o execv.o execve.o execvp.o exit.o fchmod.o fchown.o fcntl.o fork.o ftruncate.o getaddrinfo.o getcwd.o getegid.o geteuid.o getgid.o getgr.o getgroups.o gethost.o gethostname.o getlogin.o getnameinfo.o getpeername.o getpid.o getppid.o getproto.o getpw.o gettimeofday.o getserv.o getsockname.o getuid.o gmtime.o initgroups.o isatty.o itimer.o kill.o link.o listen.o lockf.o lseek.o mkdir.o mkfifo.o nice.o open.o opendir.o pipe.o putenv.o read.o readdir.o readlink.o rename.o rewinddir.o rmdir.o select.o sendrecv.o setgid.o setgroups.o setsid.o setuid.o shutdown.o signals.o sleep.o socket.o socketaddr.o socketpair.o sockopt.o stat.o strofaddr.o symlink.o termios.o time.o times.o truncate.o umask.o unixsupport.o unlink.o utimes.o wait.o write.o 
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -nolabels unix.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -nolabels unix.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -nolabels unixLabels.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -nolabels unixLabels.ml
../../boot/ocamlrun ../../tools/ocamlmklib -o unix -oc unix -ocamlc '../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib' -linkall \
	         unix.cmo unixLabels.cmo 
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun   -c strstubs.c
../../boot/ocamlrun ../../tools/ocamlmklib -oc camlstr strstubs.o 
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  str.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  str.ml
File "str.ml", line 96, characters 29-43:
Warning 3: deprecated: Char.lowercase
Use Char.lowercase_ascii instead.
File "str.ml", line 96, characters 55-69:
Warning 3: deprecated: Char.uppercase
Use Char.uppercase_ascii instead.
File "str.ml", line 219, characters 38-52:
Warning 3: deprecated: Char.lowercase
Use Char.lowercase_ascii instead.
File "str.ml", line 276, characters 43-57:
Warning 3: deprecated: Char.lowercase
Use Char.lowercase_ascii instead.
File "str.ml", line 285, characters 45-59:
Warning 3: deprecated: Char.lowercase
Use Char.lowercase_ascii instead.
File "str.ml", line 299, characters 51-67:
Warning 3: deprecated: String.lowercase
Use String.lowercase_ascii instead.
../../boot/ocamlrun ../../tools/ocamlmklib -o str -oc camlstr -ocamlc '../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib' -linkall \
	         str.cmo 
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -DBNG_ARCH_amd64 -DBNG_ASM_LEVEL=1 -c bng.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -DBNG_ARCH_amd64 -DBNG_ASM_LEVEL=1 -c nat_stubs.c
../../boot/ocamlrun ../../tools/ocamlmklib -oc nums bng.o nat_stubs.o 
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  int_misc.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  int_misc.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  nat.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  nat.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  big_int.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  big_int.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  arith_flags.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  arith_flags.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  ratio.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  ratio.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  num.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  num.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  arith_status.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  arith_status.ml
../../boot/ocamlrun ../../tools/ocamlmklib -o nums -oc nums -ocamlc '../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib' -linkall \
	         int_misc.cmo nat.cmo big_int.cmo arith_flags.cmo ratio.cmo num.cmo arith_status.cmo 
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -w +33..39 -warn-error A -bin-annot -g -safe-string -I ../../stdlib -I ../../utils -I ../../typing -I ../../bytecomp -I ../../asmcomp -pack -o dynlinkaux.cmo ../../utils/misc.cmo ../../utils/config.cmo ../../utils/clflags.cmo ../../utils/tbl.cmo ../../utils/consistbl.cmo ../../utils/terminfo.cmo ../../utils/warnings.cmo ../../parsing/asttypes.cmi ../../parsing/location.cmo ../../parsing/longident.cmo ../../parsing/docstrings.cmo ../../parsing/ast_helper.cmo ../../parsing/ast_mapper.cmo ../../parsing/attr_helper.cmo ../../typing/ident.cmo ../../typing/path.cmo ../../typing/primitive.cmo ../../typing/types.cmo ../../typing/btype.cmo ../../typing/subst.cmo ../../typing/predef.cmo ../../typing/datarepr.cmo ../../typing/cmi_format.cmo ../../typing/env.cmo ../../bytecomp/lambda.cmo ../../bytecomp/instruct.cmo ../../bytecomp/cmo_format.cmi ../../bytecomp/opcodes.cmo ../../bytecomp/runtimedef.cmo ../../bytecomp/bytesections.cmo ../../bytecomp/dll.cmo ../../bytecomp/meta.cmo ../../bytecomp/symtable.cmo
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -I ../../stdlib -I ../../utils -I ../../typing -I ../../bytecomp -I ../../asmcomp dynlink.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -I ../../stdlib -I ../../utils -I ../../typing -I ../../bytecomp -I ../../asmcomp dynlink.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -w +33..39 -warn-error A -bin-annot -g -safe-string -I ../../stdlib -I ../../utils -I ../../typing -I ../../bytecomp -I ../../asmcomp -ccopt "" -a -o dynlink.cma \
	         dynlinkaux.cmo dynlink.cmo
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -I ../../stdlib -I ../../utils -I ../../typing -I ../../bytecomp -I ../../asmcomp extract_crc.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -o extract_crc dynlink.cma extract_crc.cmo
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I../unix -DIN_OCAML_BIGARRAY -DCAML_NAME_SPACE -c bigarray_stubs.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I../unix -DIN_OCAML_BIGARRAY -DCAML_NAME_SPACE -c mmap_unix.c
../../boot/ocamlrun ../../tools/ocamlmklib -oc bigarray bigarray_stubs.o mmap_unix.o 
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -I ../unix bigarray.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string -I ../unix bigarray.ml
../../boot/ocamlrun ../../tools/ocamlmklib -o bigarray -oc bigarray -ocamlc '../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib' -linkall \
	         bigarray.cmo 
gcc -I../../byterun -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT  \
	   -c st_stubs.c
mv st_stubs.o st_stubs_b.o
../../boot/ocamlrun ../../tools/ocamlmklib -o threads st_stubs_b.o -lpthread
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string thread.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string thread.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string mutex.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string mutex.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string condition.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string condition.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string event.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string event.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string threadUnix.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -g -bin-annot -safe-string threadUnix.ml
../../boot/ocamlrun ../../tools/ocamlmklib -ocamlc '../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix' -o threads thread.cmo mutex.cmo condition.cmo event.cmo threadUnix.cmo \
	  -cclib -lunix -cclib -lpthread
gcc -I../../byterun -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT  -g   -c -o scheduler.o scheduler.c
../../boot/ocamlrun ../../tools/ocamlmklib -o threads -oc vmthreads scheduler.o
ln -s -f ../unix/unix.cmi unix.cmi
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string thread.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string thread.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string mutex.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string mutex.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string condition.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string condition.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string event.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string event.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string threadUnix.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -c -w +33..39 -warn-error A -bin-annot -g -safe-string threadUnix.ml
../../boot/ocamlrun ../../tools/ocamlmklib -ocamlc '../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix' -o threads -oc vmthreads thread.cmo mutex.cmo condition.cmo event.cmo threadUnix.cmo
ln -s ../../stdlib/pervasives.mli pervasives.mli
ln -s ../../stdlib/pervasives.cmi pervasives.cmi
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -w +33..39 -warn-error A -bin-annot -g -safe-string -nopervasives -c pervasives.ml
ln -s ../../stdlib/marshal.mli marshal.mli
ln -s ../../stdlib/marshal.cmi marshal.cmi
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -w +33..39 -warn-error A -bin-annot -g -safe-string -c marshal.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -a -o stdlib.cma ../../stdlib/camlinternalFormatBasics.cmo pervasives.cmo ../../stdlib/array.cmo ../../stdlib/list.cmo ../../stdlib/char.cmo ../../stdlib/bytes.cmo ../../stdlib/string.cmo ../../stdlib/sys.cmo ../../stdlib/sort.cmo marshal.cmo ../../stdlib/obj.cmo ../../stdlib/int32.cmo ../../stdlib/int64.cmo ../../stdlib/nativeint.cmo ../../stdlib/lexing.cmo ../../stdlib/parsing.cmo ../../stdlib/set.cmo ../../stdlib/map.cmo ../../stdlib/stack.cmo ../../stdlib/queue.cmo ../../stdlib/camlinternalLazy.cmo ../../stdlib/lazy.cmo ../../stdlib/stream.cmo ../../stdlib/buffer.cmo ../../stdlib/camlinternalFormat.cmo ../../stdlib/printf.cmo ../../stdlib/arg.cmo ../../stdlib/printexc.cmo ../../stdlib/gc.cmo ../../stdlib/digest.cmo ../../stdlib/random.cmo ../../stdlib/hashtbl.cmo ../../stdlib/format.cmo ../../stdlib/scanf.cmo ../../stdlib/callback.cmo ../../stdlib/camlinternalOO.cmo ../../stdlib/oo.cmo ../../stdlib/camlinternalMod.cmo ../../stdlib/genlex.cmo ../../stdlib/weak.cmo ../../stdlib/filename.cmo ../../stdlib/complex.cmo ../../stdlib/arrayLabels.cmo ../../stdlib/listLabels.cmo ../../stdlib/bytesLabels.cmo ../../stdlib/stringLabels.cmo ../../stdlib/moreLabels.cmo ../../stdlib/stdLabels.cmo
ln -s -f ../unix/unix.mli unix.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix -w +33..39 -warn-error A -bin-annot -g -safe-string -c unix.ml
../../boot/ocamlrun ../../tools/ocamlmklib -ocamlc '../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -I ../../otherlibs/unix' -o unix -linkall unix.cmo ../unix/unixLabels.cmo
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c open.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c draw.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c fill.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c color.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c text.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c image.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c make_img.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c dump_img.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c point_col.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c sound.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c events.c
gcc -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT -I../../byterun  -I/opt/X11/include -c subwindow.c
../../boot/ocamlrun ../../tools/ocamlmklib -oc graphics open.o draw.o fill.o color.o text.o image.o make_img.o dump_img.o point_col.o sound.o events.o subwindow.o -ldopt "-L/opt/X11/lib -lX11"
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  graphics.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  graphics.ml
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  graphicsX11.mli
../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib -c -w +33..39 -warn-error A -bin-annot -g -safe-string  graphicsX11.ml
../../boot/ocamlrun ../../tools/ocamlmklib -o graphics -oc graphics -ocamlc '../../boot/ocamlrun ../../ocamlc -nostdlib -I ../../stdlib' -linkall \
	         graphics.cmo graphicsX11.cmo -cclib "\"-L/opt/X11/lib -lX11\""
cd ocamlbuild && /Applications/Xcode.app/Contents/Developer/usr/bin/make all
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c const.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c loc.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c loc.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c discard_printf.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c discard_printf.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c signatures.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c my_std.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c my_std.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c my_unix.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c my_unix.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c tags.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c tags.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c display.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c display.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c log.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c log.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c shell.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c shell.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c bool.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c bool.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c glob_ast.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c glob_ast.ml
../boot/ocamlrun ../boot/ocamllex glob_lexer.mll
55 states, 419 transitions, table size 2006 bytes
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c glob_lexer.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c glob_lexer.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c glob.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c glob.ml
../boot/ocamlrun ../boot/ocamllex lexers.mll
251 states, 1051 transitions, table size 5710 bytes
4334 additional bytes used for bindings
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c lexers.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c lexers.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c param_tags.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c param_tags.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c command.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c command.ml
(echo 'let bindir = "/usr/local/bin"'; \
	 echo 'let libdir = "/usr/local/lib/ocaml"'; \
	 echo 'let supports_shared_libraries = true';\
	 echo 'let a = "a"'; \
	 echo 'let o = "o"'; \
	 echo 'let so = "so"'; \
	 echo 'let ext_dll = ".so"'; \
	 echo 'let exe = ""'; \
	) > ocamlbuild_config.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_config.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_where.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_where.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c slurp.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c slurp.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c options.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c options.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c pathname.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c pathname.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c configuration.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c configuration.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c flags.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c flags.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c hygiene.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c hygiene.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c digest_cache.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c digest_cache.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c resource.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c resource.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c rule.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c rule.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c solver.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c solver.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c report.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c report.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c tools.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c tools.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c fda.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c fda.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c findlib.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c findlib.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_arch.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_arch.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_utils.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_utils.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_dependencies.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_dependencies.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_compiler.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_compiler.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_tools.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_tools.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_specific.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocaml_specific.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c plugin.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c plugin.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c exit_codes.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c exit_codes.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c hooks.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c hooks.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c main.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c main.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pack const.cmo loc.cmo discard_printf.cmo signatures.cmi my_std.cmo my_unix.cmo tags.cmo display.cmo log.cmo shell.cmo bool.cmo glob_ast.cmo glob_lexer.cmo glob.cmo lexers.cmo param_tags.cmo command.cmo ocamlbuild_config.cmo ocamlbuild_where.cmo slurp.cmo options.cmo pathname.cmo configuration.cmo flags.cmo hygiene.cmo digest_cache.cmo resource.cmo rule.cmo solver.cmo report.cmo tools.cmo fda.cmo findlib.cmo ocaml_arch.cmo ocaml_utils.cmo ocaml_dependencies.cmo ocaml_compiler.cmo ocaml_tools.cmo ocaml_specific.cmo plugin.cmo exit_codes.cmo hooks.cmo main.cmo -o ocamlbuild_pack.cmo
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_plugin.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_plugin.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_executor.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_executor.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_unix_plugin.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild_unix_plugin.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -warn-error A -w L -w R -w Z -I ../otherlibs/unix -safe-string -c ocamlbuild.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -I ../otherlibs/unix -o ocamlbuild.byte \
          unix.cma ocamlbuild_pack.cmo ocamlbuild_plugin.cmo ocamlbuild_executor.cmo ocamlbuild_unix_plugin.cmo ocamlbuild.cmo
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -a -o ocamlbuildlib.cma \
          ocamlbuild_pack.cmo ocamlbuild_plugin.cmo ocamlbuild_executor.cmo ocamlbuild_unix_plugin.cmo
cd debugger; /Applications/Xcode.app/Contents/Developer/usr/bin/make all
grep -v 'REMOVE_ME for ../../debugger/dynlink.ml' \
	     ../otherlibs/dynlink/dynlink.ml >dynlink.ml
cp ../otherlibs/dynlink/dynlink.mli .
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix dynlink.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix dynlink.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix int64ops.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix int64ops.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix primitives.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix primitives.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix unix_tools.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix unix_tools.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix debugger_config.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix debugger_config.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix parameters.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix parameters.ml
../boot/ocamlrun ../boot/ocamllex lexer.mll
41 states, 1026 transitions, table size 4350 bytes
1285 additional bytes used for bindings
../boot/ocamlyacc parser.mly
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix parser_aux.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix parser.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix lexer.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix lexer.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix input_handling.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix input_handling.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix question.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix question.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix debugcom.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix debugcom.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix exec.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix exec.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix source.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix source.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix pos.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix pos.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix checkpoints.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix checkpoints.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix events.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix events.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix program_loading.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix program_loading.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix symbols.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix symbols.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix breakpoints.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix breakpoints.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix trap_barrier.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix trap_barrier.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix history.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix history.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix printval.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix printval.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix show_source.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix show_source.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix time_travel.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix time_travel.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix program_management.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix program_management.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix frames.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix frames.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix eval.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix eval.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix show_information.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix show_information.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix loadprinter.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix loadprinter.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix parser.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix command_line.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix command_line.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -c -warn-error A -safe-string -I ../utils -I ../parsing -I ../typing -I ../bytecomp -I ../toplevel -I ../otherlibs/unix main.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -linkall -I ../otherlibs/unix -o ocamldebug -linkall ../otherlibs/unix/unix.cma ../utils/misc.cmo ../utils/config.cmo ../utils/tbl.cmo ../utils/clflags.cmo ../utils/consistbl.cmo ../utils/warnings.cmo ../parsing/location.cmo ../parsing/longident.cmo ../parsing/docstrings.cmo ../parsing/ast_helper.cmo ../parsing/ast_mapper.cmo ../parsing/attr_helper.cmo ../typing/ident.cmo ../typing/path.cmo ../typing/types.cmo ../typing/btype.cmo ../typing/primitive.cmo ../typing/typedtree.cmo ../typing/subst.cmo ../typing/predef.cmo ../typing/datarepr.cmo ../typing/cmi_format.cmo ../typing/env.cmo ../typing/oprint.cmo ../typing/ctype.cmo ../typing/printtyp.cmo ../typing/mtype.cmo ../typing/envaux.cmo ../bytecomp/runtimedef.cmo ../bytecomp/bytesections.cmo ../bytecomp/dll.cmo ../bytecomp/meta.cmo ../bytecomp/symtable.cmo ../bytecomp/opcodes.cmo ../toplevel/genprintval.cmo dynlink.cmo int64ops.cmo primitives.cmo unix_tools.cmo debugger_config.cmo parameters.cmo lexer.cmo input_handling.cmo question.cmo debugcom.cmo exec.cmo source.cmo pos.cmo checkpoints.cmo events.cmo program_loading.cmo symbols.cmo breakpoints.cmo trap_barrier.cmo history.cmo printval.cmo show_source.cmo time_travel.cmo program_management.cmo frames.cmo eval.cmo show_information.cmo loadprinter.cmo parser.cmo command_line.cmo main.cmo
cd ocamldoc && /Applications/Xcode.app/Contents/Developer/usr/bin/make all
/Applications/Xcode.app/Contents/Developer/usr/bin/make exe
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_config.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_config.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_messages.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_types.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_global.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_global.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_types.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_misc.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_misc.ml
../boot/ocamlyacc -v odoc_text_parser.mly
5 shift/reduce conflicts.
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_text_parser.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_text_parser.ml
../boot/ocamlrun ../boot/ocamllex odoc_text_lexer.mll
251 states, 2458 transitions, table size 11338 bytes
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_text_lexer.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_text.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_text.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_name.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_name.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_parameter.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_value.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_type.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_extension.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_exception.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_class.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_module.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_print.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_print.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_str.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_str.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_comments_global.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_comments_global.ml
../boot/ocamlyacc -v odoc_parser.mly
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_parser.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_parser.ml
../boot/ocamlrun ../boot/ocamllex odoc_lexer.mll
50 states, 614 transitions, table size 2756 bytes
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_lexer.ml
../boot/ocamlrun ../boot/ocamllex odoc_see_lexer.mll
20 states, 264 transitions, table size 1176 bytes
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_see_lexer.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_env.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_env.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_merge.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_merge.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_sig.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_sig.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_ast.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_ast.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_control.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_inherit.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_search.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_search.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_scan.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_cross.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_cross.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_comments.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_comments.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_dep.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_analyse.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_analyse.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_info.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_info.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_dag2html.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_dag2html.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_to_text.ml
../boot/ocamlrun ../boot/ocamllex odoc_ocamlhtml.mll
111 states, 2871 transitions, table size 12150 bytes
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_ocamlhtml.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_html.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_man.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_latex_style.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_latex.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_texi.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_dot.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_gen.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_gen.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_args.mli
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_args.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -o ocamldoc -linkall unix.cma str.cma dynlink.cma \
	          ../compilerlibs/ocamlcommon.cma \
	          -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -nostdlib ../tools/depend.cmo odoc_config.cmo odoc_messages.cmo odoc_global.cmo odoc_types.cmo odoc_misc.cmo odoc_text_parser.cmo odoc_text_lexer.cmo odoc_text.cmo odoc_name.cmo odoc_parameter.cmo odoc_value.cmo odoc_type.cmo odoc_extension.cmo odoc_exception.cmo odoc_class.cmo odoc_module.cmo odoc_print.cmo odoc_str.cmo odoc_comments_global.cmo odoc_parser.cmo odoc_lexer.cmo odoc_see_lexer.cmo odoc_env.cmo odoc_merge.cmo odoc_sig.cmo odoc_ast.cmo odoc_control.cmo odoc_inherit.cmo odoc_search.cmo odoc_scan.cmo odoc_cross.cmo odoc_comments.cmo odoc_dep.cmo odoc_analyse.cmo odoc_info.cmo odoc_dag2html.cmo odoc_to_text.cmo odoc_ocamlhtml.cmo odoc_html.cmo odoc_man.cmo odoc_latex_style.cmo odoc_latex.cmo odoc_texi.cmo odoc_dot.cmo odoc_gen.cmo odoc_args.cmo odoc.cmo
/Applications/Xcode.app/Contents/Developer/usr/bin/make lib
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -a -o odoc_info.cma -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -nostdlib ../tools/depend.cmo \
	          odoc_config.cmo odoc_messages.cmo odoc_global.cmo odoc_types.cmo odoc_misc.cmo odoc_text_parser.cmo odoc_text_lexer.cmo odoc_text.cmo odoc_name.cmo odoc_parameter.cmo odoc_value.cmo odoc_type.cmo odoc_extension.cmo odoc_exception.cmo odoc_class.cmo odoc_module.cmo odoc_print.cmo odoc_str.cmo odoc_comments_global.cmo odoc_parser.cmo odoc_lexer.cmo odoc_see_lexer.cmo odoc_env.cmo odoc_merge.cmo odoc_sig.cmo odoc_ast.cmo odoc_control.cmo odoc_inherit.cmo odoc_search.cmo odoc_scan.cmo odoc_cross.cmo odoc_comments.cmo odoc_dep.cmo odoc_analyse.cmo odoc_info.cmo
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c odoc_test.ml
/Applications/Xcode.app/Contents/Developer/usr/bin/make generators
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c generators/odoc_todo.ml
../boot/ocamlrun ../ocamlc -nostdlib -I ../stdlib -pp './remove_DEBUG' -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph -warn-error A -safe-string -c generators/odoc_literate.ml
/Applications/Xcode.app/Contents/Developer/usr/bin/make manpages
mkdir -p stdlib_man
sh ./runocamldoc true -man -d stdlib_man -I ../parsing -I ../utils -I ../typing -I ../driver -I ../bytecomp -I ../tools -I ../toplevel/ -I ../stdlib -I ../otherlibs/str -I ../otherlibs/dynlink -I ../otherlibs/unix -I ../otherlibs/num -I ../otherlibs/graph \
	-t "OCaml library" -man-mini \
	../stdlib/*.mli ../parsing/*.mli ../otherlibs/unix/unix.mli ../otherlibs/str/str.mli ../otherlibs/bigarray/bigarray.mli ../otherlibs/num/num.mli

4. Building the native code compiler
------------------------------------

5. More native code
-------------------

6. Installing
---------------


