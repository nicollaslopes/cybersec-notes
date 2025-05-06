# Infra

## Whois

O Whois é um protocolo que trabalha na porta 43 tcp. Quando consultamos um domínio no whois da IANA, ele mostra quem é a autoridade por aquele domínio.

{% embed url="https://www.iana.org/whois" %}

Ou podemos usar o whois via linha de comando:

```bash
$ whois <target>
```

## RDAP

Por conta de alguns problemas de padronizações de resposta no protocolo whois, foi criado o protocolo RDAP, afim de consertar isso. A ideia basicamente é a mesma, porém ele traz uma resposta padronizada.

{% embed url="https://rdap-web.lacnic.net/rdap/home" %}

{% embed url="https://client.rdap.org" %}

## Pesquisa por IP

Para começarmos a mapear a infra pelos ips, podemos utilizar dois sites que facilita esse processo e exibe o bloco de ip.

{% embed url="https://check-host.net" %}

{% embed url="https://search.arin.net/rdap" %}

Porém, temos que nos atentar que um determinado ip pode estar por trás de um serviço contratado de uma empresa. Se dermos um whois no ip do Itaú, por exemplo, ele irá trazer as informações da Akamai, que é a empresa contratada. Para descobrirmos o real ip para mapearmos, precisamos utilizar o comando host para resolver esse hostname.

```bash
  
$ host itau.com.br
itau.com.br has address 104.73.88.118
itau.com.br mail is handled by 1 post01.itau.com.br.
itau.com.br mail is handled by 1 post02.itau.com.br.

$ host post01.itau.com.br
post01.itau.com.br has address 200.196.152.9
```

Agora podemos utilizar o whois no endereço ip exibido que irá ser trago as informações do Itaú.

## Pesquisa por BGP

{% embed url="https://bgpview.io" %}

{% embed url="https://bgp.he.net" %}

## Shodan

{% embed url="https://www.shodan.io" %}

Shodan é um mecanismo de busca para encontrar dispositivos online. Alguns operadores que podemos utilizar na busca são:

* hostname:
* os:
* port:
* ip:
* net:
* country:
* city:
* geo:
* org:
* " "

Examples: 

* hostname:businesscorp.com.br
* ip:36.59.174.225
* net:36.59.174.225/28
* webcam country:br
* port 3306 os:Windows country:br
* port 445 country:br contabilidade
* hostname:google.com

## Censys

O Censys também é uma outra plataforma para coleta de informações de forma passiva.

{% embed url="https://search.censys.io" %}

## Transferência de zona

Um domínio precisa de no mínimo 2 servidores de nomes. O servidor primário mantém um registro com todas as entradas DNS. O servidor secundário funciona como se fosse um "backup" caso aconteça do servidor primário não funcionar e para que isso aconteça corretamente, é necessário que ambos tenham os arquivos sincronizados. Quando o servidor primário possui mais entradas DNS que o secundário, ele irá forçar uma transferência de zona para que tenham os arquivos sincronizados. Então quando encontramos a porta 53 do DNS TCP aberta, podemos tentar fazer a transferência de zona.

```bash
$ host -t ns businesscorp.com.br
businesscorp.com.br name server ns1.businesscorp.com.br.
businesscorp.com.br name server ns2.businesscorp.com.br.
```

Com isso, podemos utilizar o parâmetro -l para listar todos hosts do domínio utilizando a consulta AXFR que é a de transferência de zona.

```bash
$ host -l businesscorp.com.br ns2.businesscorp.com.br
Using domain server:
Name: ns2.businesscorp.com.br
Address: 37.59.174.226#53
Aliases: 

businesscorp.com.br name server ns1.businesscorp.com.br.
businesscorp.com.br name server ns2.businesscorp.com.br.
businesscorp.com.br has address 37.59.174.225
desafio.businesscorp.com.br has address 37.59.174.226
ftp.businesscorp.com.br has address 37.59.174.225
infrasecreta.businesscorp.com.br has address 37.59.174.225
intranet.businesscorp.com.br has address 37.59.174.228
mail.businesscorp.com.br has address 37.59.174.227
ns1.businesscorp.com.br has address 37.59.174.225
ns2.businesscorp.com.br has address 37.59.174.226
parsingok.businesscorp.com.br has address 37.59.174.225
piloto.businesscorp.com.br has address 37.59.174.230
rh.businesscorp.com.br has address 37.59.174.229
srvkey.businesscorp.com.br has address 37.59.174.235
www.businesscorp.com.br has address 37.59.174.225
```

Ou também usar o dig

```
$ dig axfr businesscorp.com.br @ns2.businesscorp.com.br

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> axfr businesscorp.com.br @ns2.businesscorp.com.br
;; global options: +cmd
businesscorp.com.br.	3600	IN	SOA	ns1.businesscorp.com.br. hostmaster.businesscorp.com.br. 2019030507 3600 900 604800 1
businesscorp.com.br.	3600	IN	NS	ns1.businesscorp.com.br.
businesscorp.com.br.	3600	IN	NS	ns2.businesscorp.com.br.
businesscorp.com.br.	3600	IN	A	37.59.174.225
businesscorp.com.br.	3600	IN	HINFO	"SERVIDOR DELL" "DEBIAN - key:0989201883299"
businesscorp.com.br.	3600	IN	MX	10 mail.businesscorp.com.br.
businesscorp.com.br.	3600	IN	TXT	"v=spf1 include:key-9283947588214 ?all"
admin.businesscorp.com.br. 3600	IN	CNAME	key.vlab.takeover.092935999311009.
desafio.businesscorp.com.br. 3600 IN	A	37.59.174.226
dev.businesscorp.com.br. 3600	IN	CNAME	devbusinesscorp.s3-sa-east-1.amazonaws.com.
documentacao.businesscorp.com.br. 3600 IN CNAME	businesscorp.github.io.
ftp.businesscorp.com.br. 3600	IN	A	37.59.174.225
infrasecreta.businesscorp.com.br. 3600 IN A	37.59.174.225
intranet.businesscorp.com.br. 3600 IN	A	37.59.174.228
mail.businesscorp.com.br. 3600	IN	A	37.59.174.227
ns1.businesscorp.com.br. 3600	IN	A	37.59.174.225
ns2.businesscorp.com.br. 3600	IN	A	37.59.174.226
parsingok.businesscorp.com.br. 3600 IN	A	37.59.174.225
piloto.businesscorp.com.br. 3600 IN	A	37.59.174.230
rh.businesscorp.com.br.	3600	IN	A	37.59.174.229
srvkey.businesscorp.com.br. 3600 IN	A	37.59.174.235
trello.businesscorp.com.br. 3600 IN	CNAME	trello.com/b/kaqiIGDl/businesscorpcombr.
wordpress.businesscorp.com.br. 3600 IN	CNAME	businesscorp.home.blog.
www.businesscorp.com.br. 3600	IN	A	37.59.174.225
businesscorp.com.br.	3600	IN	SOA	ns1.businesscorp.com.br. hostmaster.businesscorp.com.br. 2019030507 3600 900 604800 1
;; Query time: 204 msec
;; SERVER: 37.59.174.226#53(ns2.businesscorp.com.br) (TCP)
;; WHEN: Tue May 06 08:40:39 -03 2025
;; XFR size: 25 records (messages 1, bytes 821)
```