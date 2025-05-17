# Firewall

O firewall é uma solução de segurança que permite criar regras para monitorar e/ou bloquear a entrada e saída de uma rede. O firewall no Linux vem por padrão aceitando qualquer entrada e saída, você deverá configurar da forma que for necessária.

A forma mais "correta" de usar o firewall é bloquear tudo e liberar manualmente apenas o que for necessário. Nos exemplos a seguir, vamos utilizar o `iptables`. Para visualizar como estão as regras do firewall, utilizamos o comando a seguir.

```
root@server5:/# iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```

Este é um caso onde está mal configurado, estão todas com regras para aceitar o tráfego. Se quisermos aplicar a política para dropar tudo na entrada, utilizamos o comando a seguir.

```
root@server5:/# iptables -P INPUT DROP
```

Vale lembrar que é bloqueado qualquer tipo de pacote, se tentarmos pingar, por exemplo, não irá funcionar. Para liberarmos determinadas portas, fazemos dessa forma.

```
root@server5:/# iptables -A INPUT -p TCP --dport 80 -j ACCEPT
```

Para aplicar a regra para especificamente um host utilizamos o comando a seguir.

```
root@server5:/# iptables -A INPUT -p TCP --dport 80 -s <ip> -j ACCEPT
```

Para bloquear determinada porta, utilizamos.

```
root@server5:/# iptables -A INPUT -p tcp --dport 80 -j DROP
```

Para apagar todas as regras, basta utilizar.

```
root@server5:/# iptables -F
```
