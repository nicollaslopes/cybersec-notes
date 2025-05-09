# Web

Para termos sucesso em uma exploração web, a parte de reconhecimento é de suma importância para que isso aconteça. Essa é praticamente a fase mais importante que pode decidir o sucesso ou não das etapas posteriores. Precisamos então coletar as informações de uma forma que possamos levantar o maior número possível para entendermos bem o ambiente.

## Robots and Sitemap

O google possui vários crawlers (robôs que varrem a internet) que servem para indexar páginas. No entendo, você pode dizer para o google para não indexar alguma página de sua escolha. Para isso, é utilizado o arquivo **robots.txt** para essa configuração, colocando dentro dele os diretórios que não devem ser indexados. Quando o crawler acessar a página, ele irá ler primeiro o arquivo e ver quais diretórios que não vão ser indexados. No entendo, nós conseguimos ler o **robots.txt** e isso acaba se tornando uma brecha se o administrador deixar isso visível. Para isso, basta acessar o site junto com um "/robots.txt".

Já o sitemap, é basicamente um mapa do site que mapeia todas as páginas do site. Isso facilita o google saber quais são as páginas do site para indexar mais rápido. Para ler esse arquivo basta acessar o site junto com um "/sitemap.xml".

## Directory Listing

A listagem de diretórios, como o próprio nome já diz, é quando a aplicação possui um diretório onde está sendo listado os arquivos daquela pasta. Isso pode representar uma falha e ao mesmo tempo não, pois irá depender muito do que for encontrado e da criticidade de importância desses arquivos. Isso ocorre quando determinada pasta não possui um arquivo nela de index, então todos os outros arquivos nela ficam expostos.

imagem 1

## Mirror Website

O mirror website é basicamente baixar/clonar uma página, serve tanto para fazer um conhecimento da estrutura como fazer uma página para engenharia social. Podemos utilizar dessa forma:

```
$ wget -m <site>
```

Pode ser que por causa do robots.txt, não seja possível baixar corretamente os arquivos, para isso basta utilizar

```
$ wget -m -e robots=off <site>
```

## Pesquisa via requisições HTTP

Uma forma de verificarmos qual a tecnologia e suas versões utilizadas na aplicação, é enviar uma requisição para o servidor. Para isso podemos utilizar o método HEAD (para retornar apenas o cabeçalho) caso seja suportado ou até mesmo o GET (irá trazer o cabeçalho e mais o corpo da resposta).

```
$ nc www.businesscorp.com.br 80
HEAD / HTTP/1.0

HTTP/1.1 200 OK
Date: Thu, 03 Oct 2019 19:15:27 GMT
Server: Apache/2.2.22 (Debian)
Last-Modified: Wed, 25 Sep 2019 17:05:45 GMT
ETag: "20463-1bb6-59363a9ea0957"
Accept-Ranges: bytes
Content-Length: 7094
Vary: Accept-Encoding
Connection: close
Content-Type: text/html
```

Também é interessante utilizar o OPTIONS para saber quais métodos o servidor suporta.

```
$ nc -v businesscorp.com.br 80
OPTIONS / HTTP/1.0

HTTP/1.1 200 OK
Date: Sat, 28 Sep 2019 16:55:27 GMT
Server: Apache/2.2.22 (Debian)
Allow: GET,HEAD,POST,OPTIONS
Vary: Accept-Encoding
Content-Length: 0
Connection: close
Content-Type: text/html
```


*Obs:* As vezes é necessário informar a tecnologia que está sendo utilizada, ou seja, se caso a aplicação usar o PHP, fazer uma requisição para `/index.php`, assim provavelmente irá retornar mais informações. 

É importante também saber que se a aplicação tiver alguma proteção como um WAF, é necessário informar o header `host` para que não seja bloqueado.

```
$ nc -v businesscorp.com.br 80
OPTIONS / HTTP/1.0
Host: businesscorp.com.br
```

## Brute foce de diretórios e arquivos

Para conseguirmos enumerar a aplicação, é necessário fazermos um brute force para encontrar diretórios e arquivos para que possamos mapear de uma forma melhor e encontrar possíveis pontos de entrada. Mas antes de utilizar uma ferramenta para isso, é necessário saber como ela e o protocolo HTTP funcionam. Sua função, é basicamente carregar uma wordlist com vários diretórios e arquivos e assim enviando uma requisição para a aplicação e de acordo com sua resposta, dizer se esse diretório ou arquivo existe dentro do servidor. Quando enviamos uma requisição e ela é bem sucedida, ou seja, esse diretório/arquivo existe, o servidor irá responder com uma resposta do http indicando o status dessa requisição, que no caso é 200. Já quando não existe, irá ser retornado o status 404. Basicamente essa é a forma que funciona. Porém, em algumas ferramentas é utilizado o cabeçalho padrão delas próprias e existem aplicações que tem algum bloqueio que impede requisições vindo como determinado cabeçalho, para justamente impedir isso de acontecer. Há uma maneira de burlar isso que é alterando o User-Agent na própria ferramenta, assim esse filtro não irá funcionar.

## Tools
### Curl

Podemos também utilizar o `curl` para fazer requisições http/https.

```
$ curl -v businesscorp.com.br
```

### Whatweb

O whatweb é uma ferramenta que faz um reconhecimento das tecnologias utilizadas em um site/aplicação.

```
$ whatweb <target>
```

### Wappalyzer

{% embed url="https://www.wappalyzer.com" %}

### Recon

{% embed url="https://github.com/ffuf/ffuf" %}


{% embed url="https://github.com/thewhiteh4t/FinalRecon" %}