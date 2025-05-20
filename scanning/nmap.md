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

