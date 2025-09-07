
### **Remote Procedure Call (RPC)**

A Remote Procedure Call (RPC) is a protocol that one program can use to request a service from a program located on another computer on a network without having to understand the network's details. RPC is a client-server model of communication. The client makes a request to the server, and the server sends a response back to the client.

### **JSON-RPC**

JSON-RPC is a lightweight remote procedure call (RPC) protocol. It is a simple protocol that defines a few data structures and the rules for their processing. It is transport-agnostic in that the concepts can be used within the same process, over sockets, over HTTP, or in many various message passing environments. It uses JSON (JavaScript Object Notation) as its data format.

### **JSON-RPC vs REST APIs**

#### **REST APIs**

- **Resource-Oriented** REST APIs are designed around resources, which are any kind of object, data, or service that can be accessed by the client. 
- **Standard HTTP Methods** They use standard HTTP methods like GET, POST, PUT, DELETE, etc. to perform operations on these resources. 
- **Multiple Data Formats** REST APIs can use various data formats for the message body, such as JSON, XML, or plain text. 
- **Stateless** Each request from a client to a server must contain all of the information needed to understand and complete the request.

#### **JSON-RPC**

- **Action-Oriented** JSON-RPC is focused on actions rather than resources. The client requests the server to perform a specific action (method) with a set of parameters. 
- **Single HTTP Method** Typically, all JSON-RPC requests are made using the POST HTTP method. 
- **Strict JSON Format** It exclusively uses JSON for both the request and response bodies. 
- **Simplicity** The protocol is very simple, with a small set of predefined rules.

### **Key Differences**

| Feature | REST API | JSON-RPC |
| :--- | :--- | :--- |
| **Architectural Style** | Resource-oriented | Action-oriented |
| **Communication** | Uses multiple HTTP endpoints (URLs) to represent different resources | Typically uses a single endpoint for all requests |
| **Data Format** | Flexible (JSON, XML, etc.) | Strictly JSON |
| **Batching Requests** | No built-in support for batching multiple requests into one | Natively supports batching multiple calls in a single request |
| **Notifications**| No inherent concept of notifications | Supports notifications (requests that do not require a response) |

### **Example of a JSON-RPC Server in Python**

Here is a simple example of how to create a JSON-RPC server using Python:

```python
from jsonrpc.server import Method, Server

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

server = Server(
    Method('add', add),
    Method('subtract', subtract)
)

server.serve()
```

### **Client-Side Request**

A client can then send a request to this server to execute the `add` method like so:

```python
import requests
import json

url = "http://localhost:5000"
headers = {'content-type': 'application/json'}

# Example payload for a single request
payload = {
    "method": "add",
    "params": [10, 5],
    "jsonrpc": "2.0",
    "id": 1,
}

response = requests.post(url, data=json.dumps(payload), headers=headers).json()

print(response)

# Example payload for batch requests
payload = [
    {"method": "add", "params": [10, 5], "jsonrpc": "2.0", "id": 1},
    {"method": "subtract", "params": [20, 10], "jsonrpc": "2.0", "id": 2},
]

response = requests.post(url, data=json.dumps(payload), headers=headers).json()

print(response)
```

### **Response from Server**

The server will then respond with the result of the operation:

```json
{
    "jsonrpc": "2.0",
    "result": 15,
    "id": 1
}
```
For the batch request, the response would be:
```json
[
    {"jsonrpc": "2.0", "result": 15, "id": 1},
    {"jsonrpc": "2.0", "result": 10, "id": 2}
]
```

### **When to Use JSON-RPC**

JSON-RPC is an excellent choice for:
- **Simple APIs** When you have a straightforward set of actions that your API needs to perform.
- **Internal Communication** It's well-suited for communication between microservices within a larger application.
- **Performance** The ability to batch requests can improve performance by reducing the number of HTTP round-trips.
- **Notifications** When you need to send notifications from the client to the server without waiting for a response.


### **Anatomy of a JSON-RPC Request**

A JSON-RPC request is sent as a single JSON object. It contains the following members, each with a specific purpose and set of rules.

#### **1. `jsonrpc` (Required)**

This member specifies the version of the JSON-RPC protocol.
- **Requirement**: It **MUST** be exactly the string `"2.0"`. Other values, such as `"2"` or `2.0`, are not valid and will result in an error.
- **Purpose**: It ensures that both the client and server are communicating using the same protocol specification.

```json
{
  "jsonrpc": "2.0"
}
```

#### **2. `method` (Required)**

This is a string containing the name of the method to be invoked on the server.
- **How it works**: Unlike REST APIs which are resource-oriented (using GET, POST on a URL), RPC is action-oriented. You directly specify the name of the function you wish to execute.
- **Example**: If you have a Python function `def add(a, b):` on your server, the method name in the request would be `"add"`.


{
  "jsonrpc": "2.0",
  "method": "subtract"
}

#### **3. `params` (Optional)**

This member is a structured value that holds the parameter values to be used during the invocation of the method.
- **Format**: It can be either a JSON Array `[]` (for positional parameters) or a JSON Object `{}` (for named parameters).
- **Usage**:
    - **By-position (Array)**: ` "params": [42, 23]`
    - **By-name (Object)**: ` "params": {"subtrahend": 23, "minuend": 42}`
- **Function Mapping**: The number and type of parameters must match what the server-side function expects.

{
  "jsonrpc": "2.0",
  "method": "subtract",
  "params": [42, 23] 
}

#### **4. `id` (Required for standard calls, omitted for notifications)**

This member is used to correlate a response with its corresponding request.
- **Data Type**: The value can be a String, a Number, or `null`.
- **Purpose**: The `id` acts as an **identifier**. When a client sends multiple requests (e.g., in a batch or asynchronously), it needs a way to match incoming responses to the requests it originally sent. The server **must** include the exact same `id` in its response object.
- **Importance**: If a client sends two requests to the same method with different parameters but the same `id`, it will not be able to differentiate between the two responses. Therefore, using a unique `id` for each concurrent request is crucial.

json
{
  "jsonrpc": "2.0",
  "method": "subtract",
  "params": [42, 23],
  "id": 1 
}

### **Special Case: Notifications**

A Request object is treated as a **Notification** if the `id` member is omitted.
- **Behavior**: When a server receives a notification, it understands that the client does not expect a response. Therefore, the server **MUST NOT** send a Response object back.
- **Use Case**: This is useful for "fire-and-forget" operations, where the client simply needs to inform the server of an event but does not require confirmation or a result.               

Of course, here is the documentation based on the provided audio transcript.

### **Advanced JSON-RPC Concepts & Use Cases in MCP**

This document details advanced features of the JSON-RPC protocol, including notifications, parameter structures, and response/error handling, and explains why this protocol is the foundation for the Model-Context-Protocol (MCP).

---

### **1. Notifications: One-Way Communication**

A standard RPC call requires a response. However, there are scenarios where a client only needs to send information to the server without needing a confirmation or result. This is a **Notification**.

- **How it Works**: A request is identified as a notification if the `id` member is omitted.
- **Server's Role**: The server **MUST NOT** send a response to a notification.
- **Use Case**: This is ideal for one-way communication, such as:
    - **Logging**: Sending user activity logs to the server (e.g., "user opened page X").
    - **Events**: Informing the server of an event that doesn't require an immediate reply.
- **Important Consideration**: Since no response is sent, the client will have no way of knowing if an error occurred during the processing of the notification on the server side. Therefore, notifications should be used for non-critical operations.

// A notification to log an update. No 'id' is present.
{
  "jsonrpc": "2.0",
  "method": "update",
  "params": ["User logged in"]
}

### **2. Parameter Structures (`params`)**

JSON-RPC offers two flexible ways to pass parameters to a method:

- **By Position**:
    - **Format**: A JSON Array `[]`.
    - **Logic**: The values in the array are mapped to the server-side function's parameters in the order they appear. The first value goes to the first parameter, the second to the second, and so on.

      json
    // Assumes server function is: def subtract(minuend, subtrahend)
    "params": [42, 23] 

- **By Name**:
    - **Format**: A JSON Object `{}`.
    - **Logic**: The keys in the object must match the names of the server-side function's parameters. The order of the key-value pairs does not matter.

    ```json
    // Assumes server function is: def subtract(minuend, subtrahend)
    "params": { "subtrahend": 23, "minuend": 42 }
    

### **3. The Response & Error Objects**

For every standard call (one with an `id`), the server must reply with a Response Object. The structure of this response depends on whether the call was successful or not.

**A Response Object MUST contain either a `result` or an `error` member, but not both.**

#### **Success Response**
- **`result`**: Contains the value returned by the server-side method. The data type is determined by the method itself and can be anything (string, number, object, array, etc.).
- **`id`**: Must match the `id` of the original request.

json
{
  "jsonrpc": "2.0",
  "result": 19,
  "id": 1
}

#### **Error Response**
- **`result`**: This member **MUST NOT** be included if an error occurred.
- **`error`**: An object containing detailed information about the error.
- **`id`**: Must match the `id` of the original request.

json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32601,
    "message": "Method not found"
  },
  "id": "1"
}


The **Error Object** has a standardized structure:
- **`code`**: An integer indicating the error type. Predefined codes exist for common errors:
    - `-32700`: **Parse error**. Invalid JSON was received by the server.
    - `-32600`: **Invalid Request**. The JSON sent is not a valid Request object.
    - `-32601`: **Method not found**. The method does not exist/is not available.
    - `-32602`: **Invalid params**. Invalid method parameter(s).
    - `-32603`: **Internal error**. Internal JSON-RPC error.
    - `-32000 to -32099`: **Server error**. Reserved for implementation-defined server-errors.
- **`message`**: A single-sentence string describing the error.
- **`data`**: (Optional) A primitive or structured value containing additional information.

---

### **4. Batch Requests: Multiplexing**

One of the most powerful features of JSON-RPC is the ability to send multiple calls in a single HTTP request. This is known as **batching** or **multiplexing**.

- **How it Works**: The client sends a JSON Array of Request Objects.
- **Server Processing**: The server can process these requests in any order.
- **Response**: The server returns a JSON Array of the corresponding Response Objects. The order of responses in the array is not guaranteed to match the order of requests.
- **Matching**: The client **must** use the `id` in each response object to match it with its original request.

This is highly efficient as it reduces network latency by minimizing the number of HTTP round trips. REST APIs do not have a standard, built-in mechanism for this.

---

### **5. Why MCP Uses JSON-RPC over REST**

The Model-Context-Protocol (MCP) is designed for complex agentic workflows, and it chose JSON-RPC for several key reasons:

1.  **Action-Oriented Fit**: MCP involves executing commands and procedures (e.g., "start_session", "call_tool"). JSON-RPC's function-calling paradigm (`"method": "function_name"`) is a more natural and simpler fit than REST's resource-based approach.

2.  **Efficiency with Batching**: Agent workflows often require multiple sequential or parallel tool calls. JSON-RPC’s native batching allows an agent to execute several tasks in a single network request, drastically improving performance.

3.  **Support for Streaming & Notifications**: MCP needs to handle asynchronous events like progress updates or partial results from a long-running tool. JSON-RPC's notification feature is perfect for these one-way messages where a response isn't required.

4.  **Simplified Tool Definition**: With MCP, developers don't need to define complex REST endpoints (URLs, HTTP verbs) for each tool. They simply create a function, and it becomes a callable method via RPC. This is much simpler.

5.  **Stateful Workflows on a Stateless Protocol**: While JSON-RPC is stateless, MCP enables stateful conversations by passing context identifiers (like `session_id` or `context_id`) as parameters in each call. The state is managed by the agent/server, not the protocol.

6.  **Standardized Error Handling**: The strict `code`, `message`, and `data` structure for errors in JSON-RPC provides a reliable and predictable way for agents to handle failures, which is more consistent than the varied error responses found in REST APIs.

### **Future Outlook: MCP UI**
The MCP framework is evolving to include **MCP UI**, where an MCP server can return small, interactive UI components (HTML/CSS/JS) instead of just data. This allows an agent's chat interface to become dynamic, rendering buttons, forms, or sliders, making the user experience far more interactive and intuitive than plain text.