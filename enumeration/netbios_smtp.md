# NetBIOS / SMB

O serviço NetBIOS e SMB são responsáveis por compartilhamento de arquivos e diretórios. O NetBIOS funciona na porta 139 TCP e o SMB na porta 445 TCP. Quando um host utiliza o NetBIOS, possivelmente esse host é mais antigo, pois o NetBIOS veio antes. Já o SMB é utilizado em sistemas mais modernos.

## Enumerando no Linux

Primeiramente, utilizamos o nbtscan para scanear os hosts passando o range da rede.

```
$ nbtscan -r 172.16.1.0/24                             
Doing NBT name scan for addresses from 172.16.1.0/24

IP address       NetBIOS Name     Server    User             MAC address      
------------------------------------------------------------------------------
172.16.1.4       WKS01            <server>  <unknown>        00:0c:29:fb:ab:27
172.16.1.5       SERVER5          <server>  SERVER5          00:00:00:00:00:00
172.16.1.60      SRVINT           <server>  <unknown>        00:0c:29:ce:87:ee
172.16.1.107     SMB              <server>  SMB              00:00:00:00:00:00
172.16.1.108     SRVWEB                     <unknown>        00:50:56:26:3d:f0
172.16.1.165     SRV-CPD-OLD                <unknown>        00:50:56:3b:59:b2
172.16.1.233     SRVSPIDER        <server>  <unknown>        00:0c:29:fe:64:d7
```

Para enumerarmos os compartilhamentos utilizamos o smbclient. Caso não quisermos informar a senha, basta usar o parâmetro -N e se quisermos informar um certo usuário basta utilizar -U e o usuário.

```
$ smbclient -L \\<ip>
```

Para nos conectarmos em determinado diretório basta utilizar da seguinte forma.

```
$ smbclient //<ip>/<directory> -N
```

Após conectar no diretório, podemos usar o help para ver as opções.

```
$ smbclient //172.16.1.4/_DOCS -N                  
Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!              
smb: \> 
```

Podemos ter um problema de versão para enumerar determinado host. Caso precisemos especificar uma versão mais antiga, podemos utilizar da seguinte forma.

```
$ smbclient -L \\172.16.1.5 -N
protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED

$ smbclient -L \\172.16.1.5 -N --option='client min protocol=NT1'   
Anonymous login successful

      Sharename       Type      Comment
      ---------       ----      -------
      print$          Disk      Printer Drivers
      tmp             Disk      oh noes!
      opt             Disk      
      IPC$            IPC       IPC Service (server5 server (Samba 3.0.20-Debian))
      ADMIN$          IPC       IPC Service (server5 server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        GBUSINESS            WKS01
        GRANDBUSINESS        SRV-CPD-OLD
        WORKGROUP            SERVER5
```

## Enumerando no Windows

Ao acessarmos o prompt de comando e estando dentro da rede, podemos começar a explorar.

Para começo, podemos utilizar o **nbtstat**, ele mostra a estatística do protocolo utilizando o NetBIOS sobre o protocolo TCP/IP.

Usamos o comando a seguir para buscar informações dessa máquina.

```
nbtstat -A <ip>
```

Usamos o comando a seguir para tentar também visualizar informações de determinada máquina, com seus compartilhamentos. Geralmente pode dar acesso negado por motivo de permissões.

```
net view \\<ip>
```

Uma das coisas que podemos fazer quando estamos enumerando SMB e NetBIOS, é usar a Null Session, que é basicamente estabelecer uma conexão sem usuário e senha. Usamos da seguinte forma.

```
net use \\<ip> "" /u:""
```

Onde ficam as duas aspas " " deveria ser passado a senha e o usuário.

Se houver sucesso, podemos agora tentar usar o net view novamente para visualizar.

```
net view \\<ip>
```

Usamos o comando a seguir para ver o cache, exibindo todas as máquinas que esse host já se comunicou.

```
nbtstat -c
```

Vamos supor que conseguimos visualizar os compartilhamentos. Precisamos agora de definir uma letra para que o windows estabeleça uma comunicação.

```
net use <any-letter> \\<ip>\<folder-we-want-to-connect>
```

Executamos **net use** para verificar novamente, e após isso digitamos a letra e dois pontos para nos conectarmos nesse volume.

```
<any-letter>:
```

Uma outra técnica é tentarmos fazer uma autenticação com o usuário administrator sem senha.

```
net use \\<ip> "" /u:administrator
```

Se quisermos desmontar alguma das conexões usamos o comando a seguir.

```
net use h: /delete
```

Se quisermos excluir a conexão de determinada máquina usamos o comando a seguir.

```
net use \\<ip> /delete
```

# Automatizando a enumeração NetBIOS/SMB

## Enum4linux

O enum4linux é uma ferramenta que automatiza o processo de enumeração de NetBIOS/SMB que fizemos manualmente de uma forma mais organizada.

O parâmetro -U serve para trazer informações de usuários.

O parâmetro -S serve para trazer informações sobre compartilhamento.

O parâmetro -a significa all, ele vai tentar achar o máximo de informações sobre o host e retornar.


Podemos utilizar da seguinte forma.

```
$ enum4linux -a <ip>
```

## Scripts para enumeração NetBIOS/SMB

O nmap também trás alguns recursos que permite facilitar a enumeração. A pasta dos arquivos referente ao protocolo NetBIOS/SMB fica em **/usr/share/nmap/scripts/**

Nessa pasta possui alguns scripts que já exploram falhas de segurança conhecidas. Utilizamos o comando a seguir. (o * é um coringa, onde irá pesquisar todos os arquivos que tiver com a string restante passada)

```
$ nmap -v --script=smb-vuln-ms* <ip>
```
