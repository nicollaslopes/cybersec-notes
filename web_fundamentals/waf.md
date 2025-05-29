# WAF (Web Application Firewall)

Before we talk about WAF, we need to know what the Firewall is used to control a network, a server, a service inside a corporate network, or a cloud or a local network.

WAF helps protect web applications by filtering and monitoring HTTP traffic between the web application and the internet. It generally protect the web applications against attacks such as cross-site request forgery, cross-site scripting (XSS), file inclusions, SQL injection, etc.

By deploying a WAF in front of the web application, a shield is places between the application and the internet. While a proxy server protects the identity of the client machine using an intermediary, WAF is a type of reverse proxy that protects the server from exposure, as requests pass through the WAF before reaching the server. 

## Identifying WAF

Uma outra forma de identificarmos um WAF, é utilizando uma ferramenta chamada `wafw00f`.

```
$ wafw00f desecsecurity.com        

                   ______
                  /      \
                 (  Woof! )
                  \  ____/                      )
                  ,,                           ) (_
             .-. -    _______                 ( |__|
            ()``; |==|_______)                .)|__|
            / ('        /|\                  (  |__|
        (  /  )        / | \                  . |__|
         \(_)_))      /  |  \                   |__|

                    ~ WAFW00F : v2.1.0 ~
    The Web Application Firewall Fingerprinting Toolkit
    
[*] Checking https://desecsecurity.com
[+] The site https://desecsecurity.com is behind GoDaddy Website Protection (GoDaddy) WAF.
```

```
$ wafw00f https://www.serasa.com.br  

                ______
               /      \
              (  W00f! )
               \  ____/
               ,,    __            404 Hack Not Found
           |`-.__   / /                      __     __
           /"  _/  /_/                       \ \   / /
          *===*    /                          \ \_/ /  405 Not Allowed
         /     )__//                           \   /
    /|  /     /---`                        403 Forbidden
    \\/`   \ |                                 / _ \
    `\    /_\\_              502 Bad Gateway  / / \ \  500 Internal Error
      `_____``-`                             /_/   \_\

                        ~ WAFW00F : v2.1.0 ~
        The Web Application Firewall Fingerprinting Toolkit
    
[*] Checking https://www.serasa.com.br
[+] The site https://www.serasa.com.br is behind Cloudfront (Amazon) WAF.
[~] Number of requests: 2
```