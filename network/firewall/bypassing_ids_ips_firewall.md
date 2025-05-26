# Bypassing IDS/IPS/FIREWALL

Passar darmos um bypass em um IDS, IPS ou Firewall, precisamos ser o mais furtivo possível. Um dos jeitos de fazer isso, é colocarmos para scanear determinadas portas (ou top ports), passar um temporizador que irá enviar pacotes em tempo em tempo, simular vários pacotes falsos com ip falsos para eles irem para a lista de bloqueio ao invés do nosso real ip, entre outros.

O parâmetro **-T** é referente ao tempo que irá ser enviado os pacotes, sendo -T5 o mais rápido e barulhento até o -T0 o mais lento (o nmap utiliza o -T3 por padrão e o -T0 leva 15 minutos). Usar essa opção seria interessante se caso houvesse uma regra no alvo que filtra as requisições por tempo, ou seja, se um atacante enviar vários pacotes em um espaço de tempo muito curto, irá ser bloqueado.

Se apenas scanearmos todas as portas, nosso ip irá para essa lista de bloqueio. Porém, passando apenas determinadas portas, podemos ter sucesso no scan sem sermos bloqueados.

```
$ sudo nmap -sS --top-ports=10 --open -Pn 192.168.2.134
Starting Nmap 7.80 ( https://nmap.org ) at 2021-07-08 16:59 -03
Nmap scan report for 192.168.2.134
Host is up (0.00039s latency).
Not shown: 8 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:47:FB:50 (Oracle VirtualBox virtual NIC)
```

Nesse caso, funcionou pois na regra do portsentry essas portas não estavam para serem analisadas. Vale lembrar que se ele tivesse configurado para analisar todas as portas, exceto os serviços reais, ele iria pegar. Dessa forma, essa técnica não é 100% e há chance de não pegar todas os serviços, ainda mais se estiver em uma porta alta. Outra técnica, é utilizar o parâmetro **-D** para ir outros pacotes com o que iremos enviar. Precisamos também passar os IPS falsos para enganar o filtro.

```
$ sudo nmap -v -D 10.10.10.10,192.168.2.22,172.17.2.13 -sS --top-ports=25 --open -Pn 192.168.2.112
```

Ou

```
$ sudo nmap -v -D RND:20 -sS --top-ports=25 --open -Pn 192.168.2.134

Starting Nmap 7.80 ( https://nmap.org ) at 2021-07-08 19:19 -03
Initiating ARP Ping Scan at 19:19
Scanning 192.168.2.134 [1 port]
Completed ARP Ping Scan at 19:19, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 19:19
Completed Parallel DNS resolution of 1 host. at 19:19, 0.00s elapsed
Initiating SYN Stealth Scan at 19:19
Scanning 192.168.2.134 [25 ports]
Discovered open port 22/tcp on 192.168.2.134
Discovered open port 80/tcp on 192.168.2.134
Discovered open port 111/tcp on 192.168.2.134
Discovered open port 143/tcp on 192.168.2.134
Completed SYN Stealth Scan at 19:19, 0.05s elapsed (25 total ports)
Nmap scan report for 192.168.2.134
Host is up (0.0048s latency).
Not shown: 21 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
111/tcp open  rpcbind
143/tcp open  imap
MAC Address: 08:00:27:47:FB:50 (Oracle VirtualBox virtual NIC)
```

Com isso o iptables irá apenas colocar na lista de bloqueio os IPs que passamos, desconsiderando assim o original, que é o nosso.