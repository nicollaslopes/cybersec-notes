# SNMP

Basicamente, o protocolo SNMP (Simple Network Management Protocol)serve para gerenciar dispositivos e serviços de rede, funciona na porta 161 UDP. Quando conseguimos enumerar o SNMP com sucesso, é possível extrair informações sobre usuários, programas instalados, portas abertas, informações sobre o sistema operacional, entre outras. Antes de qualquer coisa, precisamos saber de alguns detalhes.

* OID - Object Identifier -> Código utilizado para identificar os objetos;
* MIB - Management Information Base -> Base contendo informações relacionadas ao gerenciamento de redes;
* Community -> Valor utilizado entre as partes snmp para troca de informações.

(Nível de acesso da community: RO - Read Only; RW - Read/Write)

Examplos valores MIBS:

```
1.3.6.1.2.1.25.1.6.0  => PROCESSOS DO SISTEMA
1.3.6.1.4.1.77.1.2.25 => CONTAS DE USUÁRIO
1.3.6.1.2.1.6.13.1.3  => PORTAS TCP
```

Community:

```
public
private
cisco
manager
access
secret
```

Precisamos identificar quais hosts na rede respondem por determinada community.

Podemos usar a ferramenta onesixtyone para fazer um "brutefoce" e identificar, passando uma wordlist com o nome das community da seguinte forma.

```
$ cat comu.txt 
public
private
cisco
manager
juniper

$ onesixtyone -c comu.txt 172.16.1.0/24
Scanning 256 hosts, 7 communities
172.16.1.4 [public] Hardware: x86 Family 6 Model 62 Stepping 4 AT/AT COMPATIBLE - Software: Windows 2000 Version 5.1 (Build 2600 Multiprocessor Free)
172.16.1.247 [manager] Vyatta VyOS 1.1.7
172.16.1.247 [juniper] Vyatta VyOS 1.1.7
```

Para enumerar as contas de usuários.

```
$ snmpwalk -c public -v1 172.16.1.4 1.3.6.1.4.1.77.1.2.25  
iso.3.6.1.4.1.77.1.2.25.1.1.4.108.117.105.115 = STRING: "luis"
iso.3.6.1.4.1.77.1.2.25.1.1.7.85.115.117.97.114.105.111 = STRING: "Usuario"
iso.3.6.1.4.1.77.1.2.25.1.1.7.114.97.102.97.101.108.97 = STRING: "rafaela"
iso.3.6.1.4.1.77.1.2.25.1.1.9.67.111.110.118.105.100.97.100.111 = STRING: "Convidado"
iso.3.6.1.4.1.77.1.2.25.1.1.13.65.100.109.105.110.105.115.116.114.97.100.111.114 = STRING: "Administrador"
iso.3.6.1.4.1.77.1.2.25.1.1.13.72.101.108.112.65.115.115.105.115.116.97.110.116 = STRING: "HelpAssistant"
iso.3.6.1.4.1.77.1.2.25.1.1.15.75.69.89.50.57.56.55.48.48.49.57.49.56.50.48 = STRING: "KEY298700191820"
iso.3.6.1.4.1.77.1.2.25.1.1.16.83.85.80.80.79.82.84.95.51.56.56.57.52.53.97.48 = STRING: "SUPPORT_388945a0"
```

Dessa forma, ele não informa exatamente o que está buscando, apenas alguns códigos no início, isso dificulta o entendimento da saída da ferramenta. Precisamos instalar um pacote para que possa traduzir essas informações. Para configurar, basta deixar o seguinte arquivo vazio.

```
$ sudo apt install snmp-mibs-downloader

$ echo "" > /etc/snmp/snmp.conf
```

Agora quando executamos, retorna com mais clareza as informações. Lembrando que pode-se passar sem o valor da MIB.

```
$ snmpwalk -c public -v1 172.16.1.4
SNMPv2-MIB::sysDescr.0 = STRING: Hardware: x86 Family 6 Model 62 Stepping 4 AT/AT COMPATIBLE - Software: Windows 2000 Version 5.1 (Build 2600 Multiprocessor Free)
SNMPv2-MIB::sysObjectID.0 = OID: SNMPv2-SMI::enterprises.311.1.1.3.1.1
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (1324872701) 153 days, 8:12:07.01
SNMPv2-MIB::sysContact.0 = STRING: 
SNMPv2-MIB::sysName.0 = STRING: WKS01
SNMPv2-MIB::sysLocation.0 = STRING: 
SNMPv2-MIB::sysServices.0 = INTEGER: 76
IF-MIB::ifNumber.0 = INTEGER: 2
IF-MIB::ifIndex.1 = INTEGER: 1
IF-MIB::ifIndex.2 = INTEGER: 2
IF-MIB::ifDescr.1 = STRING: MS TCP Loopback interface.
IF-MIB::ifDescr.2 = STRING: AMD PCNET Family PCI Ethernet Adapter - Miniporta do agendador de pacotes.
IF-MIB::ifType.1 = INTEGER: softwareLoopback(24)
IF-MIB::ifType.2 = INTEGER: ethernetCsmacd(6)
IF-MIB::ifMtu.1 = INTEGER: 1520
IF-MIB::ifMtu.2 = INTEGER: 1500
IF-MIB::ifSpeed.1 = Gauge32: 10000000
IF-MIB::ifSpeed.2 = Gauge32: 1000000000
IF-MIB::ifPhysAddress.1 = STRING: 
IF-MIB::ifPhysAddress.2 = STRING: 0:c:29:fb:ab:27
IF-MIB::ifAdminStatus.1 = INTEGER: up(1)
IF-MIB::ifAdminStatus.2 = INTEGER: up(1)
```

## Modificações através do SNMP

Quando existe uma community manager, quer dizer que temos permissão de escrita, então podemos fazer algumas alterações.

Podemos manipular os dados utilizando o snmpset. Iremos alterar onde está o valor de `SNMPv2-MIB::sysContact.0` que é "XISDE". O "s" significa que o valor é uma string.

```
$ snmpset -c manager -v1 172.16.1.247 SNMPv2-MIB::sysContact.0 s "XISDE"
```

```
$ snmpwalk -c manager -v1 172.16.1.247
SNMPv2-MIB::sysDescr.0 = STRING: Vyatta VyOS 1.1.7
SNMPv2-MIB::sysObjectID.0 = OID: SNMPv2-SMI::enterprises.30803
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (479344920) 55 days, 11:30:49.20
SNMPv2-MIB::sysContact.0 = STRING: XISDE
SNMPv2-MIB::sysName.0 = STRING: juniper
SNMPv2-MIB::sysLocation.0 = STRING: Unknown
```

Alterando o valor para "1337", temos:

```
$ snmpset -c manager -v1 172.16.1.247 SNMPv2-MIB::sysContact.0 s "1337"
SNMPv2-MIB::sysContact.0 = STRING: 1337

$ snmpwalk -c manager -v1 172.16.1.247
SNMPv2-MIB::sysDescr.0 = STRING: Vyatta VyOS 1.1.7
SNMPv2-MIB::sysObjectID.0 = OID: SNMPv2-SMI::enterprises.30803
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (479359620) 55 days, 11:33:16.20
SNMPv2-MIB::sysContact.0 = STRING: 1337
SNMPv2-MIB::sysName.0 = STRING: juniper
SNMPv2-MIB::sysLocation.0 = STRING: Unknown
SNMPv2-MIB::sysServices.0 = INTEGER: 14
```