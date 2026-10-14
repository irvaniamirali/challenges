# NET-003: From Domain Name to IP Address

**Level:** L1 — Foundation  
**Category:** Networking  
**Estimated time:** 25–35 minutes  
**Environment:** Linux  
**External services:** A working DNS resolver and internet access are required

## Overview

When you visit a website, you usually enter a domain name instead of an IP address.

But how does your computer find the IP address associated with that name?

DNS, or the Domain Name System, helps applications discover information associated with domain names, including IP addresses.

In this lab, you'll query DNS records, inspect the answers, compare IPv4 and IPv6 records, and investigate what happens when a domain name doesn't exist.

You'll also learn an important distinction: resolving a domain name and connecting to the service at that address are two separate operations.

## Learning objectives

By the end of this lab, you should be able to:

- Explain the purpose of DNS.
- Understand the role of a DNS resolver.
- Query DNS records using `dig`.
- Distinguish between `A` and `AAAA` records.
- Interpret DNS response statuses, including `NOERROR` and `NXDOMAIN`.
- Explain why a successful DNS lookup doesn't guarantee a successful network connection.

## Prerequisites

You'll need:

- A Linux machine.
- Access to a working DNS resolver.
- The `dig` command.

If `dig` isn't installed, install the DNS utilities package provided by your Linux distribution. For example, on Debian or Ubuntu:

```bash
sudo apt install dnsutils
```

Here, `sudo` runs the command with elevated privileges, while `apt install` asks the APT package manager to install the specified package.

If your distribution doesn't provide `dnsutils`, check its package manager for the package that supplies `dig`.

**Note:** These experiments use public domain names and your configured DNS resolver. Results may vary depending on your network, resolver, caching, and DNS filtering policies.

## 1. Understand what DNS does

Before running any commands, let's establish a few important concepts.

### Domain name

A domain name is a human-readable name used to identify a location or resource in the DNS namespace.

For example:

`example.com`

Applications can use domain names instead of requiring you to remember IP addresses.

### IP address

An IP address identifies a network endpoint or interface within the IP networking system.

For example, an IPv4 address looks like this:

`192.0.2.10`

This is a documentation address, not an address you should expect to receive from a real DNS lookup.

### DNS resolver

A DNS resolver is the component that helps obtain DNS answers for queries.

Your Linux system normally uses a configured resolver. Depending on the system's configuration, that resolver might be a local DNS service, a router, or a DNS server elsewhere on the network.

A recursive resolver can contact other DNS servers when it needs to find an answer that isn't already available in its cache.

You don't need to understand the entire DNS hierarchy yet. For now, remember that your query usually goes to a configured resolver, which provides an answer or an error.

### DNS record

DNS stores different types of information in records.

For this lab, we'll focus on two types:

- **A:** Maps a domain name to an IPv4 address.
- **AAAA:** Maps a domain name to an IPv6 address.

Let's observe these records instead of memorizing their definitions.

## 2. Make your first DNS query

Open a terminal and run:

```bash
dig example.com A
```

Let's break down the command:

- `dig` is a command-line utility for querying DNS.
- `example.com` is the domain name we're querying.
- `A` specifies the type of record we're requesting: an IPv4 address record.

We haven't specified a DNS server, so `dig` uses the resolver configuration available to it on your system.

Your output should contain sections resembling these:

```text
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: ...
;; QUESTION SECTION:
;example.com.                    IN      A

;; ANSWER SECTION:
example.com.             ...     IN      A       93.184.216.34

;; Query time: ... msec
;; SERVER: ...#53(...)
```

The IP address above is an illustrative example. The actual answer may contain different addresses or multiple records.

Let's understand the important parts.

### Status

The header contains a response status.

`NOERROR` means the DNS response did not report a DNS error. It doesn't necessarily mean that the requested record exists; we'll examine that distinction later.

### Question section

This section shows what was requested.

In this example, we're asking for an `A` record for `example.com`.

### Answer section

This section contains the records returned in the answer.

An answer might include an IPv4 address, a time-to-live value, and other record information.

The **TTL**, or Time to Live, tells DNS caches how long a record can be cached before it should expire. It's expressed in seconds.

The exact TTL and answer records can change over time.

### Query time

This indicates how long `dig` reports the query took. It isn't necessarily the time required for a complete website connection.

### Server

This identifies the DNS server that `dig` reports using for the query. Port `53` is the conventional DNS port.

The server shown here is the resolver that answered your query, not necessarily the authoritative DNS server for `example.com`.

## 3. Query an IPv6 record

Now run:

```bash
dig example.com AAAA
```

The `AAAA` argument requests an IPv6 address record instead of an IPv4 address record.

An IPv6 address looks different from an IPv4 address. For example:

`2001:db8::10`

This is another documentation address, not an expected answer from the live query.

Compare the two commands:

| Query type | Information requested |
|---|---|
| `A` | IPv4 address |
| `AAAA` | IPv6 address |

A domain can have multiple records of either type. It can also have one type without the other.

If the `AAAA` query returns no address, that doesn't automatically mean DNS is broken. The domain may simply have no matching IPv6 address record.

This is an important detail: **a valid DNS response can contain no record of the requested type.**

## 4. Understand DNS response statuses

So far, we've queried a domain that should exist. What happens when the name doesn't exist?

Run:

```bash
dig this-domain-should-not-exist.invalid A
```

The `.invalid` top-level domain is reserved for invalid domain names. It's used here to create a predictable negative DNS test.

Look at the response header. Under normal DNS behavior, you should see a status resembling:

```text
status: NXDOMAIN
```

### What does `NXDOMAIN` mean?

`NXDOMAIN` means the queried domain name does not exist according to the DNS response.

It is different from receiving no response at all.

Let's compare some common outcomes.

| Result | Meaning |
|---|---|
| `NOERROR` with an answer | The query succeeded and matching records were returned. |
| `NOERROR` without an answer | The name exists, but no matching record was returned for this query. |
| `NXDOMAIN` | The queried name does not exist according to the DNS response. |
| `SERVFAIL` | The server couldn't successfully complete the DNS resolution. |
| Timeout | No usable response arrived within the waiting period. This alone doesn't identify the cause. |

These distinctions matter when troubleshooting.

For example, if an application reports that it cannot resolve a hostname, the next step isn't automatically to change the server's IP address. First, determine what DNS actually returned.

**Environment note:** Some networks intercept, filter, or synthesize DNS responses. If your result differs from the expected response, record the actual status and investigate your resolver's behavior instead of assuming the command itself is wrong.

## 5. Learn one useful option: `+short`

The previous commands printed several sections of output. Sometimes you only want the returned address.

Run:

```bash
dig example.com A +short
```

The `+short` option tells `dig` to use a shortened output format.

If the query returns IPv4 records, you'll usually see only the returned addresses, one per line.

This is convenient for scripts, but it hides useful information such as the response status and the DNS server used.

For learning and troubleshooting, start with the full output. Use `+short` when you specifically need a compact result.

## 6. DNS resolution is not a connection

Let's connect this lesson to the previous two labs.

When you query DNS, you're asking for information about a domain name.

When you establish a TCP connection, you're attempting to communicate with a particular network endpoint.

When you send an HTTP request, you're asking an application server to perform an operation.

These are separate steps.

Consider this simplified sequence:

1. An application needs to reach `example.com`.
2. It obtains an IP address through DNS resolution.
3. It attempts to establish a connection to an IP address and port.
4. If the connection succeeds, it can send an HTTP request.
5. The server returns an HTTP response, if the application exchange succeeds.

A failure at one step doesn't automatically tell you what happened at another.

For example:

- DNS can return an address even when the destination service is unavailable.
- A TCP connection can fail even when DNS resolution succeeds.
- An HTTP request can receive a `404` even when DNS and TCP both work.

This is why good troubleshooting starts by identifying the stage that failed.

## 7. Check your understanding

Try answering these questions in your own words.

1. What is the difference between a domain name and an IP address?
2. What does the `A` record type request? What about `AAAA`?
3. Why can a DNS response have `NOERROR` but no answer records?
4. What is the difference between `NXDOMAIN` and a timeout?
5. If `dig` returns an IP address but `curl` cannot connect to the service, does that automatically mean DNS is broken?
6. Why is the full output of `dig` often more useful for troubleshooting than `+short`?

## Cleanup

There is no running service to stop and no configuration to restore.

All commands in this lab perform DNS queries and display their results.

## Key takeaway

DNS helps applications discover information associated with domain names, but it doesn't guarantee that a service is reachable or working.

When troubleshooting, distinguish between a DNS resolution problem, a TCP connection problem, and an application-level error. Each requires different evidence and potentially a different solution.

## Further reading

- [BIND 9 `dig` manual](https://bind9.readthedocs.io/en/latest/manpages.html#dig-dns-lookup-utility)
- [RFC 1034 — Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034)
- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 3596 — DNS Extensions to Support IP Version 6](https://www.rfc-editor.org/rfc/rfc3596)
