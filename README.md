# High-Performance Multithreaded HTTP Proxy Server with LRU Caching

A custom, low-level network proxy server implemented from scratch in C to handle concurrent HTTP requests. This systems-level project demonstrates core competencies in POSIX multi-threading, concurrency control mechanisms, TCP/IP socket programming, and custom memory-bounded caching algorithms.

---

## 🏗️ System Architecture & Workflow

The proxy server acts as an intermediary system-level service between a local client (e.g., a web browser) and the target remote server.

1. **Connection Listener:** The main execution thread initializes a network socket, binds to a specified port, and actively listens for incoming TCP client connection requests.
2. **Worker Thread Dispatch:** Upon establishing a client connection, the main thread hands off the client's file descriptor to a dedicated worker thread via POSIX threads (`pthreads`).
3. **Request Parsing:** The worker thread dynamically parses incoming HTTP request strings, extracting host names, paths, and HTTP protocol variants.
4. **Cache Lookup (LRU):** The proxy checks a shared, size-bounded internal memory cache using a Least Recently Used (LRU) eviction policy.
   - **Cache Hit:** The requested payload is directly fetched from internal memory and streamed back to the client, bypassing network latency.
   - **Cache Miss:** The proxy opens a new TCP connection to the target remote server, downloads the requested data, updates the shared cache (potentially triggering LRU eviction), and streams the response back to the client.

---

## ⚡ Core Systems Programming Components

### 1. Concurrency Control & Synchronization
* **Thread-Safe Shared Memory:** To protect the shared LRU cache memory resource from multi-threaded race conditions during insertions and lookups, strict serialization policies are enforced using standard synchronization primitives.
* **Optimized Semaphores:** Implemented explicit semaphore constructs (`sem_wait` and `sem_post`) over traditional conditional variable workflows to minimize thread synchronization overhead during resource state transitions.

### 2. Low-Level Network Socket Programming
* **API Integration:** Leveraged structural Unix socket APIs (`socket()`, `bind()`, `listen()`, `accept()`, `connect()`) to build custom system communication channels.
* **Packet Streaming:** Handled non-blocking data streams using byte-level buffer manipulation routines, optimizing network throughput across transport layers.

### 3. In-Memory Cache Optimization (LRU)
* **Data Structure:** Implemented using a linked list structure encapsulating fixed-size memory blocks to control the heap footprint.
* **Deterministic Eviction:** Employs an exact O(N) lookup and tracking mechanism that moves recently accessed nodes to the head and purges the least-frequently accessed nodes from the tail when cache limits are reached.

---

## 🛠️ Compilation & Execution

### Prerequisites
* **Operating System:** Linux / UNIX-based environment (Required for POSIX thread and socket APIs).
* **Tools:** GCC Compiler and GNU Make build automation utilities.

### Steps to Run
1. Clone the repository into your local systems environment:
   ```bash
   git clone https://github.com
   cd MultiThreaded-HTTP-Proxy-Server
   ```

2. Compile the source code using the provided automated project Makefile:
   ```bash
   make all
   ```

3. Initialize the proxy server by specifying a target execution port number:
   ```bash
   ./proxy <port_number>
   ```
   *Example:* `./proxy 8080`

4. Validate proxy routing performance via terminal or client routing utilities:
   ```bash
   curl -x http://localhost:8080 http://example.com
   ```

---

## 🚀 Future Roadmap & Scalability Extensions
* **Parallel Performance:** Transition from a purely multithreaded runtime to a hybrid multi-processing architecture leveraging distinct Linux process spaces to achieve true hardware parallelism.
* **Expanded Method Protocols:** Extend the custom parsing engine to natively support stateless `POST`, `PUT`, and `DELETE` HTTP verbs alongside standard `GET` requests.
* **Network Security Layer:** Integrate an SSL/TLS layer to support safe upstream HTTPS tunneling and encrypted traffic inspection.
