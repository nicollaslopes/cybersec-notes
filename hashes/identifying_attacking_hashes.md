# Identifying and Attacking Hashes

Existem alguns softwares que podemos utilizar para identificar os hashes, que são: `hashid` e `hash-identifier`. Basta utilizar

```
$ hash-id <hash>
```

```
$ hash-identifier 
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH:
```

## Ataques a hashes: Ferramentas

## John

Usando o john dessa forma, ele irá tentar identificar qual é o tipo de hash.

```
$ john <hash-file>
```

Para informar o formato do hash e passar uma wordlist.

```
# john --format=Raw-MD5 <hash-file> --wordlist=<path-wordlist>
```

## Hashcat

A diferença do hashcat para outras ferramentas, é que ele permite utilizar a GPU para tentar descobrir o hash, automatizando o processo.

```
$ hashcat -m <hash-mode> <hash-file> <wordlist> --force
```


Sites:

{% embed url="https://hashes.com/en/decrypt/hash" %}


