# HTTP Desync Request Smuggling

Nowadays, it is very common to use Load Balancer or Reverse Proxy, which greatly help the  performance of an application that receives multiple requests per minute or even per seconds.

The idea behind the vulnerability is to cause a desynchronization between two servers (Load Balancer and Back-end Server). This allow us to add a prefix on the request of another user who is browsing the website.

The user makes a request to `www.google.com` and the Load Balancer will establishes a TCP Connection with the back-end server to process that request. The Load Balancer connection can be made over TCP or TLS.

There is something in TCP called `Connection Reuse`. Let's imagine a scenario where the Load Balancer receives 1000 requests per second. It will need to open 1000 connections to the back-end server for each request, which is quite a lot, isn't it? To solve this, we use the `Connection Reuse`, which is implemented through the HTTP header called `kee-alive`. For example:

```
POST / HTTP/1.1
```

```
POST / HTTP/1.1
Connection: close
```

The connection `close` indicates that the request will open a connection to the back-end server and at the end, the TCP (or TLS) connection will be closed. When new request is made, the entire process of opening another new connection will be repeated.

When is set the flag `keep-alive`, the connection is reused, meaning that when a new request is made, the Load Balancer will use the same connection and it will remain open. This is more advantageous in terms of connection speed. 

In some cases, when is the Load Balancer controls the `Connection Ruse` or `keep-alive`, it can reuse the same connection to process request from different users, and this is where we can find the risk.  

img1

What if we could send the extra content? That way, we could to send the additional content that is part of second user's request.

img2

That's what desync is. The back-end will process this extra content and understand that next request is from user B, meaning the user A managed to add this content to user B's request. The back-end understands that there are two requests and that at the end A's request. B's request would begin. Therefore, user A has control over the start of the B's HTTP request. 

### How Transfer-encoding chunked works?

If we take a look at the Mozila website, we can see an example:

"Data is sent in a series of chunks. Content can be sent in streams of unknown size to be transferred as a sequence of length-delimited buffers, so the sender can keep a connection open, and let the recipient know when it has received the entire message. The [`Content-Length`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Length) header must be omitted, and at the beginning of each chunk, a string of hex digits indicate the size of the chunk-data in octets, followed by `\r\n` and then the chunk itself, followed by another `\r\n`. The terminating chunk is a zero-length chunk." 

```http
POST /home HTTP/1.1
Host: localhost
Transfer-Encoding: chunked

7\r\n
Welcome\r\n
1c\r\n
to Mozilla Developer Network\r\n
0\r\n
\r\n
```

In other words, we are sending a HTTP Request **in parts**. We can notice that `1c` was sent because it needs to be in hexadecimal (`1c` = 28 characters). Instead of specifying the `Content-Length`(which is the sum of everything we send), we specify the length of each part of the request.

## How to exploit it?

There is a way to perform this desynchronization. The following is specified in the RFC bellow:

[RFC 2616](https://tools.ietf.org/html/rfc2616#section-4.4):

> If a message is received with both a `Transfer-Encoding` header field and a `Content-Length` header field, the latter MUST be ignored.

With this, we can send both headers and whoever is processing this request will be prioritize the `Transfer-Encoding`. So, since the `Transfer-Encoding` is `chunked` and has a 0, it will understand that the request has ended and there is no more content. The character `T` passed by Load Balancer is the beginning of the next new request. 

img 3
img 4

That way we are adding a prefix to the request from another user who is accessing the website. This will work when the Load Balancer ignores the `Transfer-Encoding`, prioritizing the `Content-Length`, and the back-end prioritizes the `Transfer-Encoding`. We then have a `CL.TE`.

Existem algumas formas em determinados cenarios para que o load balancer ignore o transfer-enconding para que o backend o receba. Por ex: 

* colocando um espaco antes do transfer-encondig
* colocar um espaco a mais antes do chunked
* colocar um caractere especial antes do chunked

## Portswigger Lab

### Lab: HTTP request smuggling, confirming a CL.TE vulnerability via differential responses

This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. To solve the lab, smuggle a request to the back-end server, so that a subsequent request for `/` (the web root) triggers a 404 Not Found response.

Quando enviamos uma requiscao para a raiz do site, temos a seguinte pagina

img lab 1-1

Para fazer esse cenario funcionar, primeiro precisamos de alterar manualmente a versao do HTTP/2 para HTTP/1.1 em Request attributes.

Nesse caso, temos que enviar uma requisicao POST, adicionando no corpo da requisicao

```
Transfer-Encoding: chunked

0\r\n
\r\n
GET /AAAA HTTP/1.1\r\n
B-Ignore: C
```

lab 1-2
img-1-3

Depois de fazermos isso, o proximo usuario ao acessar o site, ira receber um erro 404.

img-4

Podemos visualizar na imagem abaixo como que ficaria a requisicao com mais detalhes. Eh importante lembrar que depois de enviar o `B-Ignore: C`, nao ha uma quebra de linha com `\r\n` pois eh importante concatenar (append) a informacao com a requisicao do usuario, para ter sucesso e a requisicao nao ter sucesso, retornando o 404.

img5


Nos enviamos 94 caracteres e precisamos passar como hexadecimal `5e` (0x5e)
```
POST / HTTP/1.1
Host: 0a0e002d03a32c1e83bdec4d003a0089.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

5e
POST /AAA HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

a=1
0

```

### Lab: HTTP request smuggling, confirming a TE.CL vulnerability via differential responses

The reason this technique works is because when we send the first request when it arrives at the front-end server (if the front-end is using Transfer-encoding `chunked`) it will read in a chunk size of three `ABC` followed by the next chunk size `X` which is a invalid chunk size. So the front-end will simply reject our request and respond with an invalid request error and that show us that the front-end server might be using Transfer-encoding `chunked`.