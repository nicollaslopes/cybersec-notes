# Application Discovery

## URL Discovery

Para fazermos uma busca por arquivos e diretórios, podemos utilizar algumas ferramentas para fazer o `fuzzing`.

### Gau

O gau utiliza motores de pesquisa (ex: google, shodan, censys, waybackmachine), compila essas informações e nos retorna as urls que foi descobertas nessas buscas.

<figure><img src="../.gitbook/assets/gau.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/lc/gau/releases" %}

### ffuf

https://github.com/ffuf/ffuf

<div align="right" data-full-width="false"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

### wfuzz

<figure><img src="../.gitbook/assets/wfuzz.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/xmendez/wfuzz" %}

## Content Discovery

### httpx

<figure><img src="../.gitbook/assets/httpx.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/projectdiscovery/httpx/releases" %}

### Aquatone

<figure><img src="../.gitbook/assets/aquatone.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/michenriksen/aquatone" %}

## Parameter Discovery

### Param Spider

Podemos rodar o param spider usando o docker, para conseguirmos achar parâmetros em uma aplicação

<figure><img src="../.gitbook/assets/param-spider.png" alt=""><figcaption></figcaption></figure>

O resultado:

<figure><img src="../.gitbook/assets/param-spider-result.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/devanshbatham/ParamSpider" %}

## Unindo ferramentas

Podemos primeiro enumerar os subdomínios da aplicação com o subfinder

<figure><img src="../.gitbook/assets/subfinder.png" alt=""><figcaption></figcaption></figure>

Depois usamos o httpx para fazer uma conexão em cada subdóminios e verificar se está ativo ou não.\
Podemos usar o parâmetro -path para testar diretórios que quisermos, exemplo: `-path /admin/`&#x20;



<figure><img src="../.gitbook/assets/httpx (1).png" alt=""><figcaption></figcaption></figure>

E agora podemos unir com o gau para descobrir quais urls existem dentro de cada subdomínio.

<figure><img src="../.gitbook/assets/httpx-gau.png" alt=""><figcaption></figcaption></figure>

Podemos usar o kxss para buscar por parâmetros (retornados no param spider)que são refletidos e com isso acharmos alguma falha de XSS.

<figure><img src="../.gitbook/assets/kxss.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/tomnomnom/hacks" %}
