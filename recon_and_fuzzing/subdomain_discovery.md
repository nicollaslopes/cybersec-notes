# Subdomain Discovery

## Tools

É interessante também procurarmos por ferramentas já prontas para fazer o reconhecimento de DNS. Uma delas é o dnsenum. Usa-se dessa forma.

```
$ dnsenum --enum businesscorp.com.br
```

### Histórico

Podemos buscar informações de subdomínio pelo histórico.

{% embed url="https://securitytrails.com/" %}

{% embed url="https://web.archive.org" %}

### Certificados e Serviços para pesquisa passiva
Existem também alguns serviços que podemos coletar informações do alvo de uma forma passiva, sem gerar log no servidor que estamos testando. Alguns deles são:

{% embed url="https://dnsdumpster.com/" %}

{% embed url="https://www.virustotal.com/gui/home/search" %}

**Obs:** *É interessante analisarmos o histórico de alteração de um subdomínio no SecurityTrails, porque vamos supor que uma aplicação hospedada na AWS e que possui um WAF e com isso, todo endereço de DNS será apontado para o WAF. Podemos assim, acompanhar ho histórico de alterações, a troca que foi feita em relação ao endereço IP e assim podemos acessar diretamente o endereço do IP do servidor, burlando o WAF*.

### Coleta através de certificados digitais
Quando um site utiliza o certificado SSL, para adicionar mais uma camada de segurança, foi criado um projeto para manter um log de todas as emissões de certificados que acontecem. Com isso, podemos descobrir subdomínios que emitiram certificados uma vez que é armazenado esse log e ele é público.

{% embed url="https://crt.sh/" %}

Podemos também utilizar o subfinder, que é uma excelente ferramenta que usa fontes de forma passiva que pesquisa também por certificados para buscar por subdomínios.

{% embed url="https://github.com/projectdiscovery/subfinder" %}

discover-subdomain-1.png

```
$ subfinder -d businesscorp.com.br

               __    _____           __         
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /    
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

                projectdiscovery.io

[INF] Current subfinder version v2.7.1 (latest)
[INF] Loading provider config from /home/user/.config/subfinder/provider-config.yaml
[INF] Enumerating subdomains for businesscorp.com.br
srvkey.businesscorp.com.br
dev.businesscorp.com.br
mail.businesscorp.com.br
ns2.businesscorp.com.br
parsingok.businesscorp.com.br
www.businesscorp.com.br
intranet.businesscorp.com.br
ns1.businesscorp.com.br
piloto.businesscorp.com.br
rh.businesscorp.com.br
[INF] Found 10 subdomains for businesscorp.com.br in 1 second 712 milliseconds
```

Podemos inserir a chave de API de alguns serviços para melhorar os resultados:

```
nano /home/user/.config/subfinder/provider-config.yaml
```

### Brute force

https://github.com/aboul3la/Sublist3r

https://github.com/owasp-amass/amass