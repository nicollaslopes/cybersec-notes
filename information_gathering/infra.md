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

```
$ host -l -a businesscorp.com.br ns2.businesscorp.com.br
Trying "businesscorp.com.br"
Using domain server:
Name: ns2.businesscorp.com.br
Address: 37.59.174.226#53
Aliases: 

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57367
;; flags: qr aa; QUERY: 1, ANSWER: 25, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;businesscorp.com.br.		IN	AXFR

;; ANSWER SECTION:
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

Received 821 bytes from 37.59.174.226#53 in 209 ms
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

## Pesquisa reversa

Algumas vezes na pesquisa reversa, pesquisando por ip (resolvendo o registro PTR) podemos encontrar alguns subdomínios. Precisamos saber qual o range de ips e utilizar o script simples abaixo.

```
$ for ip in $(seq 224 239); do host 37.59.174.$ip; done
```

## Analisando SPF

O SPF (Sender Policy Framework) é responsável por identificar quais servidores estão autorizados a enviar e-mail o nome do seu domínio. Se não tiver essa configuração ou se estiver mal feita, uma pessoa consegue fazer um email spoofing através de um e-mail desse domínio. Podemos validar isso de acordo com as configurações:

* `?all` => suscetível (neutro) O e-mail irá cair na caixa de entrada.
* `~all` => suscetível (é tratado como suspeito) O e-mail pode cair na caixa de entrada mas a grande chance é de ir para o spam.
* `-all` => configuração recomendada

Utilizamos o comando host para sabermos como está configurado o SPF no servidor.

```
$ host -t txt businesscorp.com.br

businesscorp.com.br descriptive text "v=spf1 include:key-9283947588214 ?all"
```

Podemos também gerar o e-mail falso utilizando uma ferramenta chamada emkei.

{% embed url="https://emkei.cz" %}

## Subdomain Takeover

O subdomain takeover é basicamente tomar controle de um subdomínio. Essa falha ocorre quando o CNAME de um subdomínio aponta para um outro subdomínio que está desativado. O problema é que a entrada de DNS foi criada mas o serviço que a empresa utilizava foi cancelado/eliminado/migrou de serviço e não utiliza esse domínio mais. Esse serviço que foi deixado de ser utilizado, ainda fica disponível para registro, com isso o subdomínio pode ser "tomado" e assim controlado. Para verificarmos se um subdomínio possui o alias, ou seja, se aponta para algum outro endereço, utilizamos o comando host com o parâmetro do cname. Caso for positivo, irá vir de acordo com a mensagem abaixo, após isso basta acessar o outro domínio e verificar se está sendo utilizado ou não.

```
$ host -t cname dev.businesscorp.com.br
dev.businesscorp.com.br is an alias for devbusinesscorp.s3-sa-east 1.amazonaws.com.
```

## Tools

É interessante também procurarmos por ferramentas já prontas para fazer o reconhecimento de DNS. Uma delas é o dnsenum. Usa-se dessa forma.

```
$ dnsenum --enum businesscorp.com.br
```

#### Serviços para pesquisa passiva
Existem também alguns serviços que podemos coletar informações do alvo de uma forma passiva, sem gerar log no servidor que estamos testando. Alguns deles são:

{% embed url="https://www.virustotal.com/gui/home/search" %}

{% embed url="https://dnsdumpster.com/" %}

{% embed url="https://securitytrails.com/" %}

#### Coleta através de certificados digitais
Quando um site utiliza o certificado SSL, para adicionar mais uma camada de segurança, foi criado um projeto para manter um log de todas as emissões de certificados que acontecem. Com isso, podemos descobrir subdomínios que emitiram certificados uma vez que é armazenado esse log e ele é público.

{% embed url="https://crt.sh/" %}


