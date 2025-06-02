# POP3

O POP3, que utiliza a porta 110, é um protocolo utilizado para caixa de entrada de e-mail. Podemos interagir com o serviço POP3 utilizando o utilitário telnet para isso.

```
$ telnet <ip> 110
```

Uma vez conectado, irá ser necessário passar um usuário e senha.

```
USER <usuário>

PASS <senha>
```

Podemos usar o STAT para exibir o status (quantas mensagens) e o LIST para listar todas as mensagens. Para lermos uma mensagem, precisamos digitar RETR <número> (escolher o número da mensagem).