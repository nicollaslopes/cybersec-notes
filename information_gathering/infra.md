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

