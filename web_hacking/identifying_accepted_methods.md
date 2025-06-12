# Identifying accepted methods

Quando nos deparamos com um servidor web, uma das primeiras coisas que podemos testar são os métodos que esse servidor aceita. Isso é importante pois existem algumas vulnerabilidades existentes em alguns métodos permitidos. Podemos utilizar o CURL para isso.

```
$ curl -v -X OPTIONS http://businesscorp.com.br
*   Trying 37.59.174.225:80...
* TCP_NODELAY set
* Connected to businesscorp.com.br (37.59.174.225) port 80 (#0)
> OPTIONS / HTTP/1.1
> Host: businesscorp.com.br
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Tue, 01 Oct 2019 16:19:51 GMT
< Server: Apache/2.2.22 (Debian)
< Allow: GET,HEAD,POST,OPTIONS
< Vary: Accept-Encoding
< Content-Length: 0
< Content-Type: text/html
< 
* Connection #0 to host businesscorp.com.br left intact
```

É Importante também tentar em outros diretórios que for descoberto.

Se o servidor permitir o método PUT, podemos então enviar um arquivo para explorar (se for uma aplicação em php, por exemplo, podemos utilizar a função system() nesse arquivo para conseguir comandos do sistema).

```
$ curl -X PUT <url>/<file>
```

Podemos também utilizar dessa forma para fazer upload de arquivos.

```
$ curl -v <url> --upload-file file.php
```

## Burlando autenticação via métodos

Uma forma de dar bypass em uma página que só permite um método, como GET por exemplo, é tentarmos utilizar algum outro método. Se enviamos um GET e recebemos um "não autorizado", podemos tentar enviar um método POST para verificar se nessa página em específico está aceitando.