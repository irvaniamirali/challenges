# NET-002: TCP Works, but the Request Fails

**Level:** L1 — Foundation  
**Category:** Networking  
**Estimated time:** 20–30 minutes  
**Environment:** Linux  
**External services:** Not required

## Overview

A successful network connection does not guarantee a successful application request.

Imagine opening a website and receiving a `404 Not Found` response. Is the server unreachable? Did the TCP connection fail? Or did the server receive your request but fail to find the requested resource?

In this lab, you'll run a local HTTP server, request an existing resource, and then request one that doesn't exist. By comparing the responses, you'll learn how to distinguish a successful connection from a successful HTTP request.

## Learning objectives

By the end of this lab, you should be able to:

- Distinguish a TCP connection failure from an HTTP error response.
- Explain the meaning of HTTP status codes `200` and `404`.
- Use `curl` to inspect HTTP response headers and status codes.
- Explain why an HTTP error does not necessarily indicate a network problem.
- Identify which layer to investigate based on the observed result.

## Prerequisites

You'll need:

- A Linux machine.
- Python 3.
- `curl`.

No external server, Docker, or additional infrastructure is required.

## 1. Start the HTTP server

Open a terminal and run:

```bash
python3 -m http.server 8080 --bind 127.0.0.1
```

Here's what the command means:

- `python3` runs Python 3.
- `-m http.server` runs Python's built-in HTTP server module.
- `8080` specifies the port on which the server listens.
- `--bind 127.0.0.1` restricts the server to the local loopback address.

Leave this terminal running.

The server now listens for HTTP requests on `127.0.0.1:8080`.

## 2. Request an existing resource

Open a second terminal and run:

```bash
curl -i http://127.0.0.1:8080/
```

The `-i` option tells `curl` to include the HTTP response headers in its output, rather than displaying only the response body.

You should see a response beginning with something similar to:

```http
HTTP/1.0 200 OK
Server: SimpleHTTP/...
Date: ...
Content-type: text/html; charset=utf-8
```

The exact headers and HTTP version may vary.

The response body should contain an HTML directory listing.

### What does `200 OK` tell us?

The server received the request and successfully handled it. In this case, Python's HTTP server generated a directory listing for the root path `/`.

Because we received an HTTP response, we know that the client successfully communicated with the server at the application level in this experiment.

## 3. Request a resource that doesn't exist

Now run:

```bash
curl -i http://127.0.0.1:8080/does-not-exist
```

This time, you should see a response similar to:

```http
HTTP/1.0 404 File not found
Server: SimpleHTTP/...
Date: ...
Connection: close
Content-Type: text/html;charset=utf-8
```

The response body will contain an HTML error page.

The precise response text and headers can vary by Python version.

### What happened?

The client connected to the server and sent an HTTP request. The server then tried to locate the requested resource and returned an HTTP `404` response because it couldn't find it.

The connection worked. The requested resource did not exist.

That's an important distinction.

## 4. Compare the two results

| Observation | Existing resource | Missing resource |
|---|---|---|
| Server reachable | Yes | Yes |
| HTTP response received | Yes | Yes |
| Status code | `200` | `404` |
| Request handled successfully | Yes | No, the resource wasn't found |

Both requests reached the HTTP server. The difference is how the server handled the requested path.

A `404` response is not evidence of a failed TCP connection. In this lab, it is evidence that the server returned an application-level response indicating that the requested resource could not be found.

## 5. Understand where the failure occurs

Consider these three situations:

### Situation A: Connection refused

The client attempts to connect to a port where no service is accepting the connection, and the operating system rejects the attempt.

The client may report an error such as:

```text
curl: (7) Failed to connect ...
```

The exact message depends on the environment and `curl` version.

This points toward a connection-level problem. You would investigate whether a service is listening, whether the destination address and port are correct, and whether a firewall or other network component is interfering.

### Situation B: HTTP 404 Not Found

The client establishes communication with an HTTP server and receives a `404` response.

The connection succeeded, but the server couldn't find the requested resource.

You would investigate the URL path, the resource, and the application's routing or file-serving behavior.

### Situation C: HTTP 500 Internal Server Error

The client receives an HTTP `500` response.

The server was able to return an HTTP response, but it encountered an internal error while handling the request.

You would generally investigate the server application and its logs.

These examples describe common patterns, not absolute rules. A real incident can involve multiple problems at once, so always verify your assumptions with evidence.

## 6. Check your understanding

Try answering these questions in your own words:

1. Why did the second request return `404` instead of a connection error?
2. What does `curl -i` show that a normal `curl` request doesn't show by default?
3. If a client receives an HTTP `404`, does that mean TCP necessarily failed?
4. If a client cannot connect to a server at all, would changing the URL path usually solve the problem?
5. Why should you identify the type of failure before changing a server configuration?

## Cleanup

Stop the HTTP server by pressing `Ctrl+C` in the first terminal.

No configuration changes need to be reverted.

## Key takeaway

**A successful connection and a successful request are two different things.**

When troubleshooting, first establish what actually failed. A connection error, an HTTP `404`, and an HTTP `500` point toward different areas of investigation. Understanding that distinction helps you avoid debugging the wrong layer.

## Further reading

- [curl documentation](https://curl.se/docs/)
- [Python `http.server`](https://docs.python.org/3/library/http.server.html)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
