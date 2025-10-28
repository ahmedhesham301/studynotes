# Concepts
## Layer 4 vs Layer 7 Proxying
### Layer 4 (Transport Layer)
1. Operates at the **TCP/IP** level — Nginx only sees network information, not application data. 
    Accessible details include:
    - Source IP and Port
    - Destination IP and Port
    - Basic packet info (e.g., SYN, TLS handshake)
2. Used when Nginx **does not understand** or need to interpret the application protocol (e.g., **MySQL**, **SMTP**, **Redis**).
3. Once a **TCP connection** is established with a backend, **all data** within that session is routed to the same server.
4. Nginx has **no awareness of HTTP, HTTPS, or application content** — it simply proxies raw TCP streams.    
5. Ideal for:
    - Database connections
    - HTTPS passthrough
    - Any generic TCP/UDP service
### Layer 7 (Application Layer)
1. Operates at the **application level** — Nginx understands protocols such as **HTTP**, **HTTPS**, and **gRPC**.
2. Provides access to rich **contextual data**, such as:
    - HTTP headers, cookies, and methods
    - Requested URLs and endpoints
    - Client information (user agents, referrers, etc.)
3. Requires **decryption** (e.g., terminating TLS) to inspect and act on requests.
4. Enables **advanced features** like:
    - Intelligent request routing (by path, header, or cookie)
    - Caching and compression
    - Connection sharing across multiple backend servers
5. Commonly used for:
    - Web servers
    - API gateways
    - Reverse proxies

## Socket
- A **socket** is the fundamental abstraction for **network communication**.
- It represents an **endpoint** of a two-way connection between two programs over a network.
## Listener
- Opens a **socket** and listens for new TCP connections.
- Does **not** process data — only monitors for incoming handshakes.
## Acceptor
- **Accepts** pending connections from the Listener queue using.
- Returns a **new socket** for each client connection.
- That new socket is then passed to a **Reader** or **worker**.

## Reader (Handler / Worker)
- **Reads** and **writes** data using the connected socket.
- Parses protocol-level messages (e.g., HTTP, FTP, etc.).

## Timeouts
### Frontend Timeouts

408 -> Request Timeout

| type                  | discription                                                                                                                                                                                                                                                                                                                                                                                  |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| client_header_timeout | Defines a timeout for reading client request header. If a client does not transmit the entire header within this time, the request is terminated with the 408 (Request Time-out) error. Default 60s                                                                                                                                                                                          |
| client_body_timeout   | Defines a timeout for reading client request body. The timeout is set only for a period between two successive read operations, not for the transmission of the whole request body. If a client does not transmit anything within this time, the request is terminated with the 408 (Request Time-out) error. Default 60s                                                                    |
| send_timeout          | Sets a timeout for transmitting a response to the client. The timeout is set only between two successive write operations, not for the transmission of the whole response. If the client does not receive anything within this time, the connection is closed. (Default 60s)                                                                                                                 |
| keepalive_timeout     | The first parameter sets a timeout during which a keep-alive client connection will stay open on the server side. The zero value disables keep-alive client connections.The optional second parameter sets a value in the “Keep-Alive: timeout=time” response header field. Two parameters may differ. (default 75 seconds)                                                                  |
| lingering_timeout     | When lingering_close is in effect, this directive specifies the maximum waiting time for more client data to arrive. If data are not received during this time, the connection is closed. Otherwise, the data are read and ignored, and nginx starts waiting for more data again. The “wait-read-ignore” cycle is repeated, but no longer<br>than specified by the lingering_time directive. |
| resolver_timeout      | Sets a timeout for name resolution(DNS), 30 seconds                                                                                                                                                                                                                                                                                                                                          |

### Backend Timeouts

| type                        | discription                                                                                                                                                                                                                                                                    |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| proxy_connect_timeout       | Defines a timeout for establishing a connection with a proxied server. It should be noted that this timeout cannot usually exceed 75 seconds.                                                                                                                                  |
| proxy_send_timeout          | Sets a timeout for transmitting a request to the proxied server. The timeout is set only between two successive write operations, not for the transmission of the whole request. If the proxied server does not receive anything within this time, the connection is closed.   |
| proxy_read_timeout          | Defines a timeout for reading a response from the proxied server. The timeout is set only between two successive read operations, not for the transmission of the whole response. If the proxied server does not transmit anything within this time, the connection is closed. |
| proxy_next_upstream_timeout | Limits the time during which a request can be passed to the next server. The 0 value turns off this limitation. Default 0                                                                                                                                                      |
| Keepalive_timeout           | Sets a timeout during which an idle keepalive connection to an upstream server will stay open.                                                                                                                                                                                 |

## Architecture
### Single Threaded Architecture
- The **simplest model** — one thread handles everything:
	- Listening for connections
	- Accepting them
	- Reading and writing data
- Only one thread handles **all** I/O operations.
- Easy to implement, but **scales poorly** — it can handle only one operation at a time.
- Suitable for low traffic or embedded systems.
 Drawbacks:
1. If one connection blocks (e.g., waiting for data), **all others are delayed**.
2. Inefficient on multi-core systems.

### Multiple Threads, Single Acceptor Architecture
- A single **Listener + Acceptor thread** accepts all incoming connections.
- Each accepted connection is handed off to a **pool of worker threads**.
- The acceptor thread becomes a **central dispatcher**.
- Workers can run in parallel → better CPU utilization.
Drawbacks:
1. The single Acceptor thread can become a **bottleneck** under heavy connection load.

### Multiple Threads, Multiple Acceptors Architecture
- Each thread has its **own accept loop** (each calls `accept()` on the same listening socket).
- The OS handles synchronization of `accept()` calls internally.
- Eliminates the single acceptor bottleneck.
- Each thread can both **accept** and **handle** connections independently.
- Requires careful tuning to avoid race conditions or unfair load distribution.

### Multiple Threads with Message-Based Load Balancing Architecture
- The **Acceptor thread** accepts connections.
- Instead of directly handing off sockets, it sends **messages** (or file descriptors) to worker threads through a **message queue or channel**.
- - The acceptor and workers communicate asynchronously.
- Load balancing is done **at the message level** — the acceptor can choose which worker to assign next.
Drawbacks:
1. Slightly more complex implementation.
2. Message passing overhead (e.g., via IPC or queues).

### Multiple Threads with Socket Sharding (SO_REUSEPORT)
- Each thread creates its **own listening socket** bound to the same port using the Linux option `SO_REUSEPORT`.
- The OS **load balances new connections** across all listening sockets automatically.
- Each thread runs independently — no need for shared accept queues.
- OS handles **fair distribution** of incoming connections (based on CPU or hash).
# config file
## Using Nginx as a layer 7 proxy
```nginx
# The 'http' context defines all HTTP-related configuration(layer 7).
# It can contain server blocks, upstream blocks, and global HTTP settings.
http {
	# 'upstream' defines a group of backend servers for load balancing.
	# Nginx will forward client requests to these backend servers.
	upstream allabckend {
		#ip_hash; to enable sticky sections
		server nodeapp1:9999;
		server nodeapp2:9999;
		server nodeapp3:9999;
		server nodeapp4:9999;
	}

	upstream app1backend {
		server nodeapp1:9999;
		server nodeapp2:9999;
	}

	upstream app2backend {
		server nodeapp3:9999;
		server nodeapp4:9999;
	}
	# ===== Server block =====
	# Defines a virtual host — handles client requests on a specific port or IP.
	server {
		listen 80;
		location / {
			proxy_pass http://allabckend/;
		}
		location /app1 {
			proxy_pass http://app1backend;
		}
		location /app2 {
			proxy_pass http://app2backend;
		}
		location /admin {
			return 403;
		}
	}
}

events {

}
```

## Using Nginx as a layer 4 proxy
```nginx

```