# Web Hacking Fundamentals

---

## Overview

Web Hacking Fundamentals covers how the web actually works — from the anatomy of a URL to how HTTP requests and responses are structured, how databases store data, and how tools like Burp Suite are used to intercept and manipulate web traffic. As a former front-end developer, much of the underlying web knowledge here was familiar ground. What changed was the lens — looking at the same technologies through a security perspective rather than a build perspective.

---

## How the Web is Built

Web applications are split into two sides:

**Frontend** — what the user sees and interacts with, built with HTML, CSS, and JavaScript.

**Backend** — the server-side logic, databases, and infrastructure that power the application. This includes protections like Web Application Firewalls (WAF), which filter and monitor HTTP traffic to block malicious requests before they reach the application.

---

## Anatomy of a URL

As a front-end developer, URLs were a daily constant — but this room reframed them as an attack surface. Every component of a URL has both a function and a potential vulnerability.

| Component | Description | Security Consideration |
| --------- | ----------- | ---------------------- |
| **Scheme** | The protocol — `http://` or `https://` | HTTPS encrypts the connection; HTTP does not |
| **User** | Login credentials embedded in the URL (rare) | Exposes sensitive info in plaintext — avoid |
| **Host / Domain** | The website being accessed | Watch for **typosquatting** — fake domains that mimic real ones, used in phishing |
| **Port** | Directs traffic to the right service (80 for HTTP, 443 for HTTPS) | Unexpected open ports can indicate misconfigurations |
| **Path** | Points to a specific resource on the server | Must be validated and sanitised to prevent unauthorised access |
| **Query String** | Starts with `?` — passes parameters like search terms | User-modifiable; vulnerable to **injection attacks** if not sanitised |
| **Fragment** | Starts with `#` — jumps to a section of the page | Also user-modifiable; handle carefully to prevent injection |

> **Reframe from front-end experience:** Query strings and fragments were just handy tools for passing data and navigating pages. Seeing them as user-controlled inputs that can carry malicious payloads was a genuine shift in perspective.

---

## HTTP Messages

HTTP messages are how the client (browser) and server communicate. Every request and response follows the same structure:

| Part | Description |
| ---- | ----------- |
| **Start Line** | Identifies the message type — method, path, and HTTP version for requests; status code for responses |
| **Headers** | Key-value pairs providing metadata — content type, cookies, security policies, and more |
| **Empty Line** | Separates headers from the body — required for correct parsing |
| **Body** | The actual content — form data, JSON, HTML, images, etc. |

---

## HTTP Requests

### HTTP Methods

HTTP methods define what action is being performed. From a front-end background, these were used regularly — but the security implications of each were not something that came up during development.

| Method | Purpose | Security Note |
| ------ | ------- | ------------- |
| `GET` | Retrieve data from the server | Avoid passing sensitive data (tokens, passwords) — they appear as plaintext in the URL |
| `POST` | Send data to the server | Validate and sanitise all input to prevent SQL injection and XSS |
| `PUT` | Replace or update a resource | Ensure the user is authorised before allowing changes |
| `DELETE` | Remove a resource | Same as PUT — authorisation is critical |
| `PATCH` | Partially update a resource | For small changes without replacing the whole resource |
| `HEAD` | Like GET, but returns headers only | Used to check metadata without downloading content |
| `OPTIONS` | Lists available methods for a resource | Can expose the attack surface if not restricted |
| `TRACE` | Used for debugging | Often disabled on servers — can be exploited if left on |
| `CONNECT` | Creates a tunnel for secure connections (HTTPS) | Critical for encrypted communication |

### HTTP Versions

| Version | Year | Notable Change |
| ------- | ---- | -------------- |
| HTTP/0.9 | 1991 | GET requests only |
| HTTP/1.0 | 1996 | Added headers, content type support |
| HTTP/1.1 | 1997 | Persistent connections, chunked transfer — still widely used |
| HTTP/2 | 2015 | Multiplexing, header compression, faster performance |
| HTTP/3 | 2022 | Built on QUIC protocol — quicker and more secure |

### Common Request Headers

| Header | Example | Description |
| ------ | ------- | ----------- |
| `Host` | `Host: tryhackme.com` | Specifies the target web server |
| `User-Agent` | `User-Agent: Mozilla/5.0` | Identifies the browser and OS making the request |
| `Referer` | `Referer: https://google.com/` | The page that sent the user here |
| `Cookie` | `Cookie: session=abc123` | Stored data sent back to the server — used for sessions |
| `Content-Type` | `Content-Type: application/json` | Format of the data being sent in the body |

### Request Body Formats

Data sent in `POST` and `PUT` requests can be formatted several ways:

**URL Encoded** — key-value pairs separated by `&`, special characters percent-encoded:
```
name=Aleksandra&age=27&country=US
```

**Form Data (multipart)** — used when sending files alongside text, separated by a boundary string:
```
----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

aleksandra
----WebKitFormBoundary7MA4YWxkTrZu0gW
```

**JSON** — structured key-value pairs, widely used in APIs:
```json
{
    "name": "Aleksandra",
    "age": 27,
    "country": "US"
}
```

**XML** — data wrapped in opening and closing tags:
```xml
<user>
    <name>Aleksandra</name>
</user>
```

---

## HTTP Responses

### Status Codes

| Range | Category | Meaning |
| ----- | -------- | ------- |
| 100–199 | Informational | Server received the request, client should continue |
| 200–299 | Success | Request processed successfully |
| 300–399 | Redirection | Resource has moved — response includes new URL |
| 400–499 | Client Error | Bad request, missing auth, resource not found |
| 500–599 | Server Error | Server failed to handle a valid request |

**Common codes to know:**

| Code | Meaning |
| ---- | ------- |
| `200` | OK — standard success response |
| `301` | Moved Permanently — resource is at a new URL |
| `404` | Not Found — resource doesn't exist or path is wrong |
| `500` | Internal Server Error — something went wrong on the server |

### Common Response Headers

| Header | Example | Description |
| ------ | ------- | ----------- |
| `Date` | `Fri, 23 Aug 2024 10:43:21 GMT` | When the response was generated |
| `Content-Type` | `Content-Type: text/html; charset=utf-8` | Format of the response body |
| `Server` | `Server: nginx` | Server software — often hidden to prevent footprinting |
| `Set-Cookie` | `Set-Cookie: sessionId=38af1337` | Stores data on the client; use `HttpOnly` and `Secure` flags |
| `Cache-Control` | `Cache-Control: max-age=600` | How long the client should cache the response |
| `Location` | `Location: /index.html` | Redirect destination — must be sanitised to prevent open redirect attacks |

### Security Headers

These headers are set by the server to enforce protection policies on the browser side:

**Content-Security-Policy (CSP)** — defines which sources are trusted for loading scripts, styles, and other content. Mitigates XSS by blocking unauthorised scripts.
```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tryhackme.com
```

**Strict-Transport-Security (HSTS)** — forces HTTPS-only connections, preventing protocol downgrade attacks.
```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

**X-Content-Type-Options** — prevents browsers from guessing the file type (MIME-sniffing), which can be exploited.
```
X-Content-Type-Options: nosniff
```

**Referrer-Policy** — controls how much URL information is shared when a user navigates to another site.

| Directive | Behaviour |
| --------- | --------- |
| `no-referrer` | No referrer info sent at all |
| `same-origin` | Referrer only sent for same-site links |
| `strict-origin` | Only sends the domain, not the full path |
| `strict-origin-when-cross-origin` | Full URL for same-origin, domain only for cross-origin |

---

## SQL Fundamentals

### Relational vs. Non-Relational Databases

| Type | Structure | Best For |
| ---- | --------- | -------- |
| **Relational** | Rows and columns in structured tables — relationships between tables are possible | Predictable, structured data — e.g., e-commerce orders |
| **Non-Relational** | Flexible, document-based — no fixed schema | Diverse, unpredictable data — e.g., social media content |

### What is SQL?

SQL (Structured Query Language) is used to interact with relational databases through a DBMS (Database Management System) — software like MySQL, MariaDB, or Oracle that sits between the user and the stored data.

### Core SQL Operations (CRUD)

| Operation | SQL Command | Description |
| --------- | ----------- | ----------- |
| Create | `INSERT` | Add new data |
| Read | `SELECT` | Retrieve data |
| Update | `UPDATE` | Modify existing data |
| Delete | `DELETE` | Remove data |

```sql
SHOW DATABASES;
CREATE DATABASE name;
USE name;
DROP DATABASE name;
```

---

## Burp Suite

### What is Burp Suite?

Burp Suite is a Java-based framework built for web application penetration testing. It is the industry standard for testing web apps and APIs. There is a free Community Edition and paid Professional/Enterprise versions.

Coming from a front-end background, the closest comparison is the browser's Inspect tool — but Burp Suite operates at a completely different level. Rather than observing traffic passively, it sits in the middle of the connection as a proxy, giving full control to intercept, modify, replay, and analyse every request and response before it reaches its destination.

### Core Modules

| Module | Description |
| ------ | ----------- |
| **Proxy** | Intercepts and holds requests between the browser and server — the heart of Burp Suite |
| **Repeater** | Captures a request and lets you modify and resend it as many times as needed — useful for testing payloads manually |
| **Intruder** | Automates sending large volumes of requests to an endpoint — used for brute-force and fuzzing (rate-limited in Community Edition) |
| **Decoder** | Encodes or decodes data — useful for working with Base64, URL encoding, and similar formats |
| **Comparer** | Compares two pieces of data at word or byte level — helpful for spotting differences in responses |
| **Sequencer** | Analyses the randomness of tokens like session cookies — weak randomness can expose critical attack vectors |

### Dashboard

| Panel | Description |
| ----- | ----------- |
| **Tasks** | Background tasks Burp runs while you work — Community defaults to Live Passive Crawl |
| **Event Log** | A log of Burp's own actions, including proxy start and connections made |
| **Issue Activity** | Pro only — lists vulnerabilities found by the automated scanner, ranked by severity |
| **Advisory** | Detailed information on identified vulnerabilities, including remediation suggestions |

### Proxy — Key Features

**Intercepting Requests** — requests are held before reaching the server, allowing them to be forwarded, dropped, edited, or sent to another module.

**Capture and Logging** — all requests are logged automatically, even when interception is off, enabling retrospective analysis.

**WebSocket Support** — Burp captures WebSocket traffic in addition to standard HTTP.

**Response Interception** — disabled by default; can be enabled on a per-request basis.

**Match and Replace** — uses regular expressions (regex) to modify incoming or outgoing requests automatically.

### Regular Expressions (Regex)

Regex is a pattern-matching system used within Burp's Match and Replace feature, and widely in security tooling:

| Symbol | Meaning |
| ------ | ------- |
| `^` | Start of a line |
| `$` | End of a line |
| `\d` | Any single digit (0–9) |
| `+` | One or more of the preceding character |
| `[a-z]` | Any lowercase letter |

Common security uses for regex include log analysis (finding all IPs that attempted to log in as `admin`), WAF rules (blocking suspicious characters in input fields), and DLP (detecting PII or credit card numbers in outgoing data).
