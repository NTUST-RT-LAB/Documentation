# HTTP (Hyper Text Transfer Protocol)

## HTTP Definition
HTTP (Hypertext Transfer Protocol) is a protocol that is used for communication between a client (such as a web browser) and a server. It allows the client to send a request to the server, and the server will respond with the requested data or perform a specified action. Here is a summary of the HTTP request process:

## HTTP Request Process
![](https://hackmd.io/_uploads/Hkfiwyorh.png)

HTTP request is a message sent by HTTP client (such as a web browser or an application) to a HTTP server using the HTTP protocol. It is the client's way of communicating with the server and requesting a specific action or resource. There are few things that are related to HTTP Request, such as HTTP Method, HTTP Parameter, and HTTP Response


### HTTP Method
HTTP method is the method that is specified by HTTP client to request resource from HTTP Server, the most common HTTP method are:

* GET: Requests data from a server.
* POST: Submits data to be processed by a server.
* PUT: Updates or replaces existing data on the server.
* DELETE: Deletes data on the server.

### HTTP Parameter
HTTP Parameter is the data that must be specified by HTTP client to send the HTTP Request to HTTP server. List of few HTTP Parameter:
* URL (Uniform Resource Locator) or URI (Uniform Resource Identifier): The URL identifies the specific resource on the server that the client wants to interact with.
* Headers: The request can include headers that provide additional information to the server. Headers can contain details such as the client's user agent, content type, accepted languages, and more.
* Request Body: For methods like POST or PUT, the client may include a request body that contains data to be sent to the server. The body can be in various formats, such as JSON, XML, or plain text.


### HTTP Response
Upon receiving the request, the server processes it based on the method and URL. It performs the requested action, retrieves the requested data, and it will generate a HTTP Response. The HTTP Response can includes a status code, headers, and an optional response body that is requested by the client. The status code indicates the success or failure of the request, the most common HTTP Response are:

* 200 OK: This status code indicates that the request was successful, and the server is returning the requested content. It is the standard response for successful GET requests.
* 400 Bad Request: This status code indicates that the server cannot process the request due to invalid syntax, invalid parameters, or other client-side errors.
* 401 Unauthorized: This status code indicates that the request requires authentication, and the client must provide valid credentials (e.g., username and password) to access the requested resource.
* 403 Forbidden: This status code indicates that the server understood the request, but the client does not have permission to access the requested resource.
* 404 Not Found: This status code indicates that the requested resource could not be found on the server. It is commonly used when a URL or URI does not correspond to an existing resource.
* 500 Internal Server Error: This status code indicates that an unexpected condition occurred on the server, causing it to be unable to fulfill the request.
:::info
In general, HTTP Response Code that starts with:
* 2(2XX) indicates request successful
* 4(4XX) indicates client error
* 5(5XX) indicates server error
:::

After the client receives the response from the server, it reads the status code to determine the outcome of the request and parses the headers and response body as needed. That is about the basic process of a HTTP request. It defines the communication between clients and servers on the web, enabling the retrieval and exchange of data and resources.