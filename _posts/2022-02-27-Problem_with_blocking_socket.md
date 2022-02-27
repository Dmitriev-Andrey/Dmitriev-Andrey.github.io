---
layout: post
title: What's problem with blocking sockets?
---
I described how a server with blocking sockets works [here](/Blocking_socket_server/). In general, it's a good idea to use this approach in most cases.
But what if we have blocking operations in a thread? For instance, request in another service or database, reading files, etc. The thread will be blocked, of course. And if our program has a lot of blocking operations, like [CRUD application](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete), threads will be waiting for these operations without useful work. And if we have many threads, we will spend a lot of time on switch context, CPU resources, etc. This case is called [IO-bound](https://en.wikipedia.org/wiki/I/O_bound).

How can we resolve the problem? Easy! We need to use asynchronous mechanisms. That means that we are not waiting for a new message from a socket, but telling the socket to let us know when we can read this message. In this case, a thread isn't blocked and can handle other connections. 

Describe a common pipeline for an asynchronous server:
1. Creating a server socket.
2. Tell the server socket listens port and let us know about new connections.
3. When the server gets a new connection, we create a special socket for communication and tell the socket to let us know about different events (messages, interruptions).
4. When the socket gets a new event, we create a new asynchronous operation (example request to a database), like with sockets.
5. When we get an event from the operation, send a message to the socket.

In the next article, we create a small asynchronous server.