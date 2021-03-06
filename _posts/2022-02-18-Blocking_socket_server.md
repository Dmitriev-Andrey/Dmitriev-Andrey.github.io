---
layout: post
title: Blocking socket server
excerpt: Principles of work of a simple server using blocking sockets.
---
## What is a socket?
A network socket is a good abstraction for communications on the Internet. But it's a low-level object, because we don't use different protocols, like HTTP, and can implement a more efficient specification for our cases.

## How does it work?
We have two points for communication: server and client. They have their features.
In general, the work pipeline looks like this: 
1. A server listens on a specific port and waits for a connection from a client.
2. The client tries to connect.
3. The server accepts the connection, creates channels for input and output, and waits for a message from the client.
4. The client sends the message.
5. The server reads the message, handles it, and responds.
6. The client reads and handles the answer.
7. Closing the connection or repeating the communication

## What's about code?

Let's try to create a simple server and client using sockets.

First, business logic is calculating [the Fibonacci numbers](https://en.wikipedia.org/wiki/Fibonacci_number). It's one recursive function:
```java
public class Calculator {
    public static long fib(int k) {
        if (k == 0 || k == 1) {
            return 1;
        }
        return fib(k - 1) + fib(k - 2);
    }
}
```
This algorithm isn't efficient and I implement it in a separate class because I will be using it in the next article.

Second, server. We need to listen port, establish a connection, read a message, handle and respond.
Input and output messages are numbers (for simplicity).
Okey, sounds not too hard:

```java
public class Server {

    private final int port;

    public Server(int port) {
        this.port = port;
    }

    /**
    * This method implements all logic for handling a connection.
    */
    public void run() throws IOException {
        try (ServerSocket serverSocket = new ServerSocket(port); // listen port
             Socket socket = serverSocket.accept();  // wait connection
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream())); // create input stream
             BufferedWriter out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()))) { // create output stream
            String input = in.readLine(); // wait and read input message

            // The block below parses the message, 
            // calculates the fibonacci number 
            // and forms a response message
            String output;
            try {
                int k = Integer.parseInt(input);
                long fib = Calculator.fib(k);
                output = String.valueOf(fib);
            } catch (NumberFormatException e) {
                output = "Error format: " + input; // if input isn't number
            }
            //----------------------------------------

            // Write message in buffer
            out.write(output + "\n");
            // And send message
            out.flush();
        }
    }

    public static void main(String[] args) throws IOException {
        new Server(4000).run(); // start server
    }
}
```

If you notice, I use [`try with resource`](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html) so java closes socket and streams for me.

### What do you think is the problem with our server?

When the server responds it closes the connection and stops. What if the client wants to send the following message? And what are about other clients?
Try to solve these problems:
```java
public void run() throws IOException {
    try (ServerSocket serverSocket = new ServerSocket(port)) {
        while (true) { // (1)
            try (Socket socket = serverSocket.accept();
                    BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                    BufferedWriter out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()))) {
                socket.setSoTimeout(TIMEOUT);
                while (true) { // (2)
                    
                    String input; // (3)
                    try {
                        input = in.readLine();
                    } catch (SocketTimeoutException e) {
                        System.out.println("Timeout!!!");
                        break;
                    }

                    if (input.equals("exit")) { // (4)
                        out.write("good bye!\n");
                        out.flush();
                        break;
                    }

                    String output;
                    try {
                        int k = Integer.parseInt(input);
                        long fib = Calculator.fib(k);
                        output = String.valueOf(fib);
                    } catch (NumberFormatException e) {
                        output = "Error format: " + input;
                    }

                    out.write(output + "\n");
                    out.flush();
                }
            }
        }
    }
}
```
I've added two cycles (1) and (2). (1) - this cycle handles new connections, (2) - handles new messages from clients. Also (4) - special command for close communication. (3) - timeout to wait for an incoming message, without it the server will hang if the client disconnects and does not send "exit".

### What's wrong again?

Uh, we've written a lot of code! But servers can work with several clients. But our code handle only one client at a time. Fix it, fast!!!
```java
public void run() throws IOException {
    try (ServerSocket serverSocket = new ServerSocket(port)) {
        while (true) {
            Socket socket = serverSocket.accept();
            socket.setSoTimeout(5000);
            new Thread(() -> handleSocket(socket)).start();
        }
    }
}

private void handleSocket(Socket socket) {
    try (socket;
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()))) {

        while (true) {
            String input;
            try {
                input = in.readLine();
            } catch (SocketTimeoutException e) {
                System.out.println("Timeout!!!");
                break;
            }

            if (input.equals("exit")) {
                out.write("good bye!\n");
                out.flush();
                break;
            }

            String output;
            try {
                int k = Integer.parseInt(input);
                long fib = Calculator.fib(k);
                output = String.valueOf(fib);
            } catch (NumberFormatException e) {
                output = "Error format: " + input + "\n";
            }

            out.write(output + "\n");
            out.flush();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

What is change? Not a lot. We give a new connection and handle it in another thread.
And that's all. You can test the server using the next client:
```java
public class Client {
    public static void main(String[] args) throws IOException {
        try (Scanner scanner = new Scanner(System.in)) {
            try (Socket socket = new Socket("localhost", 5000);
                 BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                 BufferedWriter out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()))) {
                     socket.setSoTimeout(TIMEOUT);
                while (true) {
                    System.out.println("Print your message:");
                    final String message = scanner.nextLine();

                    out.write(message + '\n');
                    out.flush();

                    final String s = in.readLine();
                    System.out.println("Answer: " + s);

                    if (message.equals("exit")) {
                        break;
                    }
                }
            }
        }
    }
}
```

The client is easier than the server. We create a socket, input and output streams, send and read messages.

Most servers have similar logic. But what if we want to handle a lot of clients? Hundreds of thousands! We create a lot of threads, or if we use a thread pool all threads will be busy. We will solve this problem in the next article.

All code is here: [https://github.com/Dmitriev-Andrey/sockets](https://github.com/Dmitriev-Andrey/sockets)
