# HTTP

The HTTP protocol is responsible for web communications between client and server. The client sends the HTTP request and the server responds with the HTTP response.

HTTP consists of the request header and the body, which are separated by a blank line (if it is a POST request and has a body). By default, when we access a web page, the request method is GET.

In this example below, I used a proxy called Burpsuite to intercept the request between a client (a browser, in this case) and a server, so we can better visualize how it works.

<figure><img src="../.gitbook/assets/http-1.png" alt=""><figcaption></figcaption></figure>

A GET request was made to google.com. Note that line 14 is blank. As previously stated, the HTTP protocol works this way. This blank line is necessary to separate the header from the body. In this case, since it was a GET request, there is no body to be sent.

Now, let's submit a login form to a website to test how a post request works.

<figure><img src="../.gitbook/assets/http-2.png" alt=""><figcaption></figcaption></figure>

In the image above, we can see on line 20 that a body was sent, containing the information we submitted in the form, in html encoding.

## HTTP METHODS

To communicate from the front-end to the back-end, the HTTP protocol uses HTTP methods. They are:

* **GET**: Read data from the server 
* **POST**: Send data to the server 
* **PUT**: Send data to the server (create/update) 
* **DELETE**: Delete data from the server 
* **PATCH**: Update data on the server.

## HTTP Status Codes

For every HTTP request we make to the server, it will respond with a status code. Each status code has a meaning. These are some of them:

<figure><img src="../.gitbook/assets/http-3.jpg" alt=""><figcaption></figcaption></figure>

https://developer.mozilla.org/en-US/docs/Web/HTTP/Status

## Tools

So far, we've just been using a browser to make requests to websites, but there are some useful tools we can use, for example **curl**.

```bash
$ curl -v http://tibia.com

* Host tibia.com:80 was resolved.
* IPv6: (none)
* IPv4: 3.5.138.53, 52.219.171.56, 52.219.169.152, 52.219.72.142, 52.219.47.150, 52.219.169.244, 3.5.136.58, 3.5.138.176
*   Trying 3.5.138.53:80...
* Connected to tibia.com (3.5.138.53) port 80
> GET / HTTP/1.1
> Host: tibia.com
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< x-amz-id-2: uoZcyNsehdH4AMSqu8mjBcj25crKUBdklxcDRjR4u6Zrh+Nv+/3hsy3VYATP3uXBQw7QQ3NOXyzP3clbdAINy/a8e0oyUAEg
< x-amz-request-id: 033PCC39WQ5CJV7Q
< Date: Sun, 09 Mar 2025 20:30:04 GMT
< Location: http://www.tibia.com/
< Content-Length: 0
< Server: AmazonS3
< 
* Connection #0 to host tibia.com left intact

```

Curl is really useful, we can send POST request using `-X POST` parameter. 

```bash
$ curl -X POST http://tibia.com/account/?subtopic=accountmanagement -d "loginemail=test%40email.com&loginpassword=password&page=overview"
```

We can also set some headers to make our requests with `-H` parameter.

```bash
$ curl -v http://tibia.com -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:136.0) Gecko/20100101 Firefox/136.0"
```
