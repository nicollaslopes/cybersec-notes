# Port Forwarding

**Port forwarding** is a network configuration technique that redirects incoming network traffic from one port to another.

We need to know what port forwarding is. In the following image we can see the scenario where we are in a local network and we have a router that allows us to access the internet. The router connects to the provider and provides us a public IP address fro all machines on the network to access the internet. It can be a dynamic IP or a static IP, depending on the provider.

Let's suppose we need to open a port to get a `reverse shell`. Once we open a port, 443 for example, and we want to a target connects itself to this port through a public IP, it's probably that this service is not available, because it wasn't made the port forwarding.
 
Vamos supor que precisamos abrir uma porta para receber uma `reverse shell`. Uma vez que abrimos uma porta, por exemplo 443, e se quisermos que um alvo se conecte a essa porta através do ip público, é provável que esse serviço não está liberado, porque não foi feito o encaminhamento de portas. O port forwarding serve para assim que o pacote chega no roteador, ele saiba direcionar o pacote para o IP correto que tenha a porta aberta.  

Supondo que conseguimos comprometer um servidor que possui um firewall de borda. Com isso, queremos abrir uma porta nesse servidor para nos conectarmos diretamente a ele através do nosso computador. Por causa do firewall, não conseguimos nos conectar diretamente porque ele não permite e bloqueia o acesso. Com isso, a solução é fazermos uma conexão reversa. Nós abrimos uma porta no nosso computador e fazemos o servidor se conectar na nossa máquina, já que o firewall possui portas que na saída ele libera. Para que isso funcione, é necessário ter o port forwarding funcionando. 

Uma outra solução é utilizar VPS (Virtual Private Network). Podemos alugar um servidor na nuvem, e com isso podemos liberar uma porta que ela vai estar exposta para a internet e com isso não precisamos fazer configurações no roteador.

Isso irá depender muito de qual provedor você utiliza, quantos roteadores tem no meio do caminho (nesse caso teríamos que ter acesso a todos os roteadores para aplicar a configuração correta). E mesmo assim o provedor pode bloquear.