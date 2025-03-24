

### 1. What is the difference between HTTP status codes, and explain the meaning of each of them?  

HTTP status codes are 3-digit numbers returned by servers to indicate the result of a client's request. They are grouped into 5 classes:  

#### **1xx (Informational) ℹ️**  
- **100 Continue**: The server has received the request headers, and the client should proceed to send the request body.  
- **101 Switching Protocols**: The server agrees to switch protocols (e.g., from HTTP to WebSocket).  
- **102 Processing (WebDAV)**: The server is processing the request but has not yet completed it.  

#### **2xx (Success) ✅**  
- **200 OK**: The request was successful.  
- **201 Created**: The request was fulfilled, and a new resource was created (common after POST/PUT requests).  
- **202 Accepted**: The request was accepted for processing but is not yet complete.  
- **204 No Content**: The request succeeded, but there is no content to return.  

#### **3xx (Redirection) 🔄**  
- **301 Moved Permanently**: The resource has been permanently moved to a new URL.  
- **302 Found (Temporary Redirect)**: The resource is temporarily available at a different URL.  
- **304 Not Modified**: The cached version of the resource is still valid (used for conditional requests).  
- **307 Temporary Redirect**: Similar to 302, but ensures the HTTP method remains unchanged.  

#### **4xx (Client Errors) ❌**  
- **400 Bad Request**: The server cannot process the request due to malformed syntax.  
- **401 Unauthorized**: Authentication is required to access the resource.  
- **403 Forbidden**: The client does not have permission to access the resource.  
- **404 Not Found**: The requested resource does not exist.  
- **429 Too Many Requests**: The client has exceeded rate limits.  

#### **5xx (Server Errors) 🔥**  
- **500 Internal Server Error**: A generic server-side error occurred.  
- **502 Bad Gateway**: The server received an invalid response from an upstream server.  
- **503 Service Unavailable**: The server is temporarily unable to handle the request (e.g., due to overload).  
- **504 Gateway Timeout**: The upstream server did not respond in time.  

---

### 2. What database is used by Prometheus?  

Prometheus uses its custom-built **Time Series Database (TSDB)**, designed specifically for:  
- **High-performance metric ingestion and querying**  
- **Efficient storage** with compression (as low as **1.3 bytes per sample**)  
- **Fast lookups** using labels (key-value pairs)  

---

### 3. What is the difference between different metric types (Counter, Gauge, Histogram)?  

#### **1. Counter 🔢**  
- A **monotonically increasing** value (only goes up or resets to zero on restart).  
- Used to track **cumulative totals** (e.g., requests served, errors occurred).  
- **Example Use Cases**:  
  - Total HTTP requests (`http_requests_total`)  
  - Number of errors (`errors_total`)  
  - Bytes transmitted (`network_bytes_sent_total`)  

#### **2. Gauge 📈📉**  
- A **variable value** that can increase or decrease (e.g., temperature, memory usage).  
- Represents a **current state** rather than a cumulative count.  
- **Example Use Cases**:  
  - CPU usage (`node_cpu_usage`)  
  - Memory consumption (`node_memory_free_bytes`)  
  - Active connections (`nginx_connections_active`)  

#### **3. Histogram ⏱️**  
- Measures **value distributions** (e.g., request durations, response sizes).  
- Tracks **buckets** (predefined ranges) and counts observations in each.  
- **Example Use Cases**:  
  - HTTP request latency (`http_request_duration_seconds`)  
  - Database query time (`db_query_duration_seconds`)  
  - File upload sizes (`file_upload_bytes`)  

