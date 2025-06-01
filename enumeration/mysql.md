# MySQL

O MySQL é um sistema de gerenciamento de banco de dados que roda na porta 3306.

Para conectar utilizamos o próprio utilitário do mysql. Para entrar com usuário e senha.

```
$ mysql -h <host> -u <user> -p <password>
```

Uma coisa a se fazer, é ver se o sistema está permitindo entrar com credenciais padrão. Caso quisermos conectar como root sem senha.

```
$ mysql -h <host> -u root
```

Uma vez conectado, podemos utilizar o comando `show databases;` para exibir as base de dados.

Se quisermos utilizar alguma, usamos `use <table>`

Para vermos as tabelas `show tables;`

Para vermos o conteúdo da tabela `select * from <table>`