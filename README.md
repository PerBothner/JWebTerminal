# JWebTerminal - a web-based console

JWebTerminal implements a general-purpose interactive console:
Users can type commands which gets sent to an application,
which evaluates the command, and displays the results, typically
in some kind of type-script format.

There are two basic modes:
- In line-editing mode each input line is an input field you
edit locally (in the browser).  The finished line is sent to the
application when you type Enter.
- In character mode each character is sent directly to the application,
which is also responsible for input echoing.

Applications of JWebTerminal include:
- A chat/talk window.
- A read-eval-print-loop for an interactive scripting language.
- A command console.
- A terminal emulator.

JWebTerminal is a rough and incomplete prototype.  It is meant to
demo my belief that a modern terminal emulator should be based
on a web browser engine.  It seems reasonable to re-write the "guts"
of the terminal emulator code in JavaScript (rather than Java),
to make the code useful in non-Java context.

JWebTerminal recognizes a large subset of the standard ANSI/xterm
terminal escape sequences for repositioning the cursor, erasing
previous text, changing style, etc.  Enough of a subset is recognized
that you can run programs like the emacs text editor, with styling.
This requires you specify the correct terminal type (TERM), which is
done automatically by the PtyTerminal application.  A matching
terminfo entry is provided, but it also should work ok to specify TERM
to a standard terminal name, like eterm-color, vte, gnome, konsole,
and xterm (all with color), or vt100 or vt220 (both non-color).

The terminfo directory defines the escape sequences supported by the
"jfxterm" terminal type, using a terminfo descriptor.  The terminal emulation
is a subset of the standard xterm/ansi/vt100 ones.

JWebTerminal handles automatic line-wrapping, distinguishing between
explicit line-breaks and those caused by line-wrapping.  This allows
for re-flow on windows re-size (though that is not yet implemented),
and "intelligent" cut-and-paste (also not yet implemented).

There is an escape sequence to send HTML to WebTerminal.
This allows you to embed images, forms, and other HTML elements
in the console output.

The primary class is webterminal.WebTerminal, which is
an abstract (overridable) control you embed in your JavaFX scenegraph.

## Building and running

The prototype requires not only Java, but specifically JavaFX 2.x.
The latter is bundled with JDK 8.x.  JavaFX is also distributed
with newer releases of JDK 7.x; however, you may have to manually
set the classpath to include jfxrt.jar.

Build the library and sample applications with the Ant tool:
    $ ant

The PtyConsole requires non-portable native code, so it is
not built by default.  Build it with:
    $ ant all

## Sample Applications

These sample applications are very simple.  There is currently no
menues or other "chrome" beside the actual WebTerminal window.

#### RunInConsole

The utility class webterminal.RunInConsole make it easy to
run an old-fashioned console-based Java application in a WebTerminal.
It re-binds System.in, System.out, and System.err to the WebTerminal.

For example, the jrunscript tool is a wrapper around the class
com.sun.tools.script.shell.Main in tools.jar.  So to start up
a JavaScript read-eval-print-tool, do:

    $ ant run-main -Drunmain.classpath=$JAVA_HOME/lib/tools.jar \
      -Drunmain.class="com.sun.tools.script.shell.Main" \
      -Drunmain.args="-l javascript"
or:
    $ java -cp dist/WebTerminal.jar:${JAVA_HOME}/lib/tools.jar \
      webterminal.RunInConsole com.sun.tools.script.shell.Main -l javascript

This uses line-editing mode.

#### ShellConsole

The application webterminal.ShellConsole uses java.lang.Process
to run a process inside a WebTerminal.  Currently, it is hard-wired to
run /bin/bash.  (It does work under Cygwin if you edit the
commandWithArgs field.)

You can start this using Ant:
    $ ant run
or directly:
    $ java -cp dist/WebTerminal.jar:${JAVA_HOME}/lib/tools.jar \
      webterminal.ShellConsole

This also uses line-editing mode.

#### PtyConsole

An application that implements a terminal emulator
using WebTermnal and a PTY to fork off an operating system process.

The TERM and TERMINFO environment variables are set up automatically
to use the "jfxterm" terminal type.

PtyConsole does depend on native code that assumes modern Unix-style
pseudo-teletypes. At the time of writing it has only been tested on
GNU/Linux Fedora 21; it may need some porting effort on older
or non-Linux platforms, especially Windows.

You can run this with Ant:
    $ ant run-pty

This uses character mode.
