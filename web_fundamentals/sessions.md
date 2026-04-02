# Sessions

## What are sessions? How do they work?

As sessoes sao uma forma de armazenar as informacoes de um usuario no server-side. Essas informacoes serao utilizadas conforme o usuario for navegando pela aplicacao.

When a user logs into the website, a session is created. In this session, you can created variable called “session variable” that store data in a key/value format ( like cookies ).

Em php, precisamos iniciar a sessao utilizando a funcao `session_start()` e podemos criar um determinado valor para uma sessao utilizando a variavel global `$_SESSION`, como no exemplo abaixo.

```php
<?php

session_start();

$_SESSION["name"] = "myname";
```

Com isso, ao acessarmos o browser, podemos ver na aba Application que a sessao esta salva no `PHPSESSID`.

sessions-1.png

A partir do momento que a sessao eh atribuida ao usuario, ela persiste ate que seja manualmente apagada ou expirar. Ao abrir uma aba anônima, não se acessa a mesma sessão porque essa aba não possui o cookie `PHPSESSID` da navegação normal, sendo assim criado uma nova.

Sabendo disso, podemos pensar onde que essa informacao fica salva para conseguir ser persistente? Como foi dito no inicio, essa informacao fica salvo no server-side, porem se quisermos saber exatamente onde eh salva (no caso de uma aplicacao em PHP), podemos acessar o diretorio `/var/lib/php/sessions`, que esta salvo todas as sessoes existentes:

```
/var/lib/php/sessions# ls
sess_frfi6k5j9ftddrkua2m00mnv56
```

Se dermos um cat nesse arquivo, podemos ver que fica salvo um objeto PHP serializado

```
/var/lib/php/sessions# cat sess_frfi6k5j9ftddrkua2m00mnv56 
name|s:6:"myname";
```

