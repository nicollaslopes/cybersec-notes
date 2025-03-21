# Route

Roteamento é quando uma máquina com múltiplas conexões de rede decide onde entregar os pacotes IP que recebeu. Para que assim os pacotes cheguem ao seu destino. 

Routing is when a machine with multiple network connections decides where to deliver the IP packets it has received. So that the packets reach their destination. 

## How does routing work?

The `192.168.0.1` network can't communicate with `10.10.0.0`, how does that work?

Different networks can communicate with each other through a Router. The router has interfaces that make this possible. In most cases, the router is the network gateway. So the gateway is responsible for a network `192.168.0.0` being able to communicate with another `10.10.0.0`.

"Computer A" wants to communicate with the Web Server `10.10.0.50`, but it has access to this network. Therefore, the gateway will be responsible for sending the packet to its network interface that has access to another network.

192.168.0.62 (Computer A) -> 192.168.0.1 (Router)
192.168.0.1 (Router) -> 10.10.01 (Router)
10.10.01 (Router) -> 10.10.0.50 (Web Server)

We can see this entire communication process using the `traceroute` command. For example, we can see the communication with a Cloudflare server.

```
$ traceroute 1.1.1.1
```


No meio dessa comunicação, existe um firewall, que basicamente serve para fazer o controle dentro de uma rede, permitindo, negando ou bloqueando acessos. 
Nesse exemplo abaixo, podemos "simular" uma regra no firewall onde o computador A consegue acesso ao servidor do banco de dados porque existe uma regra no firewall que permite esse acesso

Obs: não existe motivo colocar alguma regra de firewall de borda dentro de computadores da mesma rede (ex: notebook comunicando com computador A) porque isso não será filtrado pelo firewall, ou seja, a comunicação é feito direto entre eles.


Cada um dos computadores, possuem um route table

Se quiser ver a tabela de roteamento:

ip route netstat -r 



Para conseguirmos acesar uma determinada rede na internet, como por exemplo a rede do google, acessando o próprio site do google, ou o facebook e etc, precisamos de um provedor de internet, podemos chamar de ISP (Internet Service Provider), que irá fazer esse trabalho de conectar a nossa rede com outras redes.

No nosso computador temos uma route table, no exemplo, 0.0.0.0/0 significa todos os endereços, ou seja, tudo que for acessado, irá ser encaminhado para o roteador (192.168.0.1).

Quando fazemos uma requisição para o google, o roteador também não sabe qual o endereço do google porque ele não tem na sua route table. Com isso, ele irá enviar a requisição para o provedor (isp).

O ISP irá receber uma notificação do google, assim o google avisa pro provedor (vivo, claro), que se caso ele quiser comunicar com a rede (100.100.250.0/24) precisa enviar enviar
a requisição para a entrada da rede do google (100.100.250.1)

Essa comunicação entre as redes públicas, nós chamamos de sistemas autônomos (AS), porque a comunicação é  automática, não existe ninguém configurando o BGP.

Porém, esse processo é feito por várias rotas, não é diretamente do provedor para a rede do google, podemos traçar a rota com o comando traceroute google.com

E podemos consultar no https://ipinfo.io/ cada ip


Comunicação:

192.168.0.62 -> 100.100.250.130

100.100.250.130 -> 192.168.0.1
100.100.250.130 -> 2.2.2.1

Podemos ver todas as redes  da claro, que é um provedor no site; https://bgp.he.net/AS4230#_prefixes

A claro tem conexão direta com várias redes: https://bgp.he.net/AS4230#_peers

Para uma ASN conectar com outra ASN (ex: ASN do provedor se conectar com a ASN do google), é necessário que tenha uma conexão direta, com cabo, uma cabo de fibra ótica saindo daqui e conectando lá.


NAT

Network address translation

Nosso roteador faz um papel mto importante chamado nat. 
Vimos como funciona para mandarmos uma requisição para o google, mas como o google sabe para onde ele tem que devolver o pacote? Uma vez que nosso IP (192.168.0.63) é uma rede privada. Para isso, é necessário traduzir o IP interno da nossa rede para um IP público. O provedor de internet fornece um endereço de IP público, geralmente um endereço por casa.

Porque existe o NAT?

Antigamente todos os dispositivos, incluindo os dispositivos internos da rede, eram endereços públicos, ou seja, qualquer pessoa da internet conseguia chegar no seu computador dentro da sua rede interna.

Com isso, a quantidade de endereços IPs não iria dar conta, esgotou a quantidade máxima.

Por isso, a ideia foi atribuir um único endereço IP para cada rede (casa), e é usado o NAT para traduzir os endereços da rede privada para uma rede pública.