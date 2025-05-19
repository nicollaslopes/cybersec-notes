# # Identificado hosts ativos em uma rede interna utilizando o ARP

O `arping` é uma ferramenta que utiliza o protocolo arp para fazer a descoberta de hosts ativos em uma rede local. Utilizamos da seguinte forma.

```
$ arping -c <ip>
```

Como é retornado "60 bytes" como resposta, podemos automatizar esse processo.

```
$ for i in $(seq 1 255); do sudo arping -c 1 192.168.2.$i; done | grep "60 bytes"

60 bytes from 58:25:75:96:e8:86 (192.168.2.1): index=0 time=780.865 usec
60 bytes from 7c:8b:b5:06:0f:4f (192.168.2.106): index=0 time=3.433 msec
60 bytes from 1c:1b:0d:f3:e7:90 (192.168.2.107): index=0 time=844.583 usec
60 bytes from 38:01:95:18:46:64 (192.168.2.111): index=0 time=1.831 msec
60 bytes from 2e:c7:b0:2e:c7:b0 (192.168.2.136): index=0 time=1.670 msec
```

Outra ferramenta bem mais prática para utilizar é o **arp-scan**.

```
$ sudo arp-scan -l                                                      
Interface: eth0, type: EN10MB, MAC: 08:00:27:47:fb:50, IPv4: 192.168.2.134
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.2.1     58:25:75:96:e8:86       (Unknown)
192.168.2.107   1c:1b:0d:f3:e7:90       GIGA-BYTE TECHNOLOGY CO.,LTD.
192.168.2.111   38:01:95:18:46:64       Samsung Electronics Co.,Ltd
192.168.2.136   2e:c7:b0:2e:c7:b0       (Unknown: locally administered)
192.168.2.130   f6:cc:17:63:04:49       (Unknown: locally administered)

5 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 1.881 seconds (136.10 hosts/sec). 
```
