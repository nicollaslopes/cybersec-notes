# LFI - Local File Inclusion

Essa vulnerabilidade ocorre quando a aplicação permite que o usuário inclua algum arquivo. Vamos supor que no código fonte há um include que pega por parâmetro via GET e inclui esse arquivo.

```
include($_GET['file']);
```

# De LFI a RCE - Infecção de Logs

Uma vez que conseguimos visualizar arquivos do lado do servidor, podemos aproveitar disso para conseguir uma RCE através da leitura dos arquivos de logs infectando o mesmo com algum código malicioso.

Podemos então enviar uma requisição para o servidor (caso a aplicação seja em php) da seguinte forma:

```
http://vuln.app/<?php system($_GET['shell']); ?>
```

Com isso, ao lermos o arquivo de log, podemos ver o código que inserimos. Como estamos através do LFI, e essa página interpreta o PHP, ele encontra o nosso código em PHP no log e tenta executar o comando que enviamos.

