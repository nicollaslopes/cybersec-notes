# SSH

O SSH é um protocolo que serve para que o cliente e servidor troquem informações de uma forma segura.

Podemos pegar o banner que o servidor está utilizando com o netcat.

```
$ nc <ip> 22
```

Precisamos também saber quais são os possíveis métodos de autenticação que o servidor utiliza, com o parâmetro `-v`. Caso o ssh esteja configurado em outra porta sem ser a 22, podemos utilizar o parâmetro `-p` para especificar.

```
$ ssh -v <ip> -p 2222
```

## Autenticação com chave pública

Ao entrar dentro de uma rede interna, pode ser que tenhamos uma shell com comandos limitados. Com isso, uma solução que podemos recorrer, é a autenticação com chave pública do SSH.

Se quisermos alterar as configurações, o arquivo de configuração do SSH fica em `/etc/ssh/sshd_config`.

Dentro do diretório `/root` existe um diretório oculto do ssh chamado `.ssh` e dentro dele existem alguns arquivos. Um deles é o `authorized_keys` que possuem as chaves que permitem o acesso à máquina e o `known_hosts` que são os hosts conhecidos (se não houver esse diretório já criado, você terá que criar manualmente com `touch ~/.ssh/authorized_keys`). Com isso, para termos a nossa máquina acessando esse servidor via chave pública, precisaríamos criar uma chave pública e privada na nossa máquina e cadastrar a chave pública no servidor dentro do arquivo `authorized_keys`.

*(O SSH geralmente por padrão não permite conectar pelo root diretamente, para alterar isso, temos que achar no arquivo, a linha `PermitRootLogin prohibit-password` e alterar para `PermitRootLogin yes`. Lembrando que isso não é uma boa prática de segurança para quem está protegendo.)*

Para gerarmos essa chave, usamos o comando a seguir na nossa máquina. Ele escolhe um diretório padrão, podemos aceitar ou escolher um novo para ser salvo.

```
$ ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/nizyuu/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/nizyuu/.ssh/id_rsa
Your public key has been saved in /home/nizyuu/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:S4nSoyIfNlaQ56TyiTDRvFg1YkntF1W2KJ944fRgPqI nizyuu@ubuntu
The key's randomart image is:
+--[ED25519 256]--+
| .+oo  ...o      |
| +.+...  o .     |
|. *.o ..* .      |
| + B...X *       |
|= o +.* S .      |
|.= o + = o       |
|o O E   .        |
| = +             |
|  .              |
+----[SHA256]-----+

$ ls id*
id_rsa  id_rsa.pub
```

Com isso, temos a chave privada e pública, respectivamente. Basta copiar o conteúdo dentro de `id_rsa.pub` e inserir dentro do arquivo `authorized_keys` na máquina alvo.

```
$ echo "ssh-rsa AAAAC3NzaC1lZDI1NTE5AAAAILJ8QE2ORwyK6GYoyZGFsQJdRGcmub8OyCu1vogLjFJv nizyuu@ubuntu" >> authorized_keys
```

Depois de copiarmos, agora precisamos utilizar o comando a seguir na nossa máquina para ser identificado que iremos utilizar esse conjunto de chaves quando formos autenticar.

```
$ ssh-add id_rsa
```

Com isso, estamos prontos para nos autenticarmos ao servidor utilizando as chaves pública e privada.

```
$ ssh user@192.168.2.191
Linux kali 6.12.13-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.12.13-1kali1 (2025-02-11) x86_64

The programs included with the Kali GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Kali GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Jun  1 09:50:24 2025 from 192.168.2.107
┌──(user㉿kali)-[~]
└─$
```

Lembrando que muitas vezes não será possível se autenticar como root de acordo com as configurações do servidor. Uma solução é adicionar um usuário com `adduser` e fazer o mesmo processo, criando o arquivo `~/.ssh/authorized_keys` e colocando a chave pública dentro dele.
