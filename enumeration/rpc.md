# RPC

Existe um utilitário chamado RPCCLIENT no linux que permite executar funções MS-RPC em um host windows onde podemos obter informações sobre os usuários e o host.

Podemos usar da seguinte forma para conectar (o parâmetro -U informa o usuário que nesse caso é vazio e o -N para não informar a senha).

```
$ rpcclient -U "" -N <ip>
```

Caso o usuário pertença a um domínio, deverá ser usado da seguinte forma.

```
$ rpcclient -U "" -N -W <domain> <ip>
```

Usamos o ponto de interrogação (?) para verificar todos os comandos que podemos executar.

Para enumerar todos os usuários: **enumdomuser**


Para enumerar todos os grupos: **enumdomgroups**


Se quisermos informações de determinado usuário: **queryuser root**


Se quisermos ver os compartilhamentos: **netshareenumall**