# Nmap: The Network Mapper

O Nmap, é um software open source que realiza ports scan em uma rede. É o port scan mais utilizado e de longe a melhor ferramenta para ser utilizada nesse quesito.

Este é um exemplo de como podemos scanear uma rede inteira:

sudo nmap -sn 192.168.1.0/24

Para identificar se uma porta em um host está aberta, é necessário estabelecer o three-way-handshake. Funciona da seguinte forma, o cliente envia uma flag SYN, o servidor responde com um SYN/ACK e o cliente finaliza com uma flag ACK, uma troca de dados TCP é dessa forma. Com isso, a conexão é bem sucedida. O ACK irá fechar o 3WH para estabelecer a conexão, mas a partir do momento que você recebe o SYN/ACK, o serviço está ativo (a porta está aberta).

Quando o serviço está inativo, o cliente envia uma flag SYN e o servidor envia uma flag RST/ACK como resposta. Ou seja, a porta está fechada.

RESPOSTAS

```
PORTA ABERTA                               RESPONDE COM SYN / ACK (SA)
PORTA FECHADA                              RESPONDE COM RST / ACK (RA)
PORTA COM FILTRO DE FIREWALL EM DROP       SEM RESPOSTA
PORTA COM FILTRO DE FIREWALL EM REJECT     RESPONDE COM UM ICMP PORT UNREACHABLE
```

## Diferenças entre os tipos de scan

O TCP connect, consome muito tráfego em uma rede, visto que é necessário completar o three-way-handshake cada vez que for se comunicar, então acaba sendo muito barulhento. O Half Open / Syn Scan já consome menos tráfego numa rede, visto que ao invés de estabelecer o 3WH, ele envia no final uma flag RST encerrando a comunicação, sendo assim bem menos barulhento e consumindo menos tráfego em uma rede. Veremos isso na prática. Para isso, abri a porta 4444 em minha máquina para realizar os testes de scan.

Scaneando nessa porta utilizando o tcp connect.

```
$ sudo nmap -sT -p 4444 192.168.2.134
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-31 09:47 EDT
Nmap scan report for 192.168.2.134
Host is up (0.00020s latency).

PORT STATE SERVICE
4444/tcp open krb524

Nmap done: 1 IP address (1 host up) scanned in 0.26 seconds
```

Assim que o scan termina, percebe-se que a conexão foi encerrada.

```
$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.2.134] from (UNKNOWN) [192.168.2.134] 60814
```

Agora, abrindo a porta novamente e realizando um Syn Scan. Podemos perceber que a conexão não foi finalizada e não foi gerado log.

```
$ sudo nmap -sS -p 4444 192.168.2.134
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-31 09:49 EDT
Nmap scan report for 192.168.2.134
Host is up (0.000082s latency).

PORT STATE SERVICE
4444/tcp open krb524

Nmap done: 1 IP address (1 host up) scanned in 0.06 seconds
```

```
$ nc -nlvp 4444
listening on [any] 4444 ...
```

Com isso percebemos a diferença, o syn scan para realizar scan é mais silencioso.

## Analisando o consumo de um Scan

Ao realizarmos um scan, temos que ficar atento a quantidade que ele irá consumir de uma rede, é ideal que o quanto menos for, melhor. Para isso, podemos utilizar o iptables para analisar a quantidade de pacotes que será enviado/recebido em um scan.

Temos que configurar para visualizar os pacotes que entram e sai da própria máquina. Aplicando a regra para aceitar a entrada e saída e logo após isso exibindo as informações.

```
$ iptables -A INPUT -s 192.168.2.134 -j ACCEPT
$ iptables -A OUTPUT -d 192.168.2.134 -j ACCEPT
```

```
$ iptables -nvL              
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination
    0     0 ACCEPT     all  --  *      *       192.168.2.134       0.0.0.0/0 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination 
    0     0 ACCEPT     all  --  *      *       0.0.0.0/0           192.168.2.134
```

Com isso, podemos fazer o scan para realizar o teste. Vamos começar com o TCP connect.

```
$ sudo nmap -sT -p 80 -Pn 192.168.2.134
```

```
$ iptables -nvL                                
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination         
    4   224 ACCEPT     all  --  *      *       192.168.2.134       0.0.0.0/0    

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination     
Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination
    4   224 ACCEPT     all  --  *      *       0.0.0.0/0           192.168.2.134
```

Foram enviados 4 pacotes, com tamanho de 224 bytes.

Para continuar os testes, precisamos zerar as configurações porque ele irá somar as quantidades anteriores. Utilizamos **iptables -Z** para zerar.

Agora, vamos utilizar o Syn Scan, ou Half Open.

```
$ sudo nmap -sS -p 80 -Pn 192.168.2.134
```

```
$ iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination
    3   128 ACCEPT     all  --  *      *       192.168.2.134       0.0.0.0/0 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source              destination
    3   128 ACCEPT     all  --  *      *       0.0.0.0/0           192.168.2.134
```

Foram enviados 3 pacotes, com mesmo tamanho de 128 bytes.

Podemos perceber que dependendo não é vantajoso utilizar o TCP scan daquela primeira forma. O mais ideal seria criar um personalizado com determinadas portas ou definir um número determinado usando o parâmetro `--top-ports 250`. Assim haverá bem menos tráfego e será enviado uma quantidade muito menor de bytes.

```
$ nmap -sS -p 21,25,80,443,8080,3306 -Pn <ip>
```


*Obs: Quando formos fazer um scan em um host, temos que ficar atento para fazer um scan completo, visto que uma porta pode ter seu número alterado e por padrão, o nmap só varre as 1000 principais. Com isso, se não especificarmos com o parâmetro `-p-` ou `-p 1-65535`, ele não irá achar uma porta específica. Ex: alterando a porta do SSH para 22999, com o scan padrão ele não irá encontrar.*

Também é interessante fazer uma varredura por UDP, porque pode haver alguns recursos que estão rodando por UDP.

```
PORTA ABERTA                               SEM REPOSTA
PORTA FECHADA                              PORT UNREACHABLE
PORTA COM FILTRO DE FIREWALL EM DROP       SEM RESPOSTA
PORTA COM FILTRO DE FIREWALL EM REJECT     PORT UNREACHABLE
```

Mas há uma ambiguidade no UDP, porque como vemos acima, quando enviamos um pacote para a porta UDP e ela estiver aberta, vai ter o mesmo comportamento quando ela está configurada em DROP no firewall. Caso ela esteja fechada, terá o mesmo comportamento se estiver configurada em REJECT no firewall. Como podemos ter certeza então se está aberta ou não? Podemos então, ao scanear com o nmap, passar o parâmetro -V para interagir e enumerar os serviços, rodando os scripts NSE que ele possui.

```
$ nmap -v -sUV -p 161 <ip>
```