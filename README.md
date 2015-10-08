What OCaml Compiles when it Compiles
====================================

This document is written from a state of ignorance. Please forward all
corrections to john@coherentgraphics.co.uk or make an issue or pull request
here:

http://www.github.com/johnwhitington/what-ocaml-compiles


1. What's in the checkout
-------------------------

2. Running configure
--------------------

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


(state of directory now)

3. Building the bytecode compiler
---------------------------------

/Applications/Xcode.app/Contents/Developer/usr/bin/make coldstart

cd byterun; /Applications/Xcode.app/Contents/Developer/usr/bin/make all

sed -n -e '/^  /s/ \([A-Z]\)/ \&\&lbl_\1/gp' \
	       -e '/^}/q' caml/instruct.h > caml/jumptbl.h

gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o interp.o interp.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o misc.o misc.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o stacks.o stacks.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o fix_code.o fix_code.c
gcc -DCAML_NAME_SPACE -O2 -fno-strict-aliasing -fwrapv -Wall -Werror -D_FILE_OFFSET_BITS=64 -D_REENTRANT    -c -o startup_aux.o startup_aux.c

../tools/make-version-header.sh ../VERSION > caml/version.h

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

ar rc libcamlrun.a interp.o misc.o stacks.o fix_code.o startup_aux.o startup.o freelist.o major_gc.o minor_gc.o memory.o alloc.o roots.o globroots.o fail.o signals.o signals_byt.o printexc.o backtrace_prim.o backtrace.o compare.o ints.o floats.o str.o array.o io.o extern.o intern.o hash.o sys.o meta.o parsing.o gc_ctrl.o terminfo.o md5.o obj.o lexing.o callback.o debugger.o weak.o compact.o finalise.o custom.o dynlink.o unix.o main.o
ranlib libcamlrun.a

4. Building the native code compiler
------------------------------------

5. More native code
-------------------

6. Installing
---------------


