# NET-001: What Actually Happens When You Connect to a Server?

**Level:** L1 — Foundation  
**Category:** Networking  
**Estimated time:** 25–35 minutes  
**Environment:** Linux  
**External services:** Not required

## Overview

When you open a website, run `curl`, or connect an application to a server, several things happen before you receive a response.

But what does a connection actually look like?

In this lab, you'll run a small HTTP server on your own machine, capture its network traffic, and observe how TCP establishes a connection before HTTP data is exchanged.

No guessing. No intentionally broken infrastructure. First, understand the system; then observe it in action.

## Learning objectives

By the end of this lab, you should be able to:

- Explain the relationship between TCP and HTTP.
- Distinguish a listening socket from an established connection.
- Recognize the TCP three-way handshake in a packet capture.
- Identify the HTTP request inside captured traffic.
- Explain what a network port represents in this experiment.

## Prerequisites

You'll need:

- A Linux machine.
- Python 3.
- `curl`.
- `tcpdump`.
- Permission to capture packets, typically through `sudo`.

If `tcpdump` isn't installed, install it using your distribution's package manager.

## 1. Understand the moving parts

Before running anything, let's establish what we're about to observe.

**The client** initiates a connection. In this lab, `curl` is our client.

**The server** listens for incoming connections. We'll use Python's built-in HTTP server.

**The IP address** identifies the network destination. We'll use `127.0.0.1`, the IPv4 loopback address, which refers to the local machine.

**The port** identifies a particular transport endpoint on that machine. Our server will listen on port `8080`.

**TCP** provides a reliable, ordered byte stream between endpoints. Before HTTP data can be exchanged over TCP, the TCP connection must be established.

**HTTP** is the application protocol we'll use to exchange a request and response over that connection.

The relationship is straightforward:

`curl → TCP connection → HTTP request → HTTP response`

This is a simplified view of the flow, not a complete description of every networking layer.

## 2. Start capturing network traffic

Open a terminal and run:

```bash
sudo tcpdump -i lo -nn -A 'tcp port 8080'
```

Let's break down the command:

| Option            | Meaning                                                                         |
| ----------------- | ------------------------------------------------------------------------------- |
| `sudo`            | Runs the command with elevated privileges, usually needed for packet capture.   |
| `-i lo`           | Captures traffic on `lo`, Linux's loopback network interface.                   |
| `-nn`             | Prevents resolving IP addresses to hostnames and port numbers to service names. |
| `-A`              | Displays packet payloads as printable ASCII where possible.                     |
| `'tcp port 8080'` | Captures TCP packets whose source or destination port is `8080`.                |

The final expression is a capture filter. It limits the output to traffic relevant to our experiment.

Leave this terminal running.

**Security note:** `-A` can expose readable application data. Only capture traffic you intend to inspect. This lab uses a local test server and no sensitive information.

## 3. Start the HTTP server

Open a second terminal and run:

```bash
python3 -m http.server 8080 --bind 127.0.0.1
```

Here's what the important parts mean:

- `python3` runs Python 3.
- `-m http.server` runs Python's built-in `http.server` module as a program.
- `8080` specifies the port the server listens on.
- `--bind 127.0.0.1` restricts the server to the local loopback address.

The server should report that it is serving HTTP on port `8080`.

Leave it running.

At this point, the server is listening, but we haven't made an HTTP request yet.

## 4. Make a request

Open a third terminal and run:

```bash
curl http://127.0.0.1:8080/
```

You should receive an HTML directory listing.

Now look at the first terminal. `tcpdump` should have captured packets exchanged between the client and server.

Stop the capture with `Ctrl+C` when you're ready to inspect the output.

## 5. Read the packet capture

The exact output depends on your system and tool versions, but look for these TCP flags.

### The three-way handshake

A new TCP connection is normally established through three messages:

1. **SYN** — the client requests to establish a connection.
2. **SYN-ACK** — the server acknowledges the request and sends its own synchronization message.
3. **ACK** — the client acknowledges the server's response.

In `tcpdump`, these flags commonly appear as:

| Flag    | Typical representation |
| ------- | ---------------------- |
| SYN     | `[S]`                  |
| SYN-ACK | `[S.]`                 |
| ACK     | `[.]`                  |

These are examples of how `tcpdump` represents TCP flags, not complete packet output.

You should see the handshake before the HTTP request data for a newly established connection.

### The HTTP request

Because we used `-A`, the captured payload may contain readable text resembling:

```http
GET / HTTP/1.1
Host: 127.0.0.1:8080
User-Agent: curl/...
Accept: */*
```

The exact headers and HTTP version can vary.

This is the application-level request. TCP transports its bytes; TCP does not itself understand what `GET /` means.

You may also see the server's HTTP response, followed by packets involved in closing the connection.

## 6. Connect the observations

Let's separate three things that are easy to confuse.

**Listening:** The server has a socket waiting for incoming connections.

**Established connection:** A client and server have completed TCP connection establishment and can exchange data.

**HTTP request:** The client sends application-level data over the established connection.

A listening socket does not mean that a client is already connected. Likewise, a successful TCP connection does not automatically mean that the application request will succeed.

TCP can establish a connection even if the application later returns an HTTP error.

## 7. Check your understanding

Try answering these questions in your own words:

1. Why do we see TCP handshake packets before the HTTP request?
2. What does port `8080` identify in this experiment?
3. Why can `tcpdump` display `GET / HTTP/1.1` when we use `-A`?
4. What is the difference between a server listening on a port and an established TCP connection?
5. If a TCP connection succeeds but the server returns HTTP `404`, did TCP necessarily fail?

## Cleanup

Stop the HTTP server with `Ctrl+C` in the second terminal.

There is no infrastructure to tear down and no configuration to restore.

## Further reading

- [Python ](https://docs.python.org/3/library/http.server.html)[`http.server`](https://docs.python.org/3/library/http.server.html)
- [curl documentation](https://curl.se/docs/)
- [tcpdump manual](https://www.tcpdump.org/manpages/tcpdump.1.html)
- [RFC 9293 — Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293)
