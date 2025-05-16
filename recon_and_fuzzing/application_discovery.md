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

image ffuf
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

```
$ sudo docker run -it 5aea64de6803 -d testphp.vulnweb.com        

           
                                      _    __       
   ___  ___ ________ ___ _  ___ ___  (_)__/ /__ ____
  / _ \/ _ `/ __/ _ `/  ' \(_-</ _ \/ / _  / -_) __/
 / .__/\_,_/_/  \_,_/_/_/_/___/ .__/_/\_,_/\__/_/   
/_/                          /_/                    

                              with <3 by @0xasm0d3us           
    
[INFO] Fetching URLs for testphp.vulnweb.com
[INFO] Found 11394 URLs for testphp.vulnweb.com
[INFO] Cleaning URLs for testphp.vulnweb.com
[INFO] Found 860 URLs after cleaning
[INFO] Extracting URLs with parameters
[INFO] Saved cleaned URLs to results/testphp.vulnweb.com.txt
```

O resultado:

```
$sudo cat /var/lib/docker/overlay2/c87ca61fc220f9a7b9e1ccad23fa10350de07/diff/app/paramspider/results/testphp.vulnweb.com.txt         

http://testphp.vulnweb.com/index.zipA01http://testphp.vulnweb.com/listproducts.php?artist=FUZZ
http://testphp.vulnweb.com/listproducts.php?artist=FUZZ
http://testphp.vulnweb.com/search.php?%3Cscript%3Eonerror=FUZZ
http://testphp.vulnweb.com/listproducts.php?artist=FUZZ&amp%3Basdf=FUZZ&amp%3Bcat=FUZZ
http://testphp.vulnweb.com/search?q=FUZZ
http://testphp.vulnweb.com/?trk=FUZZ
http://testphp.vulnweb.com/Mod_Rewrite_Shop/buy.php?id=FUZZ
http://testphp.vulnweb.com/artists.php?%CB%93%E2%86%92artist=FUZZ
http://testphp.vulnweb.com/AJAX/infoartist.php?id=FUZZ&LDeO=FUZZ
http://testphp.vulnweb.com/listproducts.php?id=FUZZ
http://testphp.vulnweb.com/showimage.php?%EF%AC%81le=FUZZ&size=FUZZ
http://testphp.vulnweb.com/AJAX/infoartist.php?id=FUZZ&HlhS=FUZZ
http://testphp.vulnweb.com/listproducts.php?artist=FUZZ&asdf=FUZZ&cat=FUZZ
...
```

Podemos usar o kxss para buscar por parâmetros que são refletidos e com isso acharmos alguma falha de XSS. https://github.com/tomnomnom/hacks

```
$ cat result.txt | kxss

param p is reflected and allows " on http://testphp.vulnweb.com/hpp/params.php?aaaa%2F=FUZZ&p=FUZZ&pp=FUZZ
param pp is reflected and allows " on http://testphp.vulnweb.com/hpp/index.php?pp=FUZZ
param pp is reflected and allows " on http://testphp.vulnweb.com/hpp/params.php?aaaa%2F=FUZZ&p=FUZZ&pp=FUZZ
param p is reflected and allows " on http://testphp.vulnweb.com/hpp/params.php?p=FUZZ&pp=FUZZ
param p is reflected and allows ' on http://testphp.vulnweb.com/hpp/params.php?aaaa%2F=FUZZ&p=FUZZ&pp=FUZZ
param pp is reflected and allows " on http://testphp.vulnweb.com/hpp/params.php?p=FUZZ&pp=FUZZ
param pp is reflected and allows ' on http://testphp.vulnweb.com/hpp/index.php?pp=FUZZ
...
```

## Unindo ferramentas

Podemos primeiro enumerar os subdomínios da aplicação com o subfinder

```
$ subfinder -d businesscorp.com.br -o subdomains.txt

               __    _____           __         
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /    
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

                projectdiscovery.io

[INF] Current subfinder version v2.7.1 (latest)
[INF] Loading provider config from /home/user/.config/subfinder/provider-config.yaml
[INF] Enumerating subdomains for businesscorp.com.br
dev.businesscorp.com.br
srvkey.businesscorp.com.br
www.businesscorp.com.br
ns1.businesscorp.com.br
piloto.businesscorp.com.br
intranet.businesscorp.com.br
mail.businesscorp.com.br
rh.businesscorp.com.br
ns2.businesscorp.com.br
parsingok.businesscorp.com.br
```

Depois usamos o httpx para fazer uma conexão em cada subdóminios e verificar se está ativo ou não. 
Podemos usar o parâmetro -path para testar diretórios que quisermos, exemplo: `-path /admin/`

```
$ cat subdomains.txt | httpx -status-code

    __    __  __       _  __
   / /_  / /_/ /_____ | |/ /
  / __ \/ __/ __/ __ \|   /
 / / / / /_/ /_/ /_/ /   |
/_/ /_/\__/\__/ .___/_/|_|
             /_/

                projectdiscovery.io

[INF] Current httpx version v1.7.0 (latest)
[WRN] UI Dashboard is disabled, Use -dashboard option to enable
https://dev.businesscorp.com.br [301]
http://rh.businesscorp.com.br [200]
http://ns1.businesscorp.com.br [200]
http://intranet.businesscorp.com.br [200]
http://parsingok.businesscorp.com.br [200]
http://mail.businesscorp.com.br [200]
http://www.businesscorp.com.br [200]
```

E agora podemos unir com o gau para descobrir quais urls existem dentro de cada subdomínio.

```
cat subdomains.txt | httpx | gau
WARN[0000] error reading config: Config file /home/user/.gau.toml not found, using default config 

    __    __  __       _  __
   / /_  / /_/ /_____ | |/ /
  / __ \/ __/ __/ __ \|   /
 / / / / /_/ /_/ /_/ /   |
/_/ /_/\__/\__/ .___/_/|_|
             /_/

                projectdiscovery.io

[INF] Current httpx version v1.7.0 (latest)
[WRN] UI Dashboard is disabled, Use -dashboard option to enable
http://dev.businesscorp.com.br/
http://mail.businesscorp.com.br:80/
http://mail.businesscorp.com.br/.php
http://mail.businesscorp.com.br/.phtml
http://mail.businesscorp.com.br/favicon.ico
http://mail.businesscorp.com.br/icons/openlogo-75.png
http://mail.businesscorp.com.br/index.php
http://mail.businesscorp.com.br/robots.txt
http://mail.businesscorp.com.br/server-status
http://mail.businesscorp.com.br/squirrelmail/images/sm_logo.png
http://mail.businesscorp.com.br/squirrelmail/src/login.php
http://mail.businesscorp.com.br/squirrelmail/src/printer_friendly_main.php?passed_ent_id=0&mailbox=INBOX&passed_id=1&view_unsafe_images=
http://mail.businesscorp.com.br/squirrelmail/src/printer_friendly_main.php?passed_ent_id=0&mailbox=INBOX&passed_id=2&view_unsafe_images=
http://mail.businesscorp.com.br/squirrelmail/src/webmail.php
http://mail.businesscorp.com.br/squirrelmail/src/webmail.php?right_frame=folders.php
http://mail.businesscorp.com.br/squirrelmail/src/webmail.php?right_frame=read_body.php
http://mail.businesscorp.com.br/squirrelmail/src/webmail.php?right_frame=right_main.php
http://parsingok.businesscorp.com.br:80/
http://parsingok.businesscorp.com.br:80/css/default.css
```

