# FTP

Para interagirmos com o serviço de FTP, podemos utilizar o netcat para fazer a conexão ou também o utilitário do ftp (não precisa passar a porta se for a porta padrão 21).

```
$ nc -vn <host> 21
```

```
$ ftp <host>
```

Com isso ele irá te pedir o nome do usuário e senha.

Um processo de enumeração é tentar logar com o usuário `anonymous` ou `ftp`. Alguns administradores configuram para aceitar logar com esse usuário, que possui privilégios baixos, não é uma boa prática utilizar isso. Então, é interessante tentar entrar com usuário e senha `anonymous` ou também `ftp` e verificar se é possível.

Se caso conseguirmos conectar com sucesso e executarmos algum comando, por exemplo "ls", ele não irá responder, porque há um firewall barrando no meio do caminho, a configuração padrão do FTP funciona de forma ativa, então precisamos antes entrar no modo passivo para interagir com o serviço.

Podemos utilizar **help** para verificar as opções. Para entrar no modo passivo basta utilizar `passive` ou `pasv`. Agora pode-se executar os comandos sem problemas!