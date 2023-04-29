vtece4564-websockets
====================

This is a fork of the excellent [websockets](https://pypi.org/project/websockets/)
library, created to facilitate use of a patched version that properly
supports the threaded `WebSocketServer` implementation on Windows. 
A [pull request](https://github.com/python-websockets/websockets/pull/1349) 
has been submitted to the main project to incorporate the fix in the official
distribution.

Unless you're a student of Virginia Tech's ECE 4564 Network Applications
Design course in Spring 2023, you almost certainly want to use the official
[websockets](https://pypi.org/project/websockets/) distribution. This 
distribution will be removed from PyPI in May 2023.

Background
----------

For a senior-level university network apps design course, my students are
using the threaded version of `WebSocketServer` as part of their final 
project. While the project required their work to run successfully in 
containers on Linux, many of them use Windows for their development 
workstation. 

We discovered that `serve_forever` throws an error at startup on Windows due 
to the use of `select.poll` as the means to block while waiting for incoming
socket connections. The documentation for `poll`indicates that it is not 
supported on all platforms, and apparently Windows is one such platform.

We patched `WebSocketServer` to instead use `selectors.DefaultSelector` which 
determines the best supported mechanism for I/O multiplexing on the runtime 
platform. The functionality is the same, but it works on a wider range of 
platforms.

After switching to `selectors.DefaultSelector`, we discovered that Windows 
also doesn't support I/O multiplexing using pipes or files -- only sockets. 
we removed the use of `os.pipe` for the shutdown mechanism. It was redundant 
anyway -- simply closing the listener socket in the shutdown method is 
sufficient to cause the selector to return. Subsequently, the call to 
`socket.accept` (on the closed listener socket) causes the loop in 
`serve_forever` to terminate as expected.

We tested the change on Windows, Mac OS X, and a Linux container, and it 
seems that it correctly supports all three platforms.




