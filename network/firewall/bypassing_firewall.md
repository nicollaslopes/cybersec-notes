#  Bypassing Firewall

## Source Port Manipulation

Um dos grandes problemas ao scanear um host atrás de portas abertas, é quando o firewall bloqueia essa tentativa. Porém há maneiras mesmo havendo um firewall bloqueando, bypassar essa regra. Vamos supor que há um firewall bloqueando determinadas portas ao fazer um scan pelo nmap, ao fazermos isso não conseguimos receber a resposta dizendo que a porta está aberta. Uma técnica que pode ser feito para burlar isso, seria utilizar uma porta de origem, o tráfego de alguma determinada porta de origem pode estar sendo permitida pelo firewall. Uma das portas que poderíamos tentar: 53, 80, 443, etc. Se o firewall não foi muito bem configurado, podemos usar isso. Partindo pra prática, vamos rodar o nmap especificando a porta de origem que queremos utilizar.

Primeiro, veremos a regra do firewall que está configurada.

```
root@server5:/home/msfadmin/firewall# cat rules4.sh 
#!/bin/bash

# Ativar forward
sysctl -w net.ipv4.ip_forward=1

# Acertar as políticas
iptables -P INPUT DROP
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

# Permintir entrada de tráfego
iptables -A INPUT -i eth0 -p tcp -m tcp --dport 22 -j ACCEPT 
iptables -A INPUT -i eth0 -p tcp -m tcp --dport 80 -j ACCEPT 
iptables -A INPUT -d 172.16.1.5 -i eth0 -p tcp -m tcp --sport 53 --dport 1024:65535 -j ACCEPT 
iptables -A INPUT -d 172.16.1.5 -i eth0 -p tcp -m tcp --sport 80 --dport 1024:65535 -j ACCEPT 
iptables -A INPUT -d 172.16.1.5 -i eth0 -p tcp -m tcp --sport 443 --dport 1024:65535 -j ACCEPT 
iptables -A OUTPUT -o eth0 -p tcp -m tcp --sport 1024:65535 --dport 80 -j ACCEPT 
iptables -A OUTPUT -o eth0 -p tcp -m tcp --sport 1024:65535 --dport 443 -j ACCEPT 
iptables -A OUTPUT -o eth0 -p tcp -m tcp --sport 1024:65535 --dport 53 -j ACCEPT 

# Mascaramento
iptables -t nat -A POSTROUTING -o 172.16.1.1/24 -j MASQUERADE
```

Aplicando essa regra, e executando o nmap temos o seguinte resultado.

```
$ sudo nmap -sS -Pn 172.16.1.5
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-03 10:44 EDT
Nmap scan report for 172.16.1.5
Host is up (0.21s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 13.87 seconds
```

Apenas a porta 22 e 80 aparece aberta. Agora, passando uma porta de origem, podemos burlar isso.

```
$ sudo nmap -sS -Pn -g 53 172.16.1.5

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-03 10:57 EDT
Nmap scan report for 172.16.1.5
Host is up (0.21s latency).
Not shown: 979 closed ports
PORT     STATE    SERVICE
21/tcp   open     ftp
22/tcp   open     ssh
23/tcp   open     telnet
25/tcp   open     smtp
53/tcp   open     domain
80/tcp   filtered http
111/tcp  open     rpcbind
139/tcp  open     netbios-ssn
445/tcp  open     microsoft-ds
512/tcp  open     exec
513/tcp  open     login
514/tcp  open     shell
1099/tcp open     rmiregistry
1524/tcp open     ingreslock
2049/tcp open     nfs
3306/tcp filtered mysql
5900/tcp open     vnc
6000/tcp open     X11
6667/tcp open     irc
8009/tcp open     ajp13
8180/tcp open     unknown

Nmap done: 1 IP address (1 host up) scanned in 7.99 seconds
```

Com isso, conseguimos atingir serviços que antes não conseguimos, enviando a porta de origem junto.

Mas ao tentarmos conectar pelo browser na porta 8180, por exemplo, ele não nos mostra nada, fica apenas carregando a página. O motivo disso é porque o firewall está bloqueando, mas ainda há uma maneira de identificarmos o que está rodando no serviço utilizando o **netcat**, passando também uma porta de origem para burlar o filtro e por fim salvando num arquivo.

```
$ nc -vn -p 53 172.16.1.5 8180 > resultado.html
(UNKNOWN) [172.16.1.5] 8180 (?) open
GET / HTTP/1.0
```

Abrindo esse arquivo, podemos ver as informações do servidor.




https://nmap.org/book/firewall-subversion.html