# Hashes on Linux

Antigamente, era utilizado o arquivo chamado **passwd** que fica localizado em **/etc/passwd** para armazenar e gerenciar as senhas dos usuários e serviços. Todos os usuários do sistema conseguem visualizar esse arquivo e antes as hashes eram armazenados nesse arquivo. Isso era um problema de segurança, pois todos usuários tinham acesso as demais contas no sistema. Então foi criado um outro formato para armazenar as hashes das senhas no arquivo **/etc/shadow** que só pode ser acessado pelo root.

Exemplo de hash do arquivo **shadow**.

```
user:$6$P/7DD1iRr9ifav$O3bZsRzchC6RErtd8ySMafSXhG0jvfcGkI0Bw/yCr7WWkNcUVi6qQLqKTvAtrdOVANYD69BnTVT7XHwh95e.:18805:0:99999:7:::
```

O linux utiliza uma função crypt para criptografar a senha. As partes desse hash é composto pelo método de encriptação, salt e a parte encriptada, seguindo o padrão. Podemos verificar no link

{% embed url="https://man7.org/linux/man-pages/man3/crypt.3.html" %}

```
$id$salt$encrypted
```

Podemos utilizar o john para tentar descobrir as senhas no linux. Para isso, precisamos copiar todo o conteúdo do arquivo passwd e shadow para dois novos arquivos para que o john consiga entender o padrão que será criado. Vamos utilizar uma ferramenta chamada unshadow para criar esse padrão. 

```
$ unshadow <passwd-file> <shadow-file> > hashes
```

Com isso, podemos executar o john.

```
$ john hashes
```