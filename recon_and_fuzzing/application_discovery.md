# Application Discovery

## URL Discovery

Para fazermos uma busca por arquivos e diretórios, podemos utilizar algumas ferramentas para fazer o `fuzzing`.

### Gau

O gau utiliza motores de pesquisa (ex: google, shodan, censys, waybackmachine), compila essas informações e nos retorna as urls que foi descobertas nessas buscas.

{% embed url="https://github.com/lc/gau/releases" %}

```
$ echo "businesscorp.com.br" | gau 
WARN[0000] error reading config: Config file /home/user/.gau.toml not found, using default config 
http://businesscorp.com.br/
http://businesscorp.com.br/%C3%A3dimi/
http://businesscorp.com.br//_docs
http://businesscorp.com.br/_docs/senhas.txt
http://businesscorp.com.br///_restrito
http://businesscorp.com.br/_restrito/?C=M%3BO=A
http://www.businesscorp.com.br/access.log
http://businesscorp.com.br/AcessoRestrito/
http://www.businesscorp.com.br/admin/
http://www.businesscorp.com.br/apiClients
http://www.businesscorp.com.br/apiClients/showNames.txt
http://www.businesscorp.com.br/apiClients/showNames.xml
http://businesscorp.com.br/app
http://businesscorp.com.br/app/index.php
http://businesscorp.com.br/bkp/
http://businesscorp.com.br/bkp/script.sh
http://businesscorp.com.br/blog/robots-txt/
http://businesscorp.com.br:80/cadastro.php
http://businesscorp.com.br/cgi-bin/
http://www.businesscorp.com.br/comunicacao/projeto.txt
http://businesscorp.com.br/config
http://businesscorp.com.br/config.txt
http://businesscorp.com.br/configuracoes/
http://businesscorp.com.br/configuracoes/comunicacao/
```

### ffuf

https://github.com/ffuf/ffuf

<div align="right" data-full-width="false"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

### wfuzz

https://github.com/xmendez/wfuzz

```
$ wfuzz -t 100 -c -z file,wordlist.txt --hc 404 http://testphp.vulnweb.com/FUZZ
```

### Content Discovery

### httpx

https://github.com/projectdiscovery/httpx/releases

### Aquatone

https://github.com/michenriksen/aquatone

```
$ subfinder -d testphp.vulnweb.com -silent | aquatone
aquatone v1.7.0 started at 2025-05-14T07:52:00-04:00

Targets    : 2
Threads    : 3
Ports      : 80, 443, 8000, 8080, 8443
Output dir : .
```

### Parameter Discovery

### Param Spider

https://github.com/devanshbatham/ParamSpider

Podemos rodar o param spider usando o docker, para conseguirmos achar parâmetros em uma aplicação

<figure><img src="../.gitbook/assets/param-spider.png" alt=""><figcaption></figcaption></figure>

O resultado:

<figure><img src="../.gitbook/assets/param-spider-result.png" alt=""><figcaption></figcaption></figure>

Podemos usar o kxss para buscar por parâmetros que são refletidos e com isso acharmos alguma falha de XSS. https://github.com/tomnomnom/hacks



<figure><img src="../.gitbook/assets/kxss.png" alt=""><figcaption></figcaption></figure>

## Unindo ferramentas

Podemos primeiro enumerar os subdomínios da aplicação com o subfinder

<figure><img src="../.gitbook/assets/subfinder.png" alt=""><figcaption></figcaption></figure>

Depois usamos o httpx para fazer uma conexão em cada subdóminios e verificar se está ativo ou não.\
Podemos usar o parâmetro -path para testar diretórios que quisermos, exemplo: `-path /admin/`&#x20;



<figure><img src="../.gitbook/assets/httpx.png" alt=""><figcaption></figcaption></figure>

E agora podemos unir com o gau para descobrir quais urls existem dentro de cada subdomínio.

<figure><img src="../.gitbook/assets/httpx-gau.png" alt=""><figcaption></figcaption></figure>
