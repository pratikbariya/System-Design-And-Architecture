# TCP vs UDP, Ports, and Sockets
## The building blocks of network communication

These concepts help applications communicate over a network, whether they are web APIs, databases, or microservices.

## Key takeaways

- **TCP and UDP** describe how data is transported.
- **Ports** help direct incoming traffic to the right service.
- A **socket** is a software communication endpoint used by an application to send or receive data.

---

## 1. How they fit together

Think of your computer as a building, and applications as offices inside it.

1. **IP address** — identifies the destination device or network interface.
2. **TCP or UDP** — transports data between endpoints.
3. **Port number** — identifies the transport endpoint for a service.
4. **Socket** — the software endpoint an application uses to communicate.

Example services on one computer:

- Web API — port `8000`
- PostgreSQL database — port `5432`

An IP address can host multiple services on different ports.

---

## 2. TCP vs UDP

**TCP (Transmission Control Protocol)** provides reliable, ordered delivery as a byte stream. **UDP (User Datagram Protocol)** sends individual datagrams without built-in guarantees of delivery or ordering.

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Delivery | Acknowledges data and retransmits lost data | No built-in delivery guarantee |
| Ordering | Preserves byte order | Datagrams may arrive out of order |
| Acknowledgments | Built into the protocol | No built-in acknowledgments |
| Overhead | Generally higher | Generally lower |
| Data format | Byte stream | Individual datagrams |
| Common uses | HTTP APIs, SSH, PostgreSQL | DNS, real-time media, gaming |

### When to use TCP

Choose TCP when reliable, ordered delivery matters:

- REST APIs and web applications
- File uploads and downloads
- Git and SSH
- Database queries
- Message-broker connections

### When to use UDP

UDP is useful when datagram-based communication and low latency are important:

- DNS queries
- Online gaming
- Real-time voice and video

UDP is not automatically faster in every situation. Applications can add reliability on top of UDP; QUIC, for example, uses UDP to support reliable HTTP/3 connections.

---

## 3. What are ports?

A **port** is a 16-bit number used by TCP or UDP to direct traffic to the appropriate transport endpoint. Port numbers range from `0` to `65535`.

| Port | Common service | Typical protocol |
|---:|---|---|
| `22` | SSH | TCP |
| `53` | DNS | UDP and TCP |
| `80` | HTTP | TCP |
| `443` | HTTPS | TCP; UDP for HTTP/3 |
| `5432` | PostgreSQL | TCP |
| `6379` | Redis | TCP |
| `8000` | FastAPI development server | TCP |
| `5672` | RabbitMQ AMQP | TCP |

These are common defaults, not mandatory assignments.

### Example: accessing a FastAPI application

```text
http://localhost:8000
```

- `localhost` refers to your own computer.
- `8000` is the port where the application is listening.
- HTTP defines the application-level communication.
- TCP transports the HTTP data.

If the server is moved to port `8001`, clients must connect to that port instead.

---

## 4. What is a socket?

A **socket** is a software communication endpoint used by an application to communicate over a network.

A network connection is commonly identified by a five-part tuple:

1. Source IP address
2. Source port
3. Destination IP address
4. Destination port
5. Transport protocol (`TCP` or `UDP`)

Example TCP connection:

```text
Client: 192.168.1.10:51000  --->  Server: 192.168.1.20:8000
        temporary source port          listening application port
```

The client and server each have a socket endpoint. A TCP server can accept many simultaneous connections on port `8000`, distinguished by their connection tuples.

### Three related terms

| Term | Meaning |
|---|---|
| IP address | Identifies a network interface or host destination |
| Port | Identifies a transport-layer endpoint |
| Socket | Software endpoint used for communication |

Strictly speaking, a socket is an operating-system abstraction; it is not just another name for an IP address and port.

---

## 5. See how it works in Python

This simple example uses TCP. Run the server and client in separate terminal windows.

### TCP server — `server.py`

```python
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("127.0.0.1", 8000))
server.listen()

print("Server listening on port 8000")

conn, address = server.accept()
with conn:
    print("Connected by:", address)
    data = conn.recv(1024)
    print("Received:", data.decode())
    conn.sendall(b"Hello from server!")

server.close()
```

### TCP client — `client.py`

```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 8000))

client.sendall(b"Hello from client!")
response = client.recv(1024)

print(response.decode())
client.close()
```

### The flow

1. The server creates a socket and binds it to port `8000`.
2. The server listens for connections.
3. The client connects to the server's IP address and port.
4. Both sides exchange data.
5. The connection closes.

`SOCK_STREAM` specifies a TCP socket. For UDP, Python uses `SOCK_DGRAM`, with `sendto()` and `recvfrom()` for datagrams.

---

## 6. How this applies to backend development

A typical backend architecture might look like this:

```text
Frontend / Browser
        |
        | HTTP request over TCP
        v
FastAPI backend
Port 8000
        |
        +-------------------+
        |                   |
        v                   v
PostgreSQL              RabbitMQ
Port 5432               Port 5672
TCP                     TCP
```

The frontend sends an HTTP request to the API. The API can then communicate with PostgreSQL and RabbitMQ using their respective service ports.

### Common connection issues

- **Connection refused:** Nothing is accepting connections at that address and port, or the connection was actively rejected.
- **Connection timed out:** The destination may be unreachable, or traffic may be filtered.
- **Address already in use:** Another process may already be bound to the requested local address and port.
- **Application not reachable from Docker:** The service may be listening only on `127.0.0.1` inside its container instead of `0.0.0.0`.

### Useful Linux commands

Check listening ports and processes:

```bash
sudo ss -lntup
```

Check whether a TCP port is reachable:

```bash
nc -vz localhost 8000
```

Check which process is using port `8000`:

```bash
sudo lsof -i :8000
```

---

## In short

> An **IP address** identifies the destination, **TCP or UDP** determines how data is transported, a **port** directs data to a transport endpoint, and a **socket** is the software endpoint an application uses to communicate.

These concepts work together, but they are not interchangeable.
