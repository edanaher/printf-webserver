"Portable" printf webserver
===========================

Inspired by http://tinyhack.com/2014/03/12/implementing-a-web-server-in-a-single-printf-call/ , this is a moderately portable webserver in a single printf command.  "Portable" in this case means it is expected to work on any x86_64 Linux system using a gcc version from 4.4 to 4.7.  More portability should be possible; check for higher-numbered files that may work on more gcc versions, x86, or maybe even OS X.

Explanation
===========
This is based on the same principle as the inspiration, including the same shell code.  Check the blog post above if you want more details.  But generally, the idea is to put shellcode in a string (the string of random hex), and jump to it by combining the %* and %n format specifiers to overwrite an address.  The original used .fini_array; this uses a simpler (and, strangely enough, more portable) stack smashing attack.

web.10.c
--------

The return address lives on the stack; this by default points to system cleanup to be run after main completes.  By taking an offset from argc (apparently 12 bytes in gcc 4.4 - 4.6, 60 bytes in 4.7), we can overwrite it.  Conveniently, in gcc 4.7, 12 bytes up is null, making for easy run-time compiler detection.  To avoid printing lots and lots of characters, this sets the address by bytes, which is a bit long, but not difficult.

Finally, where does the address of the shell code come from?  It turns out that gcc will only store one copy of each string, and they seem to be adjacent if there are no other locals, except for padding to an 8-byte boundary.  So while we could just take the address of the shellcode itself, that would require copy/pasting it eight times, which is inelegant.  So instead, the format string is used for this, with the fmtlength offset.

And yeah, there are macros, but they're just for convenience.  web.10.nomacros.c isn't really that much worse.
