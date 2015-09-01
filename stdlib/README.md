Standard Library
================

`system`
--------
Mostly internal module which allows for direct calls to system-provided
functions. This is where the magic happens, so that all the remaining modules
can be implemented fully in Newt.

`console`
---------
Used for CLI applications. Allows for easy printing to the standard output
(including formatted strings) and reading from the standard input. Includes 
functions like `write`, `writeln`, `writef`, `readInteger`, `readLine`,
`readf` etc.

`io`
----
Input/output module. Handles reading and writing files, both text and binary,
reading and modifying filesystem structure.

`gfx`
-----
2D graphics module, capable of drawing simple shapes, loading and displaying
sprites from graphics files etc.

`input`
-------
Handles input from mouse, keyboard and gaming devices.

`audio`
-------
Handles audio output (and maybe input).

`net`
-----
Sockets.

`web`
-----
Simple web server framework.

`sql`
-----
Database bindings. Probably limited to sqlite, at least at the start.

`algorithm`
-----------
Implements basic algorithms.

`containers`
------------
Implements basic data structures, like stack, queue, list, map, set etc.
