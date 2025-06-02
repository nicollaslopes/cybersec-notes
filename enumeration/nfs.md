# NFS

O NFS (Network File System) é um sistema de compartilhamento de arquivos e diretórios entre computadores em uma rede que funciona na porta 2049.

Podemos ver as versões suportadas com o comando a seguir.

```
$ rpcinfo -p 172.16.1.5 | grep "nfs"
    100003    2   udp   2049  nfs
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100003    2   tcp   2049  nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
```

Nesse caso é suportado a versão 2, 3 e 4. Para vermos os compartilhamentos exportados de um host, utilizamos o comando a seguir. A raíz está sendo compartilhada nesse exemplo.

```
$ showmount -e 172.16.1.5
Export list for 172.16.1.5:
/ *
```

Nesse outro exemplo, está sendo compartilhado /home/camila. Uma vez que descobrimos esses diretórios, precisamos montar eles para conseguirmos acessar.

```
$ showmount -e 172.16.1.31
Export list for 172.16.1.31:
/home/camila *
```

Vamos criar uma pasta na nossa máquina chamada /tmp/nfs para montarmos dentro dela.

```
$ sudo mount -t nfs -o nfsvers=2 172.16.1.5:/ /tmp/nfs

$ sudo mount -t nfs -o nfsvers=2 172.16.1.31:/home/camila /tmp/nfs
```

Agora basta acessar a pasta que foi montada para acessar os arquivos. Uma vez que o diretório foi montado, podemos manipular os arquivos, com isso uma das possibilidades é manipular chaves ssh para ganhar acesso ao sistema.

Se quisermos desconectar, basta utilizarmos o seguinte comando.

```
$ umount nfs
```